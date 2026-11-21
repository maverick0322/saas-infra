# LAB-04 - Instalacion y validacion de K3s sobre WSL2

Fecha: 20 de noviembre de 2026

Estado: PASS LOCAL

## 1. Objetivo

Preparar un nodo Kubernetes mediante K3s, administrado
con Ansible, como base para el despliegue de los
microservicios de la plataforma SaaS y la futura
implementacion de CI/CD mediante GitHub Actions,
GHCR y Argo CD.

## 2. Antecedentes

Inicialmente se utilizo Ubuntu Server sobre VirtualBox.

Aunque la instalacion de K3s fue exitosa, se observaron
congelamientos recurrentes, interrupciones de SSH,
soft lockups del kernel y cambios de disponibilidad
del nodo Kubernetes.

VirtualBox utilizaba el modo NEM/WHPX debido a la
presencia del hipervisor de Windows.

La inestabilidad de la virtualizacion fue considerada
la principal hipotesis, sin confirmar una causa raiz
definitiva.

Se decidio evaluar WSL2 como alternativa, conservando
la maquina virtual original y sus snapshots.

## 3. Entorno validado

### Equipo anfitrion

- Equipo: EVICTUS.
- Sistema operativo: Windows 11 Home.
- RAM fisica: 16 GB.
- CPU: 14 nucleos y 20 procesadores logicos.

### Entorno WSL2

- WSL: 3.0.5.
- Kernel: 6.18.58.1-microsoft-standard-WSL2.
- Cgroups: version 2.
- Gestor de servicios: systemd.
- Memoria visible: aproximadamente 7.6 GiB.
- Procesadores logicos visibles: 20.

Los recursos de WSL2 son compartidos entre sus
distribuciones y no estan reservados para K3s.

### Distribuciones

- Ubuntu: controlador Ansible.
- saas-k3s-server: nodo Kubernetes.
- Debian: fuera del alcance del laboratorio.
- docker-desktop: entorno de contenedores local.

### Kubernetes

- Distribucion: K3s.
- Version: v1.37.2+k3s1.
- Runtime: containerd 2.3.7-k3s1.
- Nombre del nodo: saas-k3s-server.
- Topologia actual: un nodo control plane.
- Interfaz de red: eth0.
- Modo de red WSL2: NAT.
- Traefik: deshabilitado.
- ServiceLB: deshabilitado.

## 4. Administracion mediante Ansible

Se reutilizaron los playbooks existentes del
repositorio saas-infra.

Se incorporo un inventario independiente para WSL2:

    ansible/inventories/wsl.ini

El inventario original de VirtualBox se conserva.

La comunicacion entre el controlador Ansible y
el nodo administrado utiliza SSH local:

- Direccion: 127.0.0.1.
- Puerto: 2224.
- Usuario: labadmin.
- Autenticacion mediante clave Ed25519.
- Autenticacion por contrasena deshabilitada.
- Acceso SSH de root deshabilitado.
- Elevacion de privilegios mediante sudo.

## 5. Procedimiento reproducible

El nodo debe estar iniciado desde PowerShell:

    wsl -d saas-k3s-server

Desde la distribucion Ubuntu que contiene Ansible,
ubicarse en la raiz del repositorio y ejecutar:

    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/saas_ansible_ed25519

Verificar conectividad:

    ansible -i ansible/inventories/wsl.ini k3s_servers -m ping

Verificar el nodo:

    ansible-playbook -i ansible/inventories/wsl.ini ansible/playbooks/verify-node.yml

Ejecutar preflight:

    ansible-playbook -i ansible/inventories/wsl.ini ansible/playbooks/preflight-k3s.yml

Instalar o comprobar la configuracion de K3s:

    ansible-playbook -i ansible/inventories/wsl.ini ansible/playbooks/install-k3s-server.yml --ask-become-pass

La preparacion inicial de WSL2, del usuario Linux
y del acceso SSH se realizo manualmente.

## 6. Adaptaciones realizadas

- Inventario independiente para WSL2.
- Seleccion parametrizada de la interfaz de red.
- Nombre explicito para el nodo Kubernetes.
- Configuracion sin fijar la direccion NAT actual.
- Conservacion del comportamiento de VirtualBox
  mediante variables de Ansible.
- Omision de la descarga del instalador cuando
  el binario de K3s ya existe.

El playbook no implementa actualizaciones de
version de K3s.

## 7. Resultados de validacion

| Prueba | Resultado |
|---|---|
| Conectividad SSH | PASS |
| Ansible ping | PASS |
| Verificacion del nodo | PASS |
| Preflight K3s | PASS |
| Instalacion automatizada | PASS |
| Nodo Ready | PASS |
| API Kubernetes readyz | PASS |
| CoreDNS | PASS |
| Metrics Server | PASS |
| Despliegue Nginx | PASS |
| Respuesta HTTP | PASS |
| Resolucion DNS mediante CoreDNS | PASS |
| Comunicacion mediante ClusterIP | PASS |
| Comunicacion entre Pods del mismo nodo | PASS |
| Recuperacion individual de la distribucion | PASS |
| Recuperacion completa de WSL2 | PASS |
| Idempotencia del playbook | PASS |
| Limpieza de recursos temporales | PASS |

Resultado final de Ansible:

    ok=9 changed=0 unreachable=0 failed=0 skipped=1

No se observaron congelamientos durante las
pruebas locales de Kubernetes.

## 8. Referencia inicial de recursos

Mediciones realizadas con el cluster y una
aplicacion Nginx de prueba:

- CPU del nodo: 104 millicores.
- Memoria reportada por Kubernetes: 910 MiB.
- Memoria disponible en WSL2: 6.3 GiB.
- Swap utilizado: 0 bytes.
- Carga promedio: 0.15, 0.23, 0.20.

Estas mediciones corresponden a una carga minima
y no constituyen pruebas de capacidad ni de
rendimiento del SaaS completo.

## 9. Limitaciones y pendientes

- Incorporacion de Surface como segundo nodo.
- Comunicacion entre computadoras fisicas.
- Comunicacion entre Pods ubicados en nodos distintos.
- Recuperacion ante un cambio efectivo de direccion IP.
- Coexistencia bajo carga con Docker Desktop.
- Posibles solapamientos de subredes de contenedores.
- Estabilidad prolongada bajo carga.
- Persistencia y recuperacion de bases de datos.
- Evaluacion del mapeo de bases de datos a servidores.
- Confirmacion del entorno distribuido para la
  demostracion academica.

La topologia de un nodo no proporciona alta
disponibilidad.

La validacion corresponde a un laboratorio academico
y no certifica un entorno de produccion.

## 10. Conclusion

WSL2 supera las verificaciones funcionales locales
realizadas y constituye el candidato principal
para continuar el laboratorio de Kubernetes.

La configuracion mediante Ansible es reutilizable
y ha demostrado idempotencia en el nodo evaluado.

El siguiente frente de trabajo es establecer CI
mediante GitHub Actions y publicacion de imagenes
en GHCR, seguido del despliegue de microservicios
y la incorporacion de Argo CD.
