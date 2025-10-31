## Fase 4: Integración con Active Directory

Conectamos Nextcloud a `laboratorio.local` (en `172.20.13.100`) para la autenticación de usuarios.

### 4.1. Creación del Usuario de Servicio (En Windows Server)

1.  En "Usuarios y equipos de Active Directory", tenemos una OU llamada `EINFO`.
<img src="./assets/Imagen22.png" width="400"/>

2.  Dentro de `EINFO`, creamos los usuarios y designamos unos para Nextcloud:
    * **Nombre:** `nextcloud`
    * **Nombre de inicio de sesión (sAMAccountName):** `nextcloud`
    * Marquecamos "La contraseña nunca caduca".
    
3.  El `distinguishedName` resultante será: `CN=nextcloud,OU=EINFO,DC=laboratorio,DC=local`.
<img src="./assets/Imagen23_DN.png" width="400"/>


### 4.2. Configuración de Nextcloud

1.  Inicie sesión en Nextcloud como administrador.
2.  Vaya a "Aplicaciones" y active **"LDAP / AD Integration"**.
   <img src="./assets/Imagen24_LDPA.png" width="400"/>
3.  Vaya a "Configuración" > "Integración LDAP / AD".
<img src="./assets/Imagen25_server.png" width="400"/>
** Pestaña "Servidor":**
* **Host:** `172.20.13.100`
* **Puerto:** `389`
* **DN de Usuario:** `CN=nextcloud,OU=EINFO,DC=laboratorio,DC=local`
* **Contraseña:** La contraseña del usuario `nextcloud` de AD.
* Haga clic en **"Guardar credenciales"**.
* **Base DN:** `DC=laboratorio,DC=local`
* Hacemos clic en "Probar Base DN". Debería aparecer un indicador verde como en la imagen.

** Pestaña "Usuarios":**
* **DN Base de Usuarios:** `DC=laboratorio,DC=local`
* **Sólo estas clases de objetos:** `user`

** Pestaña "Atributos de inicio de sesión":**
* **Atributo de inicio de sesión LDAP:** `sAMAccountName`

<img src="./assets/Imagen26_usuarios.png" width="400"/>
** Prueba Final:**
1.  Volvemos a la pestaña "Usuarios" y hacemos clic en "Verificar configuración y contar usuarios". Debemos ver un recuento de los usuarios.
2.  Cerramos la sesión e iniciamos sesión con un usuario regular del Directorio Activo.
3.  Para validar que es un usuario del AD no debemos poder cambiar la contraseña.
