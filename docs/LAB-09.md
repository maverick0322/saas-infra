
# LAB-09 — GitOps de Platform con Argo CD

**Fecha:** 21 de noviembre de 2026
**Estado técnico:** PASS LOCAL
**Argo CD:** v3.6.2
**Kubernetes:** K3s v1.37.2+k3s1

## 1. Objetivo

Incorporar el Deployment y Service existentes de saas-platform a Argo CD y comprobar la sincronización automática de cambios publicados en Git.

Se utilizaron únicamente manifiestos YAML, sin Helm ni Kustomize.

## 2. Configuración

- Repositorio: https://github.com/maverick0322/saas-infra
- Rama observada: main.
- Application: argocd/saas-platform.
- Directorio de origen: k8s/apps/platform.
- Namespace de destino: saas-vital.
- Deployment existente: saas-platform.
- Service existente: saas-platform.
- Imagen: GHCR fijada por digest SHA-256.

Política de sincronización:

```yaml
syncPolicy:
  automated:
    enabled: true
    prune: false
    selfHeal: false
```

La Application fue creada inicialmente mediante kubectl apply, utilizando una definición ya fusionada en Git.

La activación posterior de Auto-Sync también requirió aplicar manualmente la configuración de la propia Application, porque Argo CD no administra todavía ese manifiesto.

## 3. Adopción inicial

PR #3: https://github.com/maverick0322/saas-infra/pull/3

Se creó la primera Application declarativa.

Argo CD reconoció los dos recursos existentes:

- Deployment/saas-platform.
- Service/saas-platform.

El estado inicial fue OutOfSync / Healthy.

Las diferencias observadas correspondían a anotaciones de seguimiento de Argo CD.

Se realizó una sincronización manual, con resultado Synced / Healthy y operación Succeeded.

No hubo cambios de imagen, reinicios del Pod ni nuevas revisiones del Deployment.

## 4. Activación de sincronización automática

PR #4: https://github.com/maverick0322/saas-infra/pull/4

Se habilitó Auto-Sync con prune y selfHeal deshabilitados.

La Application se actualizó manualmente después del merge.

Se verificó la configuración efectiva y el estado Synced / Healthy.

## 5. Prueba de propagación automática

PR #5: https://github.com/maverick0322/saas-infra/pull/5

Se agregó la anotación temporal:

```yaml
metadata:
  annotations:
    lab.saas-vital/gitops-validation: "enabled"
```

La anotación se agregó al metadata superior del Deployment, fuera de spec.template.

Commit de merge aplicado:

e4366824e696ac6274a603fa38b70f8f42670c2f

Argo CD detectó y sincronizó automáticamente esa revisión de main.

Se comprobó que la anotación apareció en Kubernetes con valor enabled, sin ejecutar kubectl apply ni iniciar una sincronización manual.

El Pod conservó su identidad, cero reinicios y la revisión 1 del Deployment.

**Resultado:** PASS.

## 6. Prueba de reversión declarativa

PR #6: https://github.com/maverick0322/saas-infra/pull/6

Se retiró la anotación temporal mediante un nuevo commit en Git.

Commit de merge aplicado:

add52580b6c41b652f424abc965ed9aa58b20614

Se observó primero la revisión Git anterior y posteriormente la nueva revisión procesada por Argo CD.

La Application alcanzó Synced / Healthy.

La consulta de la anotación temporal devolvió un resultado vacío.

El Deployment permaneció disponible, con el mismo Pod, cero reinicios y revisión 1.

**Resultado:** PASS.

Esta prueba corresponde a una reversión de metadatos declarativos, no a un rollback de imagen.

## 7. Comprobaciones

Comprobar estado de la Application:

```bash
sudo k3s kubectl -n argocd get application saas-platform
```

Comprobar revisión sincronizada:

```bash
sudo k3s kubectl -n argocd get application saas-platform \
  -o jsonpath='{.status.sync.revision}{"\n"}'
```

Comprobar estado del Deployment:

```bash
sudo k3s kubectl -n saas-vital get deployments,pods,services
sudo k3s kubectl -n saas-vital rollout history deployment/saas-platform
```

Comprobar ausencia de la anotación temporal:

```bash
sudo k3s kubectl -n saas-vital get deployment saas-platform \
  -o jsonpath='{.metadata.annotations.lab\.saas-vital/gitops-validation}{"\n"}'
```

## 8. Limitaciones y pendientes

- La instalación funciona sobre un único nodo K3s.
- No existe alta disponibilidad.
- No se ha probado una actualización o reversión real de la imagen GHCR.
- No se ha probado recuperación ante fallos.
- No se habilitó autocorrección de cambios directos al clúster.
- No se habilitó eliminación automática de recursos.
- El manifiesto de Application aún requiere administración manual.
- Se utiliza el AppProject default, sin restricciones específicas de mínimo privilegio.
- No se han configurado webhooks de GitHub; Argo CD detectó los cambios mediante su consulta periódica.
- El segundo nodo físico continúa pendiente.
- Las pruebas utilizaron metadatos y una aplicación técnica mínima, sin datos clínicos.

## 9. Conclusión

LAB-09 obtuvo PASS LOCAL.

Se validó la adopción de recursos Kubernetes existentes, la sincronización manual inicial y la sincronización automática de cambios declarativos desde Git.

También se verificó la reversión automática de una anotación, sin reemplazar el Pod ni modificar la imagen de Platform.

Git constituye la fuente declarativa de los manifiestos administrados de Platform.

Quedan pendientes las pruebas con nuevas versiones de imágenes y las mejoras de operación y seguridad del entorno.
