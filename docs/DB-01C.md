# DB-01C — Primera instancia PostgreSQL sintética (propuesta)

**Estado:** DB_ENGINE_PASS (25 de noviembre de 2026); aprovisionamiento de bases y roles, aislamiento y persistencia pendientes.
**Dependencia:** DB-01B INSTALLATION_PASS (CloudNativePG 1.30.2 en K3s 1.37.2).
**Alcance:** validar una sola primaria PostgreSQL de laboratorio y después aislar dos bases y roles por servicio. No usar datos reales.

## Separación de responsabilidades

- `saas-infra` posee los manifiestos **de prueba**, no los clústeres reales de tenants.
- El operador CloudNativePG continúa gestionado por Argo CD; `k8s/labs/db-01c/` **no pertenece a ninguna Application de Argo CD**.
- El futuro trabajador de `saas-platform` será dueño de crear recursos dinámicos. **No se le otorgan permisos en DB-01C**.
- No se modifica `saas-platform`, Identity ni Clinical.

## Recursos propuestos

- `01-namespace.yaml`: namespace exclusivo `cnpg-lab`.
- `02-cluster.yaml`: `pg-lab-01`, `instances: 1`, PostgreSQL 17, PVC de **2 GiB** sobre `local-path`. La imagen se validará contra GHCR antes de aplicar.
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

## Aprovisionamiento de bases privadas (candidato)

1. Validar por dry-run de servidor los dos nuevos manifiestos antes de aprobarlos. No aplicar el directorio completo.
2. Aplicar exclusivamente 03-roles.yaml. Para ambos recursos exigir status.applied=true y status.observedGeneration igual a metadata.generation; confirmar por SQL la existencia de los roles sin LOGIN ni privilegios administrativos.
3. Aplicar exclusivamente 04-databases.yaml. Exigir las mismas comprobaciones de generación/applied; verificar por SQL los propietarios de ambas bases.
4. Antes de habilitar LOGIN, revocar CONNECT de PUBLIC en ambas bases sintéticas y verificar los permisos efectivos de ambos roles; confirmar que los roles no tienen acceso cruzado. No cambiar plantillas globales ni permisos de aplicaciones reales.
5. En un paso posterior, generar secretos diferentes fuera de Git, asociarlos y habilitar LOGIN declarativamente. Verificar reconciliación de la revisión actual y del Secret, conexiones TCP autenticadas positivas y negativas, y escrituras sintéticas. El manifiesto inicial NOLOGIN no debe reaplicarse después de activar roles; registrar el estado final declarativo antes del cierre.
6. Probar persistencia tras recreación controlada del Pod conservando el PVC. Una instancia no proporciona HA. Esta prueba es aislamiento entre servicios dentro de un tenant simulado, no entre tenants.

El worker futuro puede usar los estados de los CRD como señales de reconciliación, pero no como prueba suficiente de conectividad o autorización. Los ACL de CONNECT requieren una operación adicional a los campos observados del CRD Database.

## Riesgos y pendientes

- Kubernetes 1.37 probado por upstream, pero no oficialmente soportado por CloudNativePG 1.30: el PASS de instalación no demuestra compatibilidad de bases.
- `local-path` guarda volúmenes en un solo nodo WSL2 y su política Delete exige control de destrucción.
- La creación inicial de los Secret y la resolución dinámica de credenciales tenant×servicio siguen sin contrato aprobado con Identity/Clinical.
- La API pública de Platform no tendrá permisos Kubernetes. RBAC mínimo del trabajador se diseñará con evidencia de operaciones requeridas.
- Los estados reales de aprovisionamiento y transición a ACTIVA no se determinan desde infraestructura.

**Éxito de DB-01C** requiere evidencia real de PostgreSQL, persistencia, bases/roles aislados y recursos, no solo que Helm o Kubernetes acepten manifiestos.
