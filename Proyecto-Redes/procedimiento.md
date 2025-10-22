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


## 3. Creación y Configuración de la VM pfSense

1.  Cree una nueva VM en Proxmox para pfSense (OS tipo "Other", etc.).
2.  Diríjase a la pestaña `Hardware` de la VM pfSense.
3.  Añada siete (7) dispositivos de red. Se recomienda usar el modelo `VirtIO (paravirtualized)` para mejor rendimiento:
    * **net0**: Bridge: `vmbr0` (Esta será la interfaz WAN).
    * **net1**: Bridge: `vmbr10` (Esta será la interfaz LAN 10 - Admin).
    * **net2**: Bridge: `vmbr11` (Esta será la interfaz LAN 11).
    * **net3**: Bridge: `vmbr12` (Esta será la interfaz LAN 12).
    * **net4**: Bridge: `vmbr13` (Esta será la interfaz LAN 13).
    * **net5**: Bridge: `vmbr14` (Esta será la interfaz LAN 14).
    * **net6**: Bridge: `vmbr15` (Esta será la interfaz LAN 15).

4.  Inicie la VM e instale pfSense.

## 4. Configuración Inicial de pfSense (Consola)

Durante el primer arranque, pfSense solicitará la asignación de interfaces.

1.  **VLANs**: Cuando pregunte por VLANs, responda **No**. La segmentación se está realizando a nivel de bridges de Proxmox, no con tags VLAN dentro de pfSense.
2.  **Asignación de WAN**: Asigne la interfaz `vtnet0` (o la que corresponda a `net0` de Proxmox) como **WAN**.
3.  **Asignación de LAN**: Asigne la interfaz `vtnet1` (o la que corresponda a `net1` de Proxmox) como **LAN**.
4.  **Asignación de Opcionales**: Asigne `vtnet2` a `vtnet6` como interfaces opcionales (OPT1, OPT2, etc.).
5.  **Configuración IP de LAN**: Una vez en el menú principal de la consola de pfSense, seleccione la opción `2) Set interface(s) IP address`.
6.  Configure la interfaz **LAN**:
    * **IP Address**: `172.20.10.1`
    * **Subnet bit count**: `24`
    * **Upstream gateway**: `none`
    * **Enable DHCP Server on LAN**: Sí.
    * **Start/End range**: Ej. `172.20.10.100` a `172.20.10.200`.

## 5. Configuración en la Interfaz Web (WebGUI)

Acceda a la WebGUI de pfSense desde una VM conectada a `vmbr10` (o temporalmente desde el host Proxmox si se crean reglas de firewall) en `http://172.20.10.1`.

### 5.1. Configuración de Interfaces Opcionales (LAN 11-15)

Deberá habilitar y configurar cada interfaz "OPT" que creó.

1.  Vaya a `Interfaces` -> `Interface Assignments`.
2.  Verá `OPT1` (asignada a `vtnet2`), `OPT2` (asignada a `vtnet3`), etc.
3.  Haga clic en `OPT1` para habilitarla.
4.  **Enable**: Marque "Enable interface".
5.  **Description**: Cámbielo a un nombre descriptivo, ej. `LAN11`.
6.  **IPv4 Configuration Type**: `Static IPv4`.
7.  **IPv4 Address**: Asigne el gateway para esta red, ej. `172.20.11.1 /24`.
8.  Guarde y aplique cambios.
9.  **Repita este proceso** para todas las interfaces opcionales, asignando sus redes correspondientes (ej. `LAN12` con `172.20.12.1/24`, etc.).

### 5.2. Configuración de DHCP para LAN 11-15

Para que las VMs internas obtengan IP automáticamente:

1.  Vaya a `Services` -> `DHCP Server`.
2.  Verá pestañas para `LAN` (ya configurada), `LAN11`, `LAN12`, etc.
3.  Haga clic en la pestaña `LAN11`.
4.  Marque `Enable DHCP server on LAN11 interface`.
5.  Defina el `Range` (ej. `172.20.11.100` a `172.20.11.200`).
6.  Guarde.
7.  **Repita este proceso** para `LAN12` a `LAN15`.

### 5.3. Configuración de Reglas de Firewall

Por defecto, pfSense bloquea todo el tráfico de las interfaces opcionales. Debe crear reglas para permitir el acceso a Internet.

1.  Vaya a `Firewall` -> `Rules`.
2.  Haga clic en la pestaña `LAN11`.
3.  Presione `Add` (el botón "up").
4.  **Action**: `Pass`.
5.  **Interface**: `LAN11`.
6.  **Protocol**: `Any`.
7.  **Source**: `LAN11 net`.
8.  **Destination**: `any`.
9.  **Description**: `Allow LAN11 to Internet`.
10. Guarde y aplique cambios.
11. **Repita este proceso** para las pestañas `LAN12` a `LAN15`.

## 6. Asignación de Máquinas Virtuales (VMs)

Ahora puede asignar sus VMs de Proxmox a las redes correctas:

* **VMs de Administración**: En la pestaña `Hardware` de la VM, configure su dispositivo de red para usar `Bridge: vmbr10`. Obtendrá una IP en el rango `172.20.10.0/24`.
* **VMs Internas (Segmento 11)**: Configure su dispositivo de red para usar `Bridge: vmbr11`. Obtendrá una IP en el rango `172.20.11.0/24`.
* ...y así sucesivamente para `vmbr12` a `vmbr15`.

## Resumen del Flujo de Tráfico

* **WAN (Infinitum)**: `Módem` -> `enpXs0` -> `vmbr0` -> `pfSense(WAN)`.
* **Gestión PVE**: `enpXs0` -> `vmbr0` (IP: `192.168.1.100`).
* **LAN 10 (Admin)**: `VMs Admin` -> `vmbr10` -> `pfSense(LAN10)`.
* **LAN 10 (AP)**: `Access Point` -> `enpXs1` -> `vmbr10` (Tráfico Untagged) -> `pfSense(LAN10)`.
* **LAN 11 (Interna)**: `VMs Internas` -> `vmbr11` (Switch Virtual) -> `pfSense(LAN11)`.
