# Firewall-MZ-DMZ-GESTION-cisco
> En este proyecto se realiza una topologia en cisco packet tracer para
> configurar y analizar las zonas militarizadas, las zonas no militarizadas y
> las zonas de gestión.

Los requerimientos son los siguientes:
REQUERIMIENTOS - FIREWALL ROUTER FISICO: ZONAS 
[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)
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

## GESTION debe tener acceso a MZ, DMZ y a INTERNET (ping, ftp, web).

## MZ debe tener Acceso a DMZ y a INTERNET (ping, ftp, web), no a GESTION.

## DMZ NO debe tener acceso a NADA (ni siquiera ping).

## INTERNET debe tener Acceso a los Servidores Web y FTP ocultos en DMZ. 
