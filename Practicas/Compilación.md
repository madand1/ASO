# Practica 2: Compilación e Instalación de un programa en C


## Introducción 

>--Vamos a hacer una  pequeña explicación para *Dummies*, porque si no, yo no me entero.

- ¿*Qué es el codigo fuente*?

Si lo traducimos al idioma coloquial para que el usuario, o en este caso yo sepa lo que es, seria como una receta para hacer una tarta. Esta escrito en un lenguaje en el que podamos entender, en este caso en el lenguaje d eprogramación _C_, pero como el pc necesita que este codigo se traduzca a un lenguaje que se pueda ejecutar, utilizaremos un _*Compilador*_, que es el que convierte el código fuente en un programa ejecutable.

- ¿Qué es un _compilador_?



## ¿Que herramientas necesitamos?

### herramientas

- Compilador GCC: Es un compilador popular para C y otros lenguajes. 
    
    - Se instala con el comando:

    ```sudo apt install gcc```

Con el que se descarga e instala tanto el programa como  sus dependencias.


* Ejecución:

```
andy@ASO:~$ sudo apt install gcc
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Se instalarán los siguientes paquetes adicionales:
  binutils binutils-common binutils-x86-64-linux-gnu cpp cpp-12
  fontconfig-config fonts-dejavu-core gcc-12 libabsl20220623 libaom3 libasan8
  libatomic1 libavif15 libbinutils libc-dev-bin libc-devtools libc6-dev
  libcc1-0 libcrypt-dev libctf-nobfd0 libctf0 libdav1d6 libde265-0 libdeflate0
  libfontconfig1 libgav1-1
  ....
```
- Autotools o make:Incluye herramientas como make, autoconf y automake, que se utilizan para automatizar el proceso de compilación. 
  
    - Se instalan con:

    ```
    sudo apt install make
    sudo apt install autoconf automake
    ```


```
andy@ASO:~$ sudo apt install make
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Paquetes sugeridos:
  make-doc
Se instalarán los sigu.......
...............

```
### Requisito opcional, si se hace a través de la clonación de directorios de GitHub:

- Git 

```sudo apt install git```

```
andy@ASO:~$ sudo apt install git
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Se instalarán los siguientes paquetes adicionales:
  git-man liberror-perl patch
Paquetes sugeridos:
  git-daemon-run | git-daemon-sysvinit git-doc git-email git-gui gitk gitweb
  git-cvs git-mediawiki git-svn ed diffutils-doc
Se instalarán los siguientes paquetes NUEVOS:
  git git-man liberror-perl patch
......
```

### Paso para compilar e instalar el programa


1. Descargar el codigo fuente:

He usado git ya que he clonado el repo de *htop*:

- Con lo que creara un directorio llamado *htop*
```
git clone https://github.com/htop-dev/htop.git

```
- Ejecución:

```
andy@ASO:~$ git clone https://github.com/htop-dev/htop.git
Clonando en 'htop'...
remote: Enumerating objects: 19389, done.
remote: Counting objects: 100% (1623/1623), done.
remote: Compressing objects: 100% (66/66), done.
remote: Total 19389 (delta 1570), reused 1557 (delta 1557), pack-reused 17766 (from 1)
Recibiendo objetos: 100% (19389/19389), 6.22 MiB | 3.59 MiB/s, listo.
Resolviendo deltas: 100% (15067/15067), listo.

```

2. Entramso en el directorio descargado:

```
andy@ASO:~$ cd htop/
andy@ASO:~/htop$ 

```

3. Generar los archivos necesarios para la compilación:

```autorencof -i```

¿Que es lo que hace esto?

Este comando prepara los archivos de configuración que se necesitan para la compilación.

>[!CAUTION]
> SE MOSTRARAN ADVERTENCIAS CON LO QUE TENDREMOS QUE METER ESTAS LIBRERIAS: ibncurses5-dev libncursesw5-dev

```
andy@ASO:~/htop$ autoreconf -i
configure.ac:385: warning: pkg.m4 is absent or older than version 0.16; this 'configure' would have incomplete pkg-config support
configure.ac:69: installing 'build-aux/compile'
configure.ac:23: installing 'build-aux/config.guess'
configure.ac:23: installing 'build-aux/config.sub'
configure.ac:24: installing 'build-aux/install-sh'
configure.ac:24: installing 'build-aux/missing'
Makefile.am: installing './INSTALL'
Makefile.am: installing 'build-aux/depcomp'
andy@ASO:~/htop$ 
```
Este nos dara una advertencia de que no hay una libreria, y tendremos que instalarla:

```
sudo apt install libncurses5-dev libncursesw5-dev

```

4. Configurar el entorno de compilación

```
./configure --prefix=/usr/local/htop
```
Con este comando  verificamos que tenemos todas las herramientas y bibliotecas necesarias para la compilación. 

Tambien nos generara un archivo *Makefile*, el cual es un archivo que contiene las reglas y configuraciones necesarias para que se haga el proceso de compilación.


```
andy@ASO:~/htop$ ./configure --prefix=/usr/local/htop
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a race-free mkdir -p... /usr/bin/mkdir -p
checking for gawk... no
checking for mawk... mawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether make supports the include directive... yes (GNU style)
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether the compiler supports GNU C... yes
checking whether gcc accepts -g... yes
checking for gcc option to enable C11 features... none needed
checking whether gcc understands -c and -o together... yes
checking dependency style of gcc... gcc3
checking for stdio.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for strings.h... yes
checking for sys/stat.h... yes
checking for sys/types.h... yes
checking for unistd.h... yes
checking for wchar.h... yes
checking for minix/config.h... no
checking whether it is safe to define __EXTENSIONS__... yes
checking whether _XOPEN_SOURCE should be defined... no
checking for special C compiler options needed for large files... no
checking for _FILE_OFFSET_BITS value needed for large files... no
checking for gcc... (cached) gcc
checking whether the compiler supports GNU C... (cached) yes
checking whether gcc accepts -g... (cached) yes
checking for gcc option to enable C11 features... (cached) none needed
checking whether gcc understands -c and -o together... (cached) yes
checking dependency style of gcc... (cached) gcc3
checking for dirent.h that defines DIR... yes
checking for library containing opendir... none required
checking for stdlib.h... (cached) yes
checking for string.h... (cached) yes
checking for strings.h... (cached) yes
checking for sys/param.h... yes
checking for sys/time.h... yes
checking for sys/utsname.h... yes
checking for unistd.h... (cached) yes
checking for sys/mkdev.h... no
checking for sys/sysmacros.h... yes
checking for execinfo.h... yes
checking for mbstate_t... yes
checking for mode_t... yes
checking for off_t... yes
checking for pid_t... yes
checking for size_t... yes
checking for ssize_t... yes
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for uid_t in sys/types.h... yes
checking for uint8_t... yes
checking for uint16_t... yes
checking for uint32_t... yes
checking for uint64_t... yes
checking for alloc_size... yes
checking for access... yes
checking for nonnull... yes
checking for returns_nonnull... yes
checking for NaN support... yes
checking for __builtin_ctz... yes
checking for library containing ceil... -lm
checking for library containing dlopen... none required
checking for library containing clock_gettime... none required
checking for clock_gettime... yes
checking for dladdr... yes
checking for faccessat... yes
checking for fstatat... yes
checking for host_get_clock_service... no
checking for memfd_create... yes
checking for openat... yes
checking for readlinkat... yes
checking for sched_getscheduler... yes
checking for sched_setscheduler... yes
checking for strchrnul... yes
found ncursesw6 information through ncursesw6-config
checking for keypad in -lncursesw -ltinfo... yes
checking for doupdate in -lncursesw -ltinfo... yes
checking for mvadd_wchnstr in -lncursesw -ltinfo... yes
checking for ncursesw/curses.h... yes
checking for ncursesw/term.h... yes
checking for set_escdelay... yes
checking for getmouse... yes
checking for usable sched_setaffinity... yes
checking for backtrace in -lunwind... no
checking for libunwind.h... no
checking for libunwind/libunwind.h... no
checking for library containing backtrace... none required
checking for cap_init in -lcap... no
checking for sys/capability.h... no
checking for netlink/attr.h... no
checking for netlink/handlers.h... no
checking for netlink/msg.h... no
checking for sensors/sensors.h... no
checking whether C compiler accepts -Wextra-semi-stmt... no
checking whether C compiler accepts -Wimplicit-int-conversion... no
checking whether C compiler accepts -Wnull-dereference... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating htop.1
config.status: creating config.h
config.status: executing depfiles commands

  htop 3.4.0-dev-3.3.0-206-g9c316cc

  platform:                  linux
  os-release file:           /etc/os-release
  (Linux) proc directory:    /proc
  (Linux) openvz:            no
  (Linux) vserver:           no
  (Linux) ancient vserver:   no
  (Linux) delay accounting:  no
  (Linux) sensors:           no
  (Linux) capabilities:      no
  unicode:                   yes
  affinity:                  yes
  unwind:                    no
  hwloc:                     no
  debug:                     no
  static:                    no


```
5. Compilar el programa

una vez que todo este configurado,  lo siguiente que haremos sera ejecutar el programa.

```make```

```
andy@ASO:~/htop$ make
make  all-am
make[1]: se entra en el directorio '/home/andy/htop'
depbase=`echo htop.o | sed 's|[^/]*$|.deps/&|;s|\.o$||'`;\
gcc -DHAVE_CONFIG_H -I.  -DNDEBUG  -std=c99 -pedantic -D_DEFAULT_SOURCE -D_XOPEN_SOURCE=600 -Wall -Wcast-align -Wcast-qual -Wextra -Wfloat-equal -Wformat=2 -Winit-self -Wmissing-format-attribute -Wmissing-noreturn -Wmissing-prototypes -Wpointer-arith -Wshadow -Wstrict-prototypes -Wundef -Wunused -Wwrite-strings -Wnull-dereference -D_XOPE....
.....
......
.....

o    -lncursesw -ltinfo -lm 
make[1]: se sale del directorio '/home/andy/htop'

```
6. Instalar make

Despues de compilarlo, lo podemos instarla con:

```sudo make install```

El cual  copia los archivos ejecutables y otros recursos a sus ubicacines aproiadas en nuestro sistema.

7. Ejecutar htop

![Htop Ejecuion](img/htop.png)


### Desinstalación del paquete que hemos compilado:

1. Entramso en el directorio del codigo fuente.

```cd htop```

```
andy@ASO:~$ sudo whereis htop
htop: /usr/local/htop
andy@ASO:~$ cd /usr/local/htop

```

2. Eliminaremos los archivos instalados, y su ejecutable

Para ello nos saldremso de su directorio, y ejecutaremnos lo siguiente para la eliminación de los archivos:

```sudo rm -rf /usr/local/htop```

- Le meteremso rf ya que lo eliminara de forma recursiva y sin pedir confoiramcion:
  
```
andy@ASO:/usr/local/htop$ cd
andy@ASO:~$ sudo rm -rf /usr/local/htop
andy@ASO:~$ 
```

Eliminación de su ejecutable:

```sudo rm -f /usr/local/bin/htop```

```
andy@ASO:~$ sudo rm -f /usr/local/bin/htop
andy@ASO:~$ 
```

3. Comprobamso que htop ha sido eliminado:

```
andy@ASO:~$ whereis htop
htop:
andy@ASO:~$ 

```
```
andy@ASO:~$ htop
-bash: htop: orden no encontrada
andy@ASO:~$ 
```
