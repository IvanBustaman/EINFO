# Proyecto: Diseño e Implementación de Infraestructura de Red Segura y Segmentada para Laboratorio de Ciberseguridad

**Versión:** 1.0
**Fecha:** 2025-10-14
**Autor:** [Tu Nombre Completo]
**Revisor:** [Nombre de tu Asesor/Profesor]

---

## 1. Resumen Ejecutivo

Este proyecto detalla el diseño, implementación y gestión de una infraestructura de red robusta, segmentada y segura sobre una plataforma de virtualización Proxmox VE. El objetivo es crear un entorno de laboratorio avanzado para la especialidad de informática, capaz de simular redes corporativas y militares de alta demanda, facilitando la formación práctica en defensa, ataque (pentesting), monitoreo (NOC/SOC) y gestión de servicios críticos. La arquitectura se centra en el uso de un firewall pfSense para la segmentación lógica a través de VLANs, garantizando el aislamiento de redes y la aplicación de políticas de seguridad granulares. El resultado final será una plataforma resiliente, documentada y escalable, lista para ser presentada a la alta dirección del instituto como un modelo de laboratorio de ciberseguridad de próxima generación.

---

## 2. Objetivos del Proyecto

### 2.1. Objetivo Principal
Implementar una infraestructura de red virtualizada, segura y multi-segmentada que sirva como laboratorio práctico para la especialidad de informática, siguiendo las mejores prácticas de la industria y estándares de alta seguridad.

### 2.2. Objetivos Específicos
- **Configurar el entorno de virtualización:** Instalar y asegurar el hipervisor Proxmox VE en el servidor físico Huawei.
- **Desplegar el Firewall Perimetral:** Virtualizar e instalar pfSense como el firewall central y router de la infraestructura.
- **Establecer la Segmentación de Red:** Crear y configurar 6 VLANs distintas para aislar el tráfico de estudiantes, servicios de seguridad, operaciones, servidores productivos, atacantes y bases de datos.
- **Implementar Políticas de Seguridad:** Definir y aplicar reglas de firewall estrictas entre las VLANs para controlar el flujo de tráfico y asegurar el principio de mínimo privilegio.
- **Habilitar Servicios de Red:** Configurar servicios esenciales como DHCP y DNS para las redes internas a través de pfSense.
- **Validar la Arquitectura:** Realizar pruebas de conectividad y penetración controladas desde la red de atacantes (LAN 14) para verificar la efectividad de los controles de seguridad.
- **Crear Documentación Exhaustiva:** Documentar cada fase del proyecto, incluyendo diagramas de red, procedimientos de configuración, políticas de seguridad y un plan de respuesta a incidentes.

---

## 3. Alcance del Proyecto

### 3.1. Dentro del Alcance (In-Scope)
- Configuración del servidor Proxmox VE y su red física.
- Instalación y configuración completa de la VM pfSense.
- Creación de todas las VLANs y puentes de red necesarios en Proxmox y pfSense.
- Definición de reglas de firewall para el tráfico inter-VLAN.
- Despliegue de al menos una máquina virtual de prueba en cada VLAN para validar la configuración.
- Documentación técnica del proyecto en este repositorio de GitHub.

### 3.2. Fuera del Alcance (Out-of-Scope)
- Configuración de aplicaciones específicas dentro de las máquinas virtuales (ej. instalación de un SIEM específico, configuración de un servidor web Apache).
- Gestión y configuración del Access Point Wi-Fi más allá de conectarlo a la interfaz LAN física.
- Adquisición de hardware o licencias de software adicionales.
- Soporte técnico a largo plazo de la infraestructura post-entrega del proyecto.

---

## 4. Arquitectura Propuesta

### 4.1. Diagrama Lógico de la Red

```mermaid
graph TD
    subgraph Servidor Físico Huawei (Proxmox VE)
        subgraph pfSense VM
            WAN --- vmbr0 (eth0 físico)
            LAN10 -- VLAN 10 --- vmbr1 (eth1 físico)
            LAN11 -- VLAN 11 --- vmbr1
            LAN12 -- VLAN 12 --- vmbr1
            LAN13 -- VLAN 13 --- vmbr1
            LAN14 -- VLAN 14 --- vmbr1
            LAN15 -- VLAN 15 --- vmbr1
        end

        subgraph "VMs en VLAN 11 (Seguridad)"
            VM_SEC1(Zabbix/SIEM)
        end

        subgraph "VMs en VLAN 12 (NOC/SOC)"
            VM_NOC1(PC Monitoreo)
        end

        subgraph "VMs en VLAN 13 (Producción)"
            VM_PROD1(Servidor Web)
            VM_PROD2(Servidor App)
        end

        subgraph "VMs en VLAN 14 (Pentesting)"
            VM_PENTEST1(Kali Linux)
        end

        subgraph "VMs en VLAN 15 (Bases de Datos)"
            VM_DB1(MySQL/PostgreSQL)
        end
    end

    Internet([Internet]) --- Modem(Modem Infinitum <br> 192.168.1.254)
    Modem --- WAN
    LAN10 --- AP(Access Point Wi-Fi)
    AP --- Estudiantes([Estudiantes])

    %% Conexiones lógicas a pfSense
    VM_SEC1 --- LAN11
    VM_NOC1 --- LAN12
    VM_PROD1 --- LAN13
    VM_PROD2 --- LAN13
    VM_PENTEST1 --- LAN14
    VM_DB1 --- LAN15
```


### 4.2. Esquema de Direccionamiento IP y VLANs

| ID VLAN | Red      | Nombre             | Gateway (pfSense IP) | Rango de Red         | Máscara de Subred | Propósito                                              |
|---------|----------|--------------------|----------------------|----------------------|-------------------|--------------------------------------------------------|
| **N/A** | WAN      | Internet (Infinitum) | `192.168.1.254`      | `192.168.1.0/24`     | `255.255.255.0`   | Conexión de salida a Internet.                         |
| **10** | LAN 10   | Red de Estudiantes | `172.20.10.254`      | `172.20.10.0/24`     | `255.255.255.0`   | Conexión Wi-Fi para usuarios finales.                  |
| **11** | LAN 11   | Servicios Seguridad  | `172.20.11.254`      | `172.20.11.0/24`     | `255.255.255.0`   | VMs con herramientas de seguridad (SIEM, IDS/IPS, etc).|
| **12** | LAN 12   | NOC / SOC          | `172.20.12.254`      | `172.20.12.0/24`     | `255.255.255.0`   | VMs para el personal de monitoreo y operaciones.       |
| **13** | LAN 13   | Servidores Prod.   | `172.20.13.254`      | `172.20.13.0/24`     | `255.255.255.0`   | Servidores que ofrecen servicios (Web, Apps).          |
| **14** | LAN 14   | Red de Atacantes   | `172.20.14.254`      | `172.20.14.0/24`     | `255.255.255.0`   | VMs para realizar pruebas de pentesting.               |
| **15** | LAN 15   | Bases de Datos     | `172.20.15.254`      | `172.20.15.0/24`     | `255.255.255.0`   | Servidores de bases de datos, red altamente restringida.|

---

## 5. Plan de Trabajo (Work Breakdown Structure - WBS)

A continuación se presenta el plan de trabajo detallado, paso a paso.

### **Fase 1: Configuración del Entorno de Virtualización (Proxmox)**
- **1.1.** Instalación limpia de Proxmox VE en el servidor Huawei.
- **1.2.** Configuración de red inicial de Proxmox.
    - Asignar la IP estática `192.168.1.90` a la interfaz de gestión (`vmbr0`) conectada al puerto físico 1 (`eth0`).
    - Verificar la conectividad a Internet y el acceso a la interfaz web de Proxmox.
- **1.3.** Crear un segundo Linux Bridge (`vmbr1`) sin dirección IP, asociado al puerto físico 2 (`eth1`) y con la opción "VLAN aware" activada. Este será el Trunk para todas las VLANs internas.
- **1.4.** Subir la imagen ISO de pfSense al almacenamiento local de Proxmox.

### **Fase 2: Despliegue y Configuración del Firewall (pfSense)**
- **2.1.** Crear una nueva máquina virtual para pfSense.
    - **Hardware:** 2 vCPU, 2 GB RAM, 32 GB Disco.
    - **Red:** Asignar dos interfaces de red virtuales:
        - `net0` conectada a `vmbr0` (Será la WAN).
        - `net1` conectada a `vmbr1` (Será el Trunk de las LANs).
- **2.2.** Instalar pfSense desde la ISO.
- **2.3.** Configuración inicial de interfaces en la consola de pfSense:
    - Asignar la interfaz `vtnet0` como WAN (obtendrá IP por DHCP de la red `192.168.1.0/24` o se configurará estáticamente).
    - Asignar la interfaz `vtnet1` como LAN. Se le puede dar una IP temporal (ej. `192.168.100.1/24`) para el acceso inicial a la GUI web.
- **2.4.** Primer acceso a la GUI web de pfSense y completar el asistente de configuración.
- **2.5.** Crear las VLANs en pfSense (Interfaces > Assignments > VLANs):
    - Crear VLAN 10, 11, 12, 13, 14, 15, todas sobre `vtnet1`.
- **2.6.** Asignar las VLANs a nuevas interfaces lógicas (Interfaces > Assignments):
    - Crear una interfaz para cada VLAN (ej. `LAN10`, `LAN11`, etc.).
    - Configurar la IP estática correspondiente para cada interfaz (ej. `172.20.10.254/24` para `LAN10`).
- **2.7.** Configurar los servicios DHCP Server para cada interfaz LAN que lo requiera (principalmente `LAN10`, `LAN12`, `LAN14`).

### **Fase 3: Implementación de Reglas de Firewall**
- **3.1.** Establecer una regla por defecto de "Denegar todo" (Default Deny) en cada interfaz LAN.
- **3.2.** Crear reglas explícitas para permitir el tráfico necesario (Principio de Mínimo Privilegio). Ejemplos:
    - **LAN10 (Estudiantes):** Permitir salida a Internet (puertos 80, 443). Bloquear todo acceso a las otras LANs internas.
    - **LAN14 (Atacantes):** Permitir acceso controlado a `LAN13` (Servidores) solo en puertos específicos para las pruebas. Bloquear el resto.
    - **LAN13 (Servidores):** Permitir el acceso desde `LAN12` (NOC/SOC) para gestión. Permitir el acceso desde `LAN13` a `LAN15` (Bases de Datos) solo en el puerto de la base de datos (ej. 3306).
    - **LAN15 (Bases de Datos):** Bloquear toda la salida a Internet. Solo permitir conexiones entrantes desde `LAN13`.
- **3.3.** Configurar Aliases en pfSense para agrupar IPs, puertos o redes y simplificar la gestión de reglas.

### **Fase 4: Creación de VMs de Prueba y Validación**
- **4.1.** Crear al menos una VM por cada VLAN en Proxmox.
- **4.2.** Al crear la VM, en la configuración de red, asociar su interfaz de red a `vmbr1` y especificar el "VLAN Tag" correspondiente (ej. `10` para la VM de estudiantes, `13` para el servidor web, etc.).
- **4.3.** Verificar que cada VM obtenga una IP del rango correcto a través del DHCP de pfSense (si está configurado) o configurarla manualmente.
- **4.4.** Realizar pruebas de conectividad (ping, traceroute) para validar que las reglas del firewall funcionan como se espera.

### **Fase 5: Documentación y Presentación**
- **5.1.** Actualizar este archivo `README.md` con capturas de pantalla de las configuraciones clave.
- **5.2.** Crear documentos adicionales en el repositorio:
    - `POLITICAS_DE_RED.md`: Descripción detallada de cada regla de firewall y su justificación.
    - `PLAN_DE_RESPUESTA_A_INCIDENTES.md`: Procedimiento a seguir en caso de una brecha de seguridad simulada.
    - `PROCEDIMIENTOS_OPERATIVOS.md`: Guías para tareas comunes como agregar una nueva VM o modificar una regla de firewall.
- **5.3.** Preparar una presentación ejecutiva para la alta dirección resumiendo el proyecto, sus beneficios y la arquitectura implementada.

---

## 6. Consideraciones "Radar" (Visión 360°)

Para elevar el proyecto a un nivel de "cumplimiento militar", se deben considerar los siguientes aspectos:

### **Tecnología**
- **Monitoreo y Logging:** Instalar paquetes en pfSense como `pfBlockerNG` para bloqueo de IPs maliciosas y `Suricata` o `Snort` para Detección/Prevención de Intrusiones (IDS/IPS). Centralizar los logs de pfSense y de las VMs en un servidor SIEM (Security Information and Event Management) como Wazuh o un stack ELK en la `LAN11`.
- **Gestión de Vulnerabilidades:** Desplegar un escáner como OpenVAS o Nessus en la `LAN11` para escanear periódicamente los servidores de la `LAN13` en busca de vulnerabilidades.
- **Backup y Recuperación:** Utilizar Proxmox Backup Server (puede ser otra VM) para programar respaldos automáticos y cifrados de todas las máquinas virtuales críticas.
- **Hardening:** Aplicar guías de fortalecimiento (hardening) tanto para el host Proxmox como para los sistemas operativos de las VMs (ej. CIS Benchmarks).

### **Procesos**
- **Gestión de Cambios:** Documentar cualquier cambio en la configuración de red o de firewall en un "log de cambios" dentro de este repositorio. Cualquier modificación debe ser solicitada, aprobada y documentada.
- **Principio de Mínimo Privilegio:** No solo aplicarlo a las reglas de red, sino también a las cuentas de usuario. Crear roles de usuario específicos en Proxmox y en las VMs.
- **Auditoría:** Planificar auditorías periódicas de las reglas del firewall y las configuraciones de seguridad para asegurar que siguen siendo relevantes y efectivas.

### **Personas**
- **Roles y Responsabilidades:** Definir claramente quién es responsable de qué (aunque seas solo tú, define los roles): Administrador de Red, Oficial de Seguridad, Analista SOC.
- **Concienciación y Formación:** Crear una breve "Política de Uso Aceptable" para los usuarios de la `LAN10` (Estudiantes), explicando qué está y qué no está permitido.
- **Plan de Comunicación:** Definir cómo se comunicarán las alertas de seguridad generadas por el SIEM o IDS al equipo responsable (Analista SOC).

---

## 7. Cronograma Estimado

| Fase                                                  | Duración Estimada |
|-------------------------------------------------------|-------------------|
| Fase 1: Configuración de Proxmox                      | 2 Días            |
| Fase 2: Despliegue y Configuración de pfSense         | 3 Días            |
| Fase 3: Implementación de Reglas de Firewall          | 3 Días            |
| Fase 4: Creación de VMs de Prueba y Validación        | 2 Días            |
| Fase 5: Documentación y Preparación de la Presentación | 4 Días            |
| **Total Estimado** | **14 Días** |
