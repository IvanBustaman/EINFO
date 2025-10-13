# Guía de Instalación del Sistema Operativo SUSE Linux Enterprise Server

**Versión del Documento:** 1.0
**Fecha de Última Modificación:** 2025-10-13

---

### **Resumen**

Este documento detalla el procedimiento técnico estandarizado para la instalación y configuración inicial del sistema operativo SUSE Linux Enterprise Server (SLES), versión 15 SP6. El objetivo es proporcionar una guía unívoca que garantice una implementación consistente y verificable. En caso de error puede dejar un mensaje. 

---

### **Índice** 📘

1.  [Prerrequisitos](#1-prerrequisitos)
2.  [Preparación del Medio de Instalación](#2-preparación-del-medio-de-instalación)
3.  [Procedimiento de Instalación Detallado](#3-procedimiento-de-instalación-detallado) 🔬
    * [Fase 1: Arranque e Inicio](#fase-1-arranque-e-inicio)
    * [Fase 2: Configuración](#fase-2-configuración)
    * [Fase 3: Creación de usuario](#fase-3-creación-de-usuario)
4.  [Verificación Post-Instalación](#4-verificación-post-instalación) 🔁
5.  [Resolución de Problemas Comunes](#5-resolución-de-problemas-comunes) 🔧

---

### **1. Prerrequisitos**

#### **1.1. Requisitos de Hardware**

| Componente | Mínimo Requerido | Recomendado |
| :--- | :--- | :--- |
| **Procesador** | x86-64 a 1 GHz | Procesador de 2 núcleos o superior a 2.4 GHz |
| **Memoria RAM** | 1.5 GB | 4 GB o superior |
| **Espacio en Disco**| 10 GB | 40 GB o superior |
| **Resolución** | 800x600 | 1024x768 o superior |

#### **1.2. Requisitos de Software**

* Imagen ISO oficial de SUSE Linux Enterprise Server, obtenida desde [SUSE](https://www.suse.com/download/sles/).
* Para ello será necesario registrarnos para obtener una prueba gratuita de 60 días.
* <img src="./assets/Imagen1.png" alt="Página de descarga de SUSE" width="400"/>.
* Elegimos nuestra versión deseada:
* <img src="./assets/Imagen2.png" alt="Página de descarga de SUSE" width="400"/>.
* En este caso como va a ser instalada en una máquina virtual de nuestro servidor, lo primero que deberemos hacer es cargar el archivo ISO a nuestro servidor en PROXMOX.
* Entramos a nuestro servidor de PROXMOX.
* <img src="./assets/Imagen3.png" alt="Entramos a nuestro servidor de Poxmox" width="400"/>.
* Cargamos el archivo ISO de forma local.
* <img src="./assets/Imagen4.png" alt="Cargamos el archivo ISO de forma local" width="400"/>.
*  Es posible que por el tamaño del archivo no se cargue correctamente, ya que pesa entre 15 y 16 GB, por lo que deberemos cargarlo desde SSH con SCP.
* <img src="./assets/Imagen6.png" alt="Cargamos el archivo ISO de forma local" width="800"/>.
* 
  ``` scp "C:\Users\ivan2\Downloads\SUSE\SLE-15-SP6-Full-x86_64-GM-Media1-capnovelo.iso" root@192.168.1.90:/var/lib/vz/template/iso/ ```
* Creamos nuestra máquina virtual con las características deseadas segun nos permita nuestro servidor.
* <img src="./assets/Imagen5.png" alt="Creamos la MV" width="400"/>.

---

### **2. Procedimiento de Instalación Detallado**

Siga los siguientes pasos de forma secuencial. Cada paso incluye una descripción y una captura de pantalla de referencia.

#### **Fase 1: Arranque e Inicio**

1.  Iniciamos desde nuestro CD/DVD.
2.  En el menú de arranque, seleccionar la opción **"Installation"** y presionar `Enter`.

    <img src="./assets/Imagen7.png" alt="Instalar" width="400"/>.
#### **Fase 2: Configuración** ⚙️
3.  **Seleccionar** la configuración deseada así como los productos adicionales deseados.

    <img src="./assets/Imagen8.png" alt="Instalar" width="400"/>.
4.  **Iniciamos la instalación** Seleccionamos en siguiente para confirmar.

    <img src="./assets/Imagen9.png" alt="Instalar" width="400"/>.
    #### **Fase 3: Creación de usuario**

6.  **Creamos un usuario** Creamos nuestro usuario y definimos una contraseña segura para nuestro SO.

    <img src="./assets/Imagen10.png" alt="Instalar" width="400"/>.
7.  **Instalamos** Confirmamos la configuración que designamos previamente.

    <img src="./assets/Imagen11.png" alt="Instalar" width="400"/>.
---

### **3. Verificación Post-Instalación**

Una vez finalizada la instalación y reiniciado el sistema, realice las siguientes comprobaciones para validar el éxito del proceso:
<img src="./assets/Imagen12.png" alt="Instalar" width="400"/>.
- [ ] El sistema arranca correctamente sin el medio de instalación.
- <img src="./assets/Imagen13.png" alt="Instalar" width="400"/>.
- [ ] Se puede iniciar sesión con el usuario `root` y la contraseña definida (En nuestro caso usamos un usuario diferente).
- [ ] El sistema tiene conectividad a la red. Ejecutar `ping google.com`.
- <img src="./assets/Imagen15.png" alt="Instalar" width="400"/>.


---

### **4. Resolución de Problemas Comunes** 🔧

| Problema | Causa Probable | Solución Propuesta |
| :--- | :--- | :--- |
| **El sistema no arranca desde el USB** | Configuración incorrecta del orden de arranque en la BIOS/UEFI. | Acceder a la configuración de la BIOS/UEFI y priorizar el arranque desde dispositivos USB. |
| **No se detecta el disco duro** | Falta de controladores de almacenamiento o disco dañado. | Verificar la conexión física del disco. Cargar controladores adicionales si es necesario desde el instalador. |
| **Archivo ISO Corrupto** | No se cargó correctamente el archivo ISO de forma local. | Cargalor desde SSH con Copia segura (SCP) como se muestra en [Requisitos de Software](#12-requisitos-de-software). |
