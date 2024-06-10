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
![[Pasted image 20240606234745.png]]

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
![[Pasted image 20240606234840.png]]

Como es un servicio HTTP nos vamos a google y buscamos la ip y obtenemos esto de aquí, se ve que podemos subir archivos y código que funcione de tipo `.php` ya que la url nos da una pista `/upload.php` , pero vamos a ver que más podemos encontrar por aquí dentro
![[Pasted image 20240606235033.png]]

Para ello hacemos fuzzing haciendo uso de gobuster y encontramos el directorio `/uploads` allí podría ser donde se almacena todo lo que subamos en la página web
![[Pasted image 20240607001132.png]]

Subo una imagen de prueba y veo que me deja subirla
![[Pasted image 20240606235816.png]]

Me voy a `/uploads` para ver si se almacena y así es:
![[Pasted image 20240607001333.png]]

Podemos abrir lo que subamos desde `/uploads` 
![[Pasted image 20240607001352.png]]

## Explotación

Como antes en la URL hemos visto que se podía ejecutar código de tipo .php vamos a probar a colar una reverse shell
![[Pasted image 20240607003802.png]]

La subimos a la web...
![[Pasted image 20240607003855.png]]
![[Pasted image 20240607003916.png]]

Nos ponemos en escucha para ver si conseguimos obtener acceso a la máquina...
![[Pasted image 20240607004002.png]]

Pero... Nos deniega la conexión... :(. Habrá que probar de otra forma
![[Pasted image 20240607004043.png]]

Probamos a subir un código para ver si tiene ejecución de comandos remota
![[Pasted image 20240607004841.png]]
![[Pasted image 20240607004926.png]]
![[Pasted image 20240607004944.png]]

Y... Como no sale nada quiere decir que ha funcionado y vamos a probar a ejecutar comandos desde la URL
![[Pasted image 20240607005001.png]]

Después del /cmd.php ponemos `?cmd=<comando de terminal>` en este caso probamos con `id` para ver que nos devuelve, y funciona perfectamente.
![[Pasted image 20240607005129.png]]

Por lo tanto vamos a probar a colar una reverse shell de bash ya que estamos ejecutando comandos desde la supuesta terminal
![[Pasted image 20240607005439.png]]

Personalmente voy a codificar la reverse shell en URL para que no me de problemas
![[Pasted image 20240607005527.png]]

Me pongo de nuevo en escucha desde mi máquina atacante y vemos que obtenemos acceso a la máquina víctima
![[Pasted image 20240607005605.png]]

Antes de hacer lo de la imagen de abajo he hecho varias cosas para hacer la TTY operativa:
`script /dev/null -c bash`
`CTRL + Z`
`stty raw --echo;fg`
`reset xterm`
![[Pasted image 20240607010046.png]]

## Escalada de privilegios

Buscamos comandos que podamos usar como sudoer y encontramos el binario `/usr/bin/env` y como hemos visto en las máquinas anteriores esto es un binario que nos puede ayudar en la escalada de privilegios
![[Pasted image 20240607010624.png]]

Lanzamos `sudo env /bin/sh` haciendo uso de ese binario y... OBTENEMOS SUPERUSUARIO ROOT!!
![[Pasted image 20240607010720.png]]
