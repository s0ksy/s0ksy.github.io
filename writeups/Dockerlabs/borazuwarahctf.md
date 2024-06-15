---
layout: post
title: "BorazuwarahCTF"
author: "s0ksy"
---
Estaremos resolviendo una máquina de Dockerlabs de dificultad muy fácil, donde nos encontraremos ante una imagen la cual habrá que hacer uso de exiftool para la extracción de sus metadatos y después hacer fuerza bruta para encontrar la contraseña para poder entrar por ssh.

## Reconocimiento

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
- `-p-` : Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
- `--open`: Indicamos que solo queremos que nos muestre los puertos abiertos
- `--min-rate 5000`: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
- `-sS`: Para que nos haga un escaneo de tipo SYN, también para agilizar el escaneo y que sea sigiloso
- `-n`: Para que no se nos aplique resolución DNS (agilizar escaneo también)
- `-Pn`: Para que no nos lance PING, también para hacer el escaneo más rápido

Podemos observar que tenemos el puerto 22 `ssh` y 80 `http` abiertos:
![ctf 1](/assets/images/ctf1.png)

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto).
![ctf 2](/assets/images/ctf2.png)

Como vemos que tiene el puerto 80 abierto procedemos a entrar a la página web para ver que contiene dentro y observamos que simplemente tiene una imagen de un huevo kinder sorpresa.
![ctf 3](/assets/images/ctf3.png)

Como solo he visto que había una imagen que se podía descargar en la web, he decidido descargarla y ver los `metadatos` que contiene. Gracias a la herramienta `exiftool` podemos ver los metadatos de la imagen y si nos fijamos bien, podemos observar que dentro tiene un user: `borazuwarah` y la password no nos la dan.
![ctf 4](/assets/images/ctf4.png)

## Explotación

Este user lo podemos usar para entrar vía `ssh`, pero como nos falta la password decido hacerle fuerza bruta con la herramienta `hydra` indicando el nombre de usuario y la wordlists de las contraseñas a probar. Finalmente obtenemos la contraseña: `123456`, una gran y difícil contraseña de adivinar.
![ctf 5](/assets/images/ctf5.png)

Como ya tenemos el user y la contraseña, accedemos vía `ssh`:
![ctf 6](/assets/images/ctf6.png)

Tratamos la TTY para que sea más funcional:
![ctf 7](/assets/images/ctf7.png)

## Escalada de privilegios

Y hacemos `sudo -l` para ver que comandos se pueden usar como sudoer y nos encuentra el binario `/bin/bash`
![ctf 8](/assets/images/ctf8.png)

Simplemente hacemos `sudo bash` y ya hemos obtenido superusuario root!!
![ctf 9](/assets/images/ctf9.png)
