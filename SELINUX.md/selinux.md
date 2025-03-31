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

```bash
[rocky@selinux-rocky ~]$ sudo dnf update -y
[rocky@selinux-rocky ~]$ sudo dnf install samba samba-common samba-client nfs-utils -y
```
Este proceso va a tardar un poco, ya que son 139 paquetes, t luego la instalación de **samba**.

Por lo que veremos esta palabra despues de cada proceso:

```bash 
Complete!
```
Ahora lo que haremos instalar sshfs, pero para ello lo primero que haré sera tener a mano el repositorio **EPEL**, para ello haré lo siguiente:

```bash
[rocky@selinux-rocky ~]$ sudo dnf install -y epel-release
```

```bash

```
```bash

```