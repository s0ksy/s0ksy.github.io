---
layout: post
title: "Vacaciones"
author: "s0ksy"
---


## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
* `-p-` : Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
* `--open`: Indicamos que solo queremos que nos muestre los puertos abiertos
* `--min-rate 5000`: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
* `-sS`: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
* `-n`: Para que no se nos aplique resolución DNS (agilizar escaneo también)
* `-Pn`: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que tenemos el puerto 22 (ssh) y 80 (http) abiertos:
![[Pasted image 20240607121958.png]]

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
Y ya de primeras gracias al script que lanza al puerto 80 podemos ver "Site doesn't have a title", esto quiere decir que puede que no veamos nada en la página web de primeras.
![[Pasted image 20240607122033.png]]

Entramos en la página web y lo dicho, no vemos nada a priory
![[Pasted image 20240607122349.png]]

Como no vemos nada decido tirar de enumeración web con gobuster, pero sin éxito porque no encuentro ningún directorio interesante
![[Pasted image 20240607122452.png]]

Vamos a inspeccionar el código fuente de la página con `CTRL + U` y vemos un mensaje bastante interesante, donde Juan le deja un mensaje a Camilo diciendole que le ha dejado un correo importante. Este mensaje nos podría servir más adelante, así que lo tendré en mente.
![[Pasted image 20240607122513.png]]

## Explotación

Bueno, como teníamos el puerto 22 abierto y gracias al mensaje que hemos visto anterior mente podríamos probar de entrar por `SSH` haciendo fuerza bruta con "Hydra" a esos 2 usuarios (juan y camilo) para obtener la contraseña para entrar por `SSH`. 
Con el usuario juan no obtenemos éxito de encontrar la password, pero con el usuario `camilo` encontramos la contraseña: `password1`.
![[Pasted image 20240607123429.png]]

Entramos al `ssh` con el usuario `camilo` y la contraseña `password1`:
![[Pasted image 20240607123603.png]]

## Escalada de privilegios

Buscamos binarios que podamos usar como sudoer, pero resulta que `camilo` no tiene permisos de sudo... Por lo tanto paso a usar `find / -perm -4000 2>/dev/null` para ver si encuentro algún binario con permisos de SUID, pero no encuentro ninguno que pueda usar y que me sea interesante. 
![[Pasted image 20240611132354.png]]!
Tratamos un poco la TTY con:
- `export TERM=xterm`
- `export SHELL=bash`
- `bash`
Antes revisando el código fuente de la web hemos visto que le estaba hablando sobre un correo y en linux siempre que veamos algo relacionado con correos hay que ir a revisar el directorio `/var/mail/`
![[Pasted image 20240607124619.png]]

Y en este directorio encuentro que me puedo meter en otro directorio llamado `camilo` y ahí dentro encuentro un archivo de texto llamado `correo.txt` y al abrirlo veo que Juan le ha dejado un texto donde le da la contraseña de su usuario para que pueda terminar un trabajo.
La contraseña es : `2k84dicb`.
![[Pasted image 20240607124754.png]]

Y como tengo la contraseña de juan decido cambiar de usuario a juan poniendo la contraseña que nos ha dejado anteriormente.
![[Pasted image 20240607124905.png]]

Exploro un poco los directorios que tiene juan en su usuario y encuentro el directorio de `camilo`, el directorio de `juan` y el de un chico nuevo que no habíamos visto llamado `pedro`.
![[Pasted image 20240607125043.png]]

Decido meterme en el directorio de `pedro` porque es el que más curiosidad me da, buscamos binarios que podamos usar como sudoer con `sudo -l` y veo que podemos usar el binario `/usr/bin/ruby`.
(`sudo -l` lo podría haber hecho desde cualquier lugar situado en el usuario `juan`)
![[Pasted image 20240607125236.png]]

Buscamos en https://gtfobins.github.io/ como podemos explotar este binario y vemos que se puede hacer haciendo uso de: `sudo ruby -e 'exec "/bin/sh"'` y al ponerlo por la terminal entramos como superusuario root!!
![[Pasted image 20240607125325.png]]
