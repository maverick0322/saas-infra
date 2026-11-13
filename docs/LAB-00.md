# LAB-00 - Diagnóstico de viabilidad

Fecha inicial: 2026-11-13
Estado: En progreso

## Objetivo

Evaluar recursos, virtualización, conectividad y
alternativas de despliegue para construir un laboratorio
K3s reproducible con dos computadoras Windows.

No se instalará Kubernetes durante esta etapa.

## 1. Inventario confirmado

### EVICTUS

| Recurso | Resultado |
|---|---|
| SO | Windows 11 Home Single Language |
| Versión | 10.0.26200 |
| CPU | Intel Core i7-13700H |
| Núcleos | 14 |
| Hilos | 20 |
| RAM total reportada | 15.65 GB |
| Disco C libre | 172.11 GB |
| Hipervisor activo | Sí |
| WSL | Versión 2 |
| VirtualBox | Disponible |
| Docker Desktop | Disponible |

Distribuciones WSL2 observadas:

- Debian: detenida.
- Ubuntu: detenida.
- docker-desktop: en ejecución.

Los estados reflejan únicamente el momento del
diagnóstico y pueden cambiar.

### Microsoft Surface

Información conocida:

- Windows.
- Aproximadamente 12 GB de RAM.
- Docker Desktop y VirtualBox disponibles.

Pendiente confirmar CPU, versión de Windows, red,
almacenamiento y virtualización.

## 2. Memoria

Primera medición en EVICTUS:

- RAM disponible: 0.58 GB.

Segunda medición:

- RAM disponible: 3371 MB.
- Memoria comprometida: 83 %.

El equipo tenía numerosas aplicaciones y procesos activos.
El usuario reporta aproximadamente 50 % de RAM disponible
tras un reinicio en condiciones de menor carga.

Conclusión provisional:

La carga observada no demuestra una insuficiencia
estructural de RAM. La viabilidad se evaluará mediante
mediciones durante la ejecución real de la VM.

No se establece un umbral rígido de 7-8 GB libres.

## 3. Virtualización

Observaciones:

- Windows reporta un hipervisor activo.
- WSL2 y Docker Desktop están disponibles.
- El procesador reportó
  VirtualizationFirmwareEnabled = False.

Este último valor no permite concluir por sí solo
que la virtualización esté deshabilitada.

Alternativas:

A. VirtualBox con Ubuntu Server:
   propuesta principal para evaluar nodos separados.

B. WSL2:
   alternativa para Linux y posible nodo de control
   de Ansible, sujeta a validar red y compatibilidad.

Decisión definitiva: pendiente.

No modificar Hyper-V ni funciones de seguridad de
Windows sin una razón comprobada.

## 4. Red

Información observada en EVICTUS:

- Wi-Fi: 192.168.100.49.
- Ethernet: 192.168.1.2 y 169.254.156.220.
- Adaptadores activos reportados:
  Ethernet 3, Wi-Fi y vEthernet de WSL.

Pendientes:

- Identificar la interfaz física adecuada.
- Comprobar conectividad con el segundo equipo.
- Evaluar red puente de VirtualBox.
- Verificar conectividad SSH entre host y VM.
- Verificar conectividad entre las futuras VMs.
- Confirmar resolución DNS y salida a Internet.

No se presupone que las interfaces observadas
pertenezcan a una misma red.

## 5. Topología propuesta

Nodo 1:

- Equipo: EVICTUS.
- Sistema: Ubuntu Server mínimo.
- Rol futuro: K3s Server.
- RAM inicial candidata: 2.5 GB.
- vCPU inicial: 2.
- Disco virtual candidato: 25 GB dinámicos.

Nodo 2:

- Equipo: Surface.
- Sistema: Ubuntu Server mínimo.
- Rol futuro: K3s Agent.
- RAM inicial candidata: 1-1.5 GB.
- vCPU inicial: 1-2.
- Disco virtual candidato: 20 GB dinámicos.

Los recursos son propuestas de prueba, no requisitos
confirmados ni asignaciones ya realizadas.

La memoria se reajustará según mediciones reales.

## 6. Bases de datos

Restricción académica:

El profesor recomendó utilizar servidores separados
para las bases de datos en la demostración.

Pendiente precisar si se refiere a:

- Instancias de base de datos diferentes.
- Máquinas virtuales independientes.
- Servidores físicos distintos.

No se ha fijado una distribución definitiva de bases
de datos entre tenants, servicios y servidores.

Este asunto no bloquea el laboratorio Linux inicial.

## 7. Criterios de aceptación LAB-00

- [x] Inventario inicial de EVICTUS registrado.
- [ ] Inventario de la Surface completado.
- [ ] Virtualización candidata validada.
- [ ] Recursos reales de la VM medidos.
- [ ] Conectividad local comprobada.
- [ ] Topología seleccionada y justificada.
- [ ] Alternativa de un nodo documentada.
- [ ] Resultado GO / GO con restricciones / NO-GO.

## 8. Riesgos iniciales

- Compatibilidad y rendimiento VirtualBox/Hyper-V.
- Conectividad de máquinas virtuales sobre Wi-Fi.
- Presión de memoria al mantener Docker y editores.
- Recursos adicionales al incorporar Argo CD.
- Disponibilidad no permanente de la Surface.
- Ambigüedad de la restricción de bases de datos.

## 9. Próximo paso

Completar INFRA-00 y preparar una prueba controlada
de Ubuntu Server en EVICTUS.

Primero validar arranque, recursos, red, SSH y reinicio.
Después automatizar mediante Ansible.

No instalar K3s antes de validar el nodo Linux.