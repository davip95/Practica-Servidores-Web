# **Práctica Servidores web**
## 1er Trimestre

### 1. Instalación del servidor Apache

>Usaremos dos dominios mediante el archivo hosts: centro.intranet y departamentos.centro.intranet. El primero servirá el contenido mediante wordpress y el segundo una aplicación en python.

Como primer paso, desde el terminal de Linux de la máquina virtual, ejecutar los siguientes comandos:

```bash
sudo apt update
sudo apt install apache2
```

Si es la primera vez que utiliza sudo en esta sesión, se le pedirá que proporcione su contraseña de usuario para confirmar que tenga los privilegios adecuados para administrar los paquetes del sistema con apt. También se le solicitará que confirme la instalación de Apache al pulsar Y y ENTER.

Una vez que la instalación se complete, deberá ajustar la configuración de su firewall para permitir tráfico HTTP y HTTPS. UFW tiene diferentes perfiles de aplicaciones que puede aprovechar para hacerlo. Para enumerar todos los perfiles de aplicaciones de UFW disponibles, puede ejecutar lo siguiente:

```bash
sudo ufw app list
```

Verá un resultado como este:

```bash
Output
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

Para permitir tráfico únicamente en el puerto 80 utilice el perfil Apache:

```bash
sudo ufw allow in "Apache"
```

Puede realizar una verificación rápida para comprobar que todo se haya realizado según lo previsto dirigiéndose a la dirección IP pública de su servidor en su navegador web o, en su defecto, introduciendo "localhost" en la barra de navegación:

```
http://127.0.0.1
o
localhost
```
Se visualizará la siguiente página por defecto de Apache

![PruebaApache](https://github.com/davip95/Practica-Servidores-Web/blob/2e949fc0b873bfdb965dc88e075a6f8a160e34c5/Instalacion%20del%20servidor%20web%20apache/pruebaInstApache.PNG)

Para añadir los dos nuevos dominios, editamos el archivo *hosts* ubicado en /etc con permisos root:

```bash
sudo nano host
```

Añadiremos las dos siguientes líneas:

```
127.0.0.1 centro.intranet
127.0.0.1 departamentos.centro.intranet
```

Deberá quedar algo similar a esta imagen:

![DominiosHosts](https://github.com/davip95/Practica-Servidores-Web/blob/9f4b8089d4b982ae54e858edfd456639b4d67985/Instalacion%20del%20servidor%20web%20apache/dominioshosts.PNG)

A continuación guardaremos los cambios en el archivo.

### 2. Activación de módulos

>Activar los módulos necesarios para ejecutar php y acceder a mysql.

Para instalar MySQL, introduciremos el comando:

```bash
sudo apt install mysql-server
```

Cuando se le solicite, confirme la instalación al escribir Y y, luego, ENTER. Al terminar, compruebe si puede iniciar sesión en la consola de MySQL al escribir lo siguiente:

```bash
sudo mysql
```

Deberá ver una pantalla parecida a la siguiente:

![MySQL](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/mysql.PNG)

Para salir de la consola de MySQL, escriba lo siguiente:

```
mysql> exit
```

Como siguiente paso, instalaremos PHP.

Además del paquete php, necesitará php-mysql, un módulo PHP que permite que este se comunique con bases de datos basadas en MySQL. También necesitará libapache2-mod-php para habilitar Apache para gestionar archivos PHP. Los paquetes PHP básicos se instalarán automáticamente como dependencias.

Para instalar estos paquetes, ejecute lo siguiente:

```bash
sudo apt install php libapache2-mod-php php-mysql
```

Una vez que la instalación se complete, podrá ejecutar el siguiente comando para confirmar su versión de PHP:

```bash
php -v
```

Le tendrá que aparecer una pantalla con información similar a la siguiente:

![PHP](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/php.PNG)

### 3. WordPress

>Instala y configura wordpress.

#### Instalación de extensiones de PHP adicionales

Comenzaremos instalando las extensiones necesarias de PHP para Wordrpess y MySQL ejecutando los dos siguientes comandos:

```bash
sudo apt update 
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

Necesitaremos reiniciar Apache para cargar estas nuevas extensiones:

```bash
sudo systemctl restart apache2
```

#### Creación de una base de datos de MySQL y un usuario para WordPress

A continuación, crearemos una base de datos de MySQL y un usuario para WordPress. Para comenzar, inicie sesión en la cuenta root de MySQL (administrativa) emitiendo este comando:

```bash
mysql -u root -p
```

Si no puede acceder a su base de datos MySQL a través de root, como usuario sudo puede actualizar la contraseña de su usuario root iniciando sesión en la base de datos de esta forma:

```bash
sudo mysql -u root
```

Una vez que reciba la instrucción de MySQL, puede actualizar la contraseña del usuario root. Aquí, sustituya new_password por una contraseña segura de su elección.

```SQL
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password';
```

Ahora puede escribir *EXIT;* y puede volver a iniciar sesión en la base de datos a través de la contraseña con el siguiente comando:

```bash
mysql -u root -p
```

En la base de datos, puede crear una base de datos exclusiva para que WordPress la controle. Puede ponerle el nombre que quiera, pero usaremos el nombre wordpress en esta guía. Cree la base de datos para WordPress escribiendo lo siguiente:

```SQL
mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

A continuación, crearemos una cuenta de usuario separada de MySQL que usaremos exclusivamente para realizar operaciones en nuestra nueva base de datos. Crear bases de datos y cuentas específicas puede ayudarnos desde el punto de vista de administración y seguridad. Usaremos el nombre wordpressuser en esta guía, pero puede usar el nombre que sea más relevante para usted.

Crearemos esta cuenta, configuraremos una contraseña y concederemos acceso a la base de datos que hemos creado. Podemos hacerlo escribiendo el siguiente comando. Recuerde elegir una contraseña segura aquí para su usuario de base de datos donde tenemos password:

```SQL
mysql> CREATE USER 'wordpressuser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

A continuación, deje saber a la base de datos que nuestro wordpressuser debería tener acceso completo a la base de datos que configuramos:

```SQL
mysql> GRANT ALL ON wordpress.* TO 'wordpressuser'@'%';
```

Ahora tiene una base de datos y una cuenta de usuario, creadas específicamente para WordPress. Debemos eliminar los privilegios de modo que la instancia actual de MySQL sepa sobre los cambios recientes que hemos realizado:

```SQL
mysql> FLUSH PRIVILEGES;
```

Por último, cierre MySQL escribiendo *EXIT;*.

####  Ajuste de la configuración de Apache para permitir reemplazos y reescrituras .htaccess

A continuación, realizaremos algunos ajustes de menor importancia en nuestra configuración de Apache. Debe tener un archivo de configuración para su sitio en el directorio */etc/apache2/sites-available/*.

Utilizaremos */etc/apache2/sites-available/wordpress.conf*, pero debe sustituir la ruta a su archivo de configuración cuando proceda. Además, emplearemos */var/www/wordpress* como el directorio root de nuestra instalación de WordPress. Debería usar el root web especificada en su propia configuración.

Con nuestras rutas identificadas, podemos pasar a trabajar con .htaccess de forma que Apache pueda manejar los cambios en la configuración directorio por directorio.

Abra el archivo de configuración de Apache para su sitio web con un editor de texto como nano.

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Para permitir archivos .htaccess, debemos configurar la directiva AllowOverride dentro de un bloque Directory orientado a nuestro root de documentos. Agregue el siguiente bloque de texto dentro del bloque VirtualHost en su archivo de configuración. Asegúrese de utilizar el directorio root web correcto:

```bash
<Directory /var/www/wordpress/>
	AllowOverride All
</Directory>
```

![ConfigWordPress](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWordpress.PNG)

Cuando termine, guarde y cierre el archivo. 

A continuación, podemos habilitar mod_rewrite para usar la característica de permalink de WordPress:

```bash
sudo a2enmod rewrite
```

Antes de implementar los cambios realizados, compruebe que no hay errores de sintaxis ejecutando la siguiente prueba.

```bash
sudo apache2ctl configtest
```

Puede recibir un resultado como el siguiente:

![ConfigWordPress2](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWordpress2.PNG)

En tanto el resultado contenga Sintaxis OK, podrá continuar.

Reinicie Apache para implementar los cambios: Asegúrese de reiniciar ahora, incluso si ha reiniciado anteriormente en este tutorial.

```bash
sudo systemctl restart apache2
```

#### Descargar WordPress

A continuación, descargaremos y configuraremos el propio WordPress.

Cambie a un directorio que permita la escritura (recomendamos uno temporal como /tmp) y descargue la versión comprimida.

```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
```

Extraiga el archivo comprimido para crear la estructura de directorios de WordPress:

```bash
tar xzvf latest.tar.gz
```

Moveremos estos archivos a nuestro root de documentos por ahora. Antes de hacerlo, podemos añadir un archivo ficticio .htaccess de modo que esté disponible para que WordPress lo use más adelante.

Cree el archivo escribiendo lo siguiente:

```bash
touch /tmp/wordpress-6.1.1/wordpress/.htaccess
```

También copiaremos sobre el archivo de configuración de muestra al nombre de archivo que lee WordPress:

```bash
cp /tmp/wordpress-6.1.1/wordpress/wp-config-sample.php /tmp/wordpress-6.1.1/wordpress/wp-config.php
```

También podemos crear el directorio de actualización, de modo que WordPress no tenga problemas de permisos al intentar hacerlo por su cuenta siguiendo una actualización a su software:

```bash
mkdir /tmp/wordpress-6.1.1/wordpress/wp-content/upgrade
```

Ahora podemos copiar todo el contenido del directorio en nuestro root de documentos. Usaremos un punto al final de nuestro directorio de origen para indicar que todo lo que está dentro del directorio debe copiarse, incluyendo archivos ocultos (como el archivo .htaccess que hemos creado):

```bash
sudo cp -a /tmp/wordpress-6.1.1/wordpress/. /var/www/wordpress
```

#### Configurar el directorio de WordPress

Antes de realizar la configuración basada en web de WordPress, debemos ajustar algunos elementos en nuestro directorio de WordPress.

Un paso importante que debemos lograr es configurar permisos de archivo razonables y la propiedad.

Empezaremos por dar la propiedad de todos los archivos al usuario y al grupo www-data. Este es el usuario como el que se ejecuta el servidor web Apache, y este último deberá poder leer y escribir archivos de WordPress para presentar el sitio web y realizar actualizaciones automáticas.

Actualice la propiedad con el comando chown que le permite modificar la propiedad del archivo. Asegúrese de apuntar al directorio relevante de su servidor.

```bash
sudo chown -R www-data:www-data /var/www/wordpress
```

A continuación, ejecutaremos dos comandos find para establecer los permisos correctos de los directorios y archivos de WordPress:

```bash
sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;
sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;
```

Estos permisos deberían hacer que pueda trabajar de forma efectiva con WordPress, pero tenga en cuenta que algunos complementos y procedimientos pueden requerir ajustes adicionales.

Ahora, debemos realizar algunos cambios en el archivo de configuración principal de WordPress.

Cuando abramos el archivo, nuestra primera tarea será ajustar algunas claves secretas para proporcionar un nivel de seguridad a nuestra instalación. WordPress proporciona un generador seguro para estos valores, para que no tenga que crear valores correctos por su cuenta. Solo se utilizan internamente, de modo que no dañará la usabilidad el tener valores complejos y seguros aquí.

Para obtener valores seguros del generador de claves secretas de WordPress, escriba lo siguiente:

```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Obtendrá valores únicos que se parecen al resultado del bloque siguiente.

![configWordpress3](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWP3.PNG)

**Advertencia: Debe solicitar valores únicos cada vez. NO copie los siguientes valores.**

Son líneas de configuración que podemos pegar directamente en nuestro archivo de configuración para establecer claves seguras. Copie el resultado que obtuvo ahora.

A continuación, abra el archivo de configuración de WordPress:

```bash
sudo nano /var/www/wordpress/wp-config.php
```
Busque la sección que contiene los valores de ejemplo para esos ajustes. Donde pone *'put your unique phrase here');*, elimine esas líneas y pegue los valores que copió de la línea de comandos anteriores:

![configWordpress4](https://github.com/davip95/Practica-Servidores-Web/blob/50e46e705852a722d155149d11bd9a98c61a0eef/Instalacion%20del%20servidor%20web%20apache/configWP4.PNG)

A continuación, vamos a modificar algunos de los ajustes de conexión de la base de datos al principio del archivo. Debe ajustar el nombre de la base de datos, su usuario y la contraseña asociada que configuramos dentro de MySQL.

El otro cambio que debemos realizar es configurar el método que debe emplear WordPress para escribir el sistema de archivos. Debido a que hemos dado permiso al servidor web para escribir donde debe hacerlo, podemos fijar de forma explícita el método del sistema de archivos a “direct”. Si no lo configuramos con nuestros ajustes actuales, WordPress solicitaría las credenciales de FTP cuando realicemos algunas acciones.

Este ajuste se puede agregar debajo de los ajustes de conexión de la base de datos o en cualquier otra parte del archivo:

```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

define('FS_METHOD', 'direct');
```

Guarde y cierre el archivo cuando termine.

#### Completar la instalación a través de la interfaz web

Ahora que la configuración del servidor está completa, podemos finalizar la instalación a través de la interfaz web.

En su navegador web, vaya al nombre de dominio o a la dirección IP pública de su servidor:

```
https://127.0.0.1
```

Seleccione el idioma que desee usar:


### 4. Activación de "wsgi"

>Activar el módulo “wsgi” para permitir la ejecución de aplicaciones Python.



### 5. Aplicación Python

>Crea y despliega una pequeña aplicación python para comprobar que funciona correctamente.



### 6. Protección de acceso a la aplicación Python

>Adicionalmente protegeremos el acceso a la aplicación python mediante autenticación.



### 7. AWStat

>Instala y configura awstat.



### 8. Instalación de segundo servidor y phpmyadmin

>Instala un segundo servidor de tu elección (nginx, lighttpd) bajo el dominio “servidor2.centro.intranet”. Debes configurarlo para que sirva en el puerto 8080 y haz los cambios necesarios para ejecutar php. Instala phpmyadmin.
