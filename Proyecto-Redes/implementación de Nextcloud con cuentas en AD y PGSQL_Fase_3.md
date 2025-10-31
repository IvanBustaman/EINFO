## Fase 3: Servidor de Aplicación (Ubuntu Server)

Instalamos Nextcloud y su stack web en `172.20.13.102`.

### 3.1. Instalación de Dependencias (LAMP + Módulos PHP)

Instalamos Apache y todos los módulos de PHP necesarios (incluyendo `php-pgsql` para PostgreSQL y `php-ldap` para Active Directory).

```
sudo apt update
sudo apt install apache2
sudo apt install php libapache2-mod-php php-gd php-mbstring php-xml php-zip \
php-curl php-intl php-bcmath php-gmp php-pgsql php-ldap
```
### 3.2. Descargamos Nextcloud
bajamos la ultima version de Nextcloud con Web Get
```
cd /tmp *cambiamos a la carpeta temporal* 
wget [https://download.nextcloud.com/server/releases/latest.zip] 
```
### 3.3. Instalamos Nextcloud
Descomprimimos y establecemos los permisos correctos para Apache.
```
sudo unzip latest.zip -d /var/www/
sudo chown -R www-data:www-data /var/www/nextcloud/
sudo chmod -R 755 /var/www/nextcloud/
```
### 3.4. Configuración de Apache (Puerto 8080)
Cambiamos el puerto a 8080 para evitar conflictos ya que en nuestro servidor ubuntu por alguna razón nos decía que el puerto 80 ya estaba ocupado. Creamos el Virtual Host.
##### 3.4.1 Cambiar Puerto Global:
```
sudo vi /etc/apache2/ports.conf
```
<img src="./assets/Imagen20.png" width="400"/>

##### 3.4.2 Creamos el vistual Host
```
/etc/apache2/sites-available/nextcloud.conf
```
<img src="./assets/Imagen21_VH.png" width="400"/>

##### 3.4.3 Habilitamos m[odulos y sitio

```
sudo a2ensite nextcloud.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite headers env dir mime
```
### 3.5 Cambiamos los permisos de Logs
```
sudo chown -R www-data:www-data /var/log/apache2/
sudo chmod -R 750 /var/log/apache2/
 y procedemos a reiniicar apache con sudo systemctl restart apache2
```
### 3.6. Asistente de Instalación Web
* Entramos a nuestro navegador y vamos a http://172.20.13.102:8080.
* Creamos la cuenta de administrador (en este casso sera einfo).
* Vamos a la sección "Almacenamiento y base de datos".
* Seleccionamos PostgreSQL.
* Completamos los datos de la base de datos remota:
** Usuario de la Base de Datos: einfo
** Contraseña: *******
** Nombre de la Base de Datos: einfo_db (que es la que creamos en PGSQL)
** Host de la Base de Datos: 172.20.15.10 (nuestra IP de la VM de opensuse que sirve como BD)
* Damos clic en "Completar instalación".
