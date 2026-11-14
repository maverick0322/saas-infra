# LAB-01 - Preparación del primer nodo Linux

Fecha de validación: 2026-11-14
Estado: Validación final pendiente

## 1. Objetivo

Preparar un nodo Linux virtualizado, con conectividad
y acceso administrativo seguro mediante SSH, para
utilizarlo posteriormente con Ansible y K3s.

Este incremento no incluye la instalación de K3s.

## 2. Entorno

### Anfitrión

- Equipo: EVICTUS.
- Sistema: Windows 11 Home Single Language.
- Virtualizador: Oracle VirtualBox 7.2.16r174877.
- Hipervisor de Windows activo.
- Docker Desktop y WSL2 disponibles.

### Máquina virtual

| Parámetro | Configuración |
|---|---|
| Nombre | saas-k3s-server |
| SO | Ubuntu Server 24.04.5 LTS |
| Arquitectura | x86_64 |
| Kernel observado | 6.8.0-149-generic |
| RAM asignada | 3072 MB |
| Procesadores virtuales | 2 |
| Disco | VDI dinámico de 25 GB |
| Sistema de archivos | ext4 |
| Punto de montaje principal | / |
| LVM | No |
| Arranque | BIOS, sin UEFI |
| Red inicial | NAT |
| Usuario administrativo | labadmin |

La ISO utilizada fue Ubuntu Server 24.04.5 LTS.
La verificación SHA-256 de la ISO no quedó
documentada durante la instalación.

## 3. Instalación

Se realizó la instalación manual de Ubuntu Server.

Decisiones aplicadas:

- Instalación estándar, sin entorno gráfico.
- Red mediante DHCP.
- Sin proxy.
- Particionado guiado del disco virtual.
- Sin LVM ni cifrado de volumen.
- Usuario local labadmin.
- Instalación de OpenSSH Server.
- Sin Featured Server Snaps adicionales.
- Sin Docker, MicroK8s ni K3s.

Se confirmó el primer arranque y el acceso
mediante la cuenta administrativa.

## 4. Recursos observados

Tras la instalación se comprobaron:

- RAM total: aproximadamente 2.9 GiB.
- RAM utilizada: aproximadamente 335 MiB.
- RAM disponible: aproximadamente 2.6 GiB.
- Disco raíz: aproximadamente 5.7 GB utilizados.
- Ocupación del disco: aproximadamente 23 %.

Estas cifras son mediciones iniciales, no estimaciones
de consumo para K3s ni garantías de rendimiento futuro.

## 5. Red y conectividad

Interfaz Linux: enp0s3.

Dirección IPv4 observada:
10.0.2.15/24

Puerta de enlace:
10.0.2.2

Configuración:
VirtualBox NAT con DHCP.

Pruebas realizadas:

- Interfaz de red activa: PASS.
- Ruta de salida configurada: PASS.
- Ping a 1.1.1.1: 3/3 recibidos.
- Resolución de archive.ubuntu.com: PASS.

La dirección 10.0.2.15 es interna del NAT y no
debe considerarse una IP permanente del laboratorio.

La conectividad entre dos nodos K3s sigue pendiente
hasta disponer de la Surface.

## 6. Acceso SSH desde Windows

OpenSSH Server instalado en Ubuntu.

Ubuntu utiliza activación mediante ssh.socket.

Comprobaciones:

- openssh-server instalado: PASS.
- ssh.socket activo: PASS.
- ssh.socket habilitado: PASS.
- Puerto TCP 22 en LISTEN: PASS.

Regla de redirección NAT configurada:

Nombre: lab-ssh
Protocolo: TCP
Host IP: 127.0.0.1
Host port: 2222
Guest port: 22

Comando aplicado en el anfitrión:

    VBoxManage controlvm "saas-k3s-server" natpf1 "lab-ssh,tcp,127.0.0.1,2222,,22"

Comprobación desde Windows:

    Test-NetConnection 127.0.0.1 -Port 2222

Resultado:
TcpTestSucceeded = True

La regla limita la entrada a la interfaz loopback
del anfitrión. No configura acceso desde la Surface.

## 7. Autenticación mediante claves

Se generó una clave Ed25519 para el laboratorio.

En el anfitrión Windows:

    ~/.ssh/saas_k3s_lab_ed25519
    ~/.ssh/saas_k3s_lab_ed25519.pub

La clave privada tiene una passphrase.

La clave pública fue instalada en:

    /home/labadmin/.ssh/authorized_keys

Permisos configurados:

    ~/.ssh                 700
    ~/.ssh/authorized_keys 600

La clave privada no debe copiarse al repositorio.

Alias SSH configurado en Windows:

    Host saas-k3s-lab
        HostName 127.0.0.1
        User labadmin
        Port 2222
        IdentityFile ~/.ssh/saas_k3s_lab_ed25519
        IdentitiesOnly yes

Comando de conexión:

    ssh saas-k3s-lab

Resultado: PASS.

## 8. Endurecimiento de OpenSSH

Configuración propuesta y aplicada durante la sesión:

    /etc/ssh/sshd_config.d/01-saas-infra.conf

Directivas:

    PubkeyAuthentication yes
    PasswordAuthentication no
    KbdInteractiveAuthentication no
    PermitRootLogin no

Pruebas de comportamiento realizadas:

1. Conexión sin autenticación por clave pública.

Resultado:
Permission denied (publickey).

Estado: PASS.

2. Conexión mediante el alias y clave Ed25519.

Resultado:
Acceso exitoso a labadmin@saas-k3s-server.

Estado: PASS.

3. Se solicitó la passphrase de la clave privada.

Estado: PASS.

Pendiente conservar la salida de sshd -T para
confirmar todas las directivas efectivas.

La contraseña del usuario local permanece disponible
para operaciones sudo y acceso por consola.

## 9. Identidad del servidor SSH

La huella ED25519 observada inicialmente por Windows fue:

    SHA256:EH+7dL9+e1eM/ZiK28p/GmUJaV51qOuIJFbgoxFdxK0

Pendiente documentar la comparación independiente
con la clave pública del servidor desde VirtualBox.

Esta huella es pública; no es una clave privada.

## 10. Validaciones

- [x] VM creada correctamente.
- [x] Ubuntu Server instalado.
- [x] Primer arranque y login.
- [x] Recursos de memoria y disco medidos.
- [x] Red NAT operativa.
- [x] Conectividad IP y DNS.
- [x] OpenSSH instalado y disponible.
- [x] Redirección SSH desde Windows.
- [x] Clave Ed25519 configurada.
- [x] Conexión mediante clave exitosa.
- [x] Conexión sin clave rechazada.
- [ ] Configuración efectiva sshd -T documentada.
- [ ] Huella del servidor contrastada.
- [ ] Reinicio completo de VM y reconexión probados.
- [ ] Resultado final de aceptación registrado.

## 11. Riesgos y limitaciones

- La red NAT actual no comunica directamente
  esta VM con un segundo nodo físico.
- La configuración de la VM se realizó manualmente.
- El entorno aún no es completamente reproducible.
- No se han automatizado paquetes ni configuración.
- La disponibilidad de la Surface sigue pendiente.
- El consumo de K3s todavía no se ha medido.
- No se ha validado persistencia de la regla NAT
  mediante un reinicio completo.

## 12. Próximos pasos

1. Completar las verificaciones pendientes.
2. Cerrar formalmente LAB-01.
3. Preparar el controlador Ansible.
4. Crear un inventario de laboratorio.
5. Ejecutar una primera prueba Ansible ping.
6. Automatizar configuración básica del nodo.
7. Evaluar la red definitiva para K3s.

No se han instalado todavía componentes Kubernetes.

La automatización deberá preservar las medidas
de seguridad SSH ya verificadas.