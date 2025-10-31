---

## Fase 5: Publicación a Internet (pfSense)

Exponemos Nextcloud de forma segura usando `nextcloud.einfo.space`.

### 5.1. Configuración de DNS (Registrador de Dominio)

En el panel de control de `einfo.space` (GoDaddy, etc.), cree un registro `A`.

* **Tipo:** `A`
* **Host (Nombre):** `nextcloud`
* **Valor (Apunta a):** `[SU_IP_PUBLICA_REAL]` (La que obtiene buscando "cuál es mi ip" en Google).

> **Nota:** Si su IP pública es dinámica, debe usar el servicio `Dynamic DNS` en pfSense en lugar de este registro `A`.

### 5.2. Instalación de Paquetes (pfSense)

Vaya a `System > Package Manager > Available Packages` e instale:
1.  `acme`
2.  `haproxy`

### 5.3. Generación de Certificado (ACME)

1.  **Crear Cuenta ACME:**
    * Vaya a `Services > ACME Certificates > Account keys`.
    * Haga clic en `+ Add`, cree una cuenta nueva (ej. `Cuenta ACME EINFO`) y regístrela.

2.  **Crear Certificado (Desafío DNS):**
    * Vaya a la pestaña **`Certificates`** y haga clic en `+ Add`.
    * **Name:** `Nextcloud_Cert`
    * **Acme Account:** Seleccione la cuenta que acaba de crear.
    * **Domain SAN List:**
        * **Domainname:** `nextcloud.einfo.space`
        * **Method:** `DNS-Manual`
    * Haga clic en **`Save`**.

3.  **Ejecutar la Validación (Parte 1):**
    * En la lista de certificados, haga clic en `Issue/Renew` para `Nextcloud_Cert`.
    * El log fallará (esto es normal) y le dará los valores para el registro `TXT`:
        * **Host:** `_acme-challenge.nextcloud`
        * **TXT value:** `[una_cadena_larga_aleatoria]`

4.  **Crear Registro TXT (Registrador de Dominio):**
    * Vuelva al panel de DNS de `einfo.space`.
    * Cree un nuevo registro:
        * **Tipo:** `TXT`
        * **Host:** `_acme-challenge.nextcloud`
        * **Valor:** `[la_cadena_larga_aleatoria_del_paso_3]`
    * Guarde el registro y **espere 10 minutos**.

5.  **Ejecutar la Validación (Parte 2):**
    * Vuelva a pfSense (`Services > ACME Certificates`) y haga clic en `Issue/Renew` nuevamente.
    * Esta vez, el log debería mostrar `Success`.

### 5.4. Configuración de HAProxy Backend

1.  Vaya a `Services > HAProxy > Backend`.
2.  Haga clic en `+ Add`.
3.  **Configuración:**
    * **Name:** `Nextcloud_Backend`
    * **Server List:**
        * **Name:** `vm_nextcloud`
        * **Address:** `172.20.13.102`
        * **Port:** `8080`
        * **SSL:** Deje esta casilla **DESMARCADA**.
4.  Guarde.

### 5.5. Configuración de HAProxy Frontend

1.  Vaya a `Services > HAProxy > Frontend`.
2.  Haga clic en `+ Add`.
3.  **Configuración:**
    * **Name:** `Nextcloud_Frontend`
    * **Listen Address:** `WAN address (SSL / 443)`
    * **Port:** `443`
    * **SSL Offloading:** **MARQUE** esta casilla.
    * **Certificate:** Seleccione el certificado `Nextcloud_Cert` que creó con ACME.
    * **Default Backend:** Seleccione `Nextcloud_Backend`.
4.  Guarde y haga clic en **`Apply Changes`**.

### 5.6. Regla de Firewall WAN (pfSense)

1.  Vaya a `Firewall > Rules > WAN`.
2.  Cree una nueva regla:
    * **Action:** `Pass`
    * **Protocol:** `TCP`
    * **Source:** `any`
    * **Destination:** `WAN address`
    * **Destination Port:** `HTTPS (443)`
    * **Description:** `Permitir HAProxy Nextcloud`
3.  Guarde y aplique los cambios.

### 5.7. Configuración Final de Nextcloud (Trusted Domains)

Debemos decirle a Nextcloud que confíe en el nuevo dominio público.

1.  Conéctese por SSH a su **VM de Ubuntu**.
2.  Edite el archivo de configuración:
    ```bash
    sudo nano /var/www/nextcloud/config/config.php
    ```
3.  Modifique `trusted_domains` y añada las líneas `overwrite` al final (antes del `);`):

    ```php
    'trusted_domains' =>
    array (
      0 => '172.20.13.102:8080',
      1 => 'nextcloud.einfo.space',
    ),

    'overwrite.cli.url' => '[https://nextcloud.einfo.space](https://nextcloud.einfo.space)',
    'overwritehost' => '[https://nextcloud.einfo.space](https://nextcloud.einfo.space)',
    'overwriteprotocol' => 'https',
    ```
4.  Guarde el archivo.

Su servicio `https://nextcloud.einfo.space` ahora está publicado y operativo.
