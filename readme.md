# Administración Red Hat

- [Cambiar hostname a servidor](#cambiar-hostname-a-servidor)
- [Configuración de red](#configuracion-de-red)
- [Aumentar seguridad en el comando sudo](#aumentar-seguridad-en-el-comando-sudo-)
- [Información del sistema](#informacion-del-sistema)
- [Administración directorios y ficheros](#administracion-directorios-y-ficheros)
- [Usuarios](#usuarios)
- [Grupos](#grupos)
- [Comandos RPM](#comandos-rpm)
- [Servicios](#servicios)
- [Administrar paquetes de forma local](#administrar-paquetes-de-forma-local)




### Cambiar hostname a servidor <a name="cambiar-hostname-a-servidor"/>

Se puede a traves de *Network Manager TUI*:

`nmtui`

O través de variables de entorno del sistema. Podemos visualizar la información de nuestro host:

`hostnamectl`

Y setear nuestro nuevo nombre de host:

`hostnamectl set-hostname nombredeservidor`

Reiniciamos servicio:

`systemctl restart systemd-hostnamed`

Cerramos la terminal y abrimos una nueva para comprobar los cambios correctos



### Configuración de red <a name="configuracion-de-red" />

Comprobar nuestra configuración de red:

`ifconfig`

Comprobar resolución de DNS:

`cat /etc/resolv.conf`

Podemos comprobar donde tenemos todos los archivos de configuración de red de nuestro servidor:

`ls /etc/sysconfig/network-scripts`

Los ficheros de nuestra tarjeta de red se muestran con un nombre similar a este: `ifcfg-ens33`

Si queremos editar la red de nuestro servidor, debemos de modificar ese fichero de red:

`vi /etc/sysconfig/network-scripts/ifcfg-ens33`

Modificamos la variable `BOOTPROTO="dchp"` para indicarle que queremos una IP estática. Debería de quedar `BOOTPROTO="static"`. 

Y añadimos  nuestra configuración de red al final del fichero:

```bash
IPADDR=192.168.0.90
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

Para que se apliquen los cambios reiniciamos nuestra tarjeta de red:

`systemctl restart newtork.service`

Podemos usar el siguiente comando, que lo que hace es lo mismo, invocar al servicio a través de systemctl:

`service network restart`

Dentro del fichero de configuración de red de nuestro servidor vemos que existe una variable llamada `ONBOOT`, es muy importante tener esa variable con valor true, ya que de lo contrario, no se va iniciar la red del servidor. Debe de estar activada `ONBOOT=yes`.



También podemos configurar la tarjeta de red a través *Network Manager TUI*:

`nmtui`



Otra forma de configurar la red es a través de `nmcli`

Para comprobar las conexiones de red:

`nmcli connection show`

Tambien podemos ver las tarjetas de red de esta forma, ya que mostrará las que están activas como las que no y de esta forma podemos saber el nombre de una tarjeta de red inactiva y poder configurarla :

`nmcli device s`

Modicar nuestra conexión de red:

`nmcli connection modify eth0 ipv4.addresses 192.168.1.50/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 connection.autoconnect yes ipv4.method manual` 

Aplicamos los cambios de la nueva configuración:

`nmcli connection down eth0`

`nmcli connection up eth0`

Cambiar la conexión a DHCP:

`nmcli connection modify eth0 connection.autoconnect yes ipv4.method auto`

Si tenemos problemas con la configuración de red, podemos eliminarla:

`nmcli connection delete eth0`

Y para añadir una nueva configuración a nuestra red:

`nmcli connection add type ethernet ifname eth0 con-name eth0 ipv4.addresses 192.168.1.50/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 connection.autoconnect yes ipv4.method manual`



### Aumentar seguridad en el comando `sudo` <a name="aumentar-seguridad-en-el-comando-sudo" />

Tenemos que editar el fichero `/etc/sudoers`:

`vi /etc/sudoers`

Y añadir al final, la línea:

`Defaults:ALL timestamp_timeout=0`

Con esto le decimos al sistema que cada vez que ejecutemos el comando `sudo`, nos pedirá siempre la contraseña de superusuario ya que lo hemos indicado que sea 0 como valor.



### Información del sistema <a name="informacion-del-sistema" />

Visualizar el tiempo de encendido de nuestro servidor:

`uptime`

Visualizar el total de memoria, usada, libre y swap:

`free`

Visualizar los discos duros conectados a nuestro servidor en /dev/

`df -h`



### Administración directorios y ficheros <a name="administracion-directorios-y-ficheros" />

Crear un directorio:

`mkdir directorio1`

Eliminar directorio:

`rmdir directorio1`

Crear estructura de directorios:

`mkdir -p directorio1/directorio2/directorio3`

Eliminar directorio y su subestructura de directorios y ficheros:

`rm -rf directorio1`

Eliminar fichero:

`rm fichero.txt`



### Usuarios <a name="usuarios" />

#### Root

Es el usuario administrador del sistema (superusuario) y su ID es la 0. 

#### Cuentas especiales

Bin, daemon, apache, shutdown, ... son usuarios especiales para determinados servicios. No son usuarios administradores del sistema pero si lo son de sus servicios para los que han sido asignados. Estos usuarios no tienen contraseñas debido a que no inician.  sesión en el sistema (nologin). Por lo general, se les asigna un usuario entre 1 y 100

#### Usuarios normales

Cuentas de usuarios para iniciar sesión en su directorio de trabajo en `/home/nombre_usuario`. Únicamente tienen permisos de administración en su directorio de trabajo. Estos usuarios normales se les asigna la ID a partir de la 1000.

La base de datos de los usuarios del sistema se crean en `/etc/passwd`



#### Creación de usuarios

Se pueden añadir usuarios con el comando `useradd` o `adduser`.

`useradd -c "Usuario normal" -d /home/matxakito -e 20191201 -g 1004 -G 1002 -u 1004 matxakito`

En el anterior comando, estamos añadiendo un usuario llamado "matxakito" con las siguientes características:

- `-c` indicamos una descripción  al usuario
- `-d` la ruta dónde se va a establecer el directorio personal de matxakito
- `-e` la fecha de caducidad de la contraseña
- `-g` el grupo primario al que va a pertnener el usuario
- `-G` grupos secundarios a los que va a pertener el usuario, deben de estar creados previamente
- `-u` el UID del usuario
- `-r` Le estariamos indicando al sistema que sería una cuenta especial. Este parámetro no lo hemos indicado en comando de ejemplo
- `-s` es el shell por defecto que se le asignará al usuario. Por defecto, si no se indica, se le asigna /bin/bash. Este parámetro no lo hemos indicado en comando de ejemplo. Si queremos que no se loguee en el sistema `/sbin/nologin`

Podemos crear usuarios de forma recursiva:

`for i in usuario1 usuario2 usuario3; do useradd $i; done`

#### Modificar usuarios

Con el comando usermod podemos modificar los distintos parametros del usuario, por ejemplo, que el anterior usuario creado, no se pueda loguear:

`usermod -s /sbin/nologin matxakito`

Modificar el nombre de usuario:

`usermod -l matxa matxakito`

Como se ha cambiado el nombre del usuario, sería recomendable modificar también el directorio HOME

`usermod -m -d /home/matxa matxa`

#### Eliminar usuarios

Para eliminar un usuario con su directorio HOME, ejecutamos el siguiente comando:

`userdel -r matxa`

#### Configuración de contraseñas de los usuarios

Para ver los parametros establecidos en la contraseña de un usuario:

`chage -l usuario`

Para modificar la expiración de la cuenta, se debe de poner una fecha:

`chage -E 2018-12-31 usuario`

Para modificar la expiración de la contraseña, se debe de poner los días:

`chage -M 35 usuario`

Para modificar la inactividad de la contraseña después de que haya expirado, se debe de poner los días:

`chage -I 5 usuario`

Días que tienen que pasar, hasta que un usuario pueda cambiar la contraseña después de cambiarla:

`chage -m 3 usuario`

Días para avisar de que el usuario tiene que modificar su contraseña:

`chage -W 5 usuario`

Forzar al usuario que modifique la contraseña en el próximo login:

`chage -d 0 usuario`

Todos estos parámetros que se cambian, se pueden consultar en:

`chage --help`



### Grupos <a name="grupos" />

#### Creación de grupos

Se pueden añadir usuarios con el comando  `groupadd`.

`groupadd -g 2001 nombregrupo`

En el anterior comando, estamos añadiendo un grupo con el GID 2001.

La base de datos de los grupos se pueden consular en `/etc/group`

#### Modificación de grupos

Para modificar el nombre de un grupo:

`groupmod -n gruponuevo grupoantiguo`

#### Eliminación de grupos

Para eliminar un grupos:

`groupdel nombregrupo`

#### Pertenencia a grupos

Para añadir usuario un usuario existente a un grupo:

`usermod -aG nombregrupo usuario`

 Podemos visualizar los miembros pertenecientes a un grupo

`groupmems -g nombregrupo -l` 

Para sacar a un miembro de un grupo:

`gpasswd -d usuario nombregrupo`

O también con este comando:

`getent group nombregrupo`





### Comandos RPM <a name="comandos-rpm" />

Con `RPM` podemos administrar los paquetes `.rpm` en los sistemas Red Hat y derivados. En Debian y derivados usariamos `DPKG` para los paquetes `.deb`.

Para instalar un paquete:

`rpm -ivh paquete.rpm`

Desinstalar paquete:

`rpm -e paquete`

Actualizar paquetes:

`rpm -Uvh paquete`

Listar paquetes instalados en el sistema:

`rpm -qa`

Buscar paquetes instalados:

- `rpm -qa | grep paquete`


Información completa de un paquete:

`rpm -qi paquete`

Listar todos los archivos que tiene un paquete:

`rpm -ql paquete`

Listar archivos de configuración de un paquete:

`rpm -qc paquete`

 

### Servicios <a name="servicios" />

El servicio `systemctl` se incorporó con la llegada de **Systemd** con lo que, el proceso de inicio del sistema era `systemd` en vez de `init`. Este último era el responsable de activar otros servicios en el sistema. Con `systemd`:

- Capacidad de arrancar procesos en paralelo, aumentando la velocidad de arranque del sistema.
- Arancado de Demonios bajo demanda sin que se requiera un servicio nuevo.
- Administración automática de dependencias de procesos, eliminando tiempos de espera.
- Un método de seguimiento de procesos relacionados usando grupos de control.

Para gestionar servicios en nuestro sistema con este nuevo método:

`systemctl status nombreservicio.service`

Podemos también indicar el nombre del servicio sin el `.service`

`systemctl status nombreservicio`



### Administrar paquetes de forma local <a name="administrar-paquetes-de-forma-local" />

Para ello deberíamos de tener una suscripción (al menos de prueba) en el sistema, estar subscritos y tenerla habilitada.

Ver repositorios en el sistema:

`yum repolist`

Descargar repositorio de una suscripción de un servidor Red Hat para no tener que conectarnos a la nube y que equipos clientes puedan conectarse directamente al servidor para actualizar los paquetes. Para ello previamente deberíamos de tener un servidor con FTP para poder compartir estos paquetes.

En este ejemplo, vamos a descargar los paquetes en un directorio llamado `updates` que está dentro del servidor FTP que ya teníamos instalado:

`reposync  -l -n -p updates/ --downloadcomps --download-metadata`

En el anterior comando indicamos con `-n` que descarge paquetes nuevos y con `-p`  vemos el progreso de la descarga.

Creamos la base de datos del repositorio descargado:

`createrepo -v updates/rhel-7-server-rpms/ -g comps.xml`



En  nuestro equipo cliente, creamos un fichero de configuración con el repositorio deseado, le llamamos en este ejemplo, `os.repo`:

`vi /etc/yum.repos.d/os.repo`

Y deberíamos de poner dentro del fichero:

```bash
[os]
name = Repositorio base RHEL 7.3
baseurl = ftp://192.168.0.10/pub/repos/rhel7/
enabled = 1
gpgcheck = 0
```

Limpiamos la cache de metadatos, paquetes, ...:

`yum clean all`

Comprobamos que lee el repositorio correctamente:

`yum repolist`

Podemos comprobar que paquetes controlan determinados ficheros de configuración:

`rpm -qf /etc/samba/smb.conf`
