# Firewall-MZ-DMZ-GESTION-cisco
> En este proyecto se realiza una topologia en cisco packet tracer para
> configurar y analizar las zonas militarizadas, las zonas no militarizadas y
> las zonas de gestión.

Los requerimientos son los siguientes:
REQUERIMIENTOS - FIREWALL ROUTER FISICO: ZONAS 
## Cada estudiante debe utilizar DIRECCIONAMIENTO IP distinto.

| ZONA | Direccion IP | Mascara |
| ------ | ------ | ------ |
| MZ | 172.16.100.1 | 255.255.0.0 |
| DMZ | 162.168.0.1 | 255.255.255.0 |
| GESTION | 10.0.0.1 | 255.0.0.0 |
| INTERNET | 203.0.113.1 | 255.255.255.0 |

## Las Zonas con IPs Privadas deben permanecer ocultas al mundo de INTERNET.
Primero se configura el RIP en la red
Para R1
```sh
~Enable
~conf t
R1(config)#router rip
R1(config)#version 2
R1(config)#network 169.250.0.0
```
Para R2
```sh
~Enable
~conf t
R2(config)#router rip
R2(config)#version 2
R2(config)#network 169.250.0.0
R2(config)#network 250.0.113.0
```
### _NAT para DMZ_
Se asigna a cada dirección privada una dirección pública.
En router 1:
```sh
~Enable
~conf t
R1(config)#ip nat inside source static 162.168.0.10 169.250.0.10
R1(config)#ip nat inside source static 162.168.0.20 169.250.0.20
```
Y se asigna la entrada y la salida.
En router 1 para la entrada:
```sh
~Enable
~conf t
R1(config)#interface fa0/1
R1(config-if)#ip nat inside
```
Se sale de la interfaz fa0/1
```sh
R1(config-if)#exit
```
Y en el mismo router se configura la salida:
```sh
~Enable
~conf t
R1(config)#interface serial0/0/0
R1(config-if)#ip nat outside
```
> Si se quiere verificar se utiliza `show running-config` y se buscan las lineas `ip nat inside source static 162.168.0.10 169.250.0.10 & ip nat inside source static 162.168.0.20 169.250.0.20`

### _NAT PARA MZ_
Se asigna a muchas dirección privadas una dirección pública.
En router 1:
```sh
~Enable
~conf t
R1(config)#access-list 1 permit 172.16.100.0 0.0.255.255
R1(config)#ip nat inside source list 1 interface serial0/0/0 overload
```
Y se asigna la entrada y la salida.
En router 1 para la entrada:
```sh
~Enable
~conf t
R1(config)#interface fa0/1
R1(config-if)#ip nat inside
```
Se sale de la interfaz fa0/1
```sh
R1(config-if)#exit
```
Y en el mismo router se configura la salida:
```sh
~Enable
~conf t
R1(config)#interface serial0/0/0
R1(config-if)#ip nat outside
```
> Si se quiere verificar se utiliza `show running-config` y se buscan la linea `access-list 1 permit 172.16.0.0 0.0.255.255` y `ip nat inside source list 1 interface Serial0/0/0 overload`

### _NAT PARA GESTION_
Se asigna a muchas dirección privadas una dirección pública.
En router 1:
```sh
~Enable
~conf t
R1(config)#access-list 2 permit 10.0.0.0 0.255.255.255
R1(config)#ip nat inside source list 2 interface serial0/0/0 overload
```
Y se asigna la entrada y la salida.
En router 1 para la entrada:
```sh
~Enable
~conf t
R1(config)#int fa1/0
R1(config-if)#ip nat inside
```
Se sale de la interfaz fa0/1
```sh
R1(config-if)#exit
```
Y en el mismo router se configura la salida:
```sh
~Enable
~conf t
R1(config)#int se0/0/0
R1(config-if)#ip nat outside
R1(config-if)#exit
```
> Si se quiere verificar se utiliza `show running-config` y se buscan la linea `ip nat inside source list 2 interface Serial0/0/0 overload` y `access-list 2 permit 10.0.0.0 0.255.255.255`

## GESTION debe ser el unico que tiene Acceso al FW mediante SSH.
### GENERAR SSH 
Primero se debe introducir las instrucciones para generar un clave para SSH.
En router 1:
```sh
~Enable
~conf t
R1(config)#int fa1/0
R1(config-if)# ip address 10.0.0.1 255.255.255.0
R1(config-if)#exit
```
Ahora generas el nombre del dominio, en este caso _gestion.com_
```sh
R1(config)#ip domain-name gestion.com
```
Se genera la clave rsa, y se escribe el numero de bits de longitud que se quiere,
en este caso es de 1024 bits.
```sh
R1(config)#crypto key generate rsa
The name for the keys will be: R1.gestion.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
```
Se entra la line vty 0 4 y se configura:
```sh
R1(config)#line vty 0 4
R1(config-line)#transport input ssh
R1(config-line)#login local
R1(config-line)#exit
```
Se crea un usuario para entrar mediante ssh, el comando es _`username <usrname> privilege 15 password <usrpassword>`_.
En este caso ambos son _`admin`_
```sh
R1(config)#username admin privilege 15 password admin
```
Se establece como secreto y se encripta la constraseña.
```sh
R1(config)#enable secret admin
R1(config)#service pass
R1(config)#service password-encryption
```
Para observar si está habilitado el SSH se aplica el comando _`do show ip ssh`_ y se observa _SSH Enabled - version_
```sh
R1(config)#do show ip ssh
SSH Enabled - version 1.99
Authentication timeout: 120 secs; Authentication retries: 3
```
### Permitir el accesso solo de GESTION a R1 mediante SSH
Luego de haber generado el SSH se realizan las reglas para que el acceso a SSH del FW(R1) solo lo tenga GESTION.
Se vuelve a entrar a la line vty 0 4 y se configuran las reglas:
```sh
R1(config)#line vty 0 4
R1(config-line)#access-class 10 in
R1(config-line)#access-list 10 permit 10.0.0.0 0.255.255.255
R1(config)#access-list 10 deny any
```
> NOTA: Por alguna razón solo deja conectar al `PC1` y no al `PC2` mediante SSH a `R1`.

## MZ debe tener Acceso a DMZ y a INTERNET (ping, ftp, web), no a GESTION.
Primero se restringirá el acceso de MZ, despues se ejecutarán las reglas para el proximo punto.
se crea una lista de acceso en este caso es la número _`100`_, donde se deniega el acceso del protocolo ICMP.
```sh
R1(config)#access-list 100 deny icmp 10.0.0.0 0.255.255.255 172.16.100.0 0.0.255.255
```
Ahora en la misma lista de acceso se permite la respuesta del ICMP y UNREACHABLE.
```sh
R1(config)#access-list 100 permit icmp 172.16.100.0 0.0.255.255 10.0.0.0 0.255.255.255 echo-reply
R1(config)#access-list 100 permit icmp 172.16.100.0 0.0.255.255 10.0.0.0 0.255.255.255 unreachable
```
Se establece la comunicación del protocolo TCP para cualquiera:
```sh
R1(config)#access-list 100 permit tcp any any established
```
Este punto se combinó con el siguiente punto así que proceda al siguiente punto
## GESTION debe tener acceso a MZ, DMZ y a INTERNET (ping, ftp, web).
Para la misma regla anterior se define la comunicación de todos los protocolos desde la red MZ a DMZ e INTERNET, como ya está bloqueado el acceso de MZ a GESTION estas reglas no afectan los comandos anteriores:
```sh
R1(config)#access-list 100 permit ip 172.16.100.0 0.0.255.255 203.0.113.0 0.0.0.255
R1(config)#access-list 100 permit ip 172.16.100.0 0.0.255.255 162.168.0.0 0.0.0.255
R1(config)#access-list 100 deny ip any any
```
Para completar la regla _`100`_ se aplica la interfaz fa0/0 como trafico en la entrada.
```sh
R1(config)#int fa0/0
R1(config-if)#ip access-group 100 in
```
## DMZ NO debe tener acceso a NADA (ni siquiera ping).
## INTERNET debe tener Acceso a los Servidores Web y FTP ocultos en DMZ. 
Estos dos puntos se unieron en la misma regla número _`101`_.
Primero se configuró la respuesta del protocolo ICMP:
```sh
R1(config)#access-list 101 permit icmp 162.168.0.0 0.0.0.255 any echo-reply
R1(config)#access-list 101 permit icmp 162.168.0.0 0.0.0.255 any unreachable
```
Luego se permitió el protocolo TCP para las conexiones establecidas, para el protocolo FTP y para la web.
```sh
R1(config)#access-list 101 permit tcp any any established
R1(config)#access-list 101 permit tcp any 162.168.0.0 0.0.0.255 eq ftp
R1(config)#access-list 101 permit tcp any 162.168.0.0 0.0.0.255 eq www
```
Ahora se denegaron todos los protocolos desde DMZ al resto de las redes y se permitieron todos protocolos para todas las redes.
```sh
R1(config)#access-list 101 deny ip 162.168.0.0 0.0.0.255 any
R1(config)#access-list 101 permit ip any any
```
Para terminar de configurar la regla _`101`_ se le aplica a la interfaz fa0/1 en la entrada.
```sh
R1(config)#int fa0/1
R1(config-if)#ip access-group 101 in
```
