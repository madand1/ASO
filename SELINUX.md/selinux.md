# Enunciado

En un servidor basado en Rocky Linux con SELinux activado en modo enforcing, asegúrate que los servicios sshfs, samba y nfs funcionan correctamente y no hay problemas de acceso con una configuración estricta y segura de SELinux. Habilita inicio de sesión de root en el acceso remoto al servidor Rocky, teniendo como nuevo puerto de acceso el no habitual en ssh. Realiza las pruebas de acceso correspondientes.


# Pasos

```bash
[rocky@selinux-rocky ~]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```
Como podemos ver, **SELinux** está activado y en mo do **enforcing**. Por lo que ahora actualizare el sistema e instalaré los paquetes necesarios.

```bash
[rocky@selinux-rocky ~]$ sudo dnf update -y
[rocky@selinux-rocky ~]$ sudo dnf install samba samba-common samba-client nfs-utils -y
```

Este proceso va a tardar un poco, ya que son 139 paquetes, t luego la instalación de **samba**.

Por lo que veremos esta palabra despues de cada proceso:

```bash 
Complete!
```
Ahora lo que haremos instalar `sshfs`, pero para ello lo primero que haré sera tener a mano el repositorio **EPEL**, para ello haré lo siguiente:

```bash
[rocky@selinux-rocky ~]$ sudo dnf install -y epel-release
```
Ahora que ya tenemos habilitado lo que es el repositsrio, instalaré `SSHFS`:

```bash
[rocky@selinux-rocky ~]$ sudo dnf install -y fuse fuse-sshfs

```

Antes de nada, yo en mi caso, al estar en una instancia, he puesto el siguiente volumen `m1.HD40medium`, ya que si ponemos alguno inferior a `m1.medium` esté habra que dstruirlo, ya que se quedará congelado.


# SSHFS

## Configuración del servidor SSH


Primero, necesitamos asegurarnos de que nuestro servidor SSH esté configurado correctamente para permitir conexiones desde el cliente. En nuestro caso, decidimos cambiar el puerto predeterminado de SSH (**22**) al puerto **2222** por razones de seguridad.  

Para ello, editamos el archivo de configuración de SSH en `/etc/ssh/sshd_config`: 

```bash
Port 2222
PermitRootLogin yes
PubkeyAuthentication yes
```

Donde:

- SSH escuchará en el puerto 2222.

- Permitiremos el inicio de sesión como root.

- Habilitaremos la autenticación mediante claves públicas.
AHora despues de haber editado el fichero, lo que hare será reicniar el servicio SSh, para hacer efectivo estos cambios:

Despues reinico el sistema y vemos el status:

```bash

[root@selinux-rocky rocky]# sudo systemctl status sshd.service
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-04-02 10:30:29 UTC; 39s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 57523 (sshd)
      Tasks: 1 (limit: 10890)
     Memory: 1.4M
        CPU: 20ms
     CGroup: /system.slice/sshd.service
             └─57523 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Apr 02 10:30:29 selinux-rocky.novalocal systemd[1]: Starting OpenSSH server daemon...
Apr 02 10:30:29 selinux-rocky.novalocal sshd[57523]: Server listening on 0.0.0.0 port 2222.
Apr 02 10:30:29 selinux-rocky.novalocal sshd[57523]: Server listening on :: port 2222.
Apr 02 10:30:29 selinux-rocky.novalocal systemd[1]: Started OpenSSH server daemon.
[root@selinux-rocky rocky]# 
```

Y habilito el firewall para que pueda permitir las conexiones al puerto 2222:

```bash
[root@selinux-rocky rocky]# sudo firewall-cmd --permanent --add-port=2222/tcp
Warning: ALREADY_ENABLED: 2222:tcp
success
[root@selinux-rocky rocky]# sudo firewall-cmd --reload
success
[root@selinux-rocky rocky]# 
```

A parte, vamos a usar `semanage` para agregar el puerto 2222 al contexto de seguridad de SSH:

```bash
[root@selinux-rocky rocky]# sudo semanage port -a -t ssh_port_t -p tcp 2222
Port tcp/2222 already defined, modifying instead
[root@selinux-rocky rocky]# semanage port -l | grep ssh
ssh_port_t                     tcp      2222, 22
[root@selinux-rocky rocky]# 
```

## Configuración de SELinux para SSHFS

En nuestro servidor, SELinux estaba habilitado, por lo que necesitamos ajustar algunas políticas para permitir el uso de SSHFS. Utilizamos los siguientes comandos para habilitar las opciones necesarias:

```bash
[root@selinux-rocky rocky]# setsebool -P use_fusefs_home_dirs on
[root@selinux-rocky rocky]# setsebool -P virt_use_fusefs on
```

Y vemso que los cambios se hicieron efectivos:

```bash
[root@selinux-rocky rocky]# getsebool -a | grep fuse
ftpd_use_fusefs --> off
glance_use_fusefs --> off
httpd_use_fusefs --> off
logrotate_use_fusefs --> off
mailman_use_fusefs --> off
rpcd_use_fusefs --> off
samba_share_fusefs --> off
sanlock_use_fusefs --> off
use_fusefs_home_dirs --> on
virt_sandbox_use_fusefs --> off
virt_use_fusefs --> on
```

Y podemos ver como `use_fusefs_home_dirs` y `virt_use_fusefs` están habilitados.

## Configuración de claves SSH

Para facilitar la conexión sin necesidad de introducir contraseñas cada vez, configuramos la autenticación mediante claves SSH. Añadimos la clave pública del cliente al archivo authorized_keys del usuario root en el servidor:

```bash
[root@selinux-rocky rocky]# cat .ssh/authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC00lFf6Jyj2o41CRmIrIvguHHYKfaByakRVvyoVjfTyZDxDA5PIsAW0JHAKW9V2+MMREjAeY74LiDMJlyc1XHSkEFmzrKXLQ9d8M81WV9vYh2aOB3yHWZq3/CwxpHgcgatlTRKaP310y4mfsbkdbuDAcgE7jwO0k6KlibpUSIet2bUXZ3zagrMNhmDrHErYh+ARW37cm2+OqQdv1l+GeOBlFCTC6yAsOdpbJ5VQ0fWwvR6bl5DUpGBr0RDWE7HQLHGt3WE2nNKCii+myzGc17kU1ERfi7C2aI80VWdQMPqwLUpam/TdNahvw1tZ3lkrjSHVS330Ll7T+uT8WXU4dHEgtlgSNEHszxrQoQcAv+KYzkzi7oMi42xEpdW378WeKcEHgcaKK+gIqR1UHBI8cfOYiduMbnqcySaSEs3YIAk3MaBtSBn4GkkMFWaYNzUK2Ic5MQIOZ/ZkQyzDYLfpVcf6i42bHTLkJvR1mNSDYiL6M3xAp76ws/3ETx0OcdiAf0= madandy@toyota-hilux
#Cliente rocky
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCsWdFfAmRszJxKMEn/6vF9iVq6Pzz2E/F0RLoQktx+2TDxulAmVm87bwn6cCI8GPHhczqAa+9/XO5nDBgkpxYg26Q1VKs2R05gvTkrQw3+MQY8BmjNCeT26r9TmwIpKITn/I93wqq/+8Q6cY3cQCwmFJ5nqJ4QTKzi2mK/92ss8Cj99z3RsPFEnJgMIdNZIRl9jJmJtaCyQNXQb59yiWdGOfkgotvq9KXNSAFI2pVeiR/viUpndLfTRS+Xc0e7QXL+9z/j1I3kYIZZ1joZqZM4ZawnxiGI2Ez+v/rj1X5wojjNnjYgeBH//razIpLn+oUvv7FbEKUZKa9b+w5XiHl4jDJhkw9DD+zsTjgbafGnxPUqivldUcPbdXyd+/N/r0VOj1LAxGKM+WtcwBmjB8m5P29BKAXWor7Z9kOLfBAayMmtLAwFizP1nyM/oEmcAOg1czmkhh/ak3nPTBtU2GRp+LrN3EHVqYvIpuSyQGMMeycrQibWAiE688ZIRndAew6sE+Q+/sYeZY9C+JJDhbZdexwCiydKiJd3ipzp2xX8BOBLod3k7xcc/dcjth+oD2n5PL4HQz2HOcMdMSENOmbSh04YJ+jdIjRZvSZvYh/c9ASXdo74Y0e9rLBolSGJ6H/aLoBdnwmOUoBY2TWmLW0w1r5EmR6UCIagySrtAUx1mQ== rocky@cliente.novalocal

```

Vemos como en este archivo contiene las claves públicas de los clientes que pueden conectarse al servidor. En nuestro caso, añadimos la clave del usuario rocky desde el cliente.

## Creación del directoroio compartido

En el servidor, creamos un directorio que queremos compartir con el cliente:

```bash
[rocky@selinux-rocky ~]$ sudo mkdir compartir
[rocky@selinux-rocky ~]$ sudo chown rocky:rocky compartir/
```
Dentro de este directorio, colocamos un archivo de prueba para verificar que todo funciona correctamente:

```bash
[rocky@selinux-rocky ~]$ sudo nano compartir/koda.txt
[rocky@selinux-rocky ~]$ sudo cat compartir/koda.txt
No es que lo quiera, es que lo necesito.
```

## Montaje del directorio remoto en el cliente

Ahora, desde el cliente, procedemos a montar el directorio remoto utilizando SSHFS. Primero, creamos un directorio donde montaremos el directorio remoto:

```bash
[rocky@cliente ~]$ mkdir montaje
```

Luego, utilizamos el comando sshfs para montar el directorio remoto:

```bash
[rocky@cliente-rocky ~]$ sshfs -p 2222 root@10.0.0.87:/home/rocky/compartir montaje/
```
Y para comprobar de que se ha montado lo que hago es coger y meter el sigueinte comando:

```bash
[rocky@cliente-rocky montaje]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 888M     0  888M   0% /dev/shm
tmpfs                                 355M  536K  355M   1% /run
/dev/vda4                              39G  1.5G   38G   4% /
/dev/vda3                             936M  429M  508M  46% /boot
/dev/vda2                             100M  7.0M   93M   8% /boot/efi
tmpfs                                 178M     0  178M   0% /run/user/1000
root@10.0.0.87:/home/rocky/compartir   39G  1.7G   38G   5% /home/rocky/montaje
```
Y podemos ver el mensaje que esta dentro del directorio:

```bash
[rocky@cliente-rocky montaje]$ cat koda.txt
No es que lo quiera, es que lo necesito.
```

# Samba

Samba es un servicio crítico para compartir archivos en redes mixtas (Linux/Windows). Vamos a configurarlo paso a paso.

Antes de realizar cambios, es una buena práctica hacer una copia de seguridad del archivo de configuración original:

```bash
[rocky@selinux-rocky ~]$ sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

Vamos a crear un directorio que será compartido a través de Samba. En este ejemplo, usaremos /mnt/samba/share:

```bash
[rocky@selinux-rocky ~]$ sudo mkdir -p /mnt/samba/share
```

Aseguramos que los permisos y la propiedad del directorio sean correctos:

```bash
[rocky@selinux-rocky ~]$ sudo chmod -R 0755 /mnt/samba/share/
[rocky@selinux-rocky ~]$ sudo chown -R rocky:rocky /mnt/samba/share/
```
SELinux utiliza contextos de seguridad para controlar el acceso a los recursos. Para que Samba pueda acceder al directorio compartido, debemos asignar el contexto correcto:

```bash
[rocky@selinux-rocky ~]$ sudo chcon -t samba_share_t /mnt/samba/share/
[rocky@selinux-rocky ~]$ 
```

Aquí, chcon cambia el contexto de seguridad de SELinux para que el directorio sea accesible por Samba.

Editamos el archivo de configuración de Samba (/etc/samba/smb.conf) para definir nuestro recurso compartido. Añadimos la siguiente sección al final del archivo:

```bash
[Anonymous]
path = /mnt/samba/share
browsable = yes
writable = yes
guest ok = yes
read only = no
valid users = rocky
```

Donde:

- path: Especifica la ruta del directorio compartido.

- browsable: Permite que el recurso sea visible en la red.

- writable: Permite la escritura en el recurso.

- guest ok: Permite el acceso a usuarios invitados.

- read only: Define si el recurso es de solo lectura.

- valid users: Especifica los usuarios permitidos para acceder al recurso.

Para realizar la comprobación de que la configuración es correcta ejecutamos el siguiente comando:

```bash
[rocky@selinux-rocky ~]$ testparm
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
	printcap name = cups
	security = USER
	workgroup = SAMBA
	idmap config * : backend = tdb
	cups options = raw


[homes]
	browseable = No
	comment = Home Directories
	inherit acls = Yes
	read only = No
	valid users = %S %D%w%S


[printers]
	browseable = No
	comment = All Printers
	create mask = 0600
	path = /var/tmp
	printable = Yes


[print$]
	comment = Printer Drivers
	create mask = 0664
	directory mask = 0775
	force group = @printadmin
	path = /var/lib/samba/drivers
	write list = @printadmin root


[Anonymous]
	guest ok = Yes
	path = /mnt/samba/share
	read only = No
	valid users = rocky

```

Este comando valida la sintaxis del archivo smb.conf y muestra un resumen de la configuración.

Permitimos el tráfico de Samba a través del firewall:

```bash
[rocky@selinux-rocky ~]$ sudo firewall-cmd --add-service=samba --zone=public --permanent
success
[rocky@selinux-rocky ~]$ sudo firewall-cmd --reload
success
[rocky@selinux-rocky ~]$ 
```

Iniciamos y habilitamos los servicios de Samba (smb y nmb):

```bash
[rocky@selinux-rocky ~]$ sudo systemctl start smb
[rocky@selinux-rocky ~]$ sudo systemctl enable smb
Created symlink /etc/systemd/system/multi-user.target.wants/smb.service → /usr/lib/systemd/system/smb.service.
[rocky@selinux-rocky ~]$ sudo systemctl start nmb
[rocky@selinux-rocky ~]$ sudo systemctl enable nmb
Created symlink /etc/systemd/system/multi-user.target.wants/nmb.service → /usr/lib/systemd/system/nmb.service.
[rocky@selinux-rocky ~]$ 
```

Añadimos un usuario de Samba y establecemos una contraseña:

```bash
[rocky@selinux-rocky ~]$ sudo smbpasswd -a rocky
New SMB password:
Retype new SMB password:
Added user rocky.
```

Este comando añade al usuario rocky a la base de datos de Samba y solicita una contraseña, yo no  he puesto ninguna.

## Prueba de funcionamiento desde el cliente

Para verificar que todo funciona correctamente, realizamos pruebas de acceso desde un cliente.

Primero debemos instalar el paquete samba-client:

```bash
[rocky@cliente-rocky ~]$ sudo dnf install samba-client -y
```

Listamos los recursos compartidos en el servidor:

```bash
[rocky@cliente-rocky ~]$ smbclient --user=rocky -L //10.0.0.87
Password for [SAMBA\rocky]:

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	Anonymous       Disk      
	IPC$            IPC       IPC Service (Samba 4.20.2)
	rocky           Disk      Home Directories
SMB1 disabled -- no workgroup available
[rocky@cliente-rocky ~]$ 
```

Esto mostrará los recursos compartidos disponibles.

Montamos el recurso compartido en el cliente, como no tengo contrseña el campo password no estara:

```bash
[rocky@cliente-rocky ~]$ sudo mount -t cifs -o username=rocky,rw,vers=2.1,file_mode=0777,dir_mode=0777 //10.0.0.87/Anonymous /home/rocky/montaje/
```

Aquí, username y password son las credenciales del usuario de Samba, y //10.0.0.87/Anonymous es la ruta del recurso compartido.

Por último, verificamos que el recurso se ha montado correctamente:

```bash
[rocky@cliente-rocky ~]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 888M     0  888M   0% /dev/shm
tmpfs                                 355M  540K  355M   1% /run
/dev/vda4                              39G  1.6G   38G   4% /
/dev/vda3                             936M  429M  508M  46% /boot
/dev/vda2                             100M  7.0M   93M   8% /boot/efi
tmpfs                                 178M     0  178M   0% /run/user/1000
root@10.0.0.87:/home/rocky/compartir   39G  1.7G   38G   5% /home/rocky/montaje
//10.0.0.87/Anonymous                  39G  1.7G   38G   5% /home/rocky/montaje
```

Y como vemos el recurso compartido está en la lista de sistemas de archivos montados.

# NFS

## Instalación y Configuración del Servidor NFS


Lo primero que debemos hacer en el servidor Rocky Linux es asegurarnos de que el servicio NFS está habilitado. Para ello, ejecutamos:

```bash
[rocky@selinux-rocky ~]$ sudo systemctl start nfs-server.service
[rocky@selinux-rocky ~]$ sudo systemctl enable nfs-server.service
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[rocky@selinux-rocky ~]$ 
```

Ahora, creamos el directorio que vamos a compartir mediante NFS:

```bash
[rocky@selinux-rocky ~]$ sudo mkdir -p /mnt/nfs/koda
```

Este será el punto de montaje donde guardaremos los archivos que estarán disponibles para los clientes NFS.

A continuación, debemos definir las reglas de exportación en el archivo /etc/exports, que indica qué directorios se comparten y con qué permisos. Lo editamos y agregamos la siguiente línea:

```bash
[rocky@selinux-rocky ~]$ sudo nano /etc/exports
[rocky@selinux-rocky ~]$ sudo cat /etc/exports
/mnt/nfs/koda  10.0.0.0/24(rw,sync,no_all_squash,no_root_squash)
```

Esto significa que estamos compartiendo el directorio /mnt/nfs/koda con la red 10.0.0.0/24, permitiendo lectura y escritura (rw), asegurando que las escrituras se realicen de forma sincronizada (sync), y permitiendo que el usuario root del cliente actúe como root en el servidor (no_root_squash).

Aplicamos la configuración ejecutando:

```bash
[rocky@selinux-rocky ~]$ sudo exportfs -arv
exporting 10.0.0.0/24:/mnt/nfs/koda
```

Y verificamos que el directorio esté efectivamente exportado:

```bash
[rocky@selinux-rocky ~]$ sudo exportfs -s
/mnt/nfs/koda  10.0.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

Si todo está correcto, deberíamos ver una salida que confirma que el directorio /mnt/nfs/koda está compartido con la red 10.0.0.0/24.

## Configuración del Firewall

Habilitamos los servicios requeridos para NFS:

```bash
[rocky@selinux-rocky ~]$ sudo firewall-cmd --permanent --add-service=nfs
success
[rocky@selinux-rocky ~]$ sudo firewall-cmd --permanent --add-service=rpc-bind
success
[rocky@selinux-rocky ~]$ sudo firewall-cmd --permanent --add-service=mountd
success
[rocky@selinux-rocky ~]$ sudo firewall-cmd --reload
success
```

Con esto, hemos asegurado que el tráfico necesario para NFS pueda pasar sin problemas.

## COnfiguración del cliente

Ahora pasamos al cliente, que también es una máquina Rocky Linux. En primer lugar, instalamos los paquetes necesarios:

```bash
[rocky@cliente-rocky ~]$ sudo dnf install nfs-utils nfs4-acl-tools
```

Verificamos que el servidor está compartiendo correctamente el directorio usando el comando showmount:

```bash
[rocky@cliente-rocky ~]$ showmount -e 10.0.0.87
Export list for 10.0.0.87:
/mnt/nfs/koda 10.0.0.0/24
```
Despues de esto, lo que tenemos que hacer es crear el directorio, y montarlo:

```bash
[rocky@cliente-rocky mnt]$ sudo mkdir -p /mnt/nfs
[rocky@cliente-rocky mnt]$ sudo mount 10.0.0.87:/mnt/nfs/koda /mnt/nfs
```

Si todo está en orden, veremos la exportación del directorio /mnt/nfs/koda para la red 10.0.0.0/24.

Desde el servidor, creamos un archivo en el directorio compartido y verificamos sus permisos:

```bash
[rocky@selinux-rocky ~]$ echo 'Hola desde el servidor comanche' | sudo tee /mnt/nfs/koda/tr3mendoCulo.txt
Hola desde el servidor comanche
[rocky@selinux-rocky ~]$ ls -l /mnt/nfs/koda/tr3mendoCulo.txt
-rw-r--r--. 1 root root 32 Apr  2 12:18 /mnt/nfs/koda/tr3mendoCulo.txt
```

Desde el cliente, comprobamos que el archivo también está disponible en el punto de montaje /mnt/nfs:

```bash
[rocky@cliente-rocky mnt]$ cd
[rocky@cliente-rocky ~]$ ls /mnt/nfs/tr3mendoCulo.txt 
/mnt/nfs/tr3mendoCulo.txt
[rocky@cliente-rocky ~]$ cat /mnt/nfs/tr3mendoCulo.txt 
Hola desde el servidor comanche
```

Y ahopra lo que hacemo ses ver si esta montado (aunque si esta montado):

```bash
[rocky@cliente-rocky ~]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 888M     0  888M   0% /dev/shm
tmpfs                                 355M  536K  355M   1% /run
/dev/vda4                              39G  1.6G   38G   4% /
/dev/vda3                             936M  429M  508M  46% /boot
/dev/vda2                             100M  7.0M   93M   8% /boot/efi
tmpfs                                 178M     0  178M   0% /run/user/1000
root@10.0.0.87:/home/rocky/compartir   39G  1.7G   38G   5% /home/rocky/montaje
//10.0.0.87/Anonymous                  39G  1.7G   38G   5% /home/rocky/montaje
10.0.0.87:/mnt/nfs/koda                39G  1.7G   38G   5% /mnt/nfs
```

Con esto, confirmamos que tanto NFS como Samba como SSHFS están configurados correctamente y permiten el acceso a los archivos compartidos entre servidor y cliente.