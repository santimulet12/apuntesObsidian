
```
chmod 600 id_rsa
```

![[Pasted image 20250704231749.png]]

![[Pasted image 20250704231912.png]]

Vemos que la clave id_rsa está protegida por contraseña, ya que al intentar conectarnos por ssh con ella, nos pide la "key" del certificado.
Ahora intentaremos descifrar la key haciendo un ataque de fuerza bruta con la herramienta `RSAcrack` y el diccionario `rockyou.txt`.

![[Pasted image 20250704235032.png]]

Vemos que nuestro ataque de fuerza bruta ha sido exitoso, la key es "unicorn". Ahora nos conectaremos por ssh proporcionando la `id_rsa`.

```
ssh -i id_rsa USER@IP
```

![[Pasted image 20250704235324.png]]



