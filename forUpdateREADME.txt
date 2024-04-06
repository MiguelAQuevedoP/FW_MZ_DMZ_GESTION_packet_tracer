DOCUMENTO PARA ACTUALIZAR EL README.md
***************************************************************************************************
Los siguiente comando deniegan la comunicacion entre 172 y 10.
Y permiten la comunicacion entre 10 y 172.

R1(config)#access-list 100 deny icmp 10.0.0.0 0.255.255.255 172.16.100.0 0.0.255.255
R1(config)#access-list 100 permit icmp 172.16.100.0 0.0.255.255 10.0.0.0 0.255.255.255 echo-reply
R1(config)#access-list 100 permit icmp 172.16.100.0 0.0.255.255 10.0.0.0 0.255.255.255 unreachable
R1(config)#access-list 100 permit tcp any any established
R1(config)#access-list 100 permit ip 172.16.100.0 0.0.255.255 203.0.113.0 0.0.0.255
R1(config)#access-list 100 permit ip 172.16.100.0 0.0.255.255 162.168.0.0 0.0.0.255
R1(config)#access-list 100 deny ip any any

R1(config)#int fa0/0
R1(config-if)#ip access-group 100 in

***************************************************************************************************
Los siguientes comandos son para que DMZ no tenga acceso a nada.
Y que internet tenga acceso a los servidores DMZ

R1(config)#access-list 101 permit icmp 162.168.0.0 0.0.0.255 any echo-reply
R1(config)#access-list 101 permit icmp 162.168.0.0 0.0.0.255 any unreachable
R1(config)#access-list 101 permit tcp any any established
R1(config)#access-list 101 permit tcp any 162.168.0.0 0.0.0.255 eq ftp
R1(config)#access-list 101 permit tcp any 162.168.0.0 0.0.0.255 eq www
R1(config)#access-list 101 deny ip 162.168.0.0 0.0.0.255 any
R1(config)#access-list 101 permit ip any any

R1(config)#int fa0/1
R1(config-if)#ip access-group 101 in

***************************************************************************************************





