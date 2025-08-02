Estando autenticado como algun usuario. Para ello explotaremos `git hook`
### Qué es Git Hook?

Los Git Hooks son scripts que se ejecutan automáticamente cada vez que ocurre un evento específico en un repositorio de Git (Commit, Push).

Sabiendo esto, podemos crear un Git Hook que envie una reverse shell a mi máquina atacante cuando se realice una modificación en el repositorio.

![[Pasted image 20250802134719.png]]

En nuestra máquina atacante nos ponemos en escucha con netcat de la siguiente forma:

```
sudo nc -lvnp 443
```

Ahora modificamos algun archivo del repositorio, en este caso el `README.md`. Y hacemos un Commit.

![[Pasted image 20250802135142.png]]
![[Pasted image 20250802135328.png]]

Y ahora hemos obtenido un acceso no autorizado a la máquina victima

![[Pasted image 20250802135413.png]]