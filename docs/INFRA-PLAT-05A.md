# INFRA-PLAT-05A — Integración real de infraestructura

Estado: PREFLIGHT_PASS; manifiestos candidatos, sin aplicación ni ensayo real aún.

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

Preparar ejecutor temporal fuera del repo de aplicación, referenciando el proyecto del commit integrado, para aplicar las cuatro migraciones existentes de EF Core y llamar a RegisterTenantHandler con RegisterTenantCommand("plat05a-lab", "Clinica Laboratorio PLAT-05A", "contacto@synthetic.test", "Administrador Sintetico", "admin@synthetic.test", 10). Usar exclusivamente el catálogo aislado, sin INSERT manual ni desactivar autenticación API. Verificar exactamente un tenant y una operación pendiente; abortar ante catálogo no esperado. El ejecutor y mecanismo de transporte se concretarán para el entorno local antes de activar. No es un E2E de API/Identity.

Escalar explícitamente a 1 réplica solo tras verificar imagen, migraciones, Secret, permisos y admisión. Observar logs controlados, eventos, Cluster/Pod/PVC; no crear manualmente Cluster de tenant ni corregir su estado por SQL. No habilitar otros adaptadores. Un rechazo o timeout inesperado se diagnostica sin ampliar RBAC ni cambiar el catálogo.

## Aceptación y cierre (C)

- Cluster único pg-{tenantId:N} creado por Worker; etiquetas managed-by, component, tenant-id, operation-id correctas; UID real y perfil auditado. CNPG Ready y fase saludable, una primaria, PVC Bound; sin tracking Argo CD. No se requieren DatabaseRole/Database de consumidores ni distribución de credenciales.
- Catálogo: infrastructure_stage_state CONFIRMED y infrastructure_resource_ref namespace/nombre@UID exacto; service_databases_stage_state, clinical_stage_state e identity_stage_state PENDING; status FAILED_RETRYABLE, last_error_code ADAPTER_NOT_CONFIGURED, manual_intervention_required true y next_attempt_at_utc NULL. Tenant PENDIENTE_ADMIN, mayAuthenticate false según modelo/consulta existente.
- Verificar ausencia de duplicados tras reinicio controlado del Worker (no equivale a probar recuperación de un CREATE incierto). Comprobar cuota de segundo Cluster con dry-run, sin creación real.
- Escalar Worker a 0 al finalizar, conservar catálogo, Cluster, PVC y DB-01C; no borrar automáticamente recursos. Verificar salud de Platform y Argo CD, registrar evidencia y cerrar.
- Quedan fuera: HA, nuevos operadores, LimitRange/cuotas CPU/RAM de pods auxiliares, NetworkPolicy definitiva, cifrado de Secrets en reposo, migraciones de servicios, pruebas multi-tenant bajo carga y activación de clínicas. La cuota de almacenamiento limita solicitudes, no uso físico real de local-path.

No se modifican repositorios de aplicaciones ni contratos; ningún recurso dinámico pertenece a una Application de Argo CD.
