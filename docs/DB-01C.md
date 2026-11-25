# DB-01C — Primera instancia PostgreSQL sintética (propuesta)

**Estado:** CANDIDATO; no aplicado al clúster.
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
- Recursos de laboratorio posteriores para `identity_lab` y `clinical_lab`: **se diseñarán tras verificar el clúster y los CRD exactos**, sin credenciales en Git.
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

## Riesgos y pendientes

- Kubernetes 1.37 probado por upstream, pero no oficialmente soportado por CloudNativePG 1.30: el PASS de instalación no demuestra compatibilidad de bases.
- `local-path` guarda volúmenes en un solo nodo WSL2 y su política Delete exige control de destrucción.
- La creación inicial de los Secret y la resolución dinámica de credenciales tenant×servicio siguen sin contrato aprobado con Identity/Clinical.
- La API pública de Platform no tendrá permisos Kubernetes. RBAC mínimo del trabajador se diseñará con evidencia de operaciones requeridas.
- Los estados reales de aprovisionamiento y transición a ACTIVA no se determinan desde infraestructura.

**Éxito de DB-01C** requiere evidencia real de PostgreSQL, persistencia, bases/roles aislados y recursos, no solo que Helm o Kubernetes acepten manifiestos.
