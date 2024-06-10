## RECON

Lanzamos un escaneo con nmap básico para poder ver que puertos tenemos abiertos
- -p- : Sirve para indicarle que quieres hacer un escaneo sobre todos los puertos (1-65535) de la ip 
- --open: Indicamos que solo queremos que nos muestre los puertos abiertos
- --min-rate 5000: Con este parámetro le indicamos que queremos que envíe paquetes no más lentos de 5000 paquetes por segundo (para agilizar el escaneo)
- -n: Para que no se nos aplique resolución DNS (agilizar escaneo también)
- -Pn: Para que no nos lance PING, también para hacer el escaneo más rápido

Y encontramos el puerto 21 (ftp) abierto:
![[Pasted image 20240606230926.png]]

Ahora lanzaremos un escaneo más específico a los puertos que hemos encontrado abiertos usando el parámetro -sCV que es una abreviación de -sC (lanza los scripts que vienen por default en nmap a los puertos indicados para poder encontrar información de utilidad de manera más sencilla) y de -sV (indica la versión del servicio que corre en ese puerto). Encontramos la versión del `ftp` que esta corriendo en ese puerto, pero me parece raro que no tengamos acceso por Anonymous
![[Pasted image 20240606232259.png]]

Por lo tanto, buscamos la versión del servicio en searchsploit para ver si se puede vulnerar eso de alguna forma y encontramos 2 formas de hacerlo (ejecutando el script del exploit manualmente con python o de forma automática con metasploit)
![[Pasted image 20240606232945.png]]
## Explotación

Decido hacerlo con metasploit
![[Pasted image 20240606233256.png]]

Y veo que solo nos basta con configurar el HOST de la víctima, algo muy sencillo
![[Pasted image 20240606233319.png]]

Lo configuramos como toca:
![[Pasted image 20240606233346.png]]

Le damos a run para ejecutar el script y tendriamos acceso ya como root!!
![[Pasted image 20240606233608.png]]
