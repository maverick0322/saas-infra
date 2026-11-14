# LAB-02 - Automatización inicial con Ansible

Fecha: 2026-11-14
Estado: En progreso

## Objetivo

Preparar un controlador Ansible y verificar que
puede administrar el primer nodo Linux mediante
SSH, inventarios y playbooks.

## Controlador

- Equipo: EVICTUS.
- Entorno: Ubuntu 24.04.4 LTS en WSL2.
- Usuario: meler.
- Python: 3.12.3.
- Ansible Core: 2.21.6.
- Instalación de Ansible: pipx.
- Autenticación: SSH con clave Ed25519 propia.

## Nodo administrado

- Nombre: saas-k3s-server.
- Sistema: Ubuntu Server 24.04 LTS.
- Usuario SSH: labadmin.
- Python remoto: /usr/bin/python3.
- RAM detectada: 2972 MiB.
- Arquitectura: x86_64.

## Conectividad

WSL2 utiliza una red virtual independiente.

Ruta validada:

WSL2 -> Windows 172.30.80.1:2223
     -> VirtualBox NAT
     -> Ubuntu Server TCP 22

La prueba de conectividad TCP resultó PASS.

La dirección de Windows puede cambiar al
reiniciar WSL2. No se considera permanente.

## Seguridad SSH

Se creó una clave independiente para Ansible:

~/.ssh/saas_ansible_ed25519

La clave privada permanece fuera del repositorio.

La clave pública fue autorizada en Ubuntu Server.

Autenticación mediante clave: PASS.
Identidad SSH del servidor: huella coincidente.

La conexión utiliza el alias saas-k3s-server,
definido en ~/.ssh/config del controlador.

Esta configuración local es una dependencia
del inventario y debe documentarse al reconstruir
el laboratorio.

## Inventario

Archivo:

ansible/inventories/lab.ini

Grupos:

- k3s_servers: un servidor.
- k3s_agents: pendiente de segundo equipo.

La ruta de Python se declara explícitamente:

ansible_python_interpreter=/usr/bin/python3

## Primera ejecución Ansible

Comando:

ansible -i ansible/inventories/lab.ini \
  k3s_servers -m ansible.builtin.ping

Resultado: SUCCESS.
Respuesta: pong.
Cambios: 0.

## Primer playbook

Archivo:

ansible/playbooks/verify-node.yml

Verificaciones:

1. Recopilar información del nodo.
2. Validar Ubuntu 24.04 o posterior.
3. Validar al menos 2048 MiB de RAM total.
4. Mostrar información del servidor.

Validación de sintaxis: PASS.

Resultado de ejecución:

ok=4
changed=0
unreachable=0
failed=0

Esta ejecución no modificó el servidor.

## Criterios de aceptación

- [x] Controlador WSL2 preparado.
- [x] Ansible Core instalado.
- [x] SSH mediante clave independiente.
- [x] Conectividad WSL2 a VirtualBox.
- [x] Inventario reconocido.
- [x] ansible.builtin.ping exitoso.
- [x] Primer playbook ejecutado.
- [x] Ejecución sin cambios ni fallos.
- [ ] Elevación de privilegios validada.
- [ ] Primera configuración automatizada.
- [ ] Idempotencia comprobada.
- [ ] Cierre de LAB-02.

## Pendientes y limitaciones

- La red NAT es provisional.
- La Surface todavía no está disponible.
- La configuración SSH local no está versionada.
- La clave privada no debe subirse a Git.
- La regla NAT de VirtualBox es manual.
- Falta verificar operaciones administrativas.
- Falta probar idempotencia con cambios reales.

## Siguiente paso

Verificar elevación de privilegios con sudo
desde Ansible y preparar una automatización
mínima para la configuración de Ubuntu.

No instalar K3s todavía.
