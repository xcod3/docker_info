#### Crea tu propia VPN a traves de TOR

Para ello se requiere tener un Hardware Dedicado, en este caso una Raspberry PI 3 con Raspbian-lite instalado. Lo que vamos a hacer es crear un Hotspot en la Raspberry PI 3, el cual nos va a redireccionar todo el trafico de la wlan0 directamente a TOR. Para los usuarios que no dispongan de una Raspberry PI o quieran usar otro Hardware Dedicado con dos tarjetas LAN solo tendran que hacer un par de modificaciones en la configuración. 

#### Instalando Dependencias
    pi@raspberrypi:~$ sudo apt-get update
    pi@raspberrypi:~$ sudo apt-get install tor hostapd udhcpd -y

#### Configurando TOR
Para configurar TOR, agregamos las siguientes lineas en /etc/tor/torrc

    Log notice file /var/log/tor/notices.log
    VirtualAddrNetwork 10.192.0.0/10
    AutomapHostsSuffixes .onion,.exit
    AutomapHostsOnResolve 1
    TransPort 9040
    TransListenAddress 172.0.16.1
    DNSPort 53
    DNSListenAddress 172.0.16.1

Luego creamos el archivo notices.log y le damos los permisos correspondientes

    pi@raspberrypi:~$ touch /var/log/tor/notices.log
    pi@raspberrypi:~$ chown debian-tor /var/log/tor/notices.log
    pi@raspberrypi:~$ chmod 644 /var/log/tor/notices.log

#### Configuración de /etc/network/interfaces
    pi@raspberrypi:~$ cat /etc/network/interfaces

    auto lo
    iface lo inet loopback

    auto eth0
    iface eth0 inet static
    address 192.168.1.20
    netmask 255.255.255.0
    gateway 192.168.1.1

    auto wlan0
    iface wlan0 inet static
    address 172.0.16.1
    netmask 255.255.255.0

#### Configuración de udhcpd
    pi@raspberrypi:~$ cat /etc/udhcpd.conf

    start 172.0.16.10		# Inicio del rango de DHCP
    end 172.0.16.20			# Fin del rango de DHCP
    interface wlan0			# interfas por la que escucha el servidor DHCP
    remaining yes
    opt dns 8.8.8.8			# conf. de los DNS
    opt subnet 255.255.255.0	# mascara de subred
    opt router 172.0.16.1		# ip de la interfas de red
    opt lease 864000		# 10 dias de DHCP indicados en segundos

#### Configuración de hostapd

Creando el archivo de configuración
    pi@raspberrypi:~$ sudo touch /etc/hostapd/hostapd.conf
    
    Agregamos las siguientes lineas en el archivo creado
    
    pi@raspberrypi:~$ cat /etc/hostapd/hostapd.conf
    interface=wlan0    
    driver=nl80211    
    ssid=TOR_VPN_Hotspot    
    hw_mode=g    
    channel=1    
    macaddr_acl=0    
    auth_algs=1    
    ignore_broadcast_ssid=0    
    wpa=2    
    wpa_passphrase=5up3r_P455w0rd    
    wpa_key_mgmt=WPA-PSK    
    rsn_pairwise=CCMP

#### Configurando iptables y activando NAT
    pi@raspberrypi:~$ iptables -F
    pi@raspberrypi:~$ iptables -t nat -F
    pi@raspberrypi:~$ iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 22 -j REDIRECT --to-ports 22
    pi@raspberrypi:~$ iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 53 -j REDIRECT --to-ports 53
    pi@raspberrypi:~$ iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040
    pi@raspberrypi:~$ sh -c "iptables-save > /etc/iptables.ipv4.nat"
    pi@raspberrypi:~$ sudo echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf`


#### Activando inicio automatico de los servicios
    pi@raspberrypi:~$ sudo update-rc.d hostapd enable
    pi@raspberrypi:~$ sudo update-rc.d udhcpd enable
    pi@raspberrypi:~$ sudo update-rc.d tor enable

#### Al reiniciar el sistema ya debemos tener nuestro TOR_VPN online
    pi@raspberrypi:~$ sudo shutdown -r now


#### Notas
En ocaciones he presentado problemas con iptables, para ello me he creado este script el cual ejecuto de forma manual via ssh

    #!/bin/bash
    sudo iptables -F
    sudo iptables -t nat -F
    sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 22 -j REDIRECT --to-ports 22
    sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 53 -j REDIRECT --to-ports 53
    sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040



Toda peticion que se haga por el puerto 22 va a ser redireccionada a la Raspberry PI o nuestro servidor VPN, o sea que si tratan de hacer un ssh user@mivps.com en realidad van a estar haciendole un ssh al servidor VPN que acaban de configurar.

De mas esta decir que deben tomar las medidas necesarias para proteger su servidor, el cual va a estar conectado directamente a TOR, para ello deben crear sus propias reglas de iptables y denegar uno que otro servicio en la WAN (eth0).
