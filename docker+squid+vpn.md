En esta guia de configuración el grueso del tráfico de docker (layers) no pasa por la vpn, solamente lo que es abssolutamente necesario para saltarnos el bloqueo.


# Descripcion General

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
Se crea una regla que diga que los pedidos que se hagan por la ip asignada por la vpn hagan lookup dicha tabla, no a la tabla por defecto donde el gateway sigue siendo el de acceso a internet directo
Finalmente se le dice al squid que a traves de una acl, haga los pedidos a las urls bloqueadas a traves del ip asignado por la vpn

# Prerequisitos

* squid
* vpn

# Pasos

## Creacion de las tablas de lookup para enrutamiento

## Configuracion del openvpn para que no se ponga como gateway por defecto

## Creacion de la regla para que los pedidos que se hagan por la ip de la vpn hagan lookup en la tabla alernativa

## Configuracion del squid para que los pedidos que haga a las urls bloquedas las haga por la ip de la vpn
