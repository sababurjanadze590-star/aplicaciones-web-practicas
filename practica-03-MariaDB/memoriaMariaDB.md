# Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04 (Completando el LAMP)

- **Módulo:** Aplicaciones Web (SMR)
- **Alumno:** Saba Burjanadze
- **Fecha:** 21/10/2026

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
Dejamos activo únicamente el sitio por defecto de Apache y actualizamos los paquetes del sistema operativo:

```bash
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
sudo apt update
sudo apt upgrade -y
systemctl is-active apache2
