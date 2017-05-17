**Uso basico de Docker**
```Crear, Renombrar, iniciar:
docker create:		crear un container sin iniciarlo
docker rename:		renombrar un container
docker run:		crea e inicia un container
docker rm:		eliminar un container
docker start:		iniciar un container
docker stop:		detener un container
docker restart:		reiniciar un container
docker pause:		pausar un container que esta ejecutandose en ese momento
docker unpause:		reanudar un container pausado
docker attach:		conectarse a un container en ejecucion
docker ps:		muestra los containers en ejecucion (con el parametro -a podemos ver todos los containers)
docker top:		muestra los procesos corriendo dentro de un container
docker cp:		copiar archivos y/o carpetar entre el container y el sistema host
docker export:		exportar un container
docker load:		cargar un container desde un archivo.tar.gz exportado previamente
docker exec:		ejecutar un comando en un container
docker images:		nos muestra todas las imagenes importadas en docker
docker import:		crear una imagen
docker build:		crear una imagen desde un Dockerfile
docker rmi:		eliminar una imagen
docker save:		savar una imagen a un archivo.tar.gz
docker pull:		hacer un pull de una imagen desde el repo de docker
docker push:		hacer un push de una imagen
```

...estos son algunos de los comandos basicos de docker, les recomiendo siempre hagan uso de nuestro mejor amigo "--help"...
