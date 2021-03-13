# Pràctica 2 TCGI - IP version 4 (IPv4)

## ex1

En aquesta pràctica ens basarem en l'escenari de la següent figura:

![escenari](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.05.46.png)

En aquest escenari estudiarem com funcionen conjuntament el switching amb Ethernet, i el routing amb IP.

Iniciem la simulació ab `simctl switching-vlan start`.

Partim del **bloc d'adreces 192.168.100.0/24**, i li assignem una IP a **bob i frank**:

`root@bob# ifconfig eth0 192.168.100.4/24`
`root@frank# ifconfig eth0 192.168.100.5/24`

Ara provem de fer un ping desde bob a frank i mirem a la SimNet1 2 i 3 el què està passant amb el wireshark:

![ping bob - frank](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.05.46.png)

![wireshark SimNet1](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.05.46.png)

A totes 3 xarxes veiem els mateixos missatges, 2 ARP per indicar on estan l'adrecça MAC de la IP amb la que ens volem comunicar, 2 ICMPs corresponents a `echo request` i `echo reply` i finalment 2 ARPs més per fer la mateixa operació de trobar la MAC corresponent al host d'origen i retornar el paquet.

A més, si ara fem `arp -n` a qualsevol dels dos hosts amb els que ens hem enviat els pings, veurem com tenen guardada la @MAC de l'altre host:


### ex1.2
- convertim L1 en un router **(FORWARDING)**:
  1. Eliminem el BRIDGE (switch) a L1: 
  `ifconfig br1 down`
  `brctl delbr br1`
  2. Fem que el sistema Linux L1 actui com a router:
  `echo 1 > /proc/sys/net/ipv4/ip_forward`
  *Nota: Si volem desactivar-lo utilitzem un **0** com a paràmetre.*

Veiem com ara hem separat la xarxa de la figura en 3 DLL diferents.

Com que ara L1 és un router **li hem d'assignar una adreça IP a cada una de les interfícies de L1**:

```
L1# ifconfig eth0 192.168.100.1  #gw per arribar a la xarxa 192.168.100.0/24
L1# ifconfig eth1 192.168.101.1 #gw per arribar a la xarxa 192.168.101.0/24
L1# ifconfig eth2 192.168.102.1 #gw per arribar a la xarxa 192.168.102.0/24
```

Reconfigurem Alice, Bob i Frank amb les IPs corresponents a cada una de les seves xarxes:

```
alice# ifconfig 192.168.100.2/24
bob# ifconfig 192.168.102.3/24
frank# ifconfig 192.168.101.5/24
```

Ara necessitem definir entrades de les rutes que van cap als hosts de cada DLL a cada una de les interfícies:
- Definim la ruta per enviar el tràfic cap a qualsevol de les adreces de la xarxa 192.168.100.0 (**alice**):
  `L1# route add -net 192.168.100.0/24 gw 192.168.100.1`
- Definim la ruta per enviar el tràfic cap a qualsevol de les adreces de la xarxa 192.168.101.0 (**frank**):
  `L1# route add -net 192.168.101.0/24 gw 192.168.101.1`
- Definim la ruta per enviar el tràfic cap a qualsevol de les adreces de la xarxa 192.168.100.0 (**bob**):
  `L1# route add -net 192.168.102.0/24 gw 192.168.102.1`

- Si volem comprovar la taula de rutes: `route -n`

Amb aquestes configuracions podem veure com ens podem enviar correctament pings entre Alice i Bob i també entre Bob i Frank, però si volguessim enviar-nos entre Alice i Frank hauriem d'afegir les entrades a les seves taules de rutes per trobar-se entre ells.

Si volguessim poder-nos enviar pings entre alice i frank, faltaria afegir el forwarding per saber arribar fins a la xarxa d'alice des de Frank, i també el forwarding per saber arribar fins a Frank desde Alice:
```
alice# route add -net 192.168.101.0/24 gw 192.168.100.1
frank# route add -net 192.168.100.0/24 gw 192.168.101.1
```

----

## Ex2 IP Subnetting

En aquest exercici treballem el concepte de **subnetting**. és a dir la divisió entre diferents xarxes de diferents tamanys, i com es poden comunicar entre elles.

Primer necessitem configurar l'escenari segons la següent figura:

![escenari 2](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.05.46.png)
