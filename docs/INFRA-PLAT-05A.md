# INFRA-PLAT-05A — Integración real de infraestructura

Estado: INTEGRATION_PASS (25 noviembre 2026). Ensayo real completado; Worker detenido. Cierre documental en PR #15.

Referencias: Platform PR #19 integrado en develop d96848230eb42e00f3834f973705b3ac720f2e74, contenido idéntico al candidato auditado 733c4e817c8bbb53ff166e7480639e15ed040a21; CI 45678963379 éxito en pruebas y Docker, publicación omitida por diseño. ADR-0002, ADR-0003 y docs/PLAT-05A-CNPG-INFRASTRUCTURE.md de Platform; DB-01C de saas-infra.

Preflight 25 noviembre 2026: K3s 1.37.2, nodo Ready, CNPG 1.30.2 saludable, Argo CD y Platform Synced/Healthy, pg-lab-01 saludable con mismo PVC. Memoria disponible 4.9 GiB, nodo 2681 MiB/34%, Windows C: libre 138.99 GiB. local-path disponible; ValidatingAdmissionPolicy v1 disponible. Namespaces de ensayo ausentes. El preflight no certifica carga ni soporte oficial.

## Preparación (A)

Aplicación manual, por orden, fuera de las rutas de Argo CD. No aplicar el directorio completo. No modificar la API existente ni DB-01C. Solo datos sintéticos.

1. Revisar PR. Aplicar 01-namespaces.yaml; dry-run de servidor y aplicar 02-worker-rbac.yaml y 03-guardrails.yaml. El Role solo permite get/create clusters.postgresql.cnpg.io en saas-tenant-databases. Comprobar permisos efectivos con impersonación del ServiceAccount y sus grupos reales; API default de saas-vital sin permisos de aprovisionamiento.
2. Comprobar quota status.hard y status.used, estado de typeChecking de la política; realizar CREATE exclusivamente con --dry-run=server de un perfil sintético válido como Worker, y otro con etiquetas/perfil incorrectos que debe ser rechazado por la política. Ninguno persiste. La cuota limita un Cluster, un PVC y 2Gi solicitados; comprobar que sigue sin existir Cluster de tenant antes del ensayo. Una segunda creación se comprobará con dry-run cuando exista el primer Cluster. La política CEL restringe CREATE en el namespace de tenants, no UPDATE/status ni otros namespaces; es guardrail del ensayo, no una validación exhaustiva de todos los campos posibles del CRD.
3. Dry-run y aplicar 04-platform-catalog.yaml. Esperar Ready. El catálogo pg-platform-05a es estático y no es un tenant; usa su propia base platform_lab y propietario platform_lab_owner. CNPG genera credenciales de bootstrap; comprobar el Secret app sin mostrar datos. No usar el catálogo ordinario ni pg-lab-01. Construir un Secret independiente platform-lab-connection con clave connection-string (Npgsql Host=pg-platform-05a-rw.saas-platform-lab.svc;Port=5432;Database=platform_lab;Username=platform_lab_owner;Password=...;SSL Mode=Require). Nunca registrar su valor ni guardarlo en Git. TLS requerido, sin certificar identidad del servidor en el laboratorio.
4. Aplicar 05-worker.yaml con replicas=0 tras dry-run; no escalar aún. Sin Service ni Ingress. No hay endpoints HTTP del Worker. La API pública existente no cambia.

## Ensayo (B), solo tras aceptar A

Construir desde checkout limpio y separado del commit integrado exacto, usando Dockerfile existente; registrar SHA, image ID y etiqueta saas-platform-lab:d96848230eb42e00f3834f973705b3ac720f2e74. Guardar TAR e importar en containerd de saas-k3s-server; imagePullPolicy Never y nodeSelector fijan ejecución al nodo con imagen. No publicar manualmente ni modificar pipeline.

Preparar ejecutor temporal fuera del repo de aplicación, referenciando el proyecto del commit integrado, para aplicar las cuatro migraciones existentes de EF Core y llamar a RegisterTenantHandler con RegisterTenantCommand("plat05a-lab", "Clinica Laboratorio PLAT-05A", "contacto@synthetic.test", "Administrador Sintetico", "admin@synthetic.test", 10). Usar exclusivamente el catálogo aislado, sin INSERT manual ni desactivar autenticación API. Verificar exactamente un tenant y una operación pendiente; abortar ante catálogo no esperado. Se utilizó un ejecutor .NET 10 temporal (fuera de los repositorios), construido mediante un ProjectReference al proyecto Platform del SHA integrado. Validó host/base/usuario esperados, aplicó Database.MigrateAsync, comprobó cuatro migraciones aplicadas y catálogo vacío, invocó el handler real y comprobó registro y cardinalidad. Se ejecutó como Job platform-05a-seed con backoffLimit=0, activeDeadlineSeconds=180, sin token de ServiceAccount y con la conexión suministrada por Secret. No es un E2E de API/Identity.

Escalar explícitamente a 1 réplica solo tras verificar imagen, migraciones, Secret, permisos y admisión. Observar logs controlados, eventos, Cluster/Pod/PVC; no crear manualmente Cluster de tenant ni corregir su estado por SQL. No habilitar otros adaptadores. Un rechazo o timeout inesperado se diagnostica sin ampliar RBAC ni cambiar el catálogo.

## Aceptación y cierre (C)

- Cluster único pg-{tenantId:N} creado por Worker; etiquetas managed-by, component, tenant-id, operation-id correctas; UID real y perfil auditado. CNPG Ready y fase saludable, una primaria, PVC Bound; sin tracking Argo CD. No se requieren DatabaseRole/Database de consumidores ni distribución de credenciales.
- Catálogo: infrastructure_stage_state CONFIRMED y infrastructure_resource_ref namespace/nombre@UID exacto; service_databases_stage_state, clinical_stage_state e identity_stage_state PENDING; status FAILED_RETRYABLE, last_error_code ADAPTER_NOT_CONFIGURED, manual_intervention_required true y next_attempt_at_utc NULL. Tenant PENDIENTE_ADMIN, mayAuthenticate false según modelo/consulta existente.
- Verificar ausencia de duplicados tras reinicio controlado del Worker (no equivale a probar recuperación de un CREATE incierto). Comprobar cuota de segundo Cluster con dry-run, sin creación real.
- Escalar Worker a 0 al finalizar, conservar catálogo, Cluster, PVC y DB-01C; no borrar automáticamente recursos. Verificar salud de Platform y Argo CD, registrar evidencia y cerrar.
- Quedan fuera: HA, nuevos operadores, LimitRange/cuotas CPU/RAM de pods auxiliares, NetworkPolicy definitiva, cifrado de Secrets en reposo, migraciones de servicios, pruebas multi-tenant bajo carga y activación de clínicas. La cuota de almacenamiento limita solicitudes, no uso físico real de local-path.

No se modifican repositorios de aplicaciones ni contratos; ningún recurso dinámico pertenece a una Application de Argo CD.

## Evidencia final — 25 noviembre 2026

**Dictamen: INTEGRATION_PASS**, acotado a InfrastructureInstance de PLAT-05A.

- MANIFEST_ADMISSION_PASS: ambos namespaces creados; los cinco manifiestos aceptados por dry-run de servidor.
- RBAC_PASS: GET y CREATE de clusters permitidos exclusivamente al Worker en saas-tenant-databases conforme al Role; LIST/WATCH/UPDATE/PATCH/DELETE/DELETECOLLECTION rechazados. Pruebas negativas de Secrets, DatabaseRole, Database, Pods, pods/exec, namespaces y Clusters en cnpg-lab/saas-platform-lab; API existente (default de saas-vital) sin GET/CREATE de Clusters en namespace de tenants. ValidatingAdmissionPolicy con observedGeneration=1 y typeChecking sin advertencias reportadas.
- ADMISSION_GUARDRAILS_PASS: dry-run válido admitido, cambio a instances=2 rechazado con PLAT05A_PROFILE; sin Cluster persistido por estas pruebas. No se ejercitaron por separado todas las expresiones CEL.
- CATALOG_INFRASTRUCTURE_PASS: pg-platform-05a Ready, primaria saludable, PVC Bound 2Gi (PV pvc-59a540fd-7c68-46ac-a42b-3610da9462d9). Worker inicialmente 0/0.
- LOCAL_IMAGES_BUILD_PASS: primer intento local falló por ausencia del commit integrado, con mensajes de éxito engañosos al pegar comandos independientes en PowerShell. Se recuperó origin/develop mediante fetch, se exportó el SHA exacto con git archive, se conservaron los archivos temporales y se verificaron ambos builds, etiquetas OCI de revisión y TAR de 110625280 bytes. No se modificó el repo de aplicación ni se publicó imagen. Índices OCI: Platform sha256:8152d7684835ea0b189c4fbc52e13363feb1565973a64429e9230ff983ef18a3; ejecutor sha256:42d049d290c75884cf60f62168dd55b698bf6c481df97ccf2b0176cc2c2e7254. Config digests CRI observados (prefijos): 6eae892e3e81c y 7b2853ec70f49.
- IMAGE_AND_CONNECTION_PASS: importación local en containerd y Secret platform-lab-connection creado sin exponer sus valores. Job completado con MIGRATIONS_PASS y SYNTHETIC_REGISTRATION_PASS; INTEGRATION_PREPARATION_PASS. Se observó aviso de biblioteca libgssapi_krb5.so.2 ausente tanto en ejecutor como en Worker, sin impedir la conexión PostgreSQL ni la prueba. No se instaló software adicional.
- REAL_CLUSTER_PROVISIONING_PASS: escalado a una réplica, adquisición de operación en intento 1 con lease version 1, GET inicial 404 esperado; Cluster creado automáticamente, Ready y fase Cluster in healthy state, una primaria, Pod 1/1 sin reinicios, PVC Bound 2Gi (PV pvc-05ed35b4-b7b1-4c7f-b3a7-5ea525775de2). Etiquetas de propiedad y operación correctas; sin etiqueta de tracking Argo CD observada. La creación no se ejecutó manualmente.

### Correlación durable

- Tenant: 01a214af-fddc-713e-a908-c1f690486d90.
- Operación: 01a214af-fddc-75c8-9c5e-7f2317a4e6c2.
- Cluster: pg-01a214affddc713ea908c1f690486d90.
- UID: 84fb9b6c-b40a-4c85-b822-810907498f73.
- Referencia persistida: saas-tenant-databases/pg-01a214affddc713ea908c1f690486d90@84fb9b6c-b40a-4c85-b822-810907498f73.

DURABLE_STATE_PASS: consulta SQL de solo lectura confirmó exactamente PENDIENTE_ADMIN, mayAuthenticate=false (derivado del estado), CONFIRMED para infraestructura y PENDING para ServiceDatabases, Clinical e Identity; FAILED_RETRYABLE / ADAPTER_NOT_CONFIGURED, manual_intervention_required=true, next_attempt_at_utc NULL. Referencia idéntica al UID real. Cardinalidad 1 tenant / 1 operación.

### Cierre operativo

- NO_DUPLICATES_PASS: rollout restart del Worker, espera de 15 segundos, único Cluster con mismo UID; snapshot de id, status, attempt_count, lease_version e infrastructure_resource_ref sin cambios. No equivale a probar recuperación de CREATE incierto ni todos los escenarios de fallo.
- CLUSTER_QUOTA_PASS: con el Worker detenido, CREATE dry-run de segundo Cluster rechazado explícitamente por exceeded quota; no se creó otro Cluster.
- FINAL_INTEGRATION_PASS: Worker escalado a 0/0; cuota 1/1 Cluster, 1/1 PVC, 2Gi/2Gi solicitados. Tres Clusters saludables (DB-01C, catálogo global y tenant) y tres PVC Bound. DB-01C conserva PV pvc-7ef0fe18-3781-4a80-bc7c-b7f903c95e4e. Argo CD reportó cloudnativepg y saas-platform Synced/Healthy.
- Medición final: nodo 227m CPU (1%), 3294Mi RAM (42%), free -h 7.6Gi total / 4.8Gi disponible, swap 556Ki utilizado. No es prueba bajo carga ni garantía de capacidad para cinco tenants.

No hubo cambios en los repositorios de aplicación, nuevas migraciones, publicación GHCR, ampliación de RBAC ni activación de clínicas. Se conservan los tres Clusters y sus volúmenes, las credenciales de laboratorio y el Worker detenido. El manifiesto de Deployment permanece en replicas=0. No reaplicar el Job de registro sobre un catálogo no vacío. No eliminar namespaces, Clusters o PVC para preparar el siguiente incremento. PLAT-05B y las bases/roles de consumidores siguen pendientes; no se certifica aislamiento completo entre tenants, identidad TLS, backup/restore ni HA.
