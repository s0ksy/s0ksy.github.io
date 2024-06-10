---
layout: post
title: "Upload"
author: "s0ksy"
---


## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
* `-p-`: Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
* `--open`: Indicamos que solo queremos que nos muestre los puertos abiertos
* `--min-rate 5000`: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
* `-sS`: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
* `-n`: Para que no se nos aplique resolución DNS (agilizar escaneo también)
* `-Pn`: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que tenemos el puerto 80 (http) abierto:
![upload1](/assets/images/upload1.png)

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
![upload2](/assets/images/upload2.png)

Como es un servicio HTTP nos vamos a google y buscamos la ip y obtenemos esto de aquí, se ve que podemos subir archivos y código que funcione de tipo `.php` ya que la url nos da una pista `/upload.php` , pero vamos a ver que más podemos encontrar por aquí dentro
![upload3](/assets/images/upload3.png)

Para ello hacemos fuzzing haciendo uso de gobuster y encontramos el directorio `/uploads` allí podría ser donde se almacena todo lo que subamos en la página web
![upload6](/assets/images/upload6.png)

Subo una imagen de prueba y veo que me deja subirla
![upload7](/assets/images/upload7.png)

Me voy a `/uploads` para ver si se almacena y así es:
![upload8](/assets/images/upload8.png)

Podemos abrir lo que subamos desde `/uploads` 
![upload9](/assets/images/upload9.png)

## Explotación

Como antes en la URL hemos visto que se podía ejecutar código de tipo .php vamos a probar a colar una reverse shell
![upload10](/assets/images/upload10.png)

La subimos a la web...
![upload11](/assets/images/upload11.png)
![upload12](/assets/images/upload12.png)

Nos ponemos en escucha para ver si conseguimos obtener acceso a la máquina...
![upload13](/assets/images/upload13.png)

Pero... Nos deniega la conexión... :(. Habrá que probar de otra forma
![upload14](/assets/images/upload14.png)

Probamos a subir un código para ver si tiene ejecución de comandos remota
![upload15](/assets/images/upload15.png)
![upload16](/assets/images/upload16.png)
![upload17](/assets/images/upload17.png)

Y... Como no sale nada quiere decir que ha funcionado y vamos a probar a ejecutar comandos desde la URL
![upload18](/assets/images/upload18.png)

Después del /cmd.php ponemos `?cmd=<comando de terminal>` en este caso probamos con `id` para ver que nos devuelve, y funciona perfectamente.
![upload19](/assets/images/upload19.png)

Por lo tanto vamos a probar a colar una reverse shell de bash ya que estamos ejecutando comandos desde la supuesta terminal
![upload20](/assets/images/upload20.png)

Personalmente voy a codificar la reverse shell en URL para que no me de problemas
![upload21](/assets/images/upload21.png)

Me pongo de nuevo en escucha desde mi máquina atacante y vemos que obtenemos acceso a la máquina víctima
![upload22](/assets/images/upload22.png)

Antes de hacer lo de la imagen de abajo he hecho varias cosas para hacer la TTY operativa:
`script /dev/null -c bash`
`CTRL + Z`
`stty raw --echo;fg`
`reset xterm`
![upload23](/assets/images/upload23.png)

## Escalada de privilegios

Buscamos comandos que podamos usar como sudoer y encontramos el binario `/usr/bin/env` y como hemos visto en las máquinas anteriores esto es un binario que nos puede ayudar en la escalada de privilegios
![upload24](/assets/images/upload24.png)

Lanzamos `sudo env /bin/sh` haciendo uso de ese binario y... OBTENEMOS SUPERUSUARIO ROOT!!
![upload25](/assets/images/upload25.png)
