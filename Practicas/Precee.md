# Instalación basada en fichero de conifiguración preseed


## Introducción 

Creación de un sistema automatizado de instalación. Se deberá configurar el sistema para que se responda automáticamente a todos los item en la instalación. Las diferentes contraseñas deberán codificarse para que no aparezcan en texto plano. Se trabajará con un esquema lvm creando volúmenes lógicos /, home y var.

1. Instalación basada en fichero de configuración preseed.
- Instalación automatizada basada en medio de almacenamiento extraíble.
- Instalación automatizada con carga de preseed desde red.
2. Instalación basada en preseed/PXE/TFTP.

## Pasos

Para realizar esta práctica, el primer paso que debemos hacer será descargarnos la imagen ISO con la que vamos a trabajar. La que yo he utilizado la podemos encontrar en  [mamela](https://www.debian.org/download)

![alt text](/img/p-1.png)

Una vez que nos hemos descargado la imagen iso, lo montamos en un directorio para poder realizar los cambios convenientes y lo haremos de la siguiente manera:

```
madandy@toyota-hilux:~$ sudo mkdir -p /mnt/iso

madandy@toyota-hilux:/mnt/iso$ sudo mount -o loop debian-12.8.0-amd64-netinst.iso /mnt/iso/
mount: /mnt/iso: ATENCIÓN: origen protegido contra escritura; se monta como solo lectura.
```
Para montar la iso desatendida, no vamos a necesitar todos los ficheros, sino que vamos a copiar los necesarios y crearemos un enlace simbólico que apuntará al resto.

```
madandy@toyota-hilux:~$ mkdir iso-preseed && cd iso-preseed      
madandy@toyota-hilux:~/iso-preseed$ cp -pr /mnt/iso/install.amd install.amd
madandy@toyota-hilux:~/iso-preseed$ cp -pr /mnt/iso/dists dists
madandy@toyota-hilux:~/iso-preseed$ cp -pr /mnt/iso/pool pool
madandy@toyota-hilux:~/iso-preseed$ cp -pr /mnt/iso/.disk .disk
madandy@toyota-hilux:~/iso-preseed$ cp -pr /mnt/iso/isolinux isolinux
madandy@toyota-hilux:~/iso-preseed$ ln -s . debian
```

Dentro de este directorio, creamos el directorio donde vamos a alojar nuesstro archivo preseed.cfg. En mi caso, tiene la siguiente estructura:
Lo primero que he hecho ha sido crear un direectorio llamado *calamardo*, y dentro hacer el fichero:

```
madandy@toyota-hilux:~/iso-preseed/calamardo$ ls
preseed.cfg
```

```
##############################################################################
#   Instalación automatizada de Debian con particiones LVM usando todo el disco
##############################################################################

# Configuración de idioma y localización
d-i debian-installer/locale string es_ES
d-i debian-installer/language string spanish
d-i debian-installer/country string Spain
d-i debian-installer/locale string es_ES.UTF-8
d-i localechooser/supported-locales es_ES.UTF-8

# Configuración del teclado
d-i keyboard-configuration/toggle select No toggling
d-i keymap select es
d-i console-setup/ask_detect boolean true
d-i keyboard-configuration/modelcode string pc105
d-i keyboard-configuration/layoutcode string es
d-i keyboard-configuration/variantcode string qwerty

# Configuración de red
d-i netcfg/choose_interface select auto

# Creación del usuario y superusuario
d-i passwd/root-password-crypted password $6$kzdzYBwYdWnnkRNc$8CFoTaa6NAfeTwM4G1hBn9Tg2gIKQC1PEBUlTtrHi59zsLmSMNZYRUCeltCki5WicMGZnDDfSshsd5BLrMaTL/

d-i passwd/root-password-again-crypted password $6$kzdzYBwYdWnnkRNc$8CFoTaa6NAfeTwM4G1hBn9Tg2gIKQC1PEBUlTtrHi59zsLmSMNZYRUCeltCki5WicMGZnDDfSshsd5BLrMaTL/


# Creación del usuario normal
d-i passwd/user-fullname string usuario
d-i passwd/username string usuario
d-i passwd/user-password-crypted password $6$kzdzYBwYdWnnkRNc$8CFoTaa6NAfeTwM4G1hBn9Tg2gIKQC1PEBUlTtrHi59zsLmSMNZYRUCeltCki5WicMGZnDDfSshsd5BLrMaTL/


# Configuración del nombre de host y dominio
d-i netcfg/get_hostname string debian
d-i netcfg/get_domain string
d-i hw-detect/load_firmware boolean false

# Configuración de zona horaria
d-i clock-setup/utc boolean true
d-i time/zone string ES/Madrid
d-i clock-setup/ntp boolean true

# Configuración de los repositorios Debian
d-i mirror/country string manual
d-i mirror/http/hostname string ftp.es.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string

# Desactivar la participación en la encuesta
popularity-contest popularity-contest/participate boolean false

# Configuración de particiones

# Usar todo el espacio disponible en LVM sin preguntar al usuario
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true  # Eliminar LVM si ya está presente
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true

# Configuración de particionamiento
d-i partman-auto/disk string /dev/vda
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-auto/method string lvm
d-i partman-lvm/confirm boolean true
d-i partman-auto/choose_recipe select mypartitioning
d-i partman-auto-lvm/new_vg_name string vg00
d-i partman-auto-lvm/guided_size string max
d-i partman-lvm/confirm_nooverwrite boolean true

# Esquema de particionamiento personalizado
d-i partman-auto/expert_recipe string \
      mypartitioning :: \
              512 1 512 xfs \
                      $primary{ } $bootable{ } \
                      method{ format } format{ } \
                      use_filesystem{ } filesystem{ xfs } \
                      mountpoint{ /boot } \
              . \
              1024 1 1024 linux-swap \
                      $defaultignore{ } \
                      $lvmok{ } \
                      lv_name{ swap } \
                      in_vg { vg00 } \
                      method{ swap } format{ } \
              . \
              3072 1 3072 xfs \
                      $defaultignore{ } \
                      $lvmok{ } \
                      lv_name{ root } \
                      in_vg { vg00 } \
                      method{ format } format{ } \
                      use_filesystem{ } filesystem{ xfs } \
                      mountpoint{ / } \
              . \
              6144 1 6144 xfs \
                      $defaultignore{ } \
                      $lvmok{ } \
                      lv_name{ var } \
                      in_vg { vg00 } \
                      method{ format } format{ } \
                      use_filesystem{ } filesystem{ xfs } \
                      mountpoint{ /var } \
              . \
              8192 1 1000000000 xfs \
                      $defaultignore{ } \
                      $lvmok{ } \
                      lv_name{ home } \
                      in_vg { vg00 } \
                      method{ format } format{ } \
                      use_filesystem{ } filesystem{ xfs } \
                      mountpoint{ /home } \
              .

# Confirmación de escritura de nueva etiqueta de partición
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true



#Análisis de medios adicionales
d-i apt-setup/cdrom/set-first boolean false
d-i apt-setup/cdrom/set-next boolean false
d-i apt-setup/cdrom/set-failed boolean false


# Instalación de GRUB
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean false
d-i grub-installer/bootdev string default

# Selección de paquetes
tasksel tasksel/first multiselect standard, ssh-server

# Finalización de la instalación
d-i finish-install/reboot_in_progress note

```
Cabe señalar que habrá que codificar las contraseñas, por lo que tendremos que meter por consola lo siguiente:

```
madandy@toyota-hilux:~$ mkpasswd --method=SHA-512 usuario
$6$kzdzYBwYdWnnkRNc$8CFoTaa6NAfeTwM4G1hBn9Tg2gIKQC1PEBUlTtrHi59zsLmSMNZYRUCeltCki5WicMGZnDDfSshsd5BLrMaTL/

```
El siguiente paso que realizaremos será el de editar el archivo txt.cfg ubicado dentro del directorio /isolinux de la siguiente forma:

```
default install
label install
        menu label ^Install
        kernel /install.amd/vmlinuz
        append vga=788 initrd=/install.amd/initrd.gz -- quiet
label unattended-gnome
 menu label ^Instalacion desatendida Debian 11
 kernel /install.amd/gtk/vmlinuz
 append vga=788 initrd=/install.amd/gtk/initrd.gz preseed/file=/cdrom/respuestas/preseed.cfg locale=es_ES console-setup/ask_detect=false keyboard-configuration/xkb-keymap=es --

```
Al realizar cambios en el directorio /isolinux, generaremos la suma de verificación que irá alojada en el CD:


```
madandy@toyota-hilux:~/iso-preseed$ md5sum `find ! -name "md5sum.txt" ! -path "./isolinux/*" -follow -type f` > md5sum.txt
find: ‘./.debian’: Demasiados niveles de enlaces simbólicos
find: Se ha detectado un bucle en el sistema de ficheros; ‘./debian’ es parte del mismo bucle de sistema de ficheros que ‘.’.
madandy@toyota-hilux:~/iso-preseed$ 

```

Ahora nos iremos fuera del directorio y hacemos lo siguiente:

```
madandy@toyota-hilux:~$ sudo genisoimage -o cd-preseed.iso -l -r -J -no-emul-boot -boot-load-size 4 -boot-info-table -b isolinux/isolinux.bin -c isolinux/boot.cat iso-preseed
I: -input-charset not specified, using utf-8 (detected in locale settings)
Size of boot image is 4 sectors -> No emulation
  1.56% done, estimate finish Thu Nov 21 16:48:05 2024
  3.12% done, estimate finish Thu Nov 21 16:48:05 2024
  4.68% done, estimate finish Thu Nov 21 16:48:05 2024
  6.24% done, estimate finish Thu Nov 21 16:48:05 2024
  7.80% done, estimate finish Thu Nov 21 16:48:05 2024
  9.36% done, estimate finish Thu Nov 21 16:48:05 2024
 10.92% done, estimate finish Thu Nov 21 16:48:05 2024
 12.48% done, estimate finish Thu Nov 21 16:48:05 2024
 14.04% done, estimate finish Thu Nov 21 16:48:05 2024
 15.60% done, estimate finish Thu Nov 21 16:48:05 2024
 17.16% done, estimate finish Thu Nov 21 16:48:05 2024
 18.72% done, estimate finish Thu Nov 21 16:48:05 2024
 20.28% done, estimate finish Thu Nov 21 16:48:05 2024
 21.84% done, estimate finish Thu Nov 21 16:48:05 2024
 23.40% done, estimate finish Thu Nov 21 16:48:05 2024
 24.96% done, estimate finish Thu Nov 21 16:48:05 2024
 26.52% done, estimate finish Thu Nov 21 16:48:05 2024
 28.08% done, estimate finish Thu Nov 21 16:48:05 2024
 29.64% done, estimate finish Thu Nov 21 16:48:08 2024
 31.20% done, estimate finish Thu Nov 21 16:48:08 2024
 32.76% done, estimate finish Thu Nov 21 16:48:08 2024
 34.33% done, estimate finish Thu Nov 21 16:48:07 2024
 35.89% done, estimate finish Thu Nov 21 16:48:07 2024
 37.44% done, estimate finish Thu Nov 21 16:48:07 2024
 39.01% done, estimate finish Thu Nov 21 16:48:07 2024
 40.56% done, estimate finish Thu Nov 21 16:48:07 2024
 42.12% done, estimate finish Thu Nov 21 16:48:07 2024
 43.68% done, estimate finish Thu Nov 21 16:48:07 2024
 45.24% done, estimate finish Thu Nov 21 16:48:07 2024
 46.81% done, estimate finish Thu Nov 21 16:48:07 2024
 48.36% done, estimate finish Thu Nov 21 16:48:07 2024
 49.92% done, estimate finish Thu Nov 21 16:48:07 2024
 51.48% done, estimate finish Thu Nov 21 16:48:06 2024
 53.04% done, estimate finish Thu Nov 21 16:48:06 2024
 54.60% done, estimate finish Thu Nov 21 16:48:06 2024
 56.17% done, estimate finish Thu Nov 21 16:48:06 2024
 57.72% done, estimate finish Thu Nov 21 16:48:06 2024
 59.29% done, estimate finish Thu Nov 21 16:48:06 2024
 60.84% done, estimate finish Thu Nov 21 16:48:06 2024
 62.41% done, estimate finish Thu Nov 21 16:48:06 2024
 63.96% done, estimate finish Thu Nov 21 16:48:06 2024
 65.53% done, estimate finish Thu Nov 21 16:48:06 2024
 67.08% done, estimate finish Thu Nov 21 16:48:06 2024
 68.64% done, estimate finish Thu Nov 21 16:48:06 2024
 70.21% done, estimate finish Thu Nov 21 16:48:06 2024
 71.77% done, estimate finish Thu Nov 21 16:48:06 2024
 73.32% done, estimate finish Thu Nov 21 16:48:06 2024
 74.88% done, estimate finish Thu Nov 21 16:48:06 2024
 76.45% done, estimate finish Thu Nov 21 16:48:06 2024
 78.01% done, estimate finish Thu Nov 21 16:48:06 2024
 79.57% done, estimate finish Thu Nov 21 16:48:06 2024
 81.12% done, estimate finish Thu Nov 21 16:48:06 2024
 82.69% done, estimate finish Thu Nov 21 16:48:06 2024
 84.25% done, estimate finish Thu Nov 21 16:48:06 2024
 85.81% done, estimate finish Thu Nov 21 16:48:06 2024
 87.37% done, estimate finish Thu Nov 21 16:48:06 2024
 88.93% done, estimate finish Thu Nov 21 16:48:06 2024
 90.49% done, estimate finish Thu Nov 21 16:48:06 2024
 92.05% done, estimate finish Thu Nov 21 16:48:06 2024
 93.61% done, estimate finish Thu Nov 21 16:48:06 2024
 95.17% done, estimate finish Thu Nov 21 16:48:06 2024
 96.72% done, estimate finish Thu Nov 21 16:48:06 2024
 98.28% done, estimate finish Thu Nov 21 16:48:06 2024
 99.85% done, estimate finish Thu Nov 21 16:48:06 2024
Total translation table size: 2048
Total rockridge attributes bytes: 199882
Total directory bytes: 1085440
Path table size(bytes): 8618
Max brk space used 1d2000
320499 extents written (625 MB)

```
Y ahora lo que tenemos que hacer es coger y pegarlo en mi caso a mi carpeta donde tengo las iso:

![alt text](/img/image2.png)

Y como podemos comprobar una vez arrancado nos sale lo siguiente:

![alt text](/img/image3.png)

Pero nos saldra este error, ya que hemos cometido el siguiente fallo:

![alt text](/img/image4.png)

Por lo que haremos lo siguiente:

```
madandy@toyota-hilux:~/iso-preseed$ mkdir -p /home/madandy/iso-preseed/respuestas
madandy@toyota-hilux:~/iso-preseed$ ls
calamardo  debian  dists  install.amd  isolinux  md5sum.txt  pool  respuestas
madandy@toyota-hilux:~/iso-preseed$ cp /home/madandy/iso-preseed/calamardo/preseed.cfg /home/madandy/iso-preseed/respuestas/
```

Y una vez hecho esto ya podemos arranacarlo y lo tendriamos automatizado y hacemos un lsblk, veriamos lo siguiente;¡:

![alt text](/img/image-9.png)


# Instalacion de forma automatizada con carga de preseed desde red

Para esta ocasión vamos a crear una máquina irtual con vagrabnt en la que configuraremos un servidor *ngninx* y añadiremos nuestra iso (la cual hicimos con anterioridad) a lo que sera nuestro servidor.

```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64" 
  config.vm.hostname = "servidorDHCP"
  
  # Configuración de red privada
  config.vm.network :private_network, 
    libvirt__network_name: "red-server",   # Nombre de la red en libvirt
    libvirt__dhcp_enabled: false,         # Deshabilitar DHCP
    ip: "172.22.2.5",                     # IP asignada estáticamente
    libvirt__forward_mode: "none"         # Sin reenvío de tráfico
  
  # Configuraciones adicionales pueden ir aquí
end

```

Ahora configuraremos en esta máquina un servidor *ngnix* donde meteremos el fichero de configuración que llamamos *preseed.cfg*, para ello primero que haremos sera pasarnolos por scp:

![alt text](image-14.png)

Ahora nos dirigiremos a la siguiente dirección:

```
http://192.168.121.3/preseed.cfg
```
Y hacemos un amáquina virtual en la que vamos a aplicar esta configuración a través del método de carga desde la red.

Cuando elegimos la iso virgen, lo que tenemos que ir es a la tercera elección, que e
![alt text](image-1.png)

y como podemos ver, tenemos lo siguiente por pantalla tiotalmente automatizado:

![alt text](image-2.png)

# Instalación basada en preseed/PXE/TFTP

