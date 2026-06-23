# Documentacion Tecnica | Laboratorio "Ciber Dioses"

## 1. Informacion general

El laboratorio está compuesto por 4 servidores Debian 13 y una máquina cliente Windows.

## 2. Como funciona

Para lograr el sistema de cibercafe, vamos a necesitar una "golden-image", o, imagen base, que va a ser el sistema operativo que corra en los 4 servidores.
Esta imagen base tendra instalados los paquetes minimos necesarios para todas las VMs. Luego, en cada una, se instalaran paquetes necesarios para cada fin, ejemplo, php en el webserver, mysql en el db-server, etc.

### Paquetes en comun para la golden image

Este es un listado de lo que necesitamos en cada VM y una descripcion breve de para que sirve cada uno

-  sudo: Ejecutar comandos como root
-  vim: Editor de texto
-  nano: Editor de texto simple
-  curl: Probar endpoints/hosts
-  wget: Descargar archivos
-  git: Clonar repositorios
-  net-tools: ifconfig, netstat, etc (pruebas networking)
-  iproute2: ip addr, ip route (mas pruebas de networking)
-  dnsutils: nslookup, dig (aun mas pruebas de networking)
-  openssh-server: Acceso SSH (ademas del paquete, debe configurarse el puerto 22035)
-  rsync: Backups y sincronización
-  unzip: Descomprimir ZIP
-  zip: Comprimir ZIP
-  htop: Ver procesos y consumo
-  tree: Ver estructura de directorios
-  less: Visualizar archivos largos
-  ufw: Administrar reglas de firewall

Nota: Si bien varios paquetes ya vienen incluidos en debian 13, es mejor instalar todo en la golden image para que luego no nos falte ninguno.

### IPs privadas para la red interna

Para que las VMs puedan verse entre si, se va a generar una red interna.
Para eso, se van a usar 2 placas de red por servidor. Una de tipo NAT, para poder salir a internet e instalar todo lo necesario, y la otra de tipo "Internal Network", para hablar con el resto de las VMs del laboratorio.

El primer paso luego de clonar la golden image y encender la VM, tiene que ser asignar una ip privada a la placa de red correspondiente

Este paso, dada la complejidad necesaria en el codigo, es el unico que no esta automatizado por scripts.
Por otro lado, el cambio de hostname, y la inclusion de los hosts en /etc/hosts, si esta automatizado

<p align="center">
  <img src="assets/dioses-00.png">
</p>

### Scripts para cada servidor

Para facilitar la instalacion de todo lo que se necesita en cada servidor, y no tener que correr cada uno de los comandos "manualmente", se armaron scripts en shell, y se subieron a github, para poder descargarlos en cada VM, y correrlos para instalar y configurar lo que se necesite.

Nota: Para poder clonarnos los repositorios sin tener que configurar usuarios de git en cada VM, subimos los scripts a repositorios publicos. Esta practica en ambitos corporativos es insegura, pero sirve para el laboratorio. Nunca se deben incluir credenciales de base de datos o informacion sensible en un repositorio publico


<p align="center">
  <img src="assets/dioses-01.png">
</p>

### Breve ejemplo de un script

Una vez que desplegamos el primer servidor, por ejemplo, ciber-db, y configuramos su ip interna, ya estamos listos para ejecutar el script de inicializacion o init.
Dentro de la VM, en la terminal, ejecutamos

~~~ bash
# para clonar el repo
git clone https://github.com/aleconde12/lab-dioses-scripts.git

# para ejecutar el script
bash ./lab-dioses-scripts/db/db-init.sh
~~~

Dicho script contiene cosas genericas (para todos los scripts) como

~~~ bash
#!/bin/bash

set -e

# Definir hostname y /etc/hosts

grep -q "# Laboratorio Ciber" /etc/hosts || cat >> /etc/hosts << 'EOF'

# Laboratorio Ciber
192.168.100.10 ciber-db
192.168.100.20 ciber-web
192.168.100.30 ciber-dhcp
192.168.100.40 ciber-files

EOF

NEW_HOSTNAME="ciber-db"

hostnamectl set-hostname "$NEW_HOSTNAME"

echo "Hostname configurado como: $NEW_HOSTNAME"

~~~

Esto lo que hace es, que el script deje de ejecutarse si se encuentra con algun error (set -e), que defina las ips y los hostnames dentro de /etc/hosts, que defina el hostname como "ciber-db" en este caso, y de ahi en mas, seguir con sus funciones. Eso lo veremos en detalle por cada uno de los servidores.

## 3. Golden image

Como se menciono anteriormente, se instalan varios paquetes en la golden image para que esten presentes en todos los servidores a la hora de clonarlos.
Esto nos da un estandar, y nos aseguramos de que a ningun servidor le va a faltar algun paquete o alguna configuracion que es comun a todo el laboratorio.

### Instalacion de paquetes

Ejecutamos

~~~ bash
apt update
apt install -y \
  sudo \
  vim \
  nano \
  curl \
  wget \
  git \
  net-tools \
  iproute2 \
  dnsutils \
  openssh-server \
  rsync \
  unzip \
  zip \
  htop \
  tree \
  ftp
~~~

Segun cada PC, esto puede llegar a demorar 

### Hostname generico

En un futuro, cada hostname tendra su propio nombre ("ciber-db", "ciber-web", etc), asique ahora para golden image, lo dejamos en "changeme", cosa de que si vemos es nombre, sabemos que nos falto cambiar el hostname

`hostnamectl set-hostname changeme`

### Layout de Teclado LATAM

Esto sirve para no tener problemas con los caracteres de la ISO debian y nuestro teclado latam. Ejecutamos `dpkg-reconfigure keyboard-configuration` y seleccionamos `Spanish (Latin American)`

### Configurar SSH en el puerto 22035

Se modifica la configuración de SSH para que el servicio escuche en el puerto 22035 en lugar del puerto 22, en todas las VMs.

~~~ bash
Port 22035 # Previamente era Port 22

# Descomentamos la linea 
PasswordAuthentication yes
~~~

### Crear usuario sudoer ciber-user

Este usuario nos permitira hacer SSH en todas las VMs, aplicando las mejores practicas, de no usar el usuario root

~~~ bash
adduser ciber-user
usermod -aG sudo ciber-user
~~~

Con estas configuraciones, estamos listos para clonar y configurar la primer VM 

## 4. Primer servidor, "ciber-db"

### Preparar la VM

Clonamos la imagen base, seleccionando "Generate new MAC addresses... ", y luego "Full Clone"

Luego de clonar la imagen base a otra VM, la nombraremos "ciber-db". En esta, configuraremos la base de datos.
En la parte de networking o redes en virtual box, debemos asignarle una segunda interfaz de red, con opcion "internal network", la cual llamaremos simplemente "ciber"

<p align="center">
  <img src="assets/dioses-02.png">
</p>

Credenciales de laboratorio:

- Usuario: `ciber-user`
- Contraseña: `ciber123`

Estas credenciales se utilizan únicamente en el entorno académico.

### Clonar el repo

Nos movemos a /tmp, para evitar ocupar espacio innecesario en el servidor, clonamos el repo, y ejecutamos el script correspondiente. En este caso, /db/db-init.sh


~~~ bash
cd /tmp
git clone https://github.com/aleconde12/lab-dioses-scripts.git
cd lab-dioses-scripts

# ejecutamos el script correspondiente

sudo bash ./db/db-init.sh
~~~

Nota: Este proceso puede tardar bastante, dependiendo de las capacidades de cada PC.
En el laboratorio, tardo aproximadamente 10 minutos, y la db estaria lista unos 7 minutos despues de la finalizacion del script.

## 5. Segundo servidor, "ciber-web"

Clonamos la imagen base, nuevamente, seleccionando "Generate new MAC addresses... ", y luego "Full Clone"

A esta la nombraremos "ciber-web". En esta, configuraremos el servidor (front + php) que hablara con la base de datos.
Recordar siempre que debemos asignarle una segunda interfaz de red, con opcion "internal network", llamada "ciber".
En este caso en particular (y solo para este servidor), vamos a configurar en la interfaz NAT, una opcion llamada "port forwarding", que nos va a permitir consumir el servidor web desde nuestra pc.

Dentro de VirtualBox, ingresamos a la VM ciber-web, luego a las interfaces de red, y en la interfaz de tipo NAT )(la primera), abirmos las configuraciones avanzadas, y vamos a "Port Forwarding" y vamos a setearle:

Name = HTTP
Protocol = TCP
Host IP = vacio
Host Port = 8080
Guest IP = vacio
Guest Port = 80


<p align="center">
  <img src="assets/dioses-03.png">
</p>

<p align="center">
  <img src="assets/dioses-04.png">
</p>


Seguimos el mismo procedimiento para clonar el repo correspondiente, como hicimos con el servidor anterior, en /tmp

~~~ bash
cd /tmp
git clone https://github.com/aleconde12/lab-dioses-scripts.git
cd lab-dioses-scripts

# ejecutamos el script correspondiente

sudo bash ./webserver/web-init.sh
~~~

Si todo funciono correctamente, podremos entrar desde nuestra PC (fuera de virtualbox) por el navegador, a `http://localhost:8080` y visualizar el front del ciber

Mas adelante, vamos a necesitar tener otro script (el de backup) en una ubicacion especifica, para poder dejarlo en crontab.
Para eso, hacemos `sudo cp /tmp/lab-dioses-scripts/webserver/ciber-backup.sh /usr/local/bin/ciber-backup.sh`
Lo vamos a utilizar luego, en el paso 9.


## 6. Tercer servidor, "ciber-files"

Clonamos la imagen base, nuevamente, seleccionando "Generate new MAC addresses... ", y luego "Full Clone"

A esta la nombraremos "ciber-files". En esta, configuraremos el servidor FTP + Samba.
Recordar siempre que debemos asignarle una segunda interfaz de red, con opcion "internal network", llamada "ciber".

Nos movemos a /tmp y volvemos a clonar el repo, y ejecutar el script correspondiente

~~~ bash
cd /tmp
git clone https://github.com/aleconde12/lab-dioses-scripts.git
cd lab-dioses-scripts

# ejecutamos el script correspondiente

sudo bash ./files/files-init.sh
~~~

al termino de ejecutar el script, utilizamos estos comandos para verificar que samba y ftp quedo corriendo ok:

~~~ bash

systemctl status smbd # debe mostrar "active, running, enabled, ready to serve connections"

systemctl status vsftpd # debe mostrar "active, running, enabled"

ss -tlnp # opcional, verificar puertos abiertos, ver 21(FTP), 139 y 445(Samba), 22035(SSH)

~~~


## 7. Cuarto servidor, "ciber-dhcp"

Clonamos la imagen base, nuevamente, seleccionando "Generate new MAC addresses... ", y luego "Full Clone"

A esta la nombraremos "ciber-dhcp".
Recordar siempre que debemos asignarle una segunda interfaz de red, con opcion "internal network", llamada "ciber".

Nos movemos a /tmp y volvemos a clonar el repo, y ejecutar el script correspondiente

~~~ bash
cd /tmp
git clone https://github.com/aleconde12/lab-dioses-scripts.git
cd lab-dioses-scripts

# ejecutamos el script correspondiente

sudo bash ./dhcp/dhcp-init.sh
~~~

Para verificar que luego de la ejecucion del script el servidor dhcp quedo funcionando ok:

~~~ bash
systemctl status isc-dhcp-server
~~~

## 8. Máquina cliente "ciber-win"

Este servidor es una PC windows, y representaria a una maquina del ciber (cliente)

La imagen de este servidor es la mas pesada del laboratorio, por lo cual, debemos asegurarnos de tener suficientes recursos en la PC antes de correr los 5 servidores en simultaneo

Importamos el archivo .ova de windows, y antes de encenderlo, configuramos su interfaz de red "internal network". En este caso, como la PC vivira internamente dentro de la red del ciber, no le asignamos interfaz NAT. Con esto tambien probamos que el servidor DHCP esta funcionando, y le asigno alguna direccion del pool de IPs.

Una vez dentro de ciber-win, entramos a cmd o powershell y ejecutamos `ipconfig /all`, y deberiamos ver una IP asignada desde nuestro servidor DHCP.

Luego, para terminar de pulir las conexiones con la red interna del ciber, realizamos algunos pasos mas dentro de esta VM:
- Editamos `C:\Windows\System32\drivers\etc\hosts` y colocamos todas las IPs y los hostnames correspondientes 
~~~ bash
192.168.100.10 ciber-db
192.168.100.20 ciber-web
192.168.100.30 ciber-dhcp
192.168.100.40 ciber-files
~~~
- Corremos este comando para que funcione el usuario en samba:
~~~ bash
net use * /delete /y
net use \\192.168.100.40\compartido /user:ciberfiles ciber123
~~~
- Y en este punto, ya deberiamos tener acceso a `\\192.168.100.40\compartido` y poder dejar archivos en toda la red interna, tambien deberiamos poder ingresar al navegador, y llegar a `http://ciber-web`, y visualizar el front del web server del ciber.

## 9. Scripts de backup + cron

### Como funciona
Los backups se ejecutan desde el servidor ciber-web, ya que desde allí se tiene acceso tanto al contenido de la aplicación web como a la base de datos ubicada en ciber-db.

El script utilizado es:

`/usr/local/bin/ciber-backup.sh` que copiamos en el paso 5 (ciber-web)

Este script realiza las siguientes tareas:

Genera un backup del directorio de la aplicación web:
/var/www/html
Genera un dump de la base de datos:
laboratorio_db
Comprime ambos respaldos en un único archivo .tar.gz.
Sube el backup final al servidor FTP ciber-files, dentro del directorio:
/srv/ftp/backups
Se desconecta automáticamente del servidor FTP al finalizar la transferencia.
Prueba manual del backup

Antes de automatizar la tarea con cron, se ejecutó el script manualmente desde ciber-web:

`sudo /usr/local/bin/ciber-backup.sh`

Si el script finaliza sin errores, luego se valida el resultado en el servidor ciber-files:

`ls -lh /srv/ftp/backups`

Si el archivo aparece dentro de ese directorio, significa que el backup fue generado correctamente, transferido al servidor FTP y almacenado en la ubicación esperada.

### Rotación de backups

Para cumplir con la consigna, se implementó una rotación circular de 5 backups mediante slots numerados del 1 al 5.
El script mantiene un contador persistente de ejecuciones. En cada ejecución, el contador aumenta en 1 y se calcula qué slot corresponde utilizar.

La lógica es la siguiente:

~~~bash
Ejecución 1  -> slot1
Ejecución 2  -> slot2
Ejecución 3  -> slot3
Ejecución 4  -> slot4
Ejecución 5  -> slot5
Ejecución 6  -> slot1, reemplaza el backup anterior del slot1
Ejecución 7  -> slot2, reemplaza el backup anterior del slot2
~~~

De esta forma, siempre se conservan como máximo los últimos 5 backups. A partir de la sexta ejecución, el backup nuevo sobrescribe el backup más antiguo según el slot correspondiente.

Antes de subir el nuevo archivo al FTP, el script elimina del servidor remoto cualquier backup anterior que pertenezca al mismo slot. Luego sube el nuevo archivo comprimido.

Ejemplo de nombres generados:
~~~bash
backup_2026-06-23_153001_run1_slot1.tar.gz
backup_2026-06-23_153010_run2_slot2.tar.gz
backup_2026-06-23_153020_run3_slot3.tar.gz
backup_2026-06-23_153030_run4_slot4.tar.gz
backup_2026-06-23_153040_run5_slot5.tar.gz
backup_2026-06-23_153050_run6_slot1.tar.gz
~~~

En este ejemplo, la ejecución 6 vuelve a utilizar el slot1, reemplazando el backup de la ejecución 1.

### Validación de la rotación

Para validar el funcionamiento de la rotación, se ejecutó el script varias veces seguidas:
~~~ bash
for i in {1..7}; do
  sudo /usr/local/bin/ciber-backup.sh
  sleep 1
done
~~~

Luego, desde ciber-files, se verificó la cantidad de archivos existentes en el directorio de backups:

~~~ bash
ls -lh /srv/ftp/backups
ls -1 /srv/ftp/backups/backup_*.tar.gz | wc -l
~~~

El resultado esperado es que existan como máximo 5 archivos de backup, incluso después de ejecutar el script 6 o 7 veces.

Ejemplo esperado luego de 7 ejecuciones:
~~~ bash
backup_..._run3_slot3.tar.gz
backup_..._run4_slot4.tar.gz
backup_..._run5_slot5.tar.gz
backup_..._run6_slot1.tar.gz
backup_..._run7_slot2.tar.gz
~~~

Esto demuestra que el sexto backup sobrescribe el primer slot y que el séptimo backup sobrescribe el segundo slot, conservando siempre una rotación máxima de 5 archivos.

### Programación con cron

Una vez validado el funcionamiento manual del script, se configuró su ejecución automática mediante cron.
Para editar el crontab del usuario root:

`sudo crontab -e`

Se agregó la siguiente línea:

`30 23 * * * /usr/local/bin/ciber-backup.sh >> /var/log/ciber-backup.log 2>&1`

Esto ejecuta el script todos los días a las 23:30 y guarda la salida en el archivo de log:

`/var/log/ciber-backup.log`

La configuración puede verificarse con:

`sudo crontab -l`

Y el log puede revisarse con:

`sudo tail -f /var/log/ciber-backup.log`

### Verificación del contenido de los backups

Para revisar el contenido del archivo .tar.gz generado:

`tar -tzf archivo.tar.gz`

Para extraer el contenido:

`tar -xzf archivo.tar.gz`

Para ver el contenido del dump SQL comprimido:

`zcat archivo.sql.gz`

Para descomprimir el archivo SQL:

`gunzip archivo.sql.gz`

### Resultado final

Con esta configuración, el laboratorio cumple con el requerimiento de generar backups automáticos diarios de la aplicación web y de la base de datos, conservar únicamente los últimos 5 backups mediante rotación circular, transferirlos a un servidor FTP y finalizar la conexión luego de la subida.

