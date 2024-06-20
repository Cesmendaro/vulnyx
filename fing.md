---
Nombre de la máquina: Fing
Sistema Operativo: Linux
Enlace de descarga: https://vulnyx.com/
---

## Escaneo de equipos de la red.

Una vez descargada la maquina y levantada en nuestro virtualizador, la buscamos con la herramienta ARP-SCAN.
```
sudo arp-scan -I (NUESTRA INTERFAZ) --localnet
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/cc792507-e35e-4d23-a315-86612237ff7e)

Nos damos cuenta ya que luego de la MAC el escaneo con arp-scan nos dice "VMware", si aun no estamos seguros podemos tirarle un ping para verificar la conexion y su ttl.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/eb6f7ab9-0473-4fb1-9018-bec0e316b4b3)


## Nmap.

Una vez tenemos claro la IP objetivo, es momento de el escaneo con nmap.
```
sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.1.105
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/01ebda0f-42f4-4834-b556-48c121ee84bf)

Como vemos, el escaneo de puertos nos dice que hay 3 puertos abiertos, 22/ssh, 79/finger y 80/http. Como se constumbre, lo primero es revisar la aplicacion web que esta corriendo en el puerto  80.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/821b6b76-6410-484a-8bf4-d7f39b1440db)

Se trata de un Apache por defecto, por lo que vamos a buscar directios en caso de haber.

## Fuzzing.

```
gobuster dir -u http://192.168.1.105 -w (DICCIONARIO)
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/790ff582-ba3a-441d-aec7-5b2f88f3c782)
