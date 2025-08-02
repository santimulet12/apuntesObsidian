![[Pasted image 20250728211635.png]]

![[Pasted image 20250728212450.png]]
![[Pasted image 20250728212509.png]]

La función de JavaScript que se está utilizando por detrás es la siguiente:

![[Pasted image 20250728212417.png]]

Lo que podemos hacer es cerrar la etiqueta de `<img src="...>` de la siguiente forma `">`
e inyectar la etiqueta `<script>`.

``` JavaScript
"><script>alert("XSS")</script>
```

![[Pasted image 20250728212949.png]]
![[Pasted image 20250728213047.png]]
![[Pasted image 20250728214514.png]]