# DB-01C — Primera instancia PostgreSQL sintética 

**Estado:** DB-01C INTEGRATION_PASS (25 de noviembre de 2026).
**Dependencia:** DB-01B INSTALLATION_PASS (CloudNativePG 1.30.2 en K3s 1.37.2).
**Alcance:** validar una sola primaria PostgreSQL de laboratorio y después aislar dos bases y roles por servicio. No usar datos reales.

## Separación de responsabilidades

- `saas-infra` posee los manifiestos **de prueba**, no los clústeres reales de tenants.
- El operador CloudNativePG continúa gestionado por Argo CD; `k8s/labs/db-01c/` **no pertenece a ninguna Application de Argo CD**.
- El futuro trabajador de `saas-platform` será dueño de crear recursos dinámicos. **No se le otorgan permisos en DB-01C**.
- No se modifica `saas-platform`, Identity ni Clinical.

## Recursos validados

- `01-namespace.yaml`: namespace exclusivo `cnpg-lab`.
- `02-cluster.yaml`: `pg-lab-01`, `instances: 1`, PostgreSQL 17, PVC de **2 GiB** sobre `local-path`. Etiqueta validada en GHCR y ejecutada; digest no fijado. Incluye regla pg_hba de rechazo a bases predeterminadas para los dos roles.
- `03-roles.yaml` y `04-databases.yaml`: roles propietarios inicialmente NOLOGIN, sin privilegios administrativos, y bases `identity_lab` y `clinical_lab` con propietarios independientes. Ambos tipos de recurso usan política `retain`; no protege frente a eliminación del Cluster/PVC. Sin credenciales en Git.
- CPU 100m solicitados / 750m límite, memoria 256Mi solicitada / 768Mi límite: hipótesis para laboratorio; medir y ajustar únicamente si la evidencia lo requiere.
- **Sin alta disponibilidad, backups productivos ni exposición de PostgreSQL fuera del clúster**.

## Fases y aceptación

1. **Revisión sin cambios:** validar YAML de ambos recursos, versión/tag/digest de la imagen PostgreSQL y capacidad disponible. Verificar salud del operador y de Platform. Inspeccionar riesgos de `local-path` (`ReclaimPolicy: Delete`, sin expansión).
2. **Admisión y reconciliación:** tras aprobación explícita, aplicar namespace y Cluster de prueba. Verificar estado de admisión (webhooks), Pod primario saludable, servicios internos, PVC `Bound` y `Cluster` Ready; recoger eventos si hay fallos.
3. **Bases y permisos:** crear dos roles con secretos diferentes **generados fuera de Git**, y dos bases propias. Probar conexiones permitidas, denegación cruzada, ausencia de superusuario y reglas de `CONNECT` para `PUBLIC`. Dos bases separadas **no implican** aislamiento suficiente sin revocar/validar permisos.
4. **Persistencia y recuperación:** escribir solo datos sintéticos, efectuar reinicio controlado del Pod manteniendo el PVC, y comprobar conservación. Registrar consumo de RAM/CPU/disco y límites para escalar a 4–5 tenants.
5. **Limpieza:** no borrar PVC ni CRD automáticamente. Registrar qué recursos fueron creados y acordar su conservación o limpieza explícita según resultados.

## Preflight de solo lectura

Desde `saas-k3s-server`:

```bash
sudo k3s kubectl -n argocd get applications
sudo k3s kubectl -n cnpg-system get deployments,pods
sudo k3s kubectl get clusters.postgresql.cnpg.io -A
sudo k3s kubectl get pvc -A
sudo k3s kubectl top nodes
free -h
```

## Evidencia inicial — 25 de noviembre de 2026

- PR #12 fusionado en main (37ffc9c). Dry-run de cliente y servidor satisfactorios; GHCR respondió HTTP 200 para la etiqueta 17.12-system-trixie (no se fijó digest).
- Cluster pg-lab-01 creado; Ready=True, primaria pg-lab-01-1, Pod 1/1 sin reinicios, PVC 2Gi Bound sobre local-path.
- SQL ejecutado como postgres por el socket local: PostgreSQL 17.12 (Debian 17.12-1.pgdg13+1), pg_is_in_recovery()=false.
- Medición puntual en reposo: primaria 8m CPU / 47Mi RAM; durante inicialización nodo 1395m / 2323Mi. No representa carga real ni capacidad certificada para varios tenants.
- CloudNativePG y saas-platform conservaron Synced/Healthy; Platform Deployment 1/1.
- kubectl explain confirma DatabaseRole.spec.cluster.name, passwordSecret.name (Secret kubernetes.io/basic-auth con username y password) y Database.spec.cluster.name, así como status.applied y status.observedGeneration para ambos recursos.

## Aprovisionamiento de bases privadas (secuencia validada)

1. Validar por dry-run de servidor los dos nuevos manifiestos antes de aprobarlos. No aplicar el directorio completo.
2. Aplicar exclusivamente 03-roles.yaml. Para ambos recursos exigir status.applied=true y status.observedGeneration igual a metadata.generation; confirmar por SQL la existencia de los roles sin LOGIN ni privilegios administrativos.
3. Aplicar exclusivamente 04-databases.yaml. Exigir las mismas comprobaciones de generación/applied; verificar por SQL los propietarios de ambas bases.
4. Antes de habilitar LOGIN, revocar CONNECT de PUBLIC en ambas bases sintéticas y verificar los permisos efectivos de ambos roles; confirmar que los roles no tienen acceso cruzado. No cambiar plantillas globales ni permisos de aplicaciones reales.
5. En un paso posterior, generar secretos diferentes fuera de Git, asociarlos y habilitar LOGIN declarativamente. Verificar reconciliación de la revisión actual y del Secret, conexiones TCP autenticadas positivas y negativas, y escrituras sintéticas. Aplicar 05-roles-active.yaml solo después de crear los Secrets y restringir los ACL; no reaplicar 03-roles.yaml a roles activos. No aplicar el directorio completo.
6. Probar persistencia tras recreación controlada del Pod conservando el PVC. Una instancia no proporciona HA. Esta prueba es aislamiento entre servicios dentro de un tenant simulado, no entre tenants.

El worker futuro puede usar los estados de los CRD como señales de reconciliación, pero no como prueba suficiente de conectividad o autorización. Los ACL de CONNECT requieren una operación adicional a los campos observados del CRD Database.

## Evidencia final — 25 de noviembre de 2026

- PR #13 fusionado (d401ba5); roles y bases admitidos por dry-run de servidor.
- Ambos roles: applied=true, generación observada 2/2 inicialmente; SQL confirmó NOLOGIN y ausencia de superuser, createdb, createrole, replication y bypassrls.
- Ambas bases: applied=true, generación 1/1; propietarios correctos. Matriz CONNECT inicial: cuatro accesos permitidos por PUBLIC. Tras REVOKE CONNECT ON DATABASE identity_lab FROM PUBLIC y REVOKE CONNECT ON DATABASE clinical_lab FROM PUBLIC, accesos propios permitidos y cruzados denegados.
- Secrets identity-lab-credentials y clinical-lab-credentials generados independientemente mediante secrets.token_urlsafe(32), tipo kubernetes.io/basic-auth, etiqueta cnpg.io/reload=true; sin contraseñas en Git.
- Regla pg_hba incorporada: host app,postgres,template0,template1 identity_lab_owner,clinical_lab_owner all reject. Roles activados con referencia a su Secret: applied=true, generación observada 3/3.
- AUTHENTICATION_AND_ISOLATION_PASS: ambos roles conectaron por TCP vía pg-lab-01-rw.cnpg-lab.svc con sslmode=require, escribieron y leyeron un registro propio; ocho rechazos esperados (base ajena, app, postgres, template1 por cada rol). template0 tiene conexiones deshabilitadas; no se probó su conexión. No se comprobó identidad TLS del servidor ni rotación de credenciales.
- Pod primario eliminado de forma controlada y recreado con UID diferente; mismo PVC UID y PV pvc-7ef0fe18-3781-4a80-bc7c-b7f903c95e4e. Cluster y Pod Ready.
- Primer intento de lectura inmediatamente posterior a recreación: timeout TCP de 5 s. Reintento posterior con timeout de 15 s exitoso en ambas bases, sin cambios adicionales: PERSISTENCE_AND_RECOVERY_PASS. Causa exacta no determinada; no se midió tiempo de recuperación.
- Medición posterior: PostgreSQL 58m CPU / 47Mi RAM; nodo 260m / 2379Mi (30% RAM); free -h: 7.6Gi total, 4.9Gi disponible. No constituye prueba de carga.
- Argo CD: cloudnativepg y saas-platform Synced/Healthy. No se conectaron los microservicios reales.

## Reproducción y conservación

Los manifiestos son una secuencia manual de laboratorio, no una Application GitOps. Aplicar 01 y 02, esperar Cluster Ready; aplicar 03, esperar reconciliación; aplicar 04 y esperar reconciliación. Ejecutar como administrador local de PostgreSQL las dos sentencias REVOKE descritas arriba y comprobar matriz CONNECT. Generar Secrets fuera de Git con los nombres y formato indicados; solo entonces aplicar 05 y comprobar reconciliación y autenticación. La regla pg_hba queda incluida en 02. Los ACL SQL no se gestionan por los CRD observados. No ejecutar kubectl apply sobre el directorio completo ni reaplicar 03 sobre el estado activo.

Se conserva el laboratorio existente; no es un tenant real ni una base de Platform. No se autorizó limpieza destructiva.

## Riesgos y pendientes

- Kubernetes 1.37 probado por upstream, pero no oficialmente soportado por CloudNativePG 1.30: la prueba demuestra compatibilidad funcional inicial en este laboratorio, no soporte oficial.
- `local-path` guarda volúmenes en un solo nodo WSL2 y su política Delete exige control de destrucción.
- La creación inicial de los Secret y la resolución dinámica de credenciales tenant×servicio siguen sin contrato aprobado con Identity/Clinical.
- La API pública de Platform no tendrá permisos Kubernetes. RBAC mínimo del trabajador se diseñará con evidencia de operaciones requeridas.
- Los estados reales de aprovisionamiento y transición a ACTIVA no se determinan desde infraestructura.

**Dictamen:** DB-01C INTEGRATION_PASS para una primaria y dos bases de servicio de un tenant sintético. No certifica aislamiento entre tenants distintos, capacidad bajo carga, alta disponibilidad, pérdida de nodo, backup/restore, cifrado de Secrets en reposo ni aprovisionamiento automático por Platform. El worker deberá resolver ACL, orden de operaciones, credenciales, reconciliación y manejo idempotente de fallos; no se fija todavía su implementación.
