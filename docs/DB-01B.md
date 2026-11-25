# DB-01B — CloudNativePG por Argo CD (candidato)

**Estado:** pendiente de revisión, instalación y pruebas; no instalado.
**Base:** DB-01A y ADR-0002 / ADR-0003 en saas-platform/develop.

## Alcance

- Chart oficial cloudnative-pg 0.29.2, operador 1.30.2, versión fijada en k8s/argocd/apps/cloudnativepg.yaml.
- Argo CD renderiza el chart; no se mantiene copia masiva del manifiesto CRD ni una instalación Helm paralela.
- Un controlador en cnpg-system, vigilancia a escala de clúster para futuros namespaces de tenants, sin dar permisos a Platform API.
- CRD instalados, una réplica, PodMonitor deshabilitado, recursos CPU 50m/500m y RAM 128Mi/512Mi (candidatos a medir).
- La Application no pertenece a la App saas-platform; sus recursos GitOps son solo el operador, no clústeres PostgreSQL por tenant.
- No se implementan provisionador, bases de tenants, credenciales ni RBAC del trabajador de Platform en DB-01B.

## Preflight y riesgos

- K3s 1.37.2 está probado pero no oficialmente soportado por CNPG 1.30: la compatibilidad debe demostrarse.
- local-path almacena datos en el nodo y tiene ReclaimPolicy Delete, sin ampliación de PVC; no cambiarlo en DB-01B.
- La instalación de CRD y webhooks cambia recursos cluster-wide. Inspeccionar el chart renderizado antes de aplicar.
- La versión del chart está fijada, pero la fuente Helm sigue siendo una dependencia externa.
- prune:false reduce poda automática, no evita borrados manuales ni sustituye respaldos.

## Operación controlada

1. Revisar PR y verificar que saas-infra/main y la Application de Platform no fueron alteradas.
2. Comprobar salud y memoria libre de K3s, Argo CD y saas-platform.
3. Revisar recursos generados por el chart, especialmente CRD, webhook, ServiceAccount y ClusterRole.
4. Tras aprobación explícita y merge, actualizar saas-infra/main y aplicar manualmente k8s/argocd/apps/cloudnativepg.yaml desde el nodo K3s.
5. La Application tiene auto-sync habilitado, por tanto la aplicación manual de su definición inicia la instalación.

## Aceptación DB-01B

- Argo CD informa cloudnativepg Synced/Healthy con chart 0.29.2 y operador 1.30.2.
- Deployment del operador disponible, Pod estable y sin reinicios reiterados.
- CRD de postgresql.cnpg.io establecidos, incluidos Cluster, Database y DatabaseRole.
- Webhooks y ServiceAccount/RBAC del operador presentes y sin fallos evidentes.
- saas-platform permanece Synced/Healthy y su Deployment 1/1.
- Se registran valores kubectl top, RAM libre, incidencias y versión de los componentes.
- No se declaran contratos ni credenciales para Identity/Clinical.

Comandos de inspección en el nodo K3s (solo lectura):

```bash
sudo k3s kubectl -n argocd get applications
sudo k3s kubectl -n cnpg-system get deployments,pods,services
sudo k3s kubectl get crd | grep postgresql.cnpg.io
sudo k3s kubectl -n saas-vital get deployments,pods
sudo k3s kubectl top nodes
```

## Recuperación y DB-01C

En un fallo, detener el incremento y conservar evidencias. No eliminar CRD u operador a ciegas: una eliminación de CRD puede eliminar los recursos PostgreSQL asociados. La limpieza requiere un plan y comprobar antes recursos Cluster, PVC y secretos.

Solo después del PASS de DB-01B, DB-01C creará una primaria PostgreSQL sintética fuera de la Application del operador y verificará PVC, persistencia, bases privadas, roles, aislamiento negativo, recuperación y consumo. El trabajador y sus permisos de aprovisionamiento se diseñarán con Platform; su API pública no recibirá permisos de Kubernetes.
