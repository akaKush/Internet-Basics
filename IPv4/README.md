# Pràctica 2 TCGI - IP version 4 (IPv4)

## ex1

En aquesta pràctica ens basarem en l'escenari de la següent figura:

![escenari](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2017.40.46.png)

En aquest escenari estudiarem com funcionen conjuntament el switching amb Ethernet, i el routing amb IP.

Iniciem la simulació ab `simctl switching-vlan start`.

Partim del **bloc d'adreces 192.168.100.0/24**, i li assignem una IP a **bob i frank**:

`root@bob# ifconfig eth0 192.168.100.4/24`
`root@frank# ifconfig eth0 192.168.100.5/24`

Ara provem de fer un ping desde bob a frank i mirem a la SimNet1 2 i 3 el què està passant amb el wireshark:

![ping bob - frank](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2017.57.53.png)

![wireshark SimNet1](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2017.58.57.png)

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

![escenari 2](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2019.25.30.png)

```
virt1# ifconfig eth1 192.168.0.32/24
virt2# ifconfig eth1 192.168.0.96/24
virt3# ifconfig eth1 192.168.0.144/24
virt4# ifconfig eth1 192.168.0.224/24
```

Si enviem un ping entre virt1 i virt2 veiem com arriba sense problemes i es guarden a la caché ARP les seves @MAC, i mirant al wireshark veiem com s'envien els 2 missatges ARP per indicar la MAC de destí a virt1, els echo request i echo reply i els altres 2 ARP per saber la MAC d'origen.


- Ara **introduim un mapeig erròni a virt1**:
`virt1# arp -d 192.168.0.96` (eliminem l'entrada de la taula ARP posterior)
`virt1# arp -s 192.168.0.96 00:70:48:29:5c:99 temp` afegim el mapeig erròni

- Tornem a enviar un ping i veiem què passa:
![2 pings, 8s](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2019.49.06.png)

En aquest cas hem enviat 2 pings separats 8 segons entre sí.
Veiem com el primer ping NO arriba a destí, però el segon sí. Això és degut a que el primer ping intenta enviar-lo a través del mapeig erròni que li hem posat prèviament, però s'adona que aquella @MAC no és la correcta, i llavors es reajusta automàticament enviant un missatge ARP a **BROADCAST**, i al segon ping ja sap on es troba l'adreça MAC correcta de virt2, la guarda ben mapejada i llavors ja el pot enviar sense problemes.

![wireshark 2 pings](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2019.52.03.png)

En aquest exercici ens plantegen la següent situació:
- virt1 i virt3 tenen màscara /24
- virt2 i virt4 tenen màscara /25
I ens pregunten quins hosts es podran comunicar entre ells?

virt1 i virt3 tenen accés a totes els hosts de la xarxa 192.168.0.0/24, per tant arribaran a cada un dels virtX.

Per altre banda, virt2 podrà arribar a virt1, perquè aquest es troba dins el seu rang d'adreces, és a dir entre 192.168.0.0 i 192.168.0.127 (192.168.0.0/25), però no podrà arribar ni a virt3 ni virt4, i el mateix ens passarà en el cas de virt4, el qual podrà arribar a virt3 però no a virt1 ni virt2.

## ex3
En aquest exercici tractarem d'establir connexió entre virt2 i virt4, tot i tenir /25 a les dues.

### 3.1
Per a fer això primer configurarem virt1 i virt3 com a Routers (`echo 1 > /proc/sys/net/ipv4/conf/all/forwarding`)

Ara afegim una nova ruta a virt4 per arribar a virt2 (a través de virt3, que actua de router de la seva xarxa):
`virt4# route add -net 192.168.0.0/25 gw 192.168.0.144`

I afegim una ruta a virt2 per arribar a virt4:
`virt2# route add -net 192.168.0.128/25 gw 192.168.0.32`

Si enviem pings entre un i l'altre veiem com arriben sense problemes.

### 3.2
Ara només utilitzarem virt1 com a router, virt3 ja no el necessitem.
Per fer això, necessitem configurar dues adreces a virt1, una per cada xarxa /25 i poder arribar bé a les dues:

`virt1# ifconfig eth1 192.168.0.32/25`aquesta és l'adreça normal
`virt1# ifconfig eth1:0 192.168-0.232/25` aquesta és l'adreça nova i particular del router1 per arribar a la segona subxarxa /25.

També desactivem el forwarding de virt3 per assegurar-nos que els paquets no s'envien per aquell router (`echo 0 > /proc...`)

Veiem com d'aquesta manera també arriben els pings sense problemes.

## ex4 ip-routing-abc

![escenari ex4](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2021.09.10.png)

Hem de configurar els hosts tal que es puguin enviar pings entre **alice bob i carla**.

Primer afegim els permisos de forwarding a cada router.

Després configurem cada un dels hosts amb les IPs que li pertany a cada interfície de xarxa.

```
alice# ifconfig eth0 10.0.0.40/25
bob# ifconfig eth0 10.0.0.240/26
carla# ifconfig eth0 10.0.0.140/26
r1# ifconfig eth0 10.0.0.31/25
r1# ifconfig eth2 10.0.0.131/26
r1# ifconfig eth1 192.168.0.1/30
r2# ifconfig eth0 192.168.0.2/30
r2# ifconfig eth1 10.0.0.202/26
```

Un cop tenim totes les interfícies configurades, afegim les rutes necessàries a cada host i router per a fer l'enviament dels paquets correctament:

```
alice# route add -net 10.0.0.128/25 gw 10.0.0.31
bob# route add -net 10.0.0.0/25 gw 10.0.0.202
bob# route add -net 10.0.0.128/26 gw 10.0.0.202
carla# route add -net 10.0.0.0/25 gw 10.0.0.131
carla# route add -net 10.0.0.192/26 gw 10.0.0.131
r1# route add -net 10.0.0.192/26 gw 192.168.0.2
r2# route add -net 10.0.0.0/25 gw 192.168.0.1
r2# route add -net 10.0.0.128/26 gw 192.168.0.1
```
Amb aquesta configuració veiem com ens podem enviar pings entre tots els hosts.

## ex5 ACME Intranet
![escenari ex5](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-13%20a%20les%2021.40.40.png)

En aquest exerici configurarem la intranet d'una empresa fictícia. A la figura anterior veiem la topologia de la xarxa, on tenim 3 departaments: Marketing, Sales i Production.

Cada departament té el seu host router, i la 10.0.0.0/24 interconnecta els routers entre ells (**backbone**).

Iniciem l'escenari amb `simctl ip-routing start`.

*Obs: No cal configurar ni el router3 ni el host3.*

Veiem que si agafessim una xarxa /23 no separariem els diferents departaments, ja que pertanyerien a la mateixa xarxa.
Per tant configurem els diferents departaments amb /24.

Per a poder comunicar-nos entre els departaments, necessitem fer la configuració necessària tant als hosts de cada departament que es volen comunicar entre sí, com als routers intermitjos, que puguin fer forwarding dels paquets que els hi envien, i afegir a cada un dels dispositius les rutes per poder enrutar els paquets cap a on toca.

La configuració final queda així:
```
host1# ifconfig eth1 192.168.0.254/24
host1# route add -net default gw 192.168.0.1 ##posem default quan TOTS els paquets s'han d'enrutar per allà.

router1# ifconfig eth1 10.0.0.1/24
router1# ifconfig eth2 192.168.0.1/24
router1# route add -net 192.168.1.0/24 gw 10.0.0.2

host2# ifconfig eth1 192.168.1.254/24
host2# route add -net default gw 192.168.1.1

router2# ifconfig eth1 10.0.0.2/24
router2# ifconfig eth2 192.168.1.1/24
router2# route add -net 192.168.0.0/24 gw 10.0.0.1
```


Hi ha altre configuracions possibles per enviar el ping d'un host a un altre, especificant el camí que ha de seguir el ping des de l'origen fins al destí.

Si afegim la opció `ping -r @IP_1 @IP_2 ... @IP_N` podem especificar quins hops ha de fer el ping per arribar al destí.


### ex5.6 NAT
En aquest apartat volem comunicar-nos amb adreces **wwww**, les quals estan a Internet, i les seves adreces són dins el rang 203.0.113.0/24, i per tant desde Internet només pot arribar a adreces dins el seu rang.

Llavors **com ho fem per connectar-nos a un rang d'adreces que no pertanyen a la nostra xarxa?**

Necessitem utilitzar el **NAT** (Network Address Tranlation). El NAT permet intercanviar datagrames entre adreces públiques i privades. A una pràctica més endevant ho explicarem més a fons, però per fer-ho necessitem routers que estiguin habilitats per fer NAT, i seran els encarregats de que quan un host envïi algun paquet cap a internet, traduir l'adreça de destí de la xarxa privada cap a l'adreça corresponent a la xarxa pública d'Internet, i viceversa.

Per **configurar el NAT a un router** ho fem de la següent manera:
```
r4# iptables -t nat -F ##Netegem la taula NAT
r4# iptables -t nat -A POSTROUTING -s 192.168.0.254 -o eth2 -j SNAT --to-source 203.0.113.5  ##Afegim una norma tq els datagrames creats amb adreça d'origen 192.168.0.254, sigui traduida a l'adreça 203.0.113.5 en el moment d'enrutar-los, i viceversa quan els datagrames van al revés
r4# ifconfig eth2:0 203.0.113.5/24  ##Habilitem r4 per rebre trafic de 203.0.113.5
```

Si ara executem **`lynx`**, podem veure des de terminal com si estiguessim utilitzant un navegador web, i visualitzar el contingut de la web allotjada a la IP 203.0.113.5. És útil per comprovar connexions, ja que no carrega imatges ni fitxers multimedia.

## ex6 FRAGMENTACIÓ

En aquest exercici practiquem la fragmentació de datagrames IP, en l'escenari de la següent figura:
![escenari ex6](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2014.43.21.png)

Veiem com la MTU de cada router és diferent. La MTU d'una interfície ethernet pot ser reduida per sota 1500 amb el comando `ifconfig`.

Configurem els routers amb els següents paràmetres:
![routers](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2014.45.44.png)


I els hosts així:
`host1# ifconfig eth1 192.168.1.3/24 mtu 1500`
`host2# ifconfig eth1 192.168.3.3/24 mtu 560`


Finalment configurem les rutes de cada dispositiu per seguir un sentit anti-horari en l'enviament de paquets.

Fem un ping de host1 a host2 amb la opció -R per veure per quins hops passa per arribar al destí:
![ping route 1-2](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.04.19.png)

i veiem que passa per host1 --> router2 --> router3 --> host2 --> router1 --> host1.

Si enviem de host2 a host1 veiem els salts:
![ping route 2-1](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.06.28.png)

I veiem com ara envia de host2 --> router1 --> host1 --> router2 --> router3 --> host2

Si analitzem al wireshark, veiem com els paquets icmp tenen una mida de 84 bytes, i per tant no cal fragmentar-los, i si ens fixem en els FLAGS, veiem com **DF=0, MF=0 i FO=0**
![84 bytes + Flags](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.13.14.png)

Ara anem a enviar paquets d'una mida més gran per veure com es comporten.

`host1# ip neigh flush cache`
`host1# ping -c2 -s 900 192.168.3.3`

![frag needed terminal](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.18.21.png)

![frag needed wireshark](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.20.40.png)

Veiem com a l'enviar els 2 pings, el primer es troba que no pot passar per router3 perquè té una mtu de 560, i envia un missatge de **Destination Unreachable (Fragmentation Needed)** a l'origen. Llavors el segon ping s'envia fragmentat des de l'origen.

Si ens fixem en els flags, veiem:
- El primer `echo request` s'envia amb el flag de DF=1, per a no fragmentar-se.
- Llavors el següent ping es fragmenta en dues parts a l'origen;
  - La primera part de l'`echo request` té el flag MF=1 ja que indica que venen més fragments
  - La segona no el té activat ja que és l'últim, però porta un Offset de FO=67 ((556-20)/8), el qual és la divisió del tamany del paquet entre 8.
- Finalment rebem l'`echo reply` fragmentat igual que l'anterior, i amb els mateixos flags corresponents a cada fragment.

**B. FRAGMENTATION by ROUTERS**
Ara provem d'enviar un ping de host1 a host2 amb payload 900 bytes, però amb DF=0, és a dir utilitzant la opció `-M dont`.
`ping -c 1 192.168.3.3 -M dont`

Analitzem a les 3 SimNets i veiem què passa:
![SimNet0](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.48.33.png)

![SimNet1](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.49.30.png)

![SimNet2](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.49.45.png)

- A SimNet0 veiem com el primer paquet (`echo request`) s'envia sense problemes ja que passa per router2 amb mtu 1500. Els següents missatges que veiem són els ARP que envia el router1 per saber on està host1 i poder retornar l'`echo reply`, però el que veiem a continuació són 2 paquets fragmentats, els quals poden haver estat fragmentats al router1 o al router3. Analitzem les altres xarxes per comprovar-ho.
- A SimNet1 simplement veiem com hi passa el `echo request`, però el reply no ja que s'envia per una altre xarxa. El request s'envia sense problemes, tot i no trobar resposta.
- Finalment a SimNet2 veiem com després que el router3 envii els paquets ARP de reconeixement, ja s'envien 2 paquets fragmentats, els quals s'han fragmentat al router3 abans d'enviar-los ja que en la interfície de sortida cap a la SimNet2 té una mtu de 560, i per tant el paquet de 900 bytes que hem enviat desde host1, s'ha fragmentat en 2 paquets de 536 i 364 bytes de payload (més les capçaleres (IP=20bytes, ICMP=8 bytes)). Finalment host2 respón amb el `echo reply` corresponent amb els 2 fragments directament separats.

![SimNet2 - inspeccionem](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.59.46.png)

Notem que si enviessim un ping de payload 1200bytes el router que faria la fragmentació seria el router2, ja que a l'hora d'enviar paquest cap a SimNet1 té una mtu de 1000.

