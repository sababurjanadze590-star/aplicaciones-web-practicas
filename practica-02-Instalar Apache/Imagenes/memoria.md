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
![lsb_release -a imagen para poner](file:///home/mati/Imatges/Captura%20de%202026-09-30%2009-41-51.png)

Con `apt update` refrescamos los repositorios y con `upgrade` actualizamos el sistema. `lsb_release -a` nos muestra la versión exacta de Ubuntu Server 24.04 LTS que estamos utilizando.


## Apartado 1. Instalación de Apache
Procedemos a instalar el servidor web Apache2 en nuestra máquina virtual.

```bash
sudo apt install apache2 -y
apache2 -v
```

