

\# DEV-01 — Entorno de desarrollo local con Docker Compose



\*\*Fecha:\*\* 21 de noviembre de 2026

\*\*Estado técnico:\*\* PASS LOCAL — EVICTUS

\*\*Validación en Surface:\*\* Pendiente

\*\*Repositorio:\*\* saas-infra



\## 1. Objetivo



Preparar una configuración de Docker Compose reutilizable para ejecutar y comprobar microservicios del proyecto SaaS Vital durante el desarrollo local.



Cada integrante debe poder trabajar en su computadora sin depender de la disponibilidad del clúster Kubernetes del otro.



Este incremento utiliza Platform como primera aplicación de prueba. No incorpora todavía Identity, Clinical ni bases de datos.



\## 2. Separación de entornos



Se mantienen dos modalidades independientes:



\*\*Desarrollo local:\*\* Docker Compose, utilizando las imágenes y dependencias necesarias para cada microservicio.



\*\*Demostración académica:\*\* K3s y Argo CD, con EVICTUS como servidor y Surface como futuro agente.



Docker Compose no administra los recursos de Kubernetes. K3s tampoco depende de que Docker Desktop permanezca encendido.



\## 3. Prerrequisitos



\- Windows con Docker Desktop configurado para contenedores Linux, o un entorno compatible con Docker Compose.

\- Docker Engine disponible.

\- Docker Compose.

\- Git.

\- Acceso a GHCR para descargar la imagen de Platform.



No es necesario instalar K3s, Ansible ni Argo CD para ejecutar este Compose.



Versiones observadas en EVICTUS:



\- Docker Desktop: 4.95.0.

\- Docker Engine: 29.8.2.

\- Docker Compose: v5.6.0.

\- Plataforma del motor: linux/amd64.



Estas son versiones de referencia del laboratorio, no requisitos mínimos universales.



\## 4. Configuración



Archivo:



`compose/compose.yaml`



Proyecto Compose:



`saas-vital-local`



Servicio disponible:



`platform`



Imagen:



`ghcr.io/maverick0322/saas-platform@sha256:104c651cd22324935c161314a28aeb7fd8728090710ad355b26cd37842b00746`



Puerto publicado:



`127.0.0.1:18080` → `8080` del contenedor.



El puerto se publica solamente en la interfaz local. No se requiere acceso externo.



La configuración no utiliza bases de datos, volúmenes persistentes, redes con subredes fijas ni secretos.



\## 5. Procedimiento de ejecución



Los comandos siguientes se ejecutan desde PowerShell, en la raíz de saas-infra.



\### 5.1. Clonar el repositorio



```powershell

git clone https://github.com/maverick0322/saas-infra.git

cd saas-infra

```



En una copia existente, actualizar la rama correspondiente antes de ejecutar las pruebas.



\### 5.2. Validar Compose



Con Docker Compose instalado:



```powershell

docker compose -f compose/compose.yaml config --quiet

```



La ausencia de errores indica que la configuración es válida. No inicia contenedores.



\### 5.3. Arrancar Platform



Con Docker Engine en ejecución:



```powershell

docker compose -f compose/compose.yaml up -d platform

```



Consultar el estado:



```powershell

docker compose -f compose/compose.yaml ps

```



\### 5.4. Verificar endpoints



```powershell

Invoke-RestMethod http://127.0.0.1:18080/health/live

Invoke-RestMethod http://127.0.0.1:18080/health/ready

```



Ambas rutas deben responder con estado `Healthy`.



Son health checks de proceso; no comprueban todavía conexiones con PostgreSQL, Identity ni lógica SaaS.



\### 5.5. Detener el entorno local



```powershell

docker compose -f compose/compose.yaml down

```



Este comando detiene los recursos del proyecto Compose. No elimina el Deployment independiente que exista en K3s.



\## 6. Evidencias de validación en EVICTUS



\### Validación de configuración



`docker compose config --quiet`: PASS.



`git diff --cached --check`: PASS.



\### Ejecución local



La imagen oficial de Platform se descargó correctamente desde GHCR.



Se creó la red `saas-vital-local\_default` y arrancó el contenedor `saas-vital-local-platform-1`.



El contenedor publicó exclusivamente:



`127.0.0.1:18080->8080/tcp`



\### Health checks



`/health/live`: Healthy.



`/health/ready`: Healthy.



\### Coexistencia con Kubernetes



Mientras Platform se ejecutaba mediante Compose, se comprobó que el clúster K3s continuaba funcionando.



Resultados:



\- Application `saas-platform`: Synced / Healthy.

\- Deployment Kubernetes: 1/1 Ready.

\- Pod original: Running.

\- Reinicios del Pod: 0.

\- Memoria del nodo reportada por Kubernetes: 2040 MiB.

\- CPU del nodo reportada por Kubernetes: 203m.



La prueba confirma coexistencia funcional, no rendimiento bajo carga.



\### Detención independiente



Se ejecutó `docker compose down`.



El contenedor y la red del proyecto local fueron eliminados.



La consulta posterior de Compose no mostró contenedores del proyecto en ejecución.



El Deployment de Kubernetes siguió 1/1 Ready, con el mismo Pod y cero reinicios.



Argo CD permaneció Synced / Healthy.



\*\*Resultado:\*\* PASS.



\## 7. Observaciones de recursos



Durante la prueba, el contenedor local Platform reportó:



\- CPU: 0.01%.

\- Memoria: 22.35 MiB.



Sin embargo, estas cifras no representan el consumo total de Docker Desktop ni de WSL2.



Windows reportó 0.44 GiB de RAM libre durante la ejecución de Compose y 0.37 GiB después de detener el proyecto.



Posteriormente, tras cerrar Docker Desktop y reducir otros procesos activos, Windows reportó 2.02 GiB libres.



La memoria de `VmmemWSL` permaneció alrededor de 4.45 GiB, mientras `saas-k3s-server` continuaba ejecutándose.



No se determinó cuánto consumo correspondía individualmente a cada componente.



Se recomienda mantener Docker Desktop encendido solo cuando sea necesario y evaluar los recursos antes de ejecutar simultáneamente múltiples servicios.



No se realizaron pruebas de carga ni ajustes de configuración de memoria.



\## 8. Reproducibilidad para el segundo integrante



El archivo Compose está diseñado para utilizarse en otra computadora con Docker Compose.



No depende de direcciones IP específicas de EVICTUS, claves SSH, Ansible ni de un clúster Kubernetes.



La Surface todavía no ha ejecutado este procedimiento, por lo que la reproducibilidad entre ambos equipos sigue pendiente de comprobación.



Identity podrá incorporarse después de acordar sus requisitos mínimos de ejecución: imagen, puertos, health checks, configuración y dependencias.



No se crearán servicios ficticios para anticipar esa integración.



\## 9. Limitaciones



\- Solo se validó Platform.

\- La imagen ejecutada corresponde a una API técnica mínima, sin lógica funcional de negocio.

\- No existen bases de datos en esta configuración.

\- Identity y Clinical todavía no están integrados.

\- No se han realizado pruebas entre múltiples microservicios.

\- No existe una prueba de carga sostenida.

\- No se ha validado la ejecución en Surface.

\- La convivencia con K3s depende de los recursos físicos disponibles.

\- La topología Kubernetes de dos nodos pertenece a LAB-10.



\## 10. Conclusión



DEV-01 obtiene PASS LOCAL para el alcance funcional probado en EVICTUS.



Se demostró que la imagen de Platform puede ejecutarse mediante Docker Compose, responder a sus health checks y coexistir con el clúster K3s sin modificarlo.



También se comprobó que detener el entorno Compose no interrumpe el despliegue de Platform administrado mediante Argo CD.



Queda preparada una base local sencilla para incorporar progresivamente otros microservicios y comprobar su reproducción en la segunda computadora.



No se modifica la arquitectura ni la topología prevista para la demostración académica.
