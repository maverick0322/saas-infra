
# LAB-08 — Instalación y validación local de Argo CD

**Fecha:** 21 de noviembre de 2026
**Estado técnico:** PASS LOCAL
**Versión:** Argo CD v3.6.2

## 1. Objetivo

Instalar Argo CD en el clúster académico K3s, validar sus componentes y comprobar el acceso seguro a su interfaz web.

No se incorporan todavía aplicaciones a GitOps.

## 2. Entorno

- Anfitrión: EVICTUS, Windows 11 con WSL2.
- Kubernetes: K3s v1.37.2+k3s1.
- Nodo: saas-k3s-server.
- Modalidad: instalación estándar no-HA.
- Namespace: argocd.
- Microservicio coexistente: saas-platform.

## 3. Artefacto de instalación

Se utilizó el manifiesto oficial:

https://raw.githubusercontent.com/argoproj/argo-cd/v3.6.2/manifests/install.yaml

SHA-256 observado:

c948064d38294d857712ef78444768749a14293613098932f8533e37d067a2a2

No se incorporó una copia completa del manifiesto oficial al repositorio.

## 4. Instalación reproducible

Descargar el manifiesto fijado:

```bash
curl -fL --retry 3 \
  -o /tmp/saas-argocd-v3.6.2-install.yaml \
  https://raw.githubusercontent.com/argoproj/argo-cd/v3.6.2/manifests/install.yaml
```

Comprobar su integridad frente al hash registrado:

```bash
echo "c948064d38294d857712ef78444768749a14293613098932f8533e37d067a2a2  /tmp/saas-argocd-v3.6.2-install.yaml" | sha256sum -c -
```

Crear el namespace:

```bash
sudo k3s kubectl create namespace argocd
```

Aplicar el manifiesto:

```bash
sudo k3s kubectl apply \
  -n argocd \
  --server-side \
  --force-conflicts \
  -f /tmp/saas-argocd-v3.6.2-install.yaml
```

La creación del namespace es necesaria solo cuando todavía no existe.

## 5. Validación

Los seis Deployments de Argo CD alcanzaron 1/1 Available.

El StatefulSet argocd-application-controller alcanzó 1/1 Ready.

Los siete Pods se encontraron Running, sin reinicios.

La aplicación saas-platform continuó disponible con una réplica 1/1.

### Recursos observados

- Memoria del nodo reportada por Kubernetes: 1681 MiB.
- CPU del nodo: 164m.
- Memoria sumada de los Pods de Argo CD: 148 MiB.
- Memoria disponible reportada por WSL2: 5.8 GiB.
- Swap utilizada: 0 B.

Las mediciones corresponden a condiciones de reposo y no son pruebas de carga.

## 6. Acceso local

Se utilizó port-forward sin exponer Argo CD en la red externa:

```bash
sudo k3s kubectl -n argocd port-forward \
  --address 127.0.0.1 \
  svc/argocd-server 8088:443
```

Dirección local: https://localhost:8088

Se confirmó acceso a la interfaz e inicio de sesión.

No se almacenaron contraseñas en el repositorio.

## 7. Estado GitOps

La consulta:

```bash
sudo k3s kubectl -n argocd get applications.argoproj.io
```

Devolvió:

No resources found in argocd namespace.

Por tanto, Argo CD todavía no administra saas-platform.

La creación de la primera Application y la sincronización declarativa corresponden a LAB-09.

## 8. Limitaciones

- Un único nodo Kubernetes.
- Sin alta disponibilidad.
- Sin sincronización GitOps todavía.
- Sin pruebas de rendimiento bajo carga.
- Sin exposición pública de Argo CD.
- La instalación estándar contiene permisos amplios a nivel de clúster.
- La incorporación de Surface continúa pendiente.

## 9. Conclusión

Argo CD v3.6.2 fue instalado y validado localmente junto con saas-platform.

Los componentes alcanzaron disponibilidad y el acceso web funcionó mediante port-forward local.

La infraestructura queda preparada para LAB-09: adopción declarativa de Platform mediante GitOps, sin duplicar el Deployment existente.
