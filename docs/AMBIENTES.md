
# Ambientes de desarrollo y demostración

## 1. Objetivo

Permitir el desarrollo independiente de dos integrantes
y la integración de sus microservicios, así como el
despliegue conjunto del SaaS en Kubernetes.

Solo se utilizarán datos ficticios.

## 2. Integración local

Tecnología: Docker Compose.

Cada desarrollador podrá ejecutar los servicios
necesarios en su propia computadora, sin depender
de que la otra esté encendida.

Se podrán utilizar:
- Servicios .NET ejecutados localmente.
- Imágenes de contenedores.
- Bases de datos de prueba.
- Configuración local no sensible.
- Pruebas de integración y contratos.

No será necesario levantar todos los servicios
para probar una funcionalidad específica.

Directorio previsto: compose/

## 3. Despliegue académico

Tecnología: K3s.

Topología objetivo:

- EVICTUS: Ubuntu WSL2 con K3s Server.
- Surface: Ubuntu WSL2 con K3s Agent.
- Ambos nodos conectados mediante una LAN accesible.

Los nodos utilizarán la misma instalación lógica
de Kubernetes, no clústeres independientes.

Directorio previsto: k8s/

La disponibilidad del servidor dependerá de EVICTUS.
Dos nodos no proporcionan alta disponibilidad.

## 4. Automatización

- Ansible: preparación de nodos e instalación K3s.
- GitHub Actions: validación, construcción e imágenes.
- GHCR: registro de imágenes.
- Argo CD: despliegue GitOps cuando esté preparado K3s.

Cada herramienta se incorporará incrementalmente.

## 5. Portabilidad

Las direcciones IP, interfaces y otros parámetros
dependientes del equipo se configurarán por entorno.

No se fijarán las direcciones de la red doméstica
directamente en los playbooks generales.

Cambiar de LAN puede requerir reconfigurar K3s,
sus certificados y la conexión de los agentes.

## 6. Restricciones y pendientes

- Surface todavía no incorporada al clúster.
- Distribución de servicios y bases de datos pendiente.
- Red final de demostración pendiente de probar.
- Límites de RAM y CPU sujetos a medición.
- Sin datos clínicos reales.
- Sin despliegue de producción comercial.

## 7. Estado

Estrategia de entornos aceptada por el equipo.

K3s de un nodo sobre WSL2 instalado y validado
localmente en EVICTUS, incluyendo recuperacion
tras reiniciar completamente WSL2.

WSL2 utiliza actualmente una red NAT.
La conectividad entre computadoras y la red
definitiva del cluster permanecen pendientes.

Docker Compose y el despliegue distribuido
de dos nodos todavia no estan implementados.

La coexistencia bajo carga de Docker Desktop
y K3s permanece pendiente de evaluacion.

Consultar LAB-04.md para conocer los resultados
de la validacion local.
