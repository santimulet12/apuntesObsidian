Objetivo: Conocer la versión de la DB de MySQL y Microsoft.

Consulta Original: `SELECT * FROM articles WHERE category='Accessories'`

![[Pasted image 20250718161533.png]]

 Para manipular la consulta con UNION debemos saber cuantas columnas tiene la tabla.
`' ORDER BY 1-- -` -> Toma los valores de la primera columna
`' ORDER BY 3-- -` -> Toma los valores de las columnas 1, 2 y 3
De manera que cuando nos excedamos de la cantidad de columnas, lanzará un error.

![[Pasted image 20250718161916.png]]

Al realizar un ordenamiento respecto a la 3 columna, vemos que lanza un error, por lo tanto la tabla en uso cuenta con menos de 3 columnas.

![[Pasted image 20250718162048.png]]

La tabla en uso cuenta con dos columnas ya que hemos realizado un `ORDER BY 2-- -`  y la web no ha lanzado ningún error. Ahora procederemos con la consulta maliciosa.

Consulta Modificada: `SELECT * FROM articles WHERE category='Accessories' UNION SELECT @@version,"abc"-- -'`

![[Pasted image 20250718163705.png]]
Core de la consulta modificada: `' UNION SELECT @@version,"abc"-- -`

En caso de tener problemas, deberías probar realizar las consultas con BurpSuite y UrlEncodear la consulta.
