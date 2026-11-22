# LAB-07 — Primer despliegue de Platform en Kubernetes

**Fecha:** 21 de noviembre de 2026

**Estado técnico:** PASS LOCAL

**Repositorio:** saas-infra

**Rama de trabajo:** `feature/lab-07-platform-deployment`

## 1. Objetivo

Desplegar por primera vez un microservicio propio del proyecto SaaS Vital en K3s, utilizando la imagen oficial publicada mediante el pipeline de saas-platform.

El laboratorio valida manifiestos Kubernetes, ejecución del contenedor, comunicación interna, health checks y recuperación automática de Pods.

## 2. Dependencias

El despliegue utiliza los resultados de los incrementos APP-00/LAB-05 y APP-01/LAB-06, implementados en el repositorio saas-platform.

Imagen utilizada:

`ghcr.io/maverick0322/saas-platform@sha256:104c651cd22324935c161314a28aeb7fd8728090710ad355b26cd37842b00746`

Commit de origen de la publicación:

`3c46822c722f889cd715daa6d05beb58e143fb61`

La imagen fue publicada y validada mediante GitHub Actions y GHCR. No se reconstruyó ni modificó desde saas-infra.

## 3. Entorno

- Anfitrión: EVICTUS, Windows 11 con WSL2.
- Distribución: Ubuntu 24.04.5 LTS.
- Kubernetes: K3s v1.37.2+k3s1.
- Nodo: saas-k3s-server.
- Topología: un nodo control-plane.
- Namespace de aplicaciones: saas-vital.
- Runtime: containerd.

## 4. Recursos declarativos

Se incorporaron tres archivos:

- `k8s/namespaces/saas-vital.yaml`: namespace de las aplicaciones.
- `k8s/apps/platform/deployment.yaml`: Deployment del microservicio.
- `k8s/apps/platform/service.yaml`: Service interno de Platform.

Configuración principal:

| Parámetro | Valor |
|---|---|
| Deployment | saas-platform |
| Réplicas | 1 |
| Service | ClusterIP |
| Puerto | TCP 8080 |
| CPU request | 50m |
| CPU limit | 500m |
| RAM request | 128Mi |
| RAM limit | 256Mi |
| Liveness | /health/live |
| Readiness | /health/ready |
| Imagen | Referencia inmutable por digest |

El Deployment incluye ejecución no root, seccomp RuntimeDefault, eliminación de capacidades Linux y deshabilitación del montaje automático del token de ServiceAccount.

Estos mecanismos están declarados en los manifiestos. No se realizó una auditoría de seguridad integral del contenedor.

## 5. Procedimiento de despliegue

Desde la raíz de saas-infra:

```bash
sudo k3s kubectl apply --dry-run=server -f k8s/namespaces/saas-vital.yaml
sudo k3s kubectl apply -f k8s/namespaces/saas-vital.yaml

sudo k3s kubectl apply --dry-run=server -f k8s/apps/platform/
sudo k3s kubectl apply -f k8s/apps/platform/

sudo k3s kubectl -n saas-vital rollout status deployment/saas-platform --timeout=180s
```

El despliegue finalizó satisfactoriamente con una réplica disponible.

## 6. Evidencias de validación

| Prueba | Resultado |
|---|---|
| Preflight del clúster | PASS |
| API Kubernetes disponible | PASS |
| Validación de manifiestos | PASS |
| Deployment 1/1 disponible | PASS |
| Pod Running y Ready | PASS |
| Service ClusterIP | PASS |
| EndpointSlice asociado | PASS |
| Digest declarado coincide con imageID | PASS |
| Resolución DNS interna | PASS |
| HTTP /health/live | PASS |
| HTTP /health/ready | PASS |
| Recuperación automática del Pod | PASS |
| Reaplicación sin cambios | PASS |

### Identidad de imagen

Se consultó la referencia de imagen del Deployment y el imageID del contenedor en ejecución.

Ambos coincidieron con el digest oficial publicado en GHCR.

### Comunicación HTTP interna

Un Pod temporal BusyBox resolvió correctamente:

`saas-platform.saas-vital.svc.cluster.local`

El DNS devolvió la dirección ClusterIP `10.43.104.92`.

Los endpoints respondieron con estado Healthy y alcance process-only.

### Recursos observados

Con la API mínima en ejecución:

- CPU: 1m.
- RAM: 30Mi.

Son mediciones iniciales en condiciones de carga mínima. No justifican todavía modificar las solicitudes y límites establecidos.

### Recuperación automática

Pod original eliminado:

`saas-platform-5c6c59b4c5-vcwq2`

Nuevo Pod creado por Kubernetes:

`saas-platform-5c6c59b4c5-l8sxg`

El nuevo Pod alcanzó Running y Ready, y el Deployment volvió a tener 1/1 réplicas disponibles.

La IP del Pod cambió de `10.42.0.18` a `10.42.0.20`.

El EndpointSlice se actualizó y un nuevo cliente temporal obtuvo respuestas Healthy de ambos endpoints mediante el Service.

La prueba demuestra recuperación del Pod en un nodo disponible; no demuestra alta disponibilidad frente a fallos del nodo.

### Reaplicación

La reaplicación de los tres manifiestos devolvió `unchanged`.

Los Pods temporales de prueba fueron eliminados. El Deployment y el Service de Platform permanecen instalados.

## 7. Limitaciones

- Platform todavía no contiene lógica de negocio funcional.
- Los health checks son process-only.
- No se desplegaron bases de datos ni volúmenes.
- No se validó aislamiento multitenant.
- No se realizaron pruebas de carga.
- No se probó recuperación ante pérdida del nodo.
- No se incorporó Surface.
- Argo CD y GitOps todavía no están configurados.
- No se habilitó exposición externa del Service.

## 8. Conclusión

LAB-07 demuestra que el flujo de CI y publicación en GHCR puede entregar una imagen oficial que se ejecuta satisfactoriamente en el clúster académico K3s.

Se verificaron disponibilidad, identidad del artefacto, comunicación interna y recuperación automática del Pod.

La configuración queda preparada para introducir Argo CD en LAB-08, sin modificar la aplicación ni duplicar los workflows existentes.
