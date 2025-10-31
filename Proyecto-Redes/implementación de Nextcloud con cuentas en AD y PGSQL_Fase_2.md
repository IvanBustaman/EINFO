# Implementación de Nextcloud con PostgreSQL, AD y HAProxy Fase 2

Este documento detalla el procedimiento completo para instalar Nextcloud en un servidor Ubuntu, utilizando un servidor PostgreSQL remoto en openSUSE para la base de datos, autenticando usuarios contra un Directorio Activo (Active Directory) de Windows Server, y publicándolo de forma segura a Internet usando pfSense con ACME (Let's Encrypt) y HAProxy.
## Fase 2: Servidor de Base de Datos (openSUSE)

Configuramos el servidor PostgreSQL dedicado en `172.20.15.10`. este servidor se encuentra en nuestra VLAN 15 la cual es exlusivamente para BD

### 2.1. Instalación de PostgreSQL
en terminal procedemos a  instalar PGSQL. Lo bueno de este sistema es que dado que fue instalada una interfaz gráfica GNOME puede ser manipulada directo en proxmox o por SSH.
Abrimos terminal donde nos parezca más comodo
```bash
# Actualizar repositorios e instalar el servidor
sudo zypper refresh
sudo zypper install postgresql16-server postgresql16-client
```

## 2.2. Inicialización y Arranque del Servicio
Usamos el método manual initdb ya que postgresql-setup no inició.
Asignar propiedad al usuario postgres
```
sudo mkdir -p /var/lib/pgsql/data
sudo chown postgres:postgres /var/lib/pgsql/data
```

#### Cambiar al usuario postgres e inicializar
```
sudo -i -u postgres
initdb -D /var/lib/pgsql/data
exit
```

#### Habilitar e iniciar el servicio
```
sudo systemctl enable --now postgresql
```

#### Cambiar al usuario postgres
```
sudo -i -u postgres
```

#### Iniciar el shell de psql
```
psql
```

## 2.3. Creación de Usuario y Base de Datos
Creamos el usuario einfo y la base de datos einfo_db
```
CREATE USER einfo WITH PASSWORD '********!!'; * Puse asteríscos para no vulnerar la contraseña
CREATE DATABASE einfo_db OWNER einfo TEMPLATE template0 ENCODING 'UTF8';
GRANT ALL PRIVILEGES ON DATABASE einfo_db TO einfo;
```

* Una recomendación es que todos los usuarios y nombres de las bases sea en minúscula porque en la configuración tendremos problemas de conexión


#### Salir
```
\q
exit
```
## 2.4 Procedemos a editar el archivo postgresql.conf
```
sudo vi /var/lib/pgsql/data/postgresql.conf
```

## Buscamos la línea listen_addresses y la configuramos como "*"
```
listen_addresses = '*'
```
### ahora el archivo pg_hba.conf
```
sudo vi /var/lib/pgsql/data/pg_hba.conf
```
añadimos la siguiente línea hasta el final
```
host    einfo_db    einfo    172.20.13.102/32    md5
```
<img src="./assets/Imagen18_pg.png" width="400"/>.
Esta línea autoriza al usuario einfo a conectarse a einfo_db desde la IP de Nextcloud (.102) usando una contraseña (md5)

## 2.5. Configuración del Firewall.
Abrimos el puerto de PostgreSQL en el firewall local de openSUSE
```
sudo firewall-cmd --zone=public --add-port=5432/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl restart postgresql    (este es oara reiniciar el servicio)
```

