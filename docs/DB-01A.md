
# DB-01A — Preflight de CloudNativePG

**Fecha:** 23 de noviembre de 2026
**Estado:** PASS DIAGNÓSTICO
**Repositorio:** saas-infra
**Alcance:** Evaluación técnica sin instalación de CloudNativePG

## 1. Objetivo

Evaluar las condiciones iniciales del laboratorio K3s para incorporar CloudNativePG como administrador de instancias PostgreSQL independientes por clínica.

Este incremento no instala operadores, bases de datos ni recursos de aprovisionamiento.

## 2. Decisiones arquitectónicas recibidas

El equipo aprobó:

- Una instancia PostgreSQL independiente por clínica.
- Bases privadas por microservicio dentro de cada instancia.
- Aplicaciones compartidas entre tenants.
- CloudNativePG para administrar PostgreSQL en Kubernetes.
- Una instancia primaria por tenant, sin réplicas HA en el laboratorio.

Se prevén cuatro o cinco tenants sintéticos, sujetos a validación de capacidad.

La infraestructura administrará mediante GitOps la instalación estática del operador.

El futuro aprovisionamiento dinámico de clústeres de tenants corresponderá a Platform, con permisos y contratos todavía pendientes.

## 3. Entorno evaluado

- Anfitrión: EVICTUS, Windows 11.
- Kubernetes: K3s v1.37.2+k3s1.
- Distribución: Ubuntu sobre WSL2.
- Nodo: saas-k3s-server.
- Topología: un nodo control-plane.
- Argo CD: instalado y operativo.
- Platform: Deployment disponible, 1/1.

La incorporación de Surface como segundo nodo físico continúa pendiente.

## 4. Compatibilidad

CloudNativePG 1.30.x es la rama candidata del análisis.

Según la información de compatibilidad revisada, Kubernetes 1.37 figura como probado, pero no oficialmente soportado por dicha rama.

Esta condición constituye un riesgo que deberá verificarse nuevamente al fijar una versión exacta del operador.

No se ha probado todavía CloudNativePG sobre el K3s del laboratorio.

No se autoriza modificar la versión actual de K3s como parte de este incremento.

## 5. Almacenamiento

StorageClass predeterminada: local-path.

Características observadas:

- Provisioner: rancher.io/local-path.
- ReclaimPolicy: Delete.
- VolumeBindingMode: WaitForFirstConsumer.
- AllowVolumeExpansion: false.
- PV existentes: ninguno.
- PVC existentes: ninguno.

Ruta efectiva del provisionador:

/var/lib/rancher/k3s/storage

Los datos se alojarán en el almacenamiento local del nodo Kubernetes, dentro del sistema de archivos de la distribución WSL2.

Este almacenamiento no es compartido automáticamente con Surface y no proporciona alta disponibilidad.

La política Delete requiere precauciones para evitar pérdidas accidentales al eliminar PVC.

## 6. Capacidad observada

### Almacenamiento

Unidad C: de Windows:

- Capacidad total: 952.51 GiB.
- Espacio libre: 146.15 GiB.

El sistema de archivos Linux reportó aproximadamente 952 GiB disponibles sobre /dev/sdf.

Esta cifra corresponde al almacenamiento virtual presentado a WSL2 y no debe confundirse con el espacio físico libre de Windows.

La ubicación exacta del archivo VHDX no fue comprobada en este incremento.

### Memoria y CPU

Mediciones del nodo:

- RAM total visible en WSL2: 7.6 GiB.
- RAM disponible: 5.5 GiB.
- Swap utilizada: 0.
- Memoria reportada por Kubernetes: 1589 MiB.
- CPU reportada por Kubernetes: 154m.

Las mediciones corresponden a una carga ligera.

No demuestran capacidad suficiente para ejecutar simultáneamente cinco instancias PostgreSQL y todos los microservicios.

## 7. Componentes existentes

Se comprobó que:

- K3s estaba activo y el nodo Ready.
- Argo CD mostraba Platform como Synced / Healthy.
- Platform estaba disponible con una réplica.
- No existían CRD de CloudNativePG.
- CloudNativePG no estaba instalado.
- local-path-provisioner estaba disponible.

Platform registró un reinicio con motivo Unknown y código de salida 255 después del arranque del entorno.

No se confirmó su causa ni se observó un estado OOMKilled.

## 8. Riesgos y restricciones

- Kubernetes 1.37 fuera del soporte oficial considerado para CloudNativePG 1.30.
- Almacenamiento local vinculado al nodo.
- Política de eliminación Delete para volúmenes.
- Ausencia de expansión automática de PVC.
- Ausencia de backups y restauraciones validados.
- Recursos físicos compartidos con Windows y otros procesos.
- Sin pruebas de carga con múltiples instancias PostgreSQL.
- Sin alta disponibilidad de PostgreSQL.
- Credenciales, RBAC y contratos de aprovisionamiento pendientes.

No se utilizarán datos clínicos reales.

## 9. Dictamen

**PASS DIAGNÓSTICO, condicionado a pruebas posteriores.**

No se identificaron impedimentos inmediatos para evaluar una primera instancia PostgreSQL sintética.

El resultado no equivale a compatibilidad operativa validada ni autoriza desplegar cuatro o cinco tenants.

No se modificaron K3s, Argo CD, Platform, almacenamiento ni recursos Kubernetes.

## 10. Próximos incrementos propuestos

### DB-01B — Instalación del operador

- Fijar versión y procedencia verificable del artefacto.
- Preparar instalación versionada mediante GitOps.
- Separar operador y clústeres dinámicos.
- Comprobar CRD, webhooks, controlador y RBAC.
- Validar compatibilidad real con K3s.
- Preservar el despliegue existente de Platform.

### DB-01C — Primera instancia sintética

- Crear una instancia PostgreSQL de prueba con una primaria.
- Evaluar PVC, persistencia y recuperación.
- Probar bases y roles privados.
- Registrar memoria, CPU y almacenamiento.
- Verificar permisos negativos y aislamiento.
- Documentar los límites antes de aumentar tenants.

La implementación del aprovisionador de Platform queda fuera de estos incrementos.

## 11. Conclusión

La infraestructura actual proporciona condiciones iniciales favorables para experimentar con CloudNativePG de forma controlada.

Se recomienda continuar el desarrollo funcional de Platform y Clinical mientras se prepara la futura integración GitOps del operador.

DB-01A no introduce dependencias nuevas ni modifica los servicios desplegados.
