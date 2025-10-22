Este documento detalla el procedimiento para configurar un firewall pfSense dentro de Proxmox VE (PVE), utilizando dos interfaces de red físicas (NICs) para segmentar múltiples redes virtuales (VLANs), de acuerdo con los requerimientos especificados.

La arquitectura final implementará un bridge (`vmbr0`) para la gestión de Proxmox y la interfaz WAN de pfSense (conectada al módem Infinitum). Un segundo bridge (`vmbr10`) se conectará a la segunda NIC física para el tráfico *untagged* hacia un Access Point (AP) y para las VMs de administración. Finalmente, se crearán bridges adicionales (de `vmbr11` a `vmbr15`) sin asignación física para la comunicación interna entre VMs en segmentos de red aislados.

---

# Configuración de Red en Proxmox VE con pfSense para Múltiples LANs

## 1. Identificación de Interfaces Físicas

Antes de proceder, es fundamental identificar los nombres de las interfaces de red físicas en el host Proxmox. Esto se puede verificar desde la consola del nodo (Shell) con el comando `ip a`.

Para este ejemplo, asumiremos:
* `enpXs0`: Conectada al módem Infinitum (WAN y Gestión PVE).
* `enpXs1`: Conectada al Access Point (LAN 10).

## 2. Configuración de Red en Proxmox VE

La configuración de red se gestiona desde la GUI de Proxmox (`Datacenter` -> `[Nodo]` -> `System` -> `Network`).

### 2.1. Creación del Bridge WAN/Management (vmbr0)

Este bridge es el punto clave. Alojará la IP de administración de Proxmox y, al mismo tiempo, servirá como "cable" virtual para la interfaz WAN de pfSense.

1.  Navegue a `Network` y seleccione `vmbr0` (o edítelo si ya existe).
2.  Presione `Edit`.
3.  **IPv4/CIDR**: Asigne la IP de Administración de Proxmox (ej. `192.168.1.100/24`).
4.  **Gateway**: Asigne el Gateway de su red Infinitum (ej. `192.168.1.254`).
5.  **Bridge ports**: Escriba el nombre de su NIC física para la WAN (ej. `enpXs0`).
6.  *Importante*: Asegúrese de que "VLAN aware" **NO** esté marcado.

### 2.2. Creación del Bridge LAN 10 (vmbr10)

Este bridge conectará la `LAN 10` de pfSense (`172.20.10.0/24`) a la NIC física del AP y a las VMs de administración.

1.  Presione `Create` -> `Linux Bridge`.
2.  **Name**: `vmbr10`.
3.  **Bridge ports**: Escriba el nombre de su NIC física para el AP (ej. `enpXs1`).
4.  *Importante*: Deje los campos de IP y Gateway en blanco. pfSense gestionará el direccionamiento de esta red.
5.  *Importante*: Asegúrese de que "VLAN aware" **NO** esté marcado, ya que se requiere tráfico *untagged* hacia el AP.

### 2.3. Creación de Bridges Internos (vmbr11 - vmbr15)

Estos bridges actuarán como switches virtuales aislados para los segmentos LAN internos.

1.  Presione `Create` -> `Linux Bridge`.
2.  **Name**: `vmbr11`.
3.  *Importante*: Deje **vacío** el campo `Bridge ports`.
4.  Deje los campos de IP y Gateway en blanco.
5.  Repita este proceso para `vmbr12`, `vmbr13`, `vmbr14` y `vmbr15`.

### 2.4. Revisión y Aplicación de Cambios

Al finalizar, la configuración de red en `/etc/network/interfaces` debería ser similar a la siguiente. Aplique los cambios (requerirá un reinicio de red o del nodo).

```bash
# /etc/network/interfaces

auto lo
iface lo inet loopback

iface enpXs0 inet manual
#NIC para WAN y Management (conectada a Infinitum)

iface enpXs1 inet manual
#NIC para LAN 10 (conectada al Access Point)

auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100/24  # IP de Gestión de Proxmox
    gateway 192.168.1.254     # Gateway (Módem Infinitum)
    bridge-ports enpXs0
    bridge-stp off
    bridge-fd 0
    # WAN (para pfSense) y Gestión PVE

auto vmbr10
iface vmbr10 inet manual
    bridge-ports enpXs1
    bridge-stp off
    bridge-fd 0
    # LAN 10 (Admin VMs) y AP Físico (Untagged)

auto vmbr11
iface vmbr11 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # LAN 11 (Interna)

auto vmbr12
iface vmbr12 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # LAN 12 (Interna)

auto vmbr13
iface vmbr13 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # LAN 13 (Interna)

auto vmbr14
iface vmbr14 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # LAN 14 (Interna)

auto vmbr15
iface vmbr15 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # LAN 15 (Interna)
