
# Guía de configuración de VPN para acceder a hub.docker.com

Acá encontrará instrucciones acerca de cómo configurar SQUID para permitir el acceso a [Docker Hub](https://hub.docker.com) desde lugares en que ese dominio esté bloqueado o sea de difícil acceso. En esta guia de configuración el grueso del tráfico de docker (layers) no pasa por la VPN que se configura, sino solamente lo que es abssolutamente necesario para saltarnos el bloqueo.


## Descripcion General

Lista de urls relacionadas con docker a las que no se puede acceder desde Cuba:
* hub.docker.com
* registry-1.docker.io
* auth.docker.io
* index.docker.io

Lista de urls a las que se puede acceder directamente:
* *.cloudfront.net => de aqui se baja el contenido de las layers, representa el grueso del tráfico de docker

La idea es hacer que de una forma u otra, los pedidos de docker a las direcciones que estan bloqueadas para Cuba pasen por una vpn o algo similar.

En este ejemplo concreto, vamos a lograrlo utilizando squid como servidor proxy, una vpn (openvpn) para poder acceder a las urls bloqueadas y la configuracion  de enrutamiento del host para seleccionar la rua que seguiran los pedidos.

El squid accederá directamente a internet en el caso de los pedidos a *.cloudfront.net y a traves de la vpn cuando el pedido sea a alguna de las urls bloqueadas.

En principio esto se puede lograr solamente con la vpn + rutas, sin utilizar squid, pero como el enrutamiento es basado en IPs de destino, se hace engorroso darle el seguimiento a todas las IPs que representan las urls bloqueadas.

Cuando se activa la vpn, se le asigna una ip al host, por lo que tendriamos dos ips, una la de la red local y la otra la asignada por la vpn.

Para mantener el grueso del tráfico por la salida de internet, lo primero que hay que hacer es decirle a la configuracion de la vpn que no enrute todo el trafico.

Luego creamos una tabla de enrutamiento (diferente de la por defecto) donde se dice que todo el trafico que pase por esta tabla lo enrute a traves de la vpn (se configura el gw por defecto como el otro endpoint de la vpn). 

Se crea una regla que diga que los pedidos que se hagan por la ip asignada por la vpn hagan lookup dicha tabla, no a la tabla por defecto donde el gateway sigue siendo el de acceso a internet directo.

Finalmente se le dice al squid que a traves de una acl, haga los pedidos a las urls bloqueadas a traves del ip asignado por la vpn

## Prerequisitos

* squid
* vpn
* iproute2

## Pasos

### Creacion de las tablas de lookup para enrutamiento

Las tablas de enrutamiento por defecto estan identificadas con un numero, pero para facilitar las confuraciones se le puede asignar un alias.
En este caso, vamos a utilizar la tabla 1000, y le vamos a crear un alias, llamandola "vpn"

Adicionamos al fichero `/etc/iproute2/rt_tables` la siguiente linea:

`
1000    vpn
`


### Configuracion del openvpn para que no se ponga como gateway por defecto

Casi todas las configuraciones de vpn, una ves que se establece la conexion, hacen que todo el trafico pase por ellas, esto lo puede hacer a traves de dos partes de la configuracion de la vpn, una es a traves del setting `redirect-gateway` y la otra forma es a traves de las rutas que el servidor de openvpn manda para el cliente. 

1- Decimos que por defecto no se ponga como gateway, comentando o eliminando del ficheron .ovpn la linea que diga: `redirect-gateway`

2- Evitamos que el servidor nos mande las rutas, adicionando al fichero de configuracion el setting `route-nopull`

### Creacion de la regla para que los pedidos que se hagan por la ip de la vpn hagan lookup en la tabla alernativa

Este paso en realidad hay que hacerlo cada vez que se conecte la vpn pues la ip que se le asigna al host y la del servidor de la vpn pueden variar entre conexion y conexion.

Para esto se puede usar las posibilidades que brinda el openvpn de que ejecute un script cuando se estableza la conexion, este script simplemente tiene que ser un script ejecutable, para claridad le pondemos poner `<fichero ovpn>-up.sh`, o sea.. que si la configuracion de la vpn esta en `docker.ovpn`, debemos tener un script ejecutable al lado que se llame `docker-up.sh`, y en el contenido de `docker.opvn` adicionamos las lineas:

`script-security 2` => permite la ejecucion de scripts

`route-up /path/to/docker-up.sh` => ejecuta este script cuando se vaya a establecer el enrutamiento (luego de que se establece la conexion)

En el contenido del fichero `docker-up.sh` ponemos:

```bash
#!/bin/bash

# Eliminamos la regla preexistente, si existe
while `ip rule del table vpn`; do
  echo deleting ip rule
  sleep 1
done

# Aqui hacemos uso de dos variables predefinidas por openvpn:
# - $ifconfig_local => ip asignado al host por el servidor de la vpn
# - $ifconfig_remote => ip remoto del servidor de la vpn

# Adicionamos la regla que dice que cuando una conexion se haga desde el ip de la vpn busque en la tabla llamada 'vpn'
ip rule add from $ifconfig_local table vpn
# Modificamos la tabla de enrutamiento 'vpn' para que su gateway sea el ip remoto de la vpn, 
# de esta forma todos los pedidos que se enruten por esa tabla pasaran por la vpn
ip route add default via $ifconfig_remote table vpn

```

### Configuracion del squid para que los pedidos que haga a las urls bloquedas las haga por la ip de la vpn

Creamos dos ficheros de configuracion que se incluiran en la configuracion de squid, esto es para facilitar la reconfiguracion de esta parte sin afectar todo lo demas que tengamos en el squid:

Adicionamos a `squid.conf`:

``` 
include /opt/vpn/squid-urls.conf 
include /opt/vpn/squid-custom.conf
```
Contenido de `squid-urls.conf` (acl con las urls bloqueadas)

```
acl blocked_sites dstdomain hub.docker.com
acl blocked_sites dstdomain registry-1.docker.io
acl blocked_sites dstdomain auth.docker.io
acl blocked_sites dstdomain index.docker.io
```

El contenido de `squid-custom.url` lo crearemos adicionandole estas lineas al fichero `docker-up.sh`:

```bash
# Sobreescribimos el fichero poniendole la linea siguiente
echo tcp_outgoing_address $ifconfig_local blocked_sites >       /opt/vpn/squid-custom.conf
# Adicionamos al fichero la linea
echo tcp_outgoing_address ip_del_host                  >>      /opt/vpn/squid-custom.conf

#reiniciamos el squid para que coja las configuraciones
service squid reload

```

Lo unico que faltaria seria configurar el docker para que utilice el squid local 


El resultado final seria algo como lo siguiente:

##### Configuracion de las tablas de enrutamiento
```bash
$ cat /etc/iproute2/rt_tables
#
# reserved values
#
255     local
254     main
253     default
0       unspec
#
# local
#
#1      inr.ruhep
1000    book

```

##### Contenido (solo con las parte relevantes) del fichero de openvpn
```bash
$ cat /opt/vpn/docker.ovpn
....
....
# redirect-gateway
route-nopull

script-security 2
route-up /opt/vpn/docker-up.sh
```

##### Contenido del script `docker-up.sh`
```bash
$ cat /opt/vpn/docker-up.sh
#!/bin/bash

# Eliminamos la regla preexistente, si existe
while `ip rule del table vpn`; do
  echo deleting ip rule
  sleep 1
done

# Aqui hacemos uso de dos variables predefinidas por openvpn:
# - $ifconfig_local => ip asignado al host por el servidor de la vpn
# - $ifconfig_remote => ip remoto del servidor de la vpn

# Adicionamos la regla que dice que cuando una conexion se haga desde el ip de la vpn 
# busque en la tabla llamada 'vpn'
ip rule add from $ifconfig_local table vpn
# Modificamos la tabla de enrutamiento 'vpn' para que su gateway sea el ip remoto de la vpn, 
# de esta forma todos los pedidos que se enruten por esa tabla pasaran por la vpn
ip route add default via $ifconfig_remote table vpn

# Sobreescribimos el fichero poniendole la linea siguiente
echo tcp_outgoing_address $ifconfig_local blocked_sites >       /opt/vpn/squid-custom.conf
# Adicionamos al fichero la linea
echo tcp_outgoing_address ip_del_host                  >>      /opt/vpn/squid-custom.conf

service squid reload
```

##### Contenido del fichero `squid-urls.conf` con las definiciones de las acls:
```bash
$ cat /opt/vpn/squid-urls.conf
acl blocked_sites dstdomain hub.docker.com
acl blocked_sites dstdomain registry-1.docker.io
acl blocked_sites dstdomain auth.docker.io
acl blocked_sites dstdomain index.docker.io
```

