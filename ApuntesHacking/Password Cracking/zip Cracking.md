Si hacemos esto nos descargara un archivo `.zip`, si lo descomprimimos veremos lo siguiente:

```
unzip password16.zip
```

Info:

```
Archive:  password16.zip
[password16.zip] password16.txt password: 
   skipping: password16.txt          incorrect password
```

Vemos que requiere de una contraseña, por lo que vamos a probar a `crackearla` con `john` de esta forma:

```
zip2john password16.zip > zip.hash
```

```
john --wordlist=<WORDLIST> zip.hash
```

Info:

```
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password1        (password16.zip/password16.txt)     
1g 0:00:00:00 DONE (2025-07-28 07:22) 50.00g/s 819200p/s 819200c/s 819200C/s 123456..cocoliso
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Veremos que nos ha `crackeado` la `password`, vamos a probarla en el archivo a ver si funciona.

```
unzip password16.zip
```

Info:

```
Archive:  password16.zip
[password16.zip] password16.txt password: 
  inflating: password16.txt
```