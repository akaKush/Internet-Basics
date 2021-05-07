# Pràctica Firewalls & NAT

![escenari](https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/escenari.png)

### Exercici 1

El primer exercici és per entendre els conceptes bàsics del filtrat de paquets (firewalls).

Iniciem amb `simctl fwnat start`, i configurem les màquines de les xarxes Net0, Net1 i Net2 amb els següents comandos:

- Per configurar les INTERFÍCIES: `simctl fwnat exec ifcfg`
- Per configurar les RUTES d'encaminament: `simctl fwnat exec routecfg`

Ara comprovem com han quedat configurades les màquines **Rbcn**, **www**, **Rint** i **host1**:

```
Rbcn:~# ifconfig
eth1      Link encap:Ethernet  HWaddr fe:fd:00:00:03:01
          inet addr:172.16.1.1  Bcast:172.16.1.255  Mask:255.255.255.0
          inet6 addr: fe80::fcfd:ff:fe00:301/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:19 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1288 (1.2 KiB)  TX bytes:338 (338.0 B)
          Interrupt:5

eth2      Link encap:Ethernet  HWaddr fe:fd:00:00:03:02
          inet addr:10.0.2.2  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::fcfd:ff:fe00:302/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:4 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:224 (224.0 B)  TX bytes:338 (338.0 B)
          Interrupt:5

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:660 (660.0 B)  TX bytes:660 (660.0 B)

Rbcn:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     172.16.1.3      255.255.255.0   UG    0      0        0 eth1
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth2
172.16.1.0      0.0.0.0         255.255.255.0   U     0      0        0 eth1
0.0.0.0         10.0.2.1        0.0.0.0         UG    0      0        0 eth2
```
**Rbcn**
Veiem com tenim 2 interfícies configurades, eth1 (172.16.1.1) i eth2 (10.0.2.2).
La taula de rutes ens indica com per arribar a les xarxes 192.168.1.0/24 (a través de 172.16.1.3) i 172.16.1.0/24 utilitzem la interfície eth1, i la eth2 per arribar a la 10.0.2.0/24 i a tota la resta (default, 0.0.0.0), aquesta última a través de 10.0.2.1.

```
www:~# ifconfig
eth1      Link encap:Ethernet  HWaddr fe:fd:00:00:04:01  
          inet addr:172.16.1.2  Bcast:172.16.1.255  Mask:255.255.255.0
          inet6 addr: fe80::fcfd:ff:fe00:401/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:13 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:964 (964.0 B)  TX bytes:468 (468.0 B)
          Interrupt:5 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:660 (660.0 B)  TX bytes:660 (660.0 B)

www:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.1.0      0.0.0.0         255.255.255.0   U     0      0        0 eth1
0.0.0.0         172.16.1.1      0.0.0.0         UG    0      0        0 eth1
```
**www**
només te la **eth1** configurada, i pot arribar a les xarxes 172.16.1.0 i a la default 0.0.0.0 a través de 172.16.1.1 mitjançant aquesta interfície.


```
Rint:~# ifconfig
eth1      Link encap:Ethernet  HWaddr fe:fd:00:00:06:01
          inet addr:172.16.1.3  Bcast:172.16.1.255  Mask:255.255.255.0
          inet6 addr: fe80::fcfd:ff:fe00:601/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:468 (468.0 B)  TX bytes:338 (338.0 B)
          Interrupt:5

eth2      Link encap:Ethernet  HWaddr fe:fd:00:00:06:02
          inet addr:192.168.1.1  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::fcfd:ff:fe00:602/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:608 (608.0 B)  TX bytes:338 (338.0 B)
          Interrupt:5

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:660 (660.0 B)  TX bytes:660 (660.0 B)

Rint:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth2
172.16.1.0      0.0.0.0         255.255.255.0   U     0      0        0 eth1
0.0.0.0         172.16.1.1      0.0.0.0         UG    0      0        0 eth1
```
**Rint**
Rint també té dues interfícies configurades, eth1 a la IP 172.16.1.3 i eth2 a 192.168.1.1
Aquest pot arribar a les xarxes 192.168.1.0/24 i 172.16.1.0 per defecte, i a la resta a través de 172.16.1.1 (eth1).

```
host1:~# ifconfig
eth1      Link encap:Ethernet  HWaddr fe:fd:00:00:07:01
          inet addr:192.168.1.7  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::fcfd:ff:fe00:701/64 Scope:Link
          UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1500  Metric:1
          RX packets:3 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:228 (228.0 B)  TX bytes:468 (468.0 B)
          Interrupt:5

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:660 (660.0 B)  TX bytes:660 (660.0 B)

host1:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth1
0.0.0.0         192.168.1.1     0.0.0.0         UG    0      0        0 eth1
```
**host1**
host1 té només eth1 configurada amb l'@IP 192.168.1.7

**Configure las tablas de filtrado de la máquina host1 de manera que no se permita ningún tipo de tráfico ICMP entrante a los procesos internos (locales) de dicha máquina.**

Per eliminar el traffic ICMP, afegim el següent comando a host1:
`host1# iptables -t filter -A INPUT -d 192.168.1.7 -p icmp -j DROP`

**(a) Si desde Rint se ejecuta un ping con destino host1 ¿se transmitirá el correspondiente mensaje ICMP echo-request por la red? ¿Es posible capturar el mensaje de respuesta ICMP echo-reply? Describa lo que ocurre en este caso.**

Posem wireshark a capturar a SimNet2 i fem un `ping -c 1 192.168.1.7`, i veiem com ens indica que NO ha arribat.
![ping rint host1 not founf](https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/Captura%20de%20Pantalla%202021-05-07%20a%20les%2015.11.46.png)

**(b) Si en lugar de enviar el ping desde Rint hacia host1, lo hacemos en sentido contrario (ping desde host1 hacia Rint) ¿se transmitirá el correspondiente mensaje ICMP echo-request? ¿y el echo-reply? Describa lo que ocurre en este caso.**

![ping host1 rint](https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/Captura%20de%20Pantalla%202021-05-07%20a%20les%2015.13.55.png)

Veiem com ara si que es transmet el echo reply a través de la xarxa, però tot i així des de terminal ens indica que el packet no ha arribat a host1 ja que aquest té el trafic icmp filtrat.

**2. Borre la configuración de filtrado anterior de host1 y vuelva a configurar sus tablas de filtrado para obtener el siguiente comportamiento:**
Primer eliminem la norma anterior: `iptables -t filter -D INPUT -d 192.168.1.7 -p icmp -j DROP`
Si ara provem un ping, aquest cop es transmet perfectament el ping amb els seus dos missatges corresponents capturats a la xarxa.

**(a) Desde host1 se debe poder realizar correctamente un ping a una máquina remota (Rint).**
**(b) host1 no debe responder a ninguna petición de ping externa.**
`host1# iptables -t filter -A INPUT -d 192.168.1.7 -p icmp --icmp-type echo-request -j DROP`
 D'aquesta manera afegim la norma que els missatges de tipus echo-request no arribin a host1.

**3. El problema de los esquemas de filtrado anteriores es que hay que configurar las tablas de filtrado en cada una de las máquinas, haciendo que la administración de la red sea compleja. La solución más utilizada es “confiar” la seguridad al router de la red, ya que todas las comunicaciones con el exterior fluyen a través de él y se puede aplicar un control centralizado a las mismas facilitando la administración. Cuando un router realiza funciones de filtrado o firewall se le conoce con el nombre de bastión de la red.**
**En este punto, usted tiene que configurar el router Rint como bastión para protejer a los hosts internos (host1). Para ello prepare el escenario realizando las siguientes tareas:**

- **Elimine las entradas de las tablas de filtrado de host1.**
- **Verifique que las redes Net1 y Net2 están correctamente configuradas (direcciones IP y tablas de encaminamiento) de forma que exista conectividad entre ellas a nivel IP.**

**En este momento, debe ser posible realizar con éxito un ping desde www o Rbcn a cualquiera de las máquinas de la Net2 y viceversa.**

Si ho comprovem enviant-nos pings entre elles arriben bé, podem comprovar la configuració a l'exercici 1.

**Ahora, añada las entradas necesarias en su bastión (Rint) para obtener el siguiente comportamiento:**

**(a) Un ping realizado desde una máquina externa a Net2 hacia una máquina perteneciente a Net2 no debe ser respondido, pero en el caso contrario, es decir un ping iniciado desde una máquina de Net2 hacia una máquina externa, sí que debe funcionar correctamente. Verifique el funcionamiento de este filtro.**


**(b) Si una máquina de una red externa a Net2, realiza un intento de conexión a un servicio TCP de un servidor alojado en Net2, este intento de conexión debe ser rechazado, pero en el caso contrario sí que debe funcionar correctamente. Verifique el funcionamiento de este filtro utilizando las máquinas de Net1 como red externa de pruebas.**

`rint# iptables -A FORWARD -d 192.168.1.0/24 -p tcp --tcp-flags ALL SYN -j DROP`

Ara intentem iniciar una sessió telnet desde www fins a host1 `www# telnet 192.168.1.7`

I veiem com NO es pot connectar, i només veiem els paquets a la NET1, a la NET2 no arriba cap packet tcp.

En canvi, si obrim la connexió telnet desde host1 cap a www SI que es pot connectar, i veiem els paquets a les dues xarxes.

**(c) Finalmente, filtre todo el tráfico UDP que entre o salga de Net2, excepto el tráfico UDP que vaya dirigido a un servidor DNS (que se supone externo a Net2)**

Volem filtrar tot el tràfic UDP excepte el dirigit a DNS. Sabem que DNS utilitza el port 53, per tant bloquejarem tot el tràfic udp entrant o sortint, excepte el que passi pel port 53.

Hem d'afegir 3 normes, una per descartar tràfic UDP, l'altre per acceptar el tràfic entrant a DNS i l'altre per acceptar el tràfic sortint de DNS:

`rint# iptables -A FORWARD -p udp --sport 53 -d 192.168.1.0/24 -j ACCEPT` --> "tots els paquets udp que el router Rint hagi de fer un FORWARD, amb port d'origen 53, i destí la xarxa Net2 seran acceptats"
`rint# iptables -A FORWARD -p udp --dport 53 -s 192.168.1.0/24 -j ACCEPT` --> "tots els paquets udp que el router Rint hagi de fer un FORWARD, amb origen la xarxa Net2, i destí un port 53 seran acceptats"
`rint# iptables -A FORWARD -p udp -j DROP` --> "tots els paquets udp que el router Rint hagi de fer FORWARD seran descartats"

*IMPORTANT: les normes afegides amb `iptable` es llegeixen en ordre, per tant si un paquet cumpleix la primera ja no passarà a les següents. *

Per comprovar-ho fem `dig 192.168.1.7`desde www per veure si el tràfic dns passa, i un `host1# nc -l -u` i `www# nc -u 192.168.1.7` per veure si ens podem connectar mitjançant netcat amb udp.


### Exercici 2

tornem a iniciar la simulació i executem els següents comandos:
```
simctl fwnat start
simctl fwnat exec ifcfg
simctl fwnat exec routecfg
simctl fwnat exec fwcfg
```

**1. Desde el host www de Net1, realice un ping a 10.0.4.2 (test) ¿funciona? ¿Es un problema de filtrado o de direccionamiento?**

No arriba. Comprovem la taula de rutes de www i veiem que el pas previ a enviar el paquet és la direcció IP 172.16.1.1, la qual correspon a la interfície eth1 del router "exterior" Rbcn. Comprovem la taula de rutes d'aquest router, i veiem que té diverses entrades, però la que hauria d'utilitzar per anar fins a **10.0.4.2** és l'entrada default, la qual diu que el gateway és el 10.0.2.1. Tot i tenir la ruta indicada, no arribem a test, ja que és una adreça pública, i nosaltres ho estem enviant des d'una adreça privada, per tant hem de afegir una regla de filtrat NAT perquè sàpiga com traduir les direccions per a encaminar bé el paquet entre xarxes públiques i privades.

**2. Para solucionar el problema anterior configure el router externo Rbcn para que realice SNAT para sus redes internas. Una vez configurado pruebe a realizar el ping a 10.0.4.2 ¿funciona ahora? Utilice las herramientas de análisis de tráfico que conoce para ver que está sucediendo en la red.**

A Rbcn afegim el següent:
`Rbcn# iptables -t nat -A POSTROUTING -o eth2 -j SNAT --to 10.0.2.2`

El qual indica a Rbcn que després de fer la decisió d'enrutament del paquet, tradueixi la direcció d'origen de la interfície eth2 a 10.0.2.2 i així el paquet sàpiga retornar.

![ping www test](https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/Captura%20de%20Pantalla%202021-05-07%20a%20les%2017.43.27.png)

Veiem com ara si que arriba bé el ping i també retorna correctament.

**3. La figura muestra el típico esquema de firewall con doble bastión (bastion externo –Rbcn– y bastión interno –Rint–), zona desmilitarizada (Net1) o DMZ (DeMilitarized Zone) para los servidores con acceso externo, y red interna (Net2). En este esquema las máquinas de la red interna pueden establecer conexiones a los servidores de la DMZ y a servidores externos (Internet), tal y como se ha configurado en el ejercicio anterior. En este esquema de doble bastión, se debe poder acceder a los servidores de la DMZ desde el exterior pero no a los hosts de la red interna.**
**Configure el bastión externo (Rbcn) para dar acceso al servidor web de www desde Internet y haga uso de la máquina externa (test) para verificar la configuración. Utilice las herramientas de análisis de tráfico que conoce para ver que está sucediendo en la red.**

Per donar accés al servidor web www desde Internet, configurem Rbcn amb la següent norma
`Rbcn# iptables -t nat -A PREROUTING -i eth2 -d 10.0.2.2 -j DNAT --to 172.16.1.2`

Això fa que el router Rbcn, abans de prendre una decisió d'enrutament amb destí a la interfície interna (-i) eth2, amb adreça IP 10.0.2.2, faci un DNAT i tradueixi l'adreça de destí a la 172.16.1.2, la qual sap arribar al servidor www, i podrà enrutar correctament el paquet.

