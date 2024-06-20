---
Nombre de la máquina: Fing
Sistema Operativo: Linux
Enlace de descarga: https://vulnyx.com/
---

## Escaneo de equipos de la red

Una vez descargada la maquina y levantada en nuestro virtualizador, la buscamos con la herramienta ARP-SCAN.
```
sudo arp-scan -I NUESTRA INTERFAZ --localnet
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/cc792507-e35e-4d23-a315-86612237ff7e)
![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/fc41b3b5-0f22-4a5a-b838-94199d068543)

Como vemos nos aparece con la IP 192.168.1.105, solo para confirmar con ping le lanzamos un paquete para determinar si hay conexion y su ttl.

## Nmap.

Una vez tenemos claro la IP objetivo, es momento de el escaneo con nmap.
```
sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.1.105
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/01ebda0f-42f4-4834-b556-48c121ee84bf)

