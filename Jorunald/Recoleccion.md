En esta práctica explicaré cómo establecer un sistema centralizado de recolección de logs en un entorno Openstack mediante systemd-journal-remote. 

Cuyo propósito es centralizar los registros de sistemas y servicios, lo que mejora la visibilidad y simplifica la gestión y el análisis de los logs. Con esta práctica, aprenderás a configurar y administrar los registros de forma eficiente, garantizando un monitoreo adecuado y un control exhaustivo de tu infraestructura.

Lo que vamos a hacer es lo siguiente:

Implementar en tu escenario de trabajo de Openstack, un sistema de recolección de log mediante journald. Para ello debemos, implementar un sistema de recolección de log mediante el paquete systemd-journal-remote, o similares.

## Instalación de systemd-journatl-remote

En nuestro entorno, el primer paso consistirá en instalar el paquete systemd-journal-remote, lo que nos permitirá acceder de manera remota a las máquinas. Para ello, utilizaremos el gestor de paquetes apt en Luffy, que está basado en Debian 12. Además, se instalará el mismo paquete en los contenedores Nami y Sanji, que están dentro de Luffy. En Zoro, que ejecuta Rocky, se usará el gestor de paquetes dnf para realizar la instalación.

El comando que vamos a usar es esto, para **Luffy, Nami y Sanji**, ya que estamos en un sistema Debian 12:

``sudo apt install systemd-journal-remote -y``

El comando que vamos a usar para **Zoro**, es el siguinet eya qye estamos en un sistema Rocky:

`sudo dnf install systemd-journal-remote -y`

En el servidor, habilitaremos y activaremos los dos componentes de systemd necesarios para recibir los mensajes de registro con el siguiente comando:

```
andy@luffy:~$ sudo systemctl enable --now systemd-journal-remote.socket
Created symlink /etc/systemd/system/sockets.target.wants/systemd-journal-remote.socket → /lib/systemd/system/systemd-journal-remote.socket.
andy@luffy:~$ sudo systemctl enable systemd-journal-remote.service
andy@luffy:~$ 
```

En el cliente, habilitaremos el componente que systemd usa para enviar los mensajes de registro al servidor:

```
[andy@zoro ~]$ sudo systemctl enable systemd-journal-upload.service
Created symlink /etc/systemd/system/multi-user.target.wants/systemd-journal-upload.service → /usr/lib/systemd/system/systemd-journal-upload.service.

```

A continuación, en el servidor, abriremos los puertos 19532 y 80 en el firewall para permitir que el servidor reciba los registros del cliente. El puerto 80 se utilizará para que certbot genere el certificado TLS. Sin embargo, dado que en nuestro caso no contamos con un cortafuegos, esta configuración no será necesaria.

Pero como no tenemos grupso de seguirdad, pues no lo vamos a abrir.

## Generación de claves y certificadoss


Como utilizaremos el servicio con cifrado para garantizar la protección de nuestros registros, vamos a generar los certificados mediante OpenSSL. Aunque es posible crear los certificados de manera manual, contamos con una herramienta llamada Easy RSA que automatiza este proceso. Me encargaré de generar todos los certificados en Luffy y luego los transferiré a las máquinas correspondientes.

Para comenzar, en Luffy ejecutaremos el siguiente comando para instalar Easy RSA y OpenSSL:

``andy@luffy:~$ sudo apt install easy-rsa openssl -y``


A continuación, vamos a crear la estructura de directorios junto a los archivos necesarios para poder comenzar a trabajar con easyRSA:

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa init-pki
* Notice:

  init-pki complete; you may now create a CA or requests.

  Your newly created PKI dir is:
  * /usr/share/easy-rsa/pki

* Notice:
  IMPORTANT: Easy-RSA 'vars' file has now been moved to your PKI above.

andy@luffy:/usr/share/easy-rsa$ 

```

Ahora construimos nuestra CA:

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa build-ca nopass
* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)

Using configuration from /usr/share/easy-rsa/pki/ea08ea07/temp.b83c5ee7
.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.........+.+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+......+.+...............+.....+.......+........+....+...+..+......+....+...........+....+...+...+..+............+...+...+.......+...............+..+....+.....+......+...+.....................+.+...............+........................+..+.......+..+...+...+.......+..+.+...............+.....+............+...+...+...............+....+.................+...+.......+...+..+.+.........+.....+................+..+.+..+....+........+...+...+................+...+...+..................+..+...+....+..+...+......+......+.........+......+...+..........+..+....+............+..+.+...........+.+..+.......+.....+.+..+.+....................+.......+...+.....+....+.......................+...+....+.....+.......+......+..................+..+.......+...+....................+...+.......+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.+.........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+...+...+..+..........+..................+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+..+.........+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:luffy.andres.gonzalonazareno.org

* Notice:

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/usr/share/easy-rsa/pki/ca.crt
```

Ahora vamos a generar la clave privada de todas las máquina, además, generaremos a la vez una solicitud de firma del certificado:

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa gen-req zoro nopass
* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)

...+....+..............+....+...+...............+.........+...+..+.+..+......+...................+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+.+.....+.+...........+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+......+...+......+..............+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+.....+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+.+...........+.+...+..+...+...........................+.......+...+....................+...+.+......+.....+...+.+..............+......+...+.+.....+................+.....+.+...+............+...+...............+.....+.........+.+.........+......+............+.....+..........+..+......+....+......+.....+....+..+.............+.....+.+.........+..................+...+.....+......+...+...+.......+..+.+........+.+..................+..+.+...+.........+.......................+...+.+......+...+...........+......+....+........+......+....+...+..............+.+..+.......+........+.............+.....+....+.....+........................+....+.....+...+...+.+...+...+...+.....+.........................+.....+....+........+...+....+.....+.+......+......+..+.......+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [zoro]:zoro.andres.gonzalonazareno.org
* Notice:

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/zoro.req
key: /usr/share/easy-rsa/pki/private/zoro.key
```

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa gen-req nami nopass
* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)

.....+........+.+...+...+.....+...+..........+.........+...+..+..........+..+....+.....+.+........+.......+.....+....+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+..........+...+.........+...............+..+...+.+............+..+.+.....+......+.+........+.......+.....+.......+.....+...+...+......+......+.........+..........+........+.......+...+.....+.+......+...+..+.+...+...............+.....+....+...+........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
..+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+........+....+...........+.......+...+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.........+.....+.+..+...+....+........+...+......+...............+.+.....+....+...+........+...+.+......+...............+.....+....+...+......+..+..................+....+...........+......+......+.......+...........+.+...+..+...............+....+.................+...+....+......+.....+.+...+...........+.......+........+.....................+....+.....+....+...+...+..+....+.........+......+......+..+...+....+..............+.........+............+.+.....+....+...+...........+.........+.+..+....+.....+..........+........+......+....+..+.+..+.+.....+.......+.....+.........+...+...+.........+......+.......+.....+.+.........+.....+.......+...........+.......+.....+.......+........+...+...+....+...............+...............+......+.................+..........+...+....................+......+...............................+.....+...+....+...............+..+......+...+..........+......+...+...+..+......+......+...+.+.........+...+.....+...+.+.....+....+..............+............+......+....+...+...+..+.+......+.....+..........+.........+..+....+......+..+...+...+.......+........+.......+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [nami]:nami.andres.gonzalonazareno.org
* Notice:

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/nami.req
key: /usr/share/easy-rsa/pki/private/nami.key
```

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa gen-req sanji nopass
* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)

.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+.....+.+...+...........+.....................+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.........+.+..+............+.+........+.......+.....+...............+.......+.....+......+.+...........+...+.........+.+......+........+.......+......+........+.+...+..+.+...+..+...+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
....+.....+..........+...............+........+......+.+..+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+..............+......+....+............+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.............+.+...+.....+......+.......+........+.+......+...............+..+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [sanji]:sanji.andres.gonzalonazareno.org
* Notice:

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/sanji.req
key: /usr/share/easy-rsa/pki/private/sanji.key
```

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa gen-req luffy nopass
* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)

.....+.+.....+...+......+.+...+..+......+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+......+.....+......+.+...+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+...+..................+.........+...+..+......+...+............+...+.........+.+..............+......+.+.....+.+..+.......+......+.................+.+...............+.........+...+.....+.......+...+..+.....................+....+...............+..+....+..+.........+.+...+...........+...+......+.......+..+......+.+.....+.+..+.......+......+...+..+...+............+...+...............+.......+.....+....+........+.+..+...+.+...+.....+......+.+.....+.+.........+........+.+.........+..+.+....................+...+..........+..+....+.........+.........+.....+......+....+......+............+..+...+...+............+...+....+..+......+.......+...+.....+....+........+...+..........+............+...........+......+...+...+....+......+..+............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
..+.....+............+.......+..+...+.+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..........+.....+.+...+..+.+..................+.....+....+......+..+....+...........+....+........+.+.....+.+.........+......+.....+.+...+..+...+......+..........+......+...+.........+............+...+........+...+...............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.......+...+.....+...+..........+......+.........+.....+....+......+.....+.......+.................+.............+..+...+..........+...+..+...............+.........+...+.+..+...+............+....+........+.......+......+..+......+.+...........+.........+.............+..+...+.+.........+..+....+.....+.......+...............+..+.......+...........+...+.+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [luffy]:luffy.andres.gonzalonazareno.org
* Notice:

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/luffy.req
key: /usr/share/easy-rsa/pki/private/luffy.key
```

Una vez generadas las claves privadas y las solicitudes de firma de los certificados, procederemos a firmarlas. Para evitar repetir el proceso en cada máquina, mostraremos cómo firmar el certificado de Luffy, ya que el procedimiento es el mismo para los demás.

Primero, ejecutaremos el siguiente comando en Luffy para firmar la solicitud de certificado:

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa sign-req server luffy
* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.15 3 Sep 2024 (Library: OpenSSL 3.0.15 3 Sep 2024)


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 825 days:

subject=
    commonName                = luffy.andres.gonzalonazareno.org


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /usr/share/easy-rsa/pki/61cf18c1/temp.c8714d52
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'luffy.andres.gonzalonazareno.org'
Certificate is to be certified until May 12 11:21:16 2027 GMT (825 days)

Write out database with 1 new entries
Database updated

* Notice:
Certificate created at: /usr/share/easy-rsa/pki/issued/luffy.crt
```

Durante el proceso, EasyRSA nos solicitará que confirmemos la firma y verifiquemos los datos de la solicitud. Aceptaremos escribiendo "yes" y, a continuación, se generará el certificado firmado.

Después de firmar el certificado de Luffy, repetiremos el mismo procedimiento para las demás máquinas, reemplazando "luffy" por el nombre correspondiente de cada una. Esto garantizará que todos los certificados de las máquinas sean firmados correctamente.

Dejo por aqui los comandos:

```
andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa sign-req server zoro

andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa sign-req server nami

andy@luffy:/usr/share/easy-rsa$ sudo ./easyrsa sign-req server sanji
```

## Mover los certificados a las máquinas correspondientes

Ahora debemos transferir los certificados y las claves privadas a las máquinas correspondientes utilizando el método de compartición que prefiramos. En mi caso, lo haré a través de un servidor temporal en python3. Para cada máquina, copiaremos el certificado firmado, que se encuentra en /usr/share/easy-rsa/pki/issued/, junto con su clave privada, ubicada en /usr/share/easy-rsa/pki/private/.

Ya que tenemos la siguiiente distribución de archivios:

```
root@luffy:/usr/share/easy-rsa/pki# tree
.
├── ca.crt
├── certs_by_serial
│   ├── 03E0B913712951C00BBEA4DA8452E600.pem
│   ├── 16666C5F04FE5243EBAE8C9060742F15.pem
│   ├── 77AE0CC718A8C6AC2D8BAD8B043883EE.pem
│   └── AACB3DC40B8A5762D991017719A50246.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│   ├── luffy.crt
│   ├── nami.crt
│   ├── sanji.crt
│   └── zoro.crt
├── openssl-easyrsa.cnf
├── private
│   ├── ca.key
│   ├── luffy.key
│   ├── nami.key
│   ├── sanji.key
│   └── zoro.key
├── reqs
│   ├── luffy.req
│   ├── nami.req
│   ├── sanji.req
│   └── zoro.req
├── revoked
│   ├── certs_by_serial
│   ├── private_by_serial
│   └── reqs_by_serial
├── safessl-easyrsa.cnf
├── serial
├── serial.old
├── vars
└── vars.example

```

Y si nos acercamos en el caso de los clientes, tendremos que enviar el certificado de Autoridad Certificadora (CA), el cual esta en el directorio ```/usr/share/easy-rsa/pki/ca.crt```:

```
root@luffy:/usr/share/easy-rsa/pki# ls -l
total 88
-rw------- 1 root root 1289 Feb  6 11:16 ca.crt
drwx------ 2 root root 4096 Feb  6 11:24 certs_by_serial
-rw------- 1 root root  378 Feb  6 11:24 index.txt
-rw------- 1 root root   20 Feb  6 11:24 index.txt.attr
-rw------- 1 root root   20 Feb  6 11:24 index.txt.attr.old
-rw------- 1 root root  283 Feb  6 11:24 index.txt.old
drwx------ 2 root root 4096 Feb  6 11:24 issued
-rw------- 1 root root 4935 Feb  6 11:15 openssl-easyrsa.cnf
drwx------ 2 root root 4096 Feb  6 11:19 private
drwx------ 2 root root 4096 Feb  6 11:19 reqs
drwx------ 5 root root 4096 Feb  6 11:16 revoked
-rw------- 1 root root 4939 Feb  6 11:24 safessl-easyrsa.cnf
-rw------- 1 root root   33 Feb  6 11:24 serial
-rw------- 1 root root   33 Feb  6 11:24 serial.old
-rw------- 1 root root 9425 Feb  6 11:15 vars
-rw------- 1 root root 9425 Feb  6 11:15 vars.example

```

Y ahora nos transferimos los objetos:

```
root@luffy:/usr/share/easy-rsa/pki# cd issued/
root@luffy:/usr/share/easy-rsa/pki/issued# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
172.16.0.200 - - [06/Feb/2025 11:39:48] "GET /zoro.crt HTTP/1.1" 200 -
192.168.0.3 - - [06/Feb/2025 11:41:20] "GET /sanji.crt HTTP/1.1" 200 -
192.168.0.2 - - [06/Feb/2025 11:41:33] "GET /nami.crt HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.
root@luffy:/usr/share/easy-rsa/pki/issued# cd ..
root@luffy:/usr/share/easy-rsa/pki# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
172.16.0.200 - - [06/Feb/2025 11:42:12] "GET /ca.crt HTTP/1.1" 200 -
192.168.0.3 - - [06/Feb/2025 11:42:22] "GET /ca.crt HTTP/1.1" 200 -
192.168.0.2 - - [06/Feb/2025 11:42:26] "GET /ca.crt HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.
root@luffy:/usr/share/easy-rsa/pki# cd private/
root@luffy:/usr/share/easy-rsa/pki/private# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
172.16.0.200 - - [06/Feb/2025 11:43:25] "GET /zoro.key HTTP/1.1" 200 -
192.168.0.3 - - [06/Feb/2025 11:43:34] code 404, message File not found
192.168.0.3 - - [06/Feb/2025 11:43:34] "GET /sanji.crt HTTP/1.1" 404 -
192.168.0.3 - - [06/Feb/2025 11:43:52] "GET /sanji.key HTTP/1.1" 200 -
192.168.0.2 - - [06/Feb/2025 11:44:12] code 404, message File not found
192.168.0.2 - - [06/Feb/2025 11:44:12] "GET /nami.crt HTTP/1.1" 404 -
192.168.0.2 - - [06/Feb/2025 11:44:18] code 404, message File not found
192.168.0.2 - - [06/Feb/2025 11:44:18] "GET /nami.crt HTTP/1.1" 404 -
192.168.0.2 - - [06/Feb/2025 11:44:24] "GET /nami.key HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.

```

Y comprobamos que nos lo hemos llevado bien:

```

andy@nami:~$ ls
ca.crt  nami.crt  nami.key

[andy@zoro ~]$ ls
ca.crt  zoro.crt  zoro.key

andy@sanji:~$ ls
ca.crt  dead.letter  mbox  sanji.crt  sanji.key
```

El directorio `/etc/letsencrypt/live/` es utilizado por Let’s Encrypt para guardar los certificados generados con certbot. Sin embargo, dado que en este caso estamos creando los certificados de forma manual utilizando EasyRSA, este directorio no se crea automáticamente. Por lo tanto, es necesario crear dicho directorio para luego trasladar los archivos correspondientes allí.

```
root@luffy:/usr/share/easy-rsa/pki/private# cd
root@luffy:~# sudo mkdir -p /etc/letsencrypt/live/andy.es
root@luffy:~# sudo mv /usr/share/easy-rsa/pki/issued/luffy.crt /etc/letsencrypt/live/andy.es/
root@luffy:~# sudo mv /usr/share/easy-rsa/pki/private/luffy.key /etc/letsencrypt/live/andy.es/
root@luffy:~# 
```

Genero ahro aun fichero  combined.pem solamente en luffy, para realizar esto debemos concatenar nuestro certificado con la clave privada en un solo archivo:

`root@luffy:~# cat /etc/letsencrypt/live/andy.es/luffy.crt /etc/letsencrypt/live/andy.es/luffy.key > /etc/letsencrypt/live/andy.es/combined.pem`

Cambiamos los permisos:

```
root@luffy:~# chown root:systemd-journal-remote /etc/letsencrypt/live/andy.es/*
root@luffy:~# chmod 640 /etc/letsencrypt/live/andy.es/luffy.crt
root@luffy:~# chmod 640 /etc/letsencrypt/live/andy.es/luffy.key
```

Quedando luffy de la siguiente forma:

```
root@luffy:~# ls -l /etc/letsencrypt/live/andy.es/
total 20
-rw-r--r-- 1 root systemd-journal-remote 6529 Feb  6 11:57 combined.pem
-rw-r----- 1 root systemd-journal-remote 4829 Feb  6 11:21 luffy.crt
-rw-r----- 1 root systemd-journal-remote 1700 Feb  6 11:19 luffy.key
```

Ahora, en las demás máquinas realizamos lo requerido para que queden las máquinas de la siguiente forma:

- Zoro
- Nami
- Sanji

```
[andy@zoro ~]$ ls -l /etc/letsencrypt/live/zoro.andy.es/
total 16
-rw-r-----. 1 root systemd-journal-remote 1289 Feb  6 11:16 ca.crt
-rw-r-----. 1 root systemd-journal-remote 4827 Feb  6 11:24 zoro.crt
-rw-r-----. 1 root systemd-journal-remote 1704 Feb  6 11:16 zoro.key
```

```
andy@nami:~$ ls -l /etc/letsencrypt/live/nami.andy.es/
total 16
-rw-r----- 1 root systemd-journal-remote 1289 Feb  6 11:16 ca.crt
-rw-r----- 1 root systemd-journal-remote 4827 Feb  6 11:24 nami.crt
-rw-r----- 1 root systemd-journal-remote 1708 Feb  6 11:17 nami.key
```

```
andy@sanji:~$ ls -l /etc/letsencrypt/live/sanji.andy.es/
total 16
-rw-r----- 1 root systemd-journal-remote 1289 Feb  6 11:16 ca.crt
-rw-r----- 1 root systemd-journal-remote 4829 Feb  6 11:24 sanji.crt
-rw-r----- 1 root systemd-journal-remote 1704 Feb  6 11:18 sanji.key
```
Esto esta mal, bueno no , tienes que hacer esto para tdoos los clienets:

```
[andy@zoro ~]$ getent passwd systemd-journal-upload
[andy@zoro ~]$ chown root:systemd-journal-update /etc/letsencrypt/live/zoro.andy.es/*
chown: invalid group: ‘root:systemd-journal-update’
[andy@zoro ~]$ sudo groupadd --system systemd-journal-upload
[andy@zoro ~]$ sudo useradd --system --no-create-home --shell /sbin/nologin -g systemd-journal-upload systemd-journal-upload
[andy@zoro ~]$ sudo chown systemd-journal-upload:systemd-journal-upload /etc/letsencrypt/live/zoro.andy.es/*
[andy@zoro ~]$ sudo chmod 640 /etc/letsencrypt/live/zoro.andy.es/*

```

Los certificados se generaron previamente de forma manual. Cabe mencionar que su ubicación de almacenamiento no es crucial, siempre que se les asignen los permisos correctos.

## Configuración del servidor

Solo queda indicar la configuración del servicio, donde tendremos que cambiar las rutas correspondientes a nuestros ficheros:

```
andy@luffy:/usr/share/easy-rsa$ cat /etc/systemd/journal-remote.conf 
[Remote]
Seal=false
SplitMode=host
ServerKeyFile=/etc/letsencrypt/live/andy.es/luffy.key
ServerCertificateFile=/etc/letsencrypt/live/andy.es/luffy.crt
TrustedCertificateFile=/etc/letsencrypt/live/andy.es/combined.pem
```

Ahora lo que hago es reiniciar el servicio y compurebo que se haya levantado:

```
andy@luffy:~$ sudo systemctl restart systemd-journal-remote.service
andy@luffy:~$ sudo systemctl status systemd-journal-remote.service
● systemd-journal-remote.service - Journal Remote Sink Service
     Loaded: loaded (/lib/systemd/system/systemd-journal-remote.service; indirect; preset: disabled)
     Active: active (running) since Thu 2025-02-06 12:29:08 UTC; 6s ago
TriggeredBy: ● systemd-journal-remote.socket
       Docs: man:systemd-journal-remote(8)
             man:journal-remote.conf(5)
   Main PID: 18689 (systemd-journal)
     Status: "Processing requests..."
      Tasks: 1 (limit: 2314)
     Memory: 1.8M
        CPU: 63ms
     CGroup: /system.slice/systemd-journal-remote.service
             └─18689 /lib/systemd/systemd-journal-remote --listen-https=-3 --output=/var/log/journal/remote/

Feb 06 12:29:08 luffy systemd[1]: Started systemd-journal-remote.service - Journal Remote Sink Service.
Feb 06 12:29:08 luffy systemd-journal-remote[18689]: Certificate checking disabled.
Feb 06 12:29:08 luffy systemd-journal-remote[18689]: microhttpd: MHD_OPTION_EXTERNAL_LOGGER is not the first option s>
lines 1-17/17 (END)
```

## COnfiguración del cliente

Ahroa pasaré a agregar los archivos necesarios para la configuración , con lo que tendre que incluir la clave privada y los certificados correspondiente, o sea todo:

```
[andy@zoro ~]$ cat /etc/systemd/journal-remote.conf 
[Upload]
URL=https://luffy.andres.gonzalonazareno.org:19532
ServerKeyFile=/etc/letsencrypt/live/zoro.andy.es/luffy.key
ServerCertificateFile=/etc/letsencrypt/live/zoro.andy.es/luffy.crt
TrustedCertificateFile=/etc/letsencrypt/live/zoro.andy.es/ca.crt

```

Reinicio los servicios y me dan que estan mal:

```
[andy@zoro ~]$ ls -l /etc/letsencrypt/live/zoro.andy.es/
total 28
-rw-r-----. 1 systemd-journal-upload systemd-journal-upload 1289 Feb  6 11:16 ca.crt
-rw-r-----. 1 systemd-journal-upload systemd-journal-upload 4829 Feb  6 11:21 luffy.crt
-rw-r-----. 1 systemd-journal-upload systemd-journal-upload 1700 Feb  6 11:19 luffy.key
-rw-r-----. 1 systemd-journal-upload systemd-journal-upload 4827 Feb  6 11:24 zoro.crt
-rw-r-----. 1 systemd-journal-upload systemd-journal-upload 1704 Feb  6 11:16 zoro.key
[andy@zoro ~]$ sudo systemctl restart systemd-journal-upload
[andy@zoro ~]$ sudo systemctl status systemd-journal-upload
● systemd-journal-upload.service - Journal Remote Upload Service
     Loaded: loaded (/usr/lib/systemd/system/systemd-journal-upload.service; enabled; preset: disabled)
     Active: active (running) since Thu 2025-02-06 13:02:49 UTC; 4s ago
       Docs: man:systemd-journal-upload(8)
   Main PID: 1575 (systemd-journal)
     Status: "Processing input..."
      Tasks: 1 (limit: 10890)
     Memory: 5.6M
        CPU: 914ms
     CGroup: /system.slice/systemd-journal-upload.service
             └─1575 /usr/lib/systemd/systemd-journal-upload --save-state

```

## Comprobación:

Una vez que ambos servicios estén en funcionamiento, se almacenará en el servidor un archivo que contendrá los registros (logs) generados por el cliente.

```
root@luffy:~# ls -la /var/log/journal/remote/
total 24588
drwxr-xr-x  2 systemd-journal-remote systemd-journal-remote     4096 Feb  6 13:02 .
drwxr-sr-x+ 4 root                   systemd-journal            4096 Feb  6 12:29 ..
-rw-r-----  1 systemd-journal-remote systemd-journal-remote 25165824 Feb  6 13:03 remote-172.16.0.200.journal
root@luffy:~# 

```

Podemos ver los logs de los diferentes servicios, por ejemplo realizaremos el filtro por httpd:

```
root@luffy:~# journalctl -u httpd --file=/var/log/journal/remote/remote-172.16.0.200.journal
Jan 28 11:12:43 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:12:43 zoro.andy.gonzalonazareno.org httpd[50215]: (2)No such file or directory: AH02291: Cannot access dire>
Jan 28 11:12:43 zoro.andy.gonzalonazareno.org httpd[50215]: AH00014: Configuration check failed
Jan 28 11:12:43 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Main process exited, code=exited, status=1/F>
Jan 28 11:12:43 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Failed with result 'exit-code'.
Jan 28 11:12:43 zoro.andy.gonzalonazareno.org systemd[1]: Failed to start The Apache HTTP Server.
Jan 28 11:14:19 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:14:19 zoro.andy.gonzalonazareno.org httpd[50238]: Server configured, listening on: port 80
Jan 28 11:14:19 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:19:38 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:19:39 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
Jan 28 11:19:39 zoro.andy.gonzalonazareno.org systemd[1]: Stopped The Apache HTTP Server.
Jan 28 11:19:39 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:19:39 zoro.andy.gonzalonazareno.org httpd[52669]: Server configured, listening on: port 80
Jan 28 11:19:39 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:21:46 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:21:47 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
Jan 28 11:21:47 zoro.andy.gonzalonazareno.org systemd[1]: Stopped The Apache HTTP Server.
Jan 28 11:21:47 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:21:47 zoro.andy.gonzalonazareno.org httpd[52891]: Server configured, listening on: port 80
Jan 28 11:21:47 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:24:16 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:24:17 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
Jan 28 11:24:17 zoro.andy.gonzalonazareno.org systemd[1]: Stopped The Apache HTTP Server.
Jan 28 11:24:17 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:24:17 zoro.andy.gonzalonazareno.org httpd[53147]: Server configured, listening on: port 80
Jan 28 11:24:17 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:27:54 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:27:55 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
Jan 28 11:27:55 zoro.andy.gonzalonazareno.org systemd[1]: Stopped The Apache HTTP Server.
Jan 28 11:27:55 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:27:55 zoro.andy.gonzalonazareno.org httpd[53600]: Server configured, listening on: port 80
Jan 28 11:27:55 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:36:47 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:36:48 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
Jan 28 11:36:48 zoro.andy.gonzalonazareno.org systemd[1]: Stopped The Apache HTTP Server.
Jan 28 11:36:48 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Consumed 1.129s CPU time.
Jan 28 11:36:48 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:36:48 zoro.andy.gonzalonazareno.org httpd[53885]: Server configured, listening on: port 80
Jan 28 11:36:48 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:39:31 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:39:32 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
Jan 28 11:39:32 zoro.andy.gonzalonazareno.org systemd[1]: Stopped The Apache HTTP Server.
Jan 28 11:39:32 zoro.andy.gonzalonazareno.org systemd[1]: Starting The Apache HTTP Server...
Jan 28 11:39:32 zoro.andy.gonzalonazareno.org httpd[54158]: Server configured, listening on: port 80
Jan 28 11:39:32 zoro.andy.gonzalonazareno.org systemd[1]: Started The Apache HTTP Server.
Jan 28 11:40:11 zoro.andy.gonzalonazareno.org systemd[1]: Stopping The Apache HTTP Server...
Jan 28 11:40:12 zoro.andy.gonzalonazareno.org systemd[1]: httpd.service: Deactivated successfully.
```