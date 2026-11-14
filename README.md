# SaaS Infra

Infraestructura reproducible y prácticas DevOps para un
proyecto académico de gestión clínica SaaS multitenant.

## Contexto

Proyecto desarrollado por dos integrantes durante ocho
semanas. El objetivo es construir y evaluar un laboratorio
de infraestructura que permita desplegar una aplicación
mínima y, posteriormente, los componentes del SaaS.

No es un entorno de producción ni está autorizado para
almacenar información clínica real. Se utilizarán
exclusivamente datos ficticios.

## Objetivos

- Preparar máquinas Linux reproducibles.
- Automatizar su configuración mediante Ansible.
- Desplegar un clúster K3s de dos nodos.
- Disponer de una alternativa funcional de un nodo.
- Construir y publicar imágenes mediante GitHub Actions y GHCR.
- Incorporar despliegues declarativos con Argo CD.
- Validar conectividad, recursos y funcionamiento básico.

## Entorno de laboratorio

### Equipo principal: EVICTUS

- Windows 11 Home.
- Intel Core i7-13700H, 14 núcleos y 20 hilos.
- Aproximadamente 16 GB de RAM.
- Docker Desktop, VirtualBox y WSL2 disponibles.
- Rol propuesto: nodo K3s Server.

### Segundo equipo: Microsoft Surface

- Windows.
- Aproximadamente 12 GB de RAM.
- Docker Desktop y VirtualBox disponibles.
- Rol propuesto: nodo K3s Agent.
- Diagnóstico técnico pendiente.

## Tecnologías

| Tecnología | Propósito | Estado |
|---|---|---|
| Git y GitHub | Versionamiento | Inicial |
| Ubuntu Server | Sistema operativo de nodos | Propuesto |
| VirtualBox | Virtualización | En evaluación |
| WSL2 | Alternativa y control Ansible | En evaluación |
| Ansible | Configuración automatizada | Planificado |
| K3s | Orquestación Kubernetes | Planificado |
| GitHub Actions | Integración continua | Planificado |
| GHCR | Registro de imágenes | Planificado |
| Argo CD | Despliegues GitOps | Planificado |
| .NET 10 | Aplicación de prueba | Tecnología confirmada |

## Topología candidata

- EVICTUS: VM Linux para K3s Server.
- Surface: VM Linux para K3s Agent.
- Comunicación entre nodos a través de la red local.
- Un nodo como alternativa cuando no esté disponible
  el segundo equipo.

Esta topología todavía requiere validación técnica.

Dos nodos con un solo K3s Server no proporcionan
alta disponibilidad del plano de control.

## Restricciones

- Recursos limitados de dos computadoras Windows.
- Evitar infraestructura innecesaria.
- Mantener el laboratorio reproducible y documentado.
- No introducir datos clínicos reales.
- No almacenar credenciales ni secretos en Git.
- Conservar la recomendación académica de separar
  servidores de bases de datos, pendiente de precisar.
- Priorizar aprendizaje y validaciones incrementales.

## Incrementos

| ID | Objetivo | Estado |
|---|---|---|
| INFRA-00 | Repositorio y documentación | En progreso |
| LAB-00 | Viabilidad y topología | En progreso |
| LAB-01 | Primer nodo Linux | Validación final |
| LAB-02 | Automatización con Ansible | En progreso |
| LAB-03 | Instalación y validación K3s | Pendiente |
| LAB-04 | Aplicación mínima .NET 10 | Pendiente |
| LAB-05 | CI con GitHub Actions y GHCR | Pendiente |
| LAB-06 | Despliegue GitOps con Argo CD | Pendiente |

Esta secuencia es provisional y podrá ajustarse
según los resultados de cada incremento.

## Documentación

- [LAB-00: Diagnóstico de viabilidad](docs/LAB-00.md)
- [LAB-01: Primer nodo Linux](docs/LAB-01.md)
- [LAB-02: Ansible](docs/LAB-02.md)

## Estado

Laboratorio en preparación. No se ha instalado ni
validado todavía ningún nodo Kubernetes.