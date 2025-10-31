# Implementación de Nextcloud con PostgreSQL, AD y HAProxy (Fase 1 implementación de reglas de nuestro firewall en pfsense)

Este documento detalla el procedimiento completo para instalar Nextcloud en un servidor Ubuntu, utilizando un servidor PostgreSQL remoto en openSUSE para la base de datos, autenticando usuarios contra un Directorio Activo (Active Directory) de Windows Server, y publicándolo de forma segura a Internet usando pfSense con ACME (Let's Encrypt) y HAProxy.

## 📋 Arquitectura de Red

* **Host Proxmox:** Aloja todas las VMs.
* **VM pfSense:** Actúa como firewall/router.
    * `WAN`: IP pública (o IP de la red principal `192.168.1.109`).
    * `VLAN 13 (IF_SERVICIOS)`: Red `172.20.13.0/24`.
                                 (La instrucción de nuestro docente fue utilizar las IP's a partir de la 100 apra servicios por lo que nuestras VM's utilizarán las .100,etc.)
    * `VLAN 15 (IF_BDS)`: Red `172.20.15.0/24`.
* **VM Ubuntu (Nextcloud):**
    * IP: `172.20.13.102` (VLAN 13 y como es de servicio tiene una IP mayor a 100 [Nota: buscar la IP 101]) 
* **VM openSUSE (PostgreSQL):**
    * IP: `172.20.15.10` (VLAN 15 esta es la VM que utilizamos como base de datos  donde instalamos PGSQL)
* **VM Windows Server (Active Directory):**
    * IP: `172.20.13.100` (VLAN 13 Aquí el equipo 1 nos hizo el favor de crear los usuarios con sus respectivas contraseñas las cuales funcionarán tambie´n para Nextcloud)
    * Dominio: `laboratorio.local`
* **Dominio Público:** `nextcloud.einfo.space` (Este dominio lo compré con Godady por un año.)

---

## Fase 1: Configuración de Red (pfSense)

El primer paso es permitir la comunicación necesaria a través del firewall.

### 1.1. Reglas de Firewall Inter-VLAN

Permitimos que Nextcloud (VLAN 13) hable con PostgreSQL (VLAN 15).

1.  Vamos a `Firewall > Rules > IF_SERVICIOS` (la interfaz de la VLAN 13).
2.  Creamos una nueva regla:
    * **Action:** `Pass`
    * **Protocol:** `TCP`
    * **Source:** `Single host or alias` -> `172.20.13.102` para decirle de dónde se va a conectar
    * **Destination:** `Single host or alias` -> `172.20.15.10` y el destino
    * **Destination Port:** `5432` (PostgreSQL) El puerto por el que se va a conectar
    * **Description:** `Permitir Nextcloud a PostgreSQL`

### 1.2. Reglas de Acceso a Internet

Permitimos que el servidor Nextcloud acceda a Internet para descargar actualizaciones y paquetes.

1.  En la misma pestaña (`IF_SERVICIOS`), creamos otra regla:
    * **Action:** `Pass`
    * **Protocol:** `any`
    * **Source:** `IF_SERVICIOS net`
    * **Destination:** `any`
    * **Description:** `Permitir Internet a VLAN 13`
2.  Tenemos que ver que el outbound esté en modo automático: `Firewall > NAT > Outbound` modo `Automatic`.
<img src="./assets/Imagen28.png" width="400"/>
   

