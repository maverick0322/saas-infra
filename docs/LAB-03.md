
# LAB-03 — Preparación de red para K3s

Fecha de validación: 2026-11-14

## Objetivo

Preparar la conectividad del nodo Ubuntu Server
para un futuro clúster K3s de dos nodos,
conservando el acceso administrativo existente.

## Entorno

- Hipervisor: VirtualBox 7.2.16
- Sistema invitado: Ubuntu Server 24.04.5 LTS
- Nombre: saas-k3s-server
- Controlador Ansible: Ubuntu WSL2
- Adaptador 1: NAT
- Adaptador 2: Puente sobre Wi-Fi MediaTek

## Configuración de red

### enp0s3 — NAT

- IPv4: 10.0.2.15/24
- Gateway: 10.0.2.2
- Uso: administración e Internet
- Administrado por cloud-init/Netplan

### enp0s8 — Puente

- IPv4 observada: 192.168.100.214/24
- Asignación: DHCP
- Uso previsto: comunicación entre nodos K3s
- No incorpora rutas ni DNS recibidos por DHCP
- Configuración: /etc/netplan/60-saas-bridge.yaml

La IP del puente no está garantizada.
Pendiente definir una asignación estable.

## Automatización Ansible

Archivos:

- ansible/files/60-saas-bridge.yaml
- ansible/playbooks/configure-network.yml

El playbook:

1. Verifica Ubuntu 24.04.
2. Comprueba la existencia de enp0s8.
3. Copia la configuración con permisos 0600.
4. Ejecuta netplan generate.
5. No aplica cambios a la red activa.

## Evidencia de validación

- Sintaxis del playbook: PASS
- Ejecución en modo check: PASS
- Primera ejecución real: ok=6, changed=0
- Segunda ejecución real: ok=6, changed=0
- Propietario: root:root
- Permisos: 0600
- Acceso SSH por NAT: PASS
- Acceso SSH por LAN: PASS
- Conectividad Ansible: PASS
- Persistencia tras reinicio: PASS
- Unidades systemd fallidas tras reinicio: 0

## Limitaciones

- No se ha probado reconstrucción desde cero.
- No se ha probado restauración del snapshot.
- No se ha probado incorporación del nodo Surface.
- No se ha probado comunicación K3s entre nodos.
- No se ha reservado la IP del servidor en DHCP.
- El playbook no ejecuta netplan apply.

## Resultado

GO técnico para continuar con la preparación
de K3s Server en un solo nodo.

No implica que el clúster Kubernetes esté
instalado o validado.
