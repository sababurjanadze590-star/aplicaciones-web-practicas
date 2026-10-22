# Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04 (Completando el LAMP)

- **Módulo:** Aplicaciones Web (SMR)[cite: 46]
- **Alumno:** Saba
- **Fecha:** Octubre 2026

---

## Índice
1. [Preparación del sistema](#apartado-1-preparación-del-sistema)
2. [Instalación de MariaDB](#apartado-2-instalación-de-mariadb)
3. [Comprobación del funcionamiento](#apartado-3-comprobación-del-funcionamiento)
4. [Comandos principales de administración](#apartado-4-comandos-principales-de-administración)
5. [¿Qué pasa si MariaDB está parado?](#apartado-5-qué-pasa-si-mariadb-está-parado)
6. [Instalación de PHP](#apartado-6-instalación-de-php)
7. [Comprobación final](#apartado-7-comprobación-final)

---

## Apartado 1. Preparación del sistema
Antes de empezar con la base de datos, dejamos activo únicamente el sitio web por defecto de Apache y actualizamos los paquetes del sistema operativo.

### Comandos ejecutados:
```bash
sudo a2ensite 000-default.conf
```
**¿Qué hace?**: Vuelve a habilitar el sitio web virtual predeterminado de Apache por seguridad.
```bash
sudo systemctl reload apache2
```
**¿Qué hace?**: Recarga la configuración del servidor web Apache para aplicar los cambios del sitio predeterminado.

```bash
sudo apt update
```
**¿Qué hace?**: Actualiza la lista local de paquetes y versiones disponibles en los repositorios oficiales de Ubuntu.

![Imagen](imagenes/1.1captura000default.png)

```bash
systemctl is-active apache2
```
**¿Qué hace?**: Comprueba de forma rápida si el servicio de Apache se encuentra ejecutándose correctamente (debe responder `active`).

![Imagen](imagenes/1.apache2active.png)


## Apartado 2. Instalación de MariaDB
Instalamos el sistema gestor de bases de datos relacionales MariaDB en nuestra máquina virtual con Ubuntu 24.04

```bash
sudo apt install mariadb-server -y
```
**¿Qué hace?**: Descarga e instala el paquete del servidor de bases de datos MariaDB de forma desatendida

![Imagen](imagenes/2.captura.instal.mariaDB.png)

```bash
mariadb --version
```
**¿Qué hace?**: Consulta la versión exacta de MariaDB que se acaba de instalar en el sistema

![Imagen](imagenes/2.1.captura.version.MariDB.png)


## Apartado 3. Comprobación del funcionamiento
Verificamos que el servicio de base de datos está activo, configurado para arrancar automáticamente con el sistema y probamos el acceso al monitor interactivo.

```bash
sudo systemctl status mariadb
```
**¿Qué hace?**: Muestra el estado detallado del servicio MariaDB, comprobando que está en ejecución (`active running`).

![Imagen](imagenes/3.1.captura.status.maridb.png)

```bash
systemctl is-enabled mariadb
```
**¿Qué hace?**: Verifica si el servicio está configurado para iniciarse automáticamente cada vez que se enciende la máquina virtual (`enabled`).

```bash
sudo mariadb
```
**¿Qué hace?**: Accede al monitor de la base de datos con privilegios de administrador. Al entrar, el prompt cambia a `MariaDB [(none)]>`. (Para salir de la consola escribimos `exit` y pulsamos Enter)

![Imagen](imagenes/3.captura.mariadb.monitor.png)

## Apartado 4. Comandos principales
Tabla resumen con los comandos esenciales para administrar el servicio de MariaDB.












