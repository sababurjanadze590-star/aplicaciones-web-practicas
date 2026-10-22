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
