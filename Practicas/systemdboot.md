# Debian 13 Trixie

![Trixie](./trixie.webp)
## Descripción

Los desarrolladores de Debian han propuesto el uso de systemd-boot para instalaciones UEFI de Debian Trixie, que se lanzará en 2025. Opción disponible, de momento, en instalaciones debian 13 en modo experto. El objetivo es agregar soporte de arranque seguro firmado a Debian para intentar resolver el problema relacionado con UEFI y Secure Boot con sistemas Debian. Proponen utilizar un gestor de arranque llamado “systemd-boot” para mejorar el proceso de arranque de Debian en sistemas UEFI.

## Instalación en una máquina virtual, debian 13 (Trixie) con systemd-boot, y familiarízate con este nuevo gestor de arranque.


En esta sección del artículo, veremos cómo instalar Debian 13 (Trixie) con el gestor de arranque systemd-boot en un sistema UEFI.

Comenzamos descargando la imagen ISO de Debian 13 Trixie desde el sitio oficial del proyecto [Debian](https://www.debian.org/devel/debian-installer/). Aquí seleccionaremos la opción que mejor se ajuste a nuestras necesidades, en mi caso la versión netinst.

Para esta instalación, usaremos QEMU/KVM como plataforma de virtualización para crear la máquina virtual. Es crucial configurar correctamente UEFI antes de iniciar la instalación, ya que Debian 13 utiliza este modo de arranque junto con systemd-boot.
Antes de ponernos en la instalación tendremos que elegir el siguiente
![Uefi](1.png)

![Elegir expert modo](2.png)

![alt text](3.png)
![alt text](image-17.png)

![alt text](4.png)

![alt text](5.png)

![alt text](5-1.png) 
![alt text](6.png) 
![alt text](7.png) 
![alt text](8.png) 
![alt text](9.png) 
![alt text](10.png) 
![alt text](11.png) 
![alt text](12.png) 
![alt text](13.png) 
![alt text](14.png) 
![alt text](15.png) 
![alt text](16.png) 
![alt text](17.png) 
![alt text](18.png) 
![alt text](19.png) 
![alt text](20.png) 
![alt text](21.png) 
![alt text](22.png) 
![alt text](23.png) 
![alt text](24.png)
![alt text](release.png) 
![alt text](tree.png)