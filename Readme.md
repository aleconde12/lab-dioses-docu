# Laboratorio "Ciber Dioses"

## Informacion general

El laboratorio "Ciber Dioses" es un proyecto que, mediante 4 servidores o maquinas virtuales debian 13, conforma un sistema de Cibercafe, un negocio muy popular de la decada del 2000.

## Como funciona

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
-  openssh-server: Acceso SSH
-  rsync: Backups y sincronización
-  unzip: Descomprimir ZIP
-  zip: Comprimir ZIP
-  htop: Ver procesos y consumo
-  tree: Ver estructura de directorios
-  less: Visualizar archivos largos

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