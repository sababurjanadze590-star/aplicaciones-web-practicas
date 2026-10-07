# Práctica: Instalación, configuración y securización de Apache en Ubuntu 24.04

- **Módulo:** Aplicaciones Web (SMR)
- **Alumno:** Saba Burjanadze
- **Fecha:** 30/09/2026

---

## Índice
1. [Preparación del sistema](#apartado-1-preparación-del-sistema)
2. [Instalación de Apache](#apartado-2-instalación-de-apache)
3. [Comprobación del funcionamiento](#apartado-3-comprobación-del-funcionamiento)
4. [Comandos principales de administración](#apartado-4-comandos-principales-de-administración)
5. [Ficheros y directorios importantes](#apartado-5-ficheros-y-directorios-importantes)
6. [Modificaciones típicas del servicio](#apartado-6-modificaciones-típicas-del-servicio)
7. [Módulos de Apache](#apartado-7-módulos-de-apache)
8. [Varios sitios en un mismo servidor (Virtual Hosts)](#apartado-8-varios-sitios-en-un-mismo-servidor-virtual-hosts)
9. [Permisos y propietarios](#apartado-9-permisos-y-propietarios)
10. [Mejoras de seguridad](#apartado-10-mejoras-de-seguridad)
11. [Comprobación del estado y diagnóstico de errores](#apartado-11-comprobación-del-estado-y-diagnóstico-de-errores)
12. [Conclusiones](#conclusiones)

---

## Apartado 1. Preparación del sistema
Actualizamos la lista de paquetes del sistema operativo y actualizamos los existentes antes de instalar ningún servicio.

```bash
sudo apt update
sudo apt upgrade -y
lsb_release -a
```
![lsb_release -a imagen para poner](imagenes/imagen 1)

Con `apt update` refrescamos los repositorios y con `upgrade` actualizamos el sistema. `lsb_release -a` nos muestra la versión exacta de Ubuntu Server 24.04 LTS que estamos utilizando.


## Apartado 2. Instalación de Apache
Procedemos a instalar el servidor web Apache2 en nuestra máquina virtual.

```bash
sudo apt install apache2 -y
apache2 -v
```
![Instalación de Apache](file:///home/mati/Imatges/Captura%20de%202026-09-30%2009-41-51.png)


Durante la instalación de Apache (`apache2`), apt instala automáticamente paquetes necesarios como `apache2-bin`, `apache2-data`, `apache2-utils`, bibliotecas libapr, y módulos de manejo de tipos MIME o registros que permiten que el servidor funcione de manera modular.

## Apartado 3. Comprobación del funcionamiento
Verificamos que el servicio está activo, los puertos abiertos y probamos el acceso local.

### 3.1. Estado del servicio y puertos
```bash
sudo systemctl status apache2
sudo ss -tulpn | grep apache2
```
### 3.2. Prueba local y cortafuegos
```bash
curl -I http://localhost
sudo ufw status
sudo ufw allow 'Apache'
```
![estado systemctl status apache2 y de la página "Apache2 Ubuntu Default Page" cargando en el navegador](file:///home/mati/Imatges/Captura%20de%202026-09-30%2009-41-51.png)

**¿Qué diferencia hay entre los perfiles Apache, Apache Full y Apache Secure?**

(Respuesta de reflexión):

-Apache: Abre únicamente el puerto 80 (tráfico HTTP sin cifrar).

-Apache Full: Abre tanto el puerto 80 (HTTP) como el puerto 443 (HTTPS).

-Apache Secure: Abre únicamente el puerto 443 (tráfico HTTPS cifrado).

