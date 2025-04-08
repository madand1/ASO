Pasos:

✅ Paso 1: Instalar dependencias necesarias

root@java:/home/debian# apt update
root@java:/home/debian# sudo apt install -y apache2 php libapache2-mod-php build-essential libgd-dev unzip curl

✅ Paso 2: Crear usuario y grupo para Nagios

root@java:/home/debian# sudo useradd nagios
root@java:/home/debian# sudo usermod -aG nagios www-data

✅ Paso 3: Descargar e instalar Nagios Core

root@java:/home/debian# cd /tmp
root@java:/tmp# curl -LO https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.14.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10.8M  100 10.8M    0     0  2325k      0  0:00:04  0:00:04 --:--:-- 2352k
root@java:/tmp# 

root@java:/tmp# tar -zxvf nagios-4.4.14.tar.gz
root@java:/tmp/nagios-4.4.14# cd nagios-4.4.14

root@java:/tmp# ./configure --with-httpd-conf=/etc/apache2/sites-enabled

Aqui vi un error y es que nos falta una dependecia, por lo que añadiremos este:

sudo apt install -y libssl-dev


Y ahora obtenemos esto:

 Web Interface Options:
 ------------------------
                 HTML URL:  http://localhost/nagios/
                  CGI URL:  http://localhost/nagios/cgi-bin/
 Traceroute (used by WAP):  /usr/sbin/traceroute


Review the options above for accuracy.  If they look okay,
type 'make all' to compile the main program and CGIs.


por loque haremos lo siguiente:

root@java:/tmp/nagios-4.4.14# make all

Esto tardara unos minutos, y a continuación nos saldrá esto al finalizar:

```bash
*** Support Notes *******************************************

If you have questions about configuring or running Nagios,
please make sure that you:

     - Look at the sample config files
     - Read the documentation on the Nagios Library at:
           https://library.nagios.com

before you post a question to one of the mailing lists.
Also make sure to include pertinent information that could
help others help you.  This might include:

     - What version of Nagios you are using
     - What version of the plugins you are using
     - Relevant snippets from your config files
     - Relevant error messages from the Nagios log file

For more information on obtaining support for Nagios, visit:

       https://support.nagios.com

*************************************************************

Enjoy.
```

Y hacemos lo siguiente;

root@java:/tmp/nagios-4.4.14# sudo make install-init
/usr/bin/install -c -m 755 -d -o root -g root /lib/systemd/system
/usr/bin/install -c -m 755 -o root -g root startup/default-service /lib/systemd/system/nagios.service
root@java:/tmp/nagios-4.4.14# sudo make install-commandmode
/usr/bin/install -c -m 775 -o nagios -g nagios -d /usr/local/nagios/var/rw
chmod g+s /usr/local/nagios/var/rw

*** External command directory configured ***

root@java:/tmp/nagios-4.4.14# sudo make install-config
/usr/bin/install -c -m 775 -o nagios -g nagios -d /usr/local/nagios/etc
/usr/bin/install -c -m 775 -o nagios -g nagios -d /usr/local/nagios/etc/objects
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/nagios.cfg /usr/local/nagios/etc/nagios.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/cgi.cfg /usr/local/nagios/etc/cgi.cfg
/usr/bin/install -c -b -m 660 -o nagios -g nagios sample-config/resource.cfg /usr/local/nagios/etc/resource.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/templates.cfg /usr/local/nagios/etc/objects/templates.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/commands.cfg /usr/local/nagios/etc/objects/commands.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/contacts.cfg /usr/local/nagios/etc/objects/contacts.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/timeperiods.cfg /usr/local/nagios/etc/objects/timeperiods.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/localhost.cfg /usr/local/nagios/etc/objects/localhost.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/windows.cfg /usr/local/nagios/etc/objects/windows.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/printer.cfg /usr/local/nagios/etc/objects/printer.cfg
/usr/bin/install -c -b -m 664 -o nagios -g nagios sample-config/template-object/switch.cfg /usr/local/nagios/etc/objects/switch.cfg

*** Config files installed ***

Remember, these are *SAMPLE* config files.  You'll need to read
the documentation for more information on how to actually define
services, hosts, etc. to fit your particular needs.

root@java:/tmp/nagios-4.4.14# sudo make install-webconf
/usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-enabled/nagios.conf
if [ 0 -eq 1 ]; then \
	ln -s /etc/apache2/sites-enabled/nagios.conf /etc/apache2/sites-enabled/nagios.conf; \
fi

*** Nagios/Apache conf file installed ***

root@java:/tmp/nagios-4.4.14# 

✅ Paso 4: Instalar los plugins de Nagios

root@java:/tmp/nagios-4.4.14# cd ..
root@java:/tmp# curl -LO https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 2717k  100 2717k    0     0  1034k      0  0:00:02  0:00:02 --:--:-- 1034k
root@java:/tmp# 
root@java:/tmp# tar -zxvf nagios-plugins-2.3.3.tar.gz
root@java:/tmp# cd nagios-plugins-2.3.3
root@java:/tmp/nagios-plugins-2.3.3# ./configure 
root@java:/tmp/nagios-plugins-2.3.3# make

Esto tardará unos minutos.

Y por último hacemos este:

root@java:/tmp/nagios-plugins-2.3.3# sudo make install


✅ Paso 5: Crear el usuario para acceso web

root@java:/tmp/nagios-plugins-2.3.3# sudo apt install -y apache2-utils
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
apache2-utils is already the newest version (2.4.62-1~deb12u2).
apache2-utils set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
root@java:/tmp/nagios-plugins-2.3.3# sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
New password: 
Re-type new password: 
Adding password for user nagiosadmin
root@java:/tmp/nagios-plugins-2.3.3# 

Admin es mi contraseña.


✅ Paso 6: Habilitar Apache y Nagios

Habilitar apache2

root@java:/tmp/nagios-plugins-2.3.3# systemctl enable apache2
Synchronizing state of apache2.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable apache2
root@java:/tmp/nagios-plugins-2.3.3# systemctl start apache2
Job for apache2.service failed because the control process exited with error code.
See "systemctl status apache2.service" and "journalctl -xeu apache2.service" for details.


Observar el problema:

root@java:/tmp/nagios-plugins-2.3.3# sudo ss -tulpn | grep :80
tcp   LISTEN 0      4096          0.0.0.0:8081       0.0.0.0:*    users:(("docker-proxy",pid=5675,fd=4))                   
tcp   LISTEN 0      511           0.0.0.0:80         0.0.0.0:*    users:(("nginx",pid=56317,fd=5),("nginx",pid=56316,fd=5))
tcp   LISTEN 0      100                 *:8080             *:*    users:(("java",pid=56339,fd=37))                         
tcp   LISTEN 0      4096             [::]:8081          [::]:*    users:(("docker-proxy",pid=5680,fd=4))                   
tcp   LISTEN 0      511              [::]:80            [::]:*    users:(("nginx",pid=56317,fd=6),("nginx",pid=56316,fd=6))


Aqui tuve que deshabilitar ngnix, ya que la instancia estaba ya tocada, por lo que me estaba dando errores de inicio.

root@java:/tmp/nagios-plugins-2.3.3# sudo systemctl start apache2
root@java:/tmp/nagios-plugins-2.3.3# sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Fri 2025-04-04 09:17:53 UTC; 4s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 82742 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 82746 (apache2)
      Tasks: 6 (limit: 1108)
     Memory: 14.5M
        CPU: 97ms
     CGroup: /system.slice/apache2.service
             ├─82746 /usr/sbin/apache2 -k start
             ├─82747 /usr/sbin/apache2 -k start
             ├─82748 /usr/sbin/apache2 -k start
             ├─82749 /usr/sbin/apache2 -k start
             ├─82750 /usr/sbin/apache2 -k start
             └─82751 /usr/sbin/apache2 -k start

Apr 04 09:17:53 java systemd[1]: Starting apache2.service - The Apache HTTP Server...
Apr 04 09:17:53 java apachectl[82745]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using fe80::f816:3eff:fe3d:e3>
Apr 04 09:17:53 java systemd[1]: Started apache2.service - The Apache HTTP Server.


A la hora de establecer nagios, haremos lo siguiiente:


root@java:/home/debian# sudo systemctl restart nagios
root@java:/home/debian# sudo systemctl status nagios
● nagios.service - Nagios Core 4.4.14
     Loaded: loaded (/lib/systemd/system/nagios.service; enabled; preset: enabl>
     Active: active (running) since Fri 2025-04-04 09:52:11 UTC; 4s ago
       Docs: https://www.nagios.org/documentation
    Process: 83652 ExecStartPre=/usr/local/nagios/bin/nagios -v /usr/local/nagi>
    Process: 83653 ExecStart=/usr/local/nagios/bin/nagios -d /usr/local/nagios/>
   Main PID: 83654 (nagios)
      Tasks: 8 (limit: 1108)
     Memory: 7.7M
        CPU: 607ms
     CGroup: /system.slice/nagios.service
             ├─83654 /usr/local/nagios/bin/nagios -d /usr/local/nagios/etc/nagi>
             ├─83655 /usr/local/nagios/bin/nagios --worker /usr/local/nagios/va>
             ├─83656 /usr/local/nagios/bin/nagios --worker /usr/local/nagios/va>
             ├─83657 /usr/local/nagios/bin/nagios --worker /usr/local/nagios/va>
             ├─83658 /usr/local/nagios/bin/nagios --worker /usr/local/nagios/va>
             ├─83659 /usr/local/nagios/bin/nagios -d /usr/local/nagios/etc/nagi>
             ├─83660 /usr/local/nagios/libexec/check_ping -H 127.0.0.1 -w 3000.>
             └─83661 /usr/bin/ping -n -U -w 30 -c 5 127.0.0.1

Apr 04 09:52:11 java nagios[83654]: qh: echo service query handler registered
Apr 04 09:52:11 java nagios[83654]: qh: help for the query handler registered
Apr 04 09:52:11 java nagios[83654]: wproc: Successfully registered manager as @>
lines 1-23
