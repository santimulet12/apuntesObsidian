
Objetivo: Conseguir la versión de la DB de Oracle.

Consulta Original: `SELECT * FROM articles WHERE category='Accessories'`

![[Pasted image 20250717133042.png]]

Las consultas de tipo UNION se utilizan cuando quieres combinar dos o más consultas SELECT y mostrarlas en una misma salida. Para manipular la consulta con UNION debemos saber cuantas columnas tiene la tabla.
`' ORDER BY 1-- -` -> Toma los valores de la primera columna
`' ORDER BY 3-- -` -> Toma los valores de las columnas 1, 2 y 3
De manera que cuando nos excedamos de la cantidad de columnas, lanzará un error.

![[Pasted image 20250717134049.png]]

Vemos que al colocar un `' ORDER BY 5-- -` lanza un error, lo que significa que la cantidad de columnas de la tabla es menos a 5.

![[Pasted image 20250717134320.png]]

Ahora vemos que al colocar un `' ORDER BY 2-- -` no lanza ningún error, por lo tanto la tabla cuenta con dos columnas.
Ahora, como ya sabemos la cantidad de columnas que tiene la DB, podemos proceder con la consulta maliciosa, para averiguar la versión de la DB de **ORACLE** en este caso.

Consulta modificada: `SELECT * FROM articles WHERE category='Accessories' UNION SELECT banner, 'NULL' FROM v$version-- -'`

Core de la consulta: `' UNION SELECT banner, 'NULL' FROM v$version-- -`
`banner` Hace referencia a la version.
`'NULL'` Es el otro valor que debe recibir la consulta ya que la tabla tiene dos columnas.

![[Pasted image 20250717141840.png]]

Vemos que la versión es **CORE 11.2.0.2.0 Production**.