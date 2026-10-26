# Práctica: Instalación de WordPress en Ubuntu 24.04 (nuestro LAMP en marcha)

| | |
|---|---|
| **Ciclo** | Sistemas Microinformáticos y Redes (SMR) |
| **Módulo** | Aplicaciones Web |
| **Alumno/a** | _TU NOMBRE Y APELLIDOS_ |
| **Curso** | _TU CURSO (p. ej. 2025/2026)_ |
| **Fecha** | _FECHA DE ENTREGA_ |

> 🔒 **Seguridad:** en esta memoria no aparece ninguna contraseña real. Donde iría una contraseña se escribe `********` y las capturas tienen las contraseñas tapadas.

---

## Índice

1. [Conexión por SSH y comprobación del servidor](#apartado-1-conexión-por-ssh-y-comprobación-del-servidor)
2. [Dejar Apache con un único sitio](#apartado-2-dejar-apache-con-un-único-sitio)
3. [Crear la base de datos de WordPress](#apartado-3-crear-la-base-de-datos-de-wordpress)
4. [Descargar e instalar los ficheros de WordPress](#apartado-4-descargar-e-instalar-los-ficheros-de-wordpress)
5. [Terminar la instalación desde el navegador](#apartado-5-terminar-la-instalación-desde-el-navegador-la-parte-gráfica)
6. [Comprobamos que WordPress funciona](#apartado-6-comprobamos-que-wordpress-funciona)
7. [Para que no se rompa si cambia la IP](#apartado-7-para-que-no-se-rompa-si-cambia-la-ip)
8. [Dos ajustes para trabajar cómodamente](#apartado-8-dos-ajustes-para-trabajar-cómodamente)
9. [¿Qué pasa si MariaDB se para?](#apartado-9-qué-pasa-si-mariadb-se-para)
10. [Comprobación final](#apartado-10-comprobación-final)
11. [Conclusiones y problemas encontrados](#conclusiones-y-problemas-encontrados)

---

## Introducción: ¿cómo usamos una herramienta gráfica en un servidor sin pantalla?

Nuestro servidor Ubuntu **no tiene escritorio, ventanas ni ratón**: solo una terminal de texto. Aun así podemos usar WordPress porque **WordPress no es un programa que se abra en el servidor, es una web**.

Analogía del restaurante:

| En el restaurante | En nuestro caso |
|---|---|
| La **cocina** prepara los platos, pero el cliente no la ve | El **servidor** guarda y prepara WordPress, pero no dibuja nada |
| El **comedor** es donde el cliente ve y toca los platos | El **navegador de mi ordenador** es donde veo y uso WordPress |
| El **camarero** lleva los platos de la cocina al comedor | **Apache** lleva las páginas del servidor al navegador |

El servidor solo envía páginas web (texto con instrucciones de cómo dibujarlas) y **es el navegador, en mi ordenador, el que las dibuja**. Por eso el servidor no necesita interfaz gráfica.

Me conecto al servidor por **dos "puertas"**:

| Puerta | Para qué la uso | Con qué programa |
|---|---|---|
| **SSH (puerto 22)** | Administrar el servidor: instalar, copiar, configurar (en texto) | Terminal (`ssh`) |
| **HTTP (puerto 80)** | Usar WordPress: ver la web y el panel de control (gráfico) | Navegador |

**¿Qué es WordPress?** Es un **gestor de contenidos (CMS)**: una aplicación web que permite crear y mantener webs sin programar (entradas, páginas, imágenes, temas y plugins). Está hecha en **PHP** y guarda todo su contenido en la base de datos de **MariaDB**.

```text
Navegador ──HTTP:80──▶ Apache ──▶ PHP (ejecuta WordPress) ──▶ MariaDB (guarda los datos)
Terminal  ──SSH:22───▶ Administración del servidor
```

---

## Apartado 1. Conexión por SSH y comprobación del servidor

### 1.1. Averiguar la IP del servidor

```bash
hostname -I
```

**Qué hace:** muestra las direcciones IP del servidor. Necesito la IP para conectarme por SSH y para abrir WordPress en el navegador. La IP de mi servidor es `192.168.56.19`.

### 1.2. Conectarme por SSH desde mi ordenador

```bash
ssh saba@192.168.56.19
```

**Qué hace:** abre una sesión de terminal remota y cifrada con el servidor por el puerto 22. La primera vez pregunta si confío en el servidor (`yes`) y después pide la contraseña (`********`). A partir de aquí todos los comandos se escriben en esta terminal.

### 1.3. Comprobar que Apache, MariaDB y PHP siguen funcionando

```bash
systemctl is-active apache2 mariadb
php -v
```

| Comando | Explicación |
|---|---|
| `systemctl is-active apache2 mariadb` | Pregunta a `systemd` si los servicios `apache2` y `mariadb` están en marcha. Debe responder `active` dos veces. |
| `php -v` | Muestra la versión de PHP instalada (PHP 8.3.x). |

📸 **Captura: conexión SSH, `systemctl is-active` y `php -v`**

![Conexión SSH y comprobación de servicios](img/01-ssh-comprobacion.png)

### ❓ ¿Qué dos "puertas" (programas y puertos) usas y para qué sirve cada una?

> Uso **SSH (puerto 22)** con la **terminal** para administrar el servidor en modo texto: instalar paquetes, copiar ficheros, crear la base de datos y dar permisos. Y uso **HTTP (puerto 80)** con el **navegador** para usar WordPress de forma gráfica: terminar la instalación con el asistente, ver la web y entrar en el panel de administración.

---

## Apartado 2. Dejar Apache con un único sitio

En la práctica de Apache se crearon dos sitios (`sitio1` y `sitio2`) y se desactivó el sitio por defecto. Ahora el servidor tendrá **un solo sitio, WordPress**, accesible con la IP del servidor. Los dos sitios antiguos se **desactivan** (sus ficheros no se borran) y se reactiva el sitio por defecto, cuya carpeta es `/var/www/html`.

### 2.1. Cambiar los sitios activos

```bash
sudo a2dissite sitio1.conf sitio2.conf
sudo a2ensite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

| Comando | Explicación |
|---|---|
| `sudo a2dissite sitio1.conf sitio2.conf` | **Desactiva** los sitios 1 y 2: elimina sus enlaces simbólicos de `sites-enabled`. Los ficheros siguen en `sites-available`. |
| `sudo a2ensite 000-default.conf` | **Activa** el sitio por defecto (crea el enlace simbólico en `sites-enabled`). Su carpeta es `/var/www/html`. |
| `sudo apache2ctl configtest` | Comprueba que la configuración no tiene errores. Debe responder `Syntax OK`. |
| `sudo systemctl reload apache2` | Recarga la configuración sin cortar las conexiones activas. |

> **Nota:** si `configtest` muestra el aviso *"Could not reliably determine the server's fully qualified domain name"*, se soluciona definiendo un `ServerName` global:
>
> ```bash
> echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/servername.conf
> sudo a2enconf servername
> sudo apache2ctl configtest
> sudo systemctl reload apache2
> ```
>
> `tee` con `sudo` crea el fichero de configuración (la redirección `>` no heredaría los permisos de `sudo`), `a2enconf` lo activa y `configtest` debe mostrar ya solo `Syntax OK`.

📸 **Captura: cambio de sitios y `configtest`**

![Cambio de sitios activos](img/02-apache-cambio-sitios.png)

### 2.2. Comprobar que solo hay un sitio activo y qué carpeta usa

```bash
ls /etc/apache2/sites-enabled/
grep DocumentRoot /etc/apache2/sites-enabled/000-default.conf
```

| Comando | Explicación |
|---|---|
| `ls /etc/apache2/sites-enabled/` | Lista los sitios activos. Debe aparecer **solo** `000-default.conf`. |
| `grep DocumentRoot /etc/apache2/sites-enabled/000-default.conf` | Busca la línea `DocumentRoot`: la carpeta desde la que Apache sirve el sitio (`/var/www/html`). |

📸 **Captura: sitios activos y `DocumentRoot`**

![Sitios activos y DocumentRoot](img/03-apache-sitio-unico.png)

### 2.3. Comprobar que el navegador llega al servidor

En el navegador de mi ordenador abro `http://192.168.56.19`. Aparece la página de Apache. Ya no hace falta usar `sitio1.local` ni `sitio2.local`.

📸 **Captura: página de Apache en el navegador**

![Página de Apache en el navegador](img/04-apache-navegador.png)

### ❓ ¿Qué diferencia hay entre desactivar un sitio con `a2dissite` y borrar sus ficheros? ¿Por qué ahora ves la página por defecto y antes veías sitio1?

> **Desactivar** con `a2dissite` solo elimina el enlace simbólico de `sites-enabled`: Apache deja de servir el sitio, pero su configuración sigue en `sites-available` y puedo reactivarlo con `a2ensite`. **Borrar** los ficheros elimina el sitio de verdad y no se recupera sin copia.
>
> Ahora veo la página por defecto porque **desactivé `sitio1` y `sitio2` y activé `000-default.conf`**, cuyo `DocumentRoot` es `/var/www/html`. Antes veía `sitio1` porque era el sitio activo que respondía a esa petición.

---

## Apartado 3. Crear la base de datos de WordPress

WordPress guarda todo en una base de datos de MariaDB. Analogía del **armario con llave**:

| Qué creamos | Nombre | Para qué |
|---|---|---|
| El armario (base de datos) | `wordpress` | Donde WordPress guarda todos sus datos |
| La llave (usuario) | `wp_user` | Con lo que WordPress entra en el armario |
| El permiso | Todos, pero solo sobre `wordpress` | Esa llave abre ese armario y ningún otro |

> La contraseña de `wp_user` (mínimo 12 caracteres: letras, números y `_`) está guardada en un sitio privado. En esta memoria aparece como `********`.

### 3.1. Entrar en MariaDB y crear la base de datos, el usuario y el permiso

```bash
sudo mariadb
```

```sql
CREATE DATABASE wordpress;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY '********';
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
exit
```

| Orden | Qué hace |
|---|---|
| `sudo mariadb` | Abre la consola de MariaDB como administrador. |
| `CREATE DATABASE wordpress;` | Crea el armario: la base de datos `wordpress`. |
| `CREATE USER 'wp_user'@'localhost' IDENTIFIED BY '********';` | Crea la llave: el usuario `wp_user`, que solo puede conectarse desde el propio servidor (`localhost`), con su contraseña. |
| `GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';` | Da permiso total al usuario **solo** sobre la base `wordpress` (`wordpress.*`). |
| `exit` | Sale de la consola de MariaDB. |

Cada una de las tres órdenes SQL responde `Query OK`.

📸 **Captura: creación de la base de datos y el usuario (contraseña tapada)**

![Creación de la base de datos y el usuario](img/05-mariadb-creacion.png)

### 3.2. Comprobar que la llave funciona

```bash
mariadb -u wp_user -p wordpress
```

| Parte | Explicación |
|---|---|
| `mariadb` | Cliente de línea de comandos de MariaDB. |
| `-u wp_user` | Entra con el usuario `wp_user`. |
| `-p` | Pide la contraseña (al escribirla no se ve nada). |
| `wordpress` | Base de datos a la que se conecta. |

Aparece el prompt `MariaDB [wordpress]>`, así que todo funciona. Salgo con `exit`.

📸 **Captura: entrada con `wp_user`**

![Entrada con wp_user](img/06-mariadb-wp-user.png)

### ❓ ¿Por qué WordPress usa su propio usuario `wp_user` y no el usuario root de MariaDB?

> Por **seguridad** (principio de mínimo privilegio). Si WordPress usara `root` y alguien explotara un fallo (por ejemplo, un plugin vulnerable), tendría control total sobre **todas** las bases de datos del servidor. Con `wp_user` el daño se limita a la base `wordpress`. Además, la contraseña queda guardada en `wp-config.php`, y es preferible que ahí no esté la de `root`.

---

## Apartado 4. Descargar e instalar los ficheros de WordPress

### 4.1. Descargar y descomprimir WordPress (en la carpeta temporal)

```bash
sudo apt install wget -y
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
ls wordpress
```

| Comando | Explicación |
|---|---|
| `sudo apt install wget -y` | Instala `wget`, el programa para descargar ficheros desde la terminal. `-y` responde "sí" automáticamente. |
| `cd /tmp` | Me muevo a la carpeta temporal para no ensuciar mi carpeta personal. |
| `wget https://wordpress.org/latest.tar.gz` | Descarga la última versión de WordPress comprimida en `.tar.gz`. |
| `tar -xzf latest.tar.gz` | Descomprime el fichero y crea la carpeta `wordpress` (`x` extraer, `z` descomprimir gzip, `f` indica el fichero). |
| `ls wordpress` | Lista el contenido: `index.php`, `wp-admin`, `wp-content`, `wp-config-sample.php`... |

📸 **Captura: descarga y descompresión**

![Descarga y descompresión de WordPress](img/4_captura_descargar_descomprimir.png)

### 4.2. Colocarlo en la carpeta de mi sitio

```bash
sudo rm /var/www/html/index.html
sudo cp -a /tmp/wordpress/. /var/www/html/
rm -r /tmp/wordpress /tmp/latest.tar.gz
ls /var/www/html
```

| Comando | Explicación |
|---|---|
| `sudo rm /var/www/html/index.html` | Borra la página de bienvenida de Apache. Si no se borra, tendría prioridad sobre WordPress (`index.html` se sirve antes que `index.php`). |
| `sudo cp -a /tmp/wordpress/. /var/www/html/` | Copia **todo el contenido** de WordPress (incluidos ficheros ocultos, por el `.` final) a la carpeta del sitio. `-a` copia de forma recursiva conservando permisos y atributos. |
| `rm -r /tmp/wordpress /tmp/latest.tar.gz` | Limpia: borra la carpeta descomprimida y el `.tar.gz`, que ya no hacen falta. `-r` borra carpetas de forma recursiva. |
| `ls /var/www/html` | Comprueba que WordPress está en la carpeta del sitio. |

📸 **Captura: copia de WordPress a `/var/www/html`**

![Copia de WordPress a /var/www/html](img/4_1_colocar_carpeta_sitio.png)

### ❓ Explica qué hace cada uno de los comandos que acabas de ejecutar.

> Están explicados en las tablas de 4.1 y 4.2: instalo `wget`, descargo WordPress en `/tmp` y lo descomprimo con `tar`; borro el `index.html` de Apache para que no tape a WordPress; copio todo el contenido a `/var/www/html` con `cp -a`; elimino los restos de `/tmp` con `rm -r` y compruebo con `ls`.

### 4.3. Dar los permisos correctos

Apache se ejecuta con el usuario `www-data`, y WordPress necesita poder escribir (guardar imágenes, instalar plugins, crear su configuración...).

```bash
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
ls -ld /var/www/html
```

| Comando | Explicación |
|---|---|
| `sudo chown -R www-data:www-data /var/www/html` | Cambia el **propietario y el grupo** de la carpeta y todo su contenido (`-R`, recursivo) a `www-data`, el usuario con el que corre Apache. |
| `sudo find /var/www/html -type d -exec chmod 755 {} \;` | Busca todas las **carpetas** (`-type d`) y les da permisos `755`: el propietario lee, escribe y entra; el resto solo lee y entra. |
| `sudo find /var/www/html -type f -exec chmod 644 {} \;` | Busca todos los **ficheros** (`-type f`) y les da permisos `644`: el propietario lee y escribe; el resto solo lee. |
| `ls -ld /var/www/html` | Muestra permisos, propietario y grupo de la propia carpeta (`-d` evita listar su contenido). |

La última orden muestra `www-data www-data` como propietario y grupo.

📸 **Capturas: permisos aplicados y resultado de `ls`**

![Permisos aplicados](img/4_2_darpermisos.png)

![Resultado de ls y ls -ld](img/4_3_resultado_ls.png)

### ❓ ¿Por qué hay que cambiar el propietario a `www-data`? ¿Qué podría fallar si siguiera siendo tu usuario?

> Porque **Apache (y por tanto PHP/WordPress) se ejecuta con `www-data`**, y WordPress necesita **escribir** en la carpeta: crear `wp-config.php` durante la instalación, subir imágenes a `wp-content/uploads`, instalar o actualizar temas y plugins y crear el `.htaccess`.
>
> Si el propietario siguiera siendo mi usuario, con permisos `755/644` `www-data` solo podría **leer**, y fallarían cosas como: el asistente no podría crear `wp-config.php`, no se podrían subir imágenes, no se podrían instalar plugins ni temas desde el panel y no se podrían actualizar.

---

## Apartado 5. Terminar la instalación desde el navegador (la parte gráfica)

En el navegador de mi ordenador abro `http://192.168.56.19`. Aparece el asistente de instalación:

| Pantalla | Qué hice |
|---|---|
| 1. Idioma | Elegí **Español** y pulsé *Continuar* |
| 2. Bienvenida | Pulsé *¡Vamos a ello!* |
| 3. Datos de la base de datos | Rellené la tabla siguiente y pulsé *Enviar* |
| 4. Todo en orden | Pulsé *Iniciar la instalación* |
| 5. Información necesaria | Rellené la segunda tabla y pulsé *Instalar WordPress* |
| 6. Éxito | Pulsé *Iniciar sesión* |

### Pantalla 3: datos de la base de datos

| Campo | Valor |
|---|---|
| Nombre de la base de datos | `wordpress` |
| Nombre de usuario | `wp_user` |
| Contraseña | `********` (la elegida en el apartado 3) |
| Servidor de la base de datos | `localhost` |
| Prefijo de tabla | `wp_` (por defecto) |

📸 **Captura: pantalla 3 (contraseña tapada)**

![Pantalla 3 del asistente](img/07-wp-pantalla3.png)

### Pantalla 5: datos del sitio

| Campo | Valor |
|---|---|
| Título del sitio | WordPress de Saba |
| Nombre de usuario | Uno propio (no `admin`, es el primero que prueban los atacantes) |
| Contraseña | `********` (fuerte, guardada en un sitio privado) |
| Correo electrónico | Un correo de práctica (tapado en la captura) |
| Visibilidad en motores de búsqueda | **Disuadir a los motores de búsqueda** marcado (sitio de prácticas) |

📸 **Captura: pantalla 5 (contraseña y correo tapados)**

![Pantalla 5 del asistente](img/08-wp-pantalla5.png)

### Qué ha pasado "por debajo"

- Al enviar la **pantalla 3**, WordPress comprobó que podía entrar en la base de datos con `wp_user` y **creó él solo su fichero de configuración**, `wp-config.php` (por eso necesitaba permisos de escritura).
- En la **pantalla 5**, WordPress **creó las tablas en MariaDB** y mi **usuario administrador**.

Inicio sesión con el usuario y la contraseña creados y accedo al **escritorio de WordPress**.

📸 **Captura: escritorio de WordPress**

![Escritorio de WordPress](img/09-wp-escritorio.png)

### ❓ ¿Qué partes de la instalación has hecho en la terminal y cuáles en el navegador? ¿Por qué el servidor no necesita interfaz gráfica?

> **En la terminal (SSH):** conectarme al servidor, dejar Apache con un único sitio, crear la base de datos y el usuario en MariaDB, descargar WordPress, copiarlo a `/var/www/html` y darle los permisos.
>
> **En el navegador:** terminar la instalación con el asistente gráfico (datos de la base de datos, título, usuario y contraseña) y, después, usar el panel de WordPress.
>
> El servidor no necesita interfaz gráfica porque **WordPress es una web**: el servidor solo envía páginas y es el **navegador de mi ordenador** el que las dibuja.

---

## Apartado 6. Comprobamos que WordPress funciona

### 6.1. La web pública y el panel de administración

- Web pública: `http://192.168.56.19` (se ve mi sitio con el título elegido).
- Panel de administración: `http://192.168.56.19/wp-admin` (inicio sesión con mi usuario).

📸 **Captura: web pública**

![Web pública de WordPress](img/10-wp-web-publica.png)

### 6.2. Publicar una entrada de prueba

En el panel: **Entradas → Añadir nueva**. Escribo un título y un texto (mi nombre y la fecha), pulso **Publicar** y compruebo que aparece en la web pública.

📸 **Captura: entrada publicada en la web pública**

![Entrada publicada](img/11-wp-entrada-publicada.png)

### 6.3. ¿Dónde ha guardado WordPress mi entrada?

```bash
sudo mariadb wordpress -e "SHOW TABLES;"
```

| Parte | Explicación |
|---|---|
| `sudo mariadb wordpress` | Entra en MariaDB, directamente en la base `wordpress`. |
| `-e "SHOW TABLES;"` | Ejecuta la orden indicada (`-e`, *execute*) y sale. `SHOW TABLES;` lista las tablas de la base de datos. |

Aparecen unas 12 tablas cuyo nombre empieza por `wp_`:

| Tabla | Qué guarda |
|---|---|
| `wp_posts` ⭐ | Entradas y páginas de la web (y datos de las imágenes subidas) |
| `wp_users` ⭐ | Usuarios: nombre, correo y contraseña cifrada |
| `wp_options` ⭐ | Configuración del sitio: título, tema, plugins y dirección web |
| `wp_postmeta`, `wp_usermeta`, `wp_commentmeta`, `wp_termmeta` | Datos extra de entradas, usuarios, comentarios y categorías |
| `wp_comments` | Comentarios de los visitantes |
| `wp_terms`, `wp_term_taxonomy`, `wp_term_relationships` | Categorías y etiquetas, y su relación con las entradas |
| `wp_links` | Enlaces (función antigua, casi no se usa) |

📸 **Captura: resultado de `SHOW TABLES`**

![Resultado de SHOW TABLES](img/12-show-tables.png)

### 6.4. Dónde está cada cosa

| Qué | Dónde |
|---|---|
| Los ficheros de WordPress | `/var/www/html/` |
| La conexión con la base de datos | `/var/www/html/wp-config.php` (**no compartir nunca**: contiene la contraseña) |
| Temas, plugins e imágenes subidas | `/var/www/html/wp-content/` (`themes`, `plugins` y `uploads`) |
| Las entradas, usuarios y ajustes | En la base de datos `wordpress` de MariaDB |

### ❓ ¿Dónde guarda WordPress las entradas: en ficheros del servidor o en la base de datos? ¿Cómo lo has comprobado?

> Las guarda **en la base de datos** (MariaDB), en la tabla `wp_posts`, y no en ficheros del servidor. Lo comprobé en la terminal con `sudo mariadb wordpress -e "SHOW TABLES;"`, que lista las tablas que WordPress creó en la base `wordpress`, entre ellas `wp_posts`. En `/var/www/html` solo están el código de WordPress, los temas, los plugins y las imágenes subidas.

---

## Apartado 7. Para que no se rompa si cambia la IP

WordPress apunta en su base de datos la dirección con la que se instaló. Si la IP de la máquina virtual cambia, WordPress seguiría usando la antigua: la web se vería sin diseño y el panel llevaría a una dirección que ya no existe. Con dos líneas se le indica que **use siempre la dirección con la que le visitan**.

### 7.1. Editar `wp-config.php`

```bash
sudo nano /var/www/html/wp-config.php
```

Busco con `Ctrl+W` la línea `/* That's all, stop editing! */` y, justo **encima**, añado estas dos líneas (dejando una línea en blanco antes del comentario):

```php
define( 'WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] );
define( 'WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] );
```

| Elemento | Explicación |
|---|---|
| `WP_HOME` | Dirección que WordPress usa para la **web pública**. |
| `WP_SITEURL` | Dirección donde están **los ficheros de WordPress**. |
| `$_SERVER['HTTP_HOST']` | Variable de PHP con la dirección con la que el navegador ha pedido la página (la IP o el nombre escrito). |

Guardo con `Ctrl+O`, `Enter`, y salgo con `Ctrl+X`.

📸 **Captura: las dos líneas dentro de `wp-config.php` (sin mostrar la contraseña)**

![Líneas añadidas en wp-config.php](img/13b-wp-config-editor.png)

### 7.2. Comprobar que no hay errores

```bash
grep -n "WP_HOME\|WP_SITEURL" /var/www/html/wp-config.php
php -l /var/www/html/wp-config.php
```

| Comando | Explicación |
|---|---|
| `grep -n "WP_HOME\|WP_SITEURL" ...` | Muestra solo las dos líneas añadidas, con su número de línea, sin enseñar el resto del fichero (que contiene la contraseña). |
| `php -l ...` | *Lint*: revisa la sintaxis del fichero PHP sin ejecutarlo. Debe responder `No syntax errors detected`. |

Recargo la web en el navegador y sigue funcionando.

> Estas dos líneas son cómodas para un laboratorio. En un servidor real se escribiría la dirección fija del sitio.

📸 **Captura: `grep` y `php -l`**

![grep y php -l sobre wp-config.php](img/13-wp-config-php-l.png)

---

## Apartado 8. Dos ajustes para trabajar cómodamente

### 8.1. Direcciones "bonitas" (enlaces permanentes)

Por defecto, las entradas tienen direcciones como `http://IP/?p=1`. WordPress puede usar direcciones claras como `http://IP/mi-entrada/`, pero para eso crea un fichero `.htaccess` que Apache **ignora por defecto**. Si no se permite, las entradas dan **error 404**.

**1. Permitir a Apache leer el `.htaccess`:**

```bash
sudo nano /etc/apache2/conf-available/wordpress.conf
```

Contenido del fichero:

```apache
<Directory /var/www/html>
    AllowOverride All
</Directory>
```

| Elemento | Explicación |
|---|---|
| `<Directory /var/www/html>` | Aplica la configuración a esa carpeta. |
| `AllowOverride All` | Permite que los ficheros `.htaccess` de esa carpeta modifiquen la configuración de Apache. |

**2. Activarlo:**

```bash
sudo a2enmod rewrite
sudo a2enconf wordpress
sudo apache2ctl configtest
sudo systemctl restart apache2
```

| Comando | Explicación |
|---|---|
| `sudo a2enmod rewrite` | Activa el módulo `rewrite`, que reescribe direcciones (necesario para las URL bonitas). Ya estaba activo de la práctica de Apache; así me aseguro. |
| `sudo a2enconf wordpress` | Activa el fichero `wordpress.conf`. |
| `sudo apache2ctl configtest` | Comprueba la sintaxis (`Syntax OK`). |
| `sudo systemctl restart apache2` | Reinicia Apache para aplicar los cambios. |

📸 **Captura: configuración de Apache y su activación**

![Configuración wordpress.conf y activación](img/14-apache-wordpress-conf.png)

**3. Cambiar el formato en WordPress:** **Ajustes → Enlaces permanentes → Nombre de la entrada → Guardar cambios**. Abro contenido desde la portada y compruebo que se ve y que su dirección ya es "bonita".

📸 **Capturas: direcciones bonitas**

![Dirección bonita de una página](img/15-enlace-permanente.png)

![Dirección bonita de una entrada](img/15b-enlace-entrada.png)

**4. Comprobar que WordPress creó el `.htaccess`:**

```bash
ls -la /var/www/html/.htaccess
```

**Qué hace:** lista el fichero `.htaccess` (oculto por empezar por punto) con sus permisos y propietario, para confirmar que WordPress lo ha creado.

📸 **Captura: fichero `.htaccess`**

![Fichero .htaccess](img/16-htaccess.png)

### 8.2. Poder subir ficheros grandes

PHP solo deja subir ficheros de 2 MB por defecto. Subo el límite a 64 MB:

```bash
printf 'upload_max_filesize = 64M\npost_max_size = 64M\n' | sudo tee /etc/php/8.3/apache2/conf.d/99-wordpress.ini
sudo systemctl restart apache2
```

| Elemento | Explicación |
|---|---|
| `printf '...\n...\n'` | Escribe dos líneas de configuración (`\n` es un salto de línea). |
| `upload_max_filesize = 64M` | Tamaño máximo de **un fichero** subido. |
| `post_max_size = 64M` | Tamaño máximo de **todos los datos** de una petición (igual o mayor que el anterior). |
| `\| sudo tee /etc/php/8.3/apache2/conf.d/99-wordpress.ini` | Guarda el texto en un fichero de configuración adicional de PHP. El `99-` hace que se cargue el último y prevalezca sobre los valores por defecto. |
| `sudo systemctl restart apache2` | Reinicia Apache para que PHP lea la nueva configuración. |

Compruebo en el panel: **Medios → Añadir nuevo** indica un tamaño máximo de **64 MB**.

📸 **Captura: tamaño máximo de subida**

![Tamaño máximo de subida de 64 MB](img/17-subida-64mb.png)

### ❓ ¿Por qué las direcciones "bonitas" dan error 404 si Apache no tiene permitido leer el `.htaccess`?

> Las direcciones bonitas (`/mi-entrada/`) no corresponden a ningún fichero ni carpeta real. WordPress las hace funcionar con reglas de reescritura en el `.htaccess`, que redirigen esas peticiones a `index.php`, y este ya busca la entrada en la base de datos. Si Apache tiene `AllowOverride None` (el valor por defecto), **ignora el `.htaccess`**, no se aplican las reglas y Apache busca un fichero o carpeta llamada `mi-entrada`, que no existe, y responde **404**.

---

## Apartado 9. ¿Qué pasa si MariaDB se para?

WordPress depende de MariaDB: sin base de datos no puede mostrar nada. Lo compruebo parándola a propósito.

### 9.1. Parar MariaDB y ver el error

```bash
sudo systemctl stop mariadb
```

**Qué hace:** detiene el servicio de MariaDB.

Al recargar `http://192.168.56.19` aparece el mensaje **"Error al establecer una conexión con la base de datos"**.

📸 **Captura: error en el navegador**

![Error de conexión con la base de datos](img/18-error-base-datos.png)

### 9.2. Arrancar MariaDB de nuevo

```bash
sudo systemctl start mariadb
systemctl is-active mariadb
```

| Comando | Explicación |
|---|---|
| `sudo systemctl start mariadb` | Arranca de nuevo el servicio de MariaDB. |
| `systemctl is-active mariadb` | Comprueba que está en marcha (responde `active`). |

📸 **Captura: `stop`, `start` e `is-active`**

![Parar y arrancar MariaDB](img/19-mariadb-stop-start.png)

Al recargar la web, vuelve a funcionar.

📸 **Captura: web recuperada**

![Web recuperada](img/20-web-recuperada.png)

### Guía rápida cuando WordPress muestre ese error

1. **¿Está MariaDB en marcha?** → `systemctl is-active mariadb` (si no, `sudo systemctl start mariadb`).
2. **¿Siguen siendo correctos el usuario y la contraseña?** → pruebo a entrar con `mariadb -u wp_user -p wordpress`.

### ❓ Cuando WordPress muestra "Error al establecer una conexión con la base de datos", ¿qué dos cosas comprobarías y en qué orden?

> 1. **Primero, si MariaDB está en marcha:** `systemctl is-active mariadb`. Si está parada, la arranco con `sudo systemctl start mariadb`. Es lo primero porque es la causa más habitual y la más rápida de comprobar.
> 2. **Después, si el usuario y la contraseña siguen siendo correctos:** intento entrar con `mariadb -u wp_user -p wordpress`. Si falla, las credenciales de `wp-config.php` no coinciden con las de MariaDB.

---

## Apartado 10. Comprobación final

```bash
systemctl is-active apache2 mariadb
php -v
ls /etc/apache2/sites-enabled/
sudo mariadb wordpress -e "SHOW TABLES;"
```

| Comando | Qué comprueba |
|---|---|
| `systemctl is-active apache2 mariadb` | Apache y MariaDB están activos |
| `php -v` | Versión de PHP instalada |
| `ls /etc/apache2/sites-enabled/` | Solo hay un sitio activo |
| `sudo mariadb wordpress -e "SHOW TABLES;"` | WordPress creó sus tablas en la base de datos |

📸 **Captura: comprobación final**

![Comprobación final](img/21-comprobacion-final.png)

### Estado final del servidor

| Pieza | Programa | Estado |
|---|---|---|
| Linux | Ubuntu 24.04 | Instalado |
| Apache | Apache2, con un único sitio | Funcionando |
| MariaDB | MariaDB 10.11 | Funcionando, con la base `wordpress` |
| PHP | PHP 8.3 | Funcionando |
| WordPress | Instalado en `/var/www/html` | Funcionando y accesible desde el navegador |

### Datos para trabajar con WordPress

| Dato | Valor |
|---|---|
| Dirección de la web | `http://192.168.56.19` |
| Panel de administración | `http://192.168.56.19/wp-admin` |
| Usuario administrador | El creado en el apartado 5 |
| Contraseña | `********` (guardada en privado) |

Para terminar, hice una **nueva instantánea (snapshot)** de la máquina virtual, que será mi punto de partida seguro para trabajar con WordPress.

---

## Conclusiones y problemas encontrados

**Conclusiones**

- WordPress es una aplicación web: el servidor la prepara y el navegador la muestra. Por eso se instala y administra por SSH (texto) y se usa por HTTP (gráfico).
- El funcionamiento depende de la cadena **Apache → PHP → MariaDB**: Apache sirve la web, PHP ejecuta WordPress y MariaDB guarda los datos. Si falla una pieza (por ejemplo MariaDB), la web deja de funcionar, como comprobé en el apartado 9.
- Los **permisos** (`www-data`) y la **seguridad** (usuario de base de datos propio con permisos mínimos, administrador distinto de `admin`, contraseñas fuera de GitHub) son parte esencial de una instalación correcta.
- Aprendí a editar `wp-config.php` y a comprobar su sintaxis con `php -l` sin exponer la contraseña de la base de datos.

**Problemas encontrados**

No tuve ningún problema durante la práctica: todos los pasos funcionaron siguiendo el enunciado.

---

## Referencias

- Documentación de WordPress: <https://wordpress.org/documentation/>
- Requisitos de WordPress: <https://wordpress.org/about/requirements/>
