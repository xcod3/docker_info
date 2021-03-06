# Configuracion de proxy para Docker:

`$ sudo mkdir -p /etc/systemd/system/docker.service.d`

`$ sudo touch /etc/systemd/system/docker.service.d/http-proxy.conf	`

> esto habilita la variable HTTP_PROXY, para HTTPS_PROXY creariamos el archivo https-proxy.conf
...luego agregamos las siguientes lineas en **/etc/systemd/system/docker.service.d/http-proxy.conf**

`$ cat /etc/systemd/system/docker.service.d/http-proxy.conf`

```
[Service]
Environment="HTTP_PROXY=http://proxy_host:port/"
```

> Reiniciamos los procesos:

`$ sudo systemctl daemon-reload && sudo systemctl restart docker`

> Verificamos que la variable HTTP_PROXY esta configurada correctamente:

`$ systemctl show --property=Environment docker`
```
Environment=HTTP_PROXY=http://proxy_host:port/
```
