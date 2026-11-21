 # SaaS Infra

Infraestructura reproducible y prácticas DevOps para un proyecto académico de gestión clínica SaaS multitenant.

## Contexto

Proyecto desarrollado por dos integrantes durante ocho semanas. El objetivo es construir y evaluar una infraestructura que permita desplegar los componentes del SaaS mediante prácticas de automatización, integración continua y despliegue declarativo.

No es un entorno de producción ni está autorizado para almacenar información clínica real. Se utilizarán exclusivamente datos ficticios.

## Objetivos

- Preparar entornos Linux reproducibles.
- Automatizar su configuración mediante Ansible.
- Desplegar un clúster K3s de dos nodos físicos.
- Disponer de una alternativa funcional de un nodo.
- Construir y publicar imágenes mediante GitHub Actions y GHCR.
- Incorporar despliegues declarativos con Argo CD.
- Validar conectividad, recursos, recuperación y funcionamiento básico.

## Entorno de laboratorio

### Equipo principal: EVICTUS

- Windows 11 Home.
- Intel Core i7-13700H, 14 núcleos y 20 hilos.
- 16 GB de RAM.
- WSL 3.0.5, kernel Linux 6.18.58.1 y cgroups v2.
- Ubuntu: controlador Ansible.
- saas-k3s-server: distribución Ubuntu 24.04 dedicada a K3s Server.
- Docker Desktop disponible para integración local.
- Nodo Kubernetes instalado y validado localmente.

### Segundo equipo: Microsoft Surface

- Windows.
- Aproximadamente 12 GB de RAM.
- WSL 2.6.2 y kernel Linux 6.6.87.2.
- Ubuntu 24.04 con systemd y cgroups v2.
- WSL2 configurado inicialmente con 4 GB de RAM y 2 procesadores.
- Rol propuesto: K3s Agent.
- Preflight realizado.
- Instalación de K3s y conectividad entre nodos pendientes.

## Tecnologías

| Tecnología | Propósito | Estado |
|---|---|---|
| Git y GitHub | Versionamiento | Implementado |
| Ubuntu 24.04 | Entornos Linux | Validado localmente |
| VirtualBox | Virtualización tradicional | Suspendido por inestabilidad |
| WSL2 | Ejecución de nodos y controlador Ansible | Validado localmente |
| Ansible | Configuración automatizada | Implementado |
| K3s | Orquestación Kubernetes | Un nodo validado |
| containerd | Runtime de contenedores | Implementado |
| Docker Compose | Integración local | Pendiente |
| GitHub Actions | Integración continua | Pendiente |
| GHCR | Registro de imágenes | Pendiente |
| Argo CD | Despliegues GitOps | Pendiente |
| .NET 10 | Backend de microservicios | Tecnología confirmada |

## Topología

### Estado actual

EVICTUS ejecuta un clúster K3s de un nodo mediante WSL2.

La administración se realiza desde una distribución Ubuntu independiente mediante Ansible y SSH.

Se han validado:

- Instalación automatizada e idempotencia.
- Salud de los componentes de Kubernetes.
- Despliegue de contenedores.
- Resolución DNS interna.
- Comunicación mediante Services.
- Recuperación tras reiniciar la distribución.
- Recuperación tras detener completamente WSL2.

### Topología objetivo

- EVICTUS: Ubuntu WSL2 con K3s Server.
- Surface: Ubuntu WSL2 con K3s Agent.
- Comunicación mediante una red accesible entre ambos equipos.
- Un único clúster Kubernetes distribuido.

La incorporación del segundo nodo y la comunicación entre Pods ubicados en computadoras diferentes permanecen pendientes de validación.

Dos nodos con un solo K3s Server no proporcionan alta disponibilidad del plano de control.

## Estrategia de ambientes

El desarrollo y las pruebas de integración se realizarán de forma independiente mediante Docker Compose.

K3s se utilizará como entorno académico para el despliegue y la demostración conjunta de los microservicios.

Consultar [Ambientes de desarrollo y demostración](docs/AMBIENTES.md).

## Restricciones

- Recursos limitados de dos computadoras Windows.
- Evitar infraestructura innecesaria y sobreingeniería.
- Mantener el laboratorio reproducible y documentado.
- No introducir datos clínicos reales.
- No almacenar credenciales ni secretos en Git.
- Evaluar la separación de servidores de bases de datos conforme a la recomendación académica.
- Configurar por entorno los parámetros dependientes de la red.
- Priorizar aprendizaje y validaciones incrementales.

## Incrementos

| ID | Objetivo | Estado |
|---|---|---|
| INFRA-00 | Repositorio y documentación | Implementado |
| LAB-00 | Viabilidad y topología | Diagnóstico inicial realizado |
| LAB-01 | Primer nodo Linux y SSH | Validado |
| LAB-02 | Automatización con Ansible | Validado |
| LAB-03 | Configuración y experimentación de red | Validado en el entorno original |
| LAB-04 | Instalación y validación local de K3s sobre WSL2 | PASS LOCAL |
| LAB-05 | CI con GitHub Actions | Propuesto |
| LAB-06 | Construcción y publicación de imágenes en GHCR | Propuesto |
| LAB-07 | Manifiestos Kubernetes y despliegue de aplicaciones | Propuesto |
| LAB-08 | Instalación y configuración de Argo CD | Propuesto |
| LAB-09 | Automatización del CD mediante GitOps | Propuesto |
| LAB-10 | Incorporación del segundo nodo físico | Pendiente |

La secuencia de los incrementos siguientes es provisional y podrá ajustarse según las necesidades del proyecto.

## Documentación

- [LAB-00: Diagnóstico de viabilidad](docs/LAB-00.md)
- [LAB-01: Primer nodo Linux](docs/LAB-01.md)
- [LAB-02: Ansible](docs/LAB-02.md)
- [LAB-03: Configuración de red](docs/LAB-03.md)
- [LAB-04: Instalación y validación de K3s sobre WSL2](docs/LAB-04.md)
- [Ambientes de desarrollo y demostración](docs/AMBIENTES.md)

## Estado

Infraestructura Kubernetes de un nodo instalada y validada localmente sobre WSL2.

La máquina VirtualBox original y sus snapshots se conservan como respaldo histórico del laboratorio.

El siguiente objetivo es configurar la integración continua y publicación de imágenes de los microservicios, seguida de su despliegue mediante Kubernetes y Argo CD.

La validación distribuida de dos nodos físicos permanece pendiente.
