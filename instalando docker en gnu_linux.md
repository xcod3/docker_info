# Cómo instalar Docker CE en GNU/Linux desde Cuba y hacer uso del repo de docker online.


 Esto es un breve tutorial que he hecho a petición de los Grupos @sysadmincuba & @dockercuba de telegram, este tutorial no va a cubrir la parte de trabajar con docker, solo lo que es instalación de Docker en varias distribuciones GNU/Linux y evitar las restricciones que tenemos con los repositorios de Docker Online. El principal problema que tenemos como cubanos, no es la instalación de Docker, sino a la hora de hacer "pull" para bajar containers. En este tutorial estare cubriendo algunas ideas para poder hacer uso de los repositorios oficiales de Docker a los cuales no tenemos acceso y tambien como importar y exportar containers desde Docker sin necesidad de usar los repositorios oficiales. Por ahora solo voy a explicar la instalación de Docker en las tres distribuciones que considero son las más usadas en nuestro país (Ubuntu, Debian & Fedora). 

 Los usuarios que tengan alguna duda o quieran hacer alguna recomendación pueden contactarme via telegram @xc0d3, últimamente no he tenido mucho tiempo para dedicarlo a esto, es algo que he hecho apurado así que no se fijen mucho en los posibles errores que puedan encontrar... infórmenme de ello..

**Instalando Docker en Ubuntu**

Requisitos:
Ubuntu 16.10 (Yakkety)
Ubuntu 16.04-LTS (Xenial)
Ubuntu 14.04-LTS (Trusty)

Nota: Docker CE soporta las versióndes x86_64 & armhf de Ubuntu

**Preparandonos para la instalación**

Paquetes recomendados para Ubuntu-14.04 (Trusty)

`$ sudo apt-get update`

`$ sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual`

**Instalando Docker CE**
Antes que nada, debemos agregar el repositorio de Docker a nuestro *sources.list* para luego tener la posibilidad de Instalar & Actualizar Docker desde el repositorio

- Configurando el repositorio

	`$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common`

- Agregando las Claves GPG oficiales de Docker

	`$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

- Verificamos que el ID sea 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
	
	`$ sudo apt-key fingerprint 0EBFCD88`

- Agregando el repositorio stable

	* Para arquitecturas amd64:

		`$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`

	* Para arquitecturas armhf:

		`$ sudo add-apt-repository "deb [arch=armhf] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`

**Instalando Docker**

`$ sudo apt-get update`

`$ sudo apt-get install docker-ce`

***Nota:*** Para sistemas de producción en ocasiones hay que instalar una versión específica de Docker en lugar de instalar la última versión estable. Para ello instalamos docker especificándole la versión:

`$ sudo apt-get install docker-ce=<versión>`


**Instalando Docker en Debian**

Requisitos:
Debian Stretch (testing)
Debian Jessie 8.0 (LTS) / Raspbian Jessie
Debian Wheezy 7.7 (LTS)

***Nota:*** Al igual que en Ubuntu, docker corre tanto en las versiones tanto x86_64 y armhf de Debian.

- Requisitos extras para la instalación en Debian 	
	* Wheezy:

	 	Docker requiere una versión del kernel >=3.10, Debian Wheezy viene con Kernel 3.2, asi que chequeamos nuestra versión del kernel:
			
		`$ uname -r`
		
	... y si es necesario actualizamos el kernel.


**Instalando Docker CE en Debian**

Al igual que en Ubuntu, pordemos Instalar Docker desde el repositorio o bajarnos el paquete .deb y instalar todo a mano, esto es un poco molesto ya que tenemos que luego a la hora de actualizar tenemos que bajar nuevamente el .deb y upgradear de forma manual.

- Agregando el Repositorio:

	* Jessie or Stretch:

		`$ sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common`

	* Wheezy:

		`$ sudo apt-get install apt-transport-https ca-certificates curl python-software-properties`

- Agregando las Claves GPG oficiales de Docker:

	`$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -`

- Verificamos que el ID sea **9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88**

	`$ sudo apt-key fingerprint 0EBFCD88`

- Agregando el repositorio stable

	* Para arquitecturas amd64:
		`$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"`

	* Para arquitecturas armhf:

		`$ echo "deb [arch=armhf] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list`


***Nota solo para Wheezy:*** la versión de **add-apt-repository** para Wheezy nos agrega un repositorio deb-src en nuestro *sources.list* el cual no existe. Es necesario comentar esta línea o eliminarla de lo contrario **apt-get** update fallaría:
Buscamos la siguiente línea en */etc/apt/sources.list* y la comentamos o la eliminamos.
```
deb-src [arch=amd64] https://download.docker.com/linux/debian wheezy stable
```


**Intalando Docker CE en Debian**

`$ sudo apt-get update`

`$ sudo apt-get install docker-ce`


***Nota:*** al igual que para Ubuntu si queremos instalar una versión específica de Docker, primeramente la buscamos y luego le especificamos que versión queremos instalar.

`$ apt-cache madison docker-ce`

`$ sudo apt-get install docker-ce=<versión_STRING>`

**Instalando Docker en Raspbian**
Una ves que agregamos el repo de docker en /etc/apt/sources.list.d/ vamos a ver el archivo docker.list, y su contenido va a ser el siguiente:

```
deb [arch=armhf] https://apt.dockerproject.org/repo raspbian-jessie main
```

si el contenido de este archivo no es así, pues lo editamos y luego seguimos con las instalación de Docker.

`$ sudo apt-get update`

`$ sudo apt-get install docker`

***Nota:*** La versión de Docker para Raspbian es Docker CE, por lo que no es necesario especificarlo.

**Instalando Docker CE en Fedora**

Requisitos:
Fedora 24
Fedora 25

*Instalando desde el repositorio*

Primeramente instalamos el paquete **dnf-plugins-core** el cual nos permite configurar nuestros repositorios desde la línea de comandos.

`$ sudo dnf -y install dnf-plugins-core`

- Configurando el Repositorio estable:

	`$ sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo`

- Instalando Docker CE

	`$ sudo dnf makecache fast`

si las claves GPG conenciden con estas "060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35" aceptamos y continuamos con la instalación.

- Instalando Docker CE

	`$ sudo dnf install docker-ce`

Si queremos instalar una versión específica de Docker y no la última versión, podemos buscar e instalar la versión que necesitamos.

`$ dnf list docker-ce.x86_64  --showduplicates |sort -r`  	 
Nos mostrara una lista de las versiones disponibles...
```
docker-ce.x86_64  17.03.0.fc24                docker-ce-stable  
```
Luego instalamos a versión que 
nos acomode...

`$ sudo dnf -y install docker-ce-<versión>`


Hasta ahora se ha explicado la instalación de Docker en las tres distribuciones más usadas en nuestro país, luego agregaré más distros al tutorial, pero bueno lo que mas nos preocupa es el problema del bloqueo a la hora de hacer "pull" para bajar los containers. Primero estaremos probando métodos offline y luego pasaremos a la parte de instalar y configurar algunas herramientas que nos permitan evadir las restricciones que tenemos con https://hub.docker.com/*

