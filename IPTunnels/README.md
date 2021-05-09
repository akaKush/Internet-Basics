# IP Tunnels Theory

- [IP Tunnels Theory](#ip-tunnels-theory)
- [The Interconnection Problem](#the-interconnection-problem)
- [The Tunneling Solution](#the-tunneling-solution)
- [IP Tunnels](#ip-tunnels)
- [2021](#2021)
  - [Pràctica](#pràctica)
  - [CONFIGURACIÓ de TUNELS IPIP](#configuració-de-tunels-ipip)
  - [TESTING DELS TUNNELS: camps de PROTOCOL i TTL](#testing-dels-tunnels-camps-de-protocol-i-ttl)
  - [TESTING NOPMTUDISC (fragmentation)](#testing-nopmtudisc-fragmentation)
    - [Fragmentation FIELDS and options](#fragmentation-fields-and-options)
  - [-M want](#-m-want)
  - [-M do](#-m-do)
  - [-M dont](#-m-dont)
  - [PMTUDISC](#pmtudisc)
  - [-M want](#-m-want-1)
  - [-M do](#-m-do-1)
  - [-M dont](#-m-dont-1)
  - [Testing tunnels: TCP and MSS](#testing-tunnels-tcp-and-mss)
  - [Netcat and TCP](#netcat-and-tcp)
  - [Netcat to transfer files](#netcat-to-transfer-files)
  - [TCP, tunnels and filtering](#tcp-tunnels-and-filtering)


---

# The Interconnection Problem
<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/problem.png"/>

Des d'una xarxa pública NO podem enrutar a les xarxes privades directament. Això produeix pèrdues de paquets als routers públics.

# The Tunneling Solution

<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/solution.png"/>

Els tunels s'utilitzen per transportar un altre protocol de xarxa encapsulant els seus paquets.

Es poden veure com "Source Routes" que no els hi importen els mecanismes de routing tradicionals.

Quan el tunel es construeix a la capa IP, s'anomena **IP tunnel**.

- L'adreça IP "exterior" (pública) del router, identifica els "endpoints" del tunel.


# IP Tunnels

Qualsevol paquet IP (incloent la informació de les adreces) és encapsulada a dins d'un altre paquet, el qual té el format natiu de la xarxa pública.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/tunnel.png"/>

**Mètode d'encapsulació de paquets**:
Quan un "border router" ha d'enviar paquets a través d'un tunel IP, aquest crea un nou "pseudo-dispositiu de xarxa", associat a la @IP exterior.

- Quan un paquet **s'envia** a través d'aquest "pseudo-dispositiu", és encapsulat en un paquet IP i enviat a l'altre "border-router".
- Quan un paquet és **rebut** a través d'aquest pseudo-dispositiu, el payload s'extreu del paquet IP, i es tracta com un paquet real.

*Obs: Els "pseudo-dispositius" només accepten paquets IP de altres "border-routers".*

La **taula de rutes** configurada per saber quins paquets ha d'enviar a través del tunel IP és tal que:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/table.png"/>

Llavors, qualsevol paquet que s'envii a `192.168.2.0/24`, s'enviarà a través del "pseudo-dispositiu" `tunnel0`, encapsulat a dins d'un altre paquet IP, i aquest s'enviarà a l'altre "border-router".


----

# 2021

Un tunnel és un canal de comunicació entre dues xarxes. Un tunnel IP és un tunnel de comunicació mitjançant el protocol IP.

Existeixen varies tècniques per aplicar els tunnels, com VPNs, multicast, IPv6...

En el tunneling IP s'encapsulen els paquets, tal que, cada paquet IP, incloent les adreces d'origen i destí, està encapsulat dins un altre paquet de format natiu el qual es pot utilitzar per transmetrel a la xarxa.

Tenim diversos tipus de tunnels, cada un a la seva capa de protocol corresponent, p.ex: IPIP, GRE, IPSec al nivell de xarxa, SSL o TLS al nivell de transport, o SSH a nivell d'aplicació.

En aquesta pràctica utilitzarem el més simple, IPIP (també se li diu ipencap o encap).
Utilitzant IPIP tant els paquets enviats com els que els encapsulen són paquets IP.

Per fer-nos una idea de la sintaxis dels components que té un IPIP tunnel:

```
<< ... In the most general tunneling case we have
    
    source ---> encapsulator --------> decapsulator ---> destination
    
with the source, encapsulator, decapsulator, and destination being separate nodes. 
The encapsulator node is considered the "entry point" of the tunnel, and the decapsulator node is considered the "exit point" of the tunnel ... >>
```

Per encapsular un paquet dins un altre, es necessita una capçalera externa, en aquesta capçalera **l'adreça d'origen és l'encapsulador** (el punt d'entrada al tunnel) i **l'adreça de destí és el desencapsulador**, mentres que en el paquet intern, juntament amb la seva capçalera queden intactes.

En els exercicis següents veurem els problemes típics del IPIP Tunneling:
- network error messages treatment (`ICMP Relaying` and `sof state`)
- fragmentation management in the encapsulator/decapsulator
- problems when using TCP inside tunnels


*Nota: Per entendre bé la pràctica cal tenir un coneixement elevat dels **diferents camps de la capçalera IP**, especialment els relacionats amb la fragmentació de paquets (consultar [RFC 791](https://tools.ietf.org/html/rfc791)), també saber com funcionen les **connexions TCP** i entendre completament el concepte de **MSS** (consultar [RFC 793](https://tools.ietf.org/html/rfc793) i [RFC 879](https://tools.ietf.org/html/rfc879)). Coneixement sobre **regles de filtrat** (`iptables`) i **netcat** (`nc`) també són necessaris.*


---

## Pràctica

![escenari](https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/escenari.png)

Treballarem sobre l'anterior escenari, on tindrem una connexió d'una xarxa IP remota i privada, la qual utilitza una xarxa pública intermediaria per connectar-se amb una altre xarxa privada.

- SimNet0 i SimNet3 són les xarxes privades, representen una certa branca d'un negoci.
- Connectarem aquestes branques a través seu mitjançant tunnels IP.
- SimNet1 i SimNet2 representen l'internet.

Assignació de @IP:

**Branques**
- SimNet3 --> 172.16.1.0/24 (private addressing)
- SimNet0 --> 192.168.0.0/24 (private addressing)

Els routers **R1** i **R2** utilitzen SNAT per permetre als seus usuaris interns accedir a hosts externs.
Al principi de la pràctica NO utilitzarem cap regla de filtrat, però més endevant inclourem alguna `iptables` per filtrar el trafic.

**Internet**
- SimNet1 --> 192.0.2.0/24 (public adderssing)
- SimNet2 --> 198.51.100.0/24 (public addressing)

Suposem que **només tenim un router per l'Internet, RC**. El problema d'aquest escenari és que no podem enrutar packets IP directament a través d'Internet perquè tenim private addressing.
Per això configurarem un tunnel IPIP entre els "edge routers" per poder enrutar els paquets IP a través d'Internet.

<br><br>

---
<br><br>

Iniciem amb `simctl iptunnel start` i comprovem que tinguem les adreces especificades anteriorment ben configurades.
<br><br>

## CONFIGURACIÓ de TUNELS IPIP

<br>
Per configurar els tunels IPIP hem d'executar alguns comandos als border routers (encapsulador i desencapsulador).

A cada border router s'ha de:

1. Configurar la interfície del tunel mitjançant el comando `ip tunnel` (consultar el manual d'aquest comando per saber-ne més).

Configurarem els tunels en mode **ipip**, i amb la opció **nopmtudisc** activada per evitar el Path MTU Discovery.

A R2 executem:<br>
`R2$ ip tunnel add tunnel0 mode ipip local 198.51.100.2 remote 192.0.2.2 ttl 0 nopmtudisc dev eth2`

A R1:<br>
`R1$ ip tunnel add tunnel0 mode ipip local 192.0.2.2 remote 198.51.100.2 ttl 0 nopmtudisc dev eth2`

2. Activar la nova interfície virtual del tunel. Per fer-ho també li hem d'assignar una nova IP amb ifconfig.

`R2$ ifconfig tunnel0 1.2.3.4`

`R1$ ifconfig tunnel0 1.2.3.4`

3. Actualitzar la taula de rutes per assegurar-nos que els paquets que ens interessen siguin enrutats pel tunel. Per exemple en el nostre cas volem que els paquets que arribin a R2 amb destí la xarxa `192.168.0.0/24` utilitzin el tunnel0:

`R2$ route add -net 192.168.0.0/24 dev tunnel0`

`R1$ route add -net 172.16.1.0/24 dev tunnel0`

<br><br>

## TESTING DELS TUNNELS: camps de PROTOCOL i TTL

<br><br>

Per comprovar que els tunnels funcionen bé, obrim 4 wiresharks, un a cada xarxa, i **desactivem** la següent opció a cada un: `Edit-Preferences-Protocols-IPv4-Reassemble fragmented IPv4 datagrams`.

Ara enviem un ping desde **host2** a **host3**, i veiem que aquest arriba sense problemes (NO EL VEIG A SIMNET3, PQ???)

Un cop enviat anem a comprovar que estigui ben encapsulat, per fer-ho hauriem de mirar-ho a les xarxes SimNet1 i SimNet2, ja que són les dues que pertanyen al tunnel:

![paquet amb 2 headers SimNet1](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2014.58.45.png)

Veiem com tenim 2 headers, el primer que indica com arribar d'una punta a l'altre del tunnel (192.0.2.2 (R1) --> 198.51.100.2 (R2)), i l'altre desde l'origen i fins al destí que hem indicat al ping (192.168.0.2 (host2) --> 172.16.1.3 (host3)).

Si ens fixem en el valor del PROTOCOL, veiem com a la capçalera exterior ens indica IPIP, mentres que en la interior ICMP, corresponent al ping.

(A SimNet2 veiem el mateix).

**Per comprovar la diferència entre la mida dels paquets, mirem a SimNet0, ja que així trobem el paquet sense estar encapsulat i els podem comparar**:

![mida paquet simnet0](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2015.05.26.png)

![mida paquet simnet1](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2015.07.40.png)

Veiem com a dins el tunel, el paquet ha augmentat 20bytes, els quals són exactament el tamany de la capçalera IP exterior.

Finalment fixem-nos en que tenim el **Flag Don't Fragment** activat, i ara mirem els diferents TTL que tenim tant al paquet interior com a l'exterior al llarg de les diferents xarxes.

![TTL](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2017.58.58.png)

*La capçalera exterior utilitza el TTL de la interior. Durant el tunnel només es decrementa el TTL de la exterior, el de la interior queda congelat.*

**Exercici 4** Reiniciem les captures dels wiresharks i fem:
`host2$ ping -c 1 -t 1 172.16.1.3` (ping de host2 a host3 amb un TTL=1).

Analitzant el wireshark veiem que el paquet no passa de la SimNet0, i rebem un missatge TTL exceeded que ens l'envia R1.

![TTL exceeded R1](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2016.28.32.png)

Si ara posem TTL=2, el que envia el missatge de TTL exceeded aquest cop és RC (192.0.2.1):

![TTL exceeded RC](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2016.28.41.png)

Si finalment fem el mateix test amb TTL=3, aquest arriba al destí i rebem echo-reply, però és interessant veure els diferents TTL que trobem:

![TTL SimNet2](https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/Captura%20de%20Pantalla%202021-05-08%20a%20les%2016.30.47.png)

Veiem com tenim per el outer header un TTL de 2 salts, el qual fa el primer entre R1 i RC, i el segon entre RC i R2.

En canvi, al camí de tornada (`echo reply`) trobem com aquest té un TTL=64 (default).

<br><br>

## TESTING NOPMTUDISC (fragmentation)

Amb el comando ifconfig podem veure la MTU de cada dispositiu. Tenim 1480bytes a cada cantó del tunnel.

Aquesta MTU ve de 1500B (MTU eth) - 20B (IP Header).

Eliminem el tunnel amb `simctl iptunnel exec deltun`

Configurem un valor MTU a SimNet2 de 996B.
- `R2$ ifconfig eth2 mtu 996`
- `RC$ ifconfig eth2 mtu 996`

Restablim el tunnel0: `simctl iptunnel exec addtun_nopmtu`.

Ara ja tenim una MTU als dos costats del tunnel de 976B (996 - 20).

### Fragmentation FIELDS and options

Fem un ping de host2 a hos3: `ping -c1 -s 500 -M want 172.16.1.3`
**L'opció "-M want" permet la fragmentació LOCAL adaptant-se al camí.**
En el mode **-M want**, al principi **sempre tenim DF=set** (No fragmenta).

Com que el tamany del ping és inferior a la MTU, podem veure el echo request i reply tant en SimNet0 com en SimNet1.

Veiem que a SimNet0 el paquet és de 542B (500B dades + 20B IPHeader + 14B EthHeader + 8B ICMPHeader), mentres que en SimNet1 és de 562B, ja que tenim una capçalera IP extra.




`host2$ ping -c 1 -s 500 -M dont 172.16.1.3`
**L'opció "-M dont" PERMET (NO PROHIBEIX) la fragmentació durant tot el camí.**

`host2$ ping -c1 -s 500 -M do 172.16.1.3`
**L'opció "-M do" PROHIBEIX la fragmentació, fins i tot la local.**

En aquests tres casos no hi ha hagut diferència ja que la MTU sempre era més gran que la mida dels paquets.

## -M want
Netegem la caché de la taula de rutes: `simctl iptunnel exec flush cache`, o amb `ip route flush cache` a cada un dels hosts que volguem eliminar-la.

- Executem `host2$ ping -c1 -s 1000 -M want 172.16.1.3`.

Veiem com el ping no s'envia, podem veure un missatge echo-request amb DF=Set a la SimNet0.

A SimNet1 veiem 2 paquets ICMP, un ICMP request amb DF=Set tant en la inner header com outer, i seguidament un altre paquet que va dirigit desde RC a R1 (NO a host2), amb DU (Fragmentation Needed).

- Netegem caché i tornem a enviar ping `host2$ ping -c2 -s 1000 -M want -i 1 172.16.1.3` (amb -i 1 li indiquem que s'esperi un interval de 1 segon abans de enviar el segon ping).

Veiem com el ping NO arriba al destí.

A SimNet0 veiem 3 missatges ICMP, dos echo-request (amb el flag DF=Set), i un últim missatge de Destination Unreachable, fragmentation needed (amb DF=Not Set).

A SimNet1 tenim 2 ICMP, un echo-request amb DF=Set als dos headers, i un DU-FN.

A SimNet0 també podem veure un que el missatge de fragmentation needed va dirigit de R1 a host2, i que indica una MTU de 976B.

- `host2$ ping -c 3 -M want -s 1000 -i 1 172.16.1.3`

Finalment veiem com SÍ que funciona el ping, ja que rebem un echo-reply a SimNet0.

**SN0:** Primer echo-request amb DF=Set, segon echo-request amb DF=Set, tercer echo-request amb DF=Not Set i MF=Set. Veiem com el 3r paquet s'ha pogut fragmentar a l'origen (host2), amb 2 paquets, els quals tenen el primer 986B (944B dades + 20 IP_H + 8 ICMP_H + 14B Eth_H), i el segon 90B (56B dades + 20 + 14).

**SN1**: A SimNet1 hi veiem 4 fragments, corresponents a 2 paquets:
- echo request: 1r fragment de 1010B (948B dades + 8B ICMP + 20 IP + 20 IPIP + 14 Eth) i 2n fragment de 86B (56B de dades + 20B IP + 14 eth)
- echo reply: 1r fragment de 1006B (944 + 8 + 20 + 20 + 14) i 2n de 110B (56 + 20 + 20 + 14).

Veiem que el echo request té 1010+86 (1096B), i en canvi el echo reply té 1006 + 110 (1116B). **Això es produeix perquè a la tornada PRIMER posa la capçalera IP i després fragmenta, i en canvi a l'anada fragmentem i després incloem la capçalera IP**

**SN3:** Només hi veiem UN echo-request (el que s'ha enviat últim) i un echo reply corresponent. Els dos tenen mida 1042B (992B + 16B ICMP + 20B IP + 14B eth).


Netegem cache de nou i veiem l'últim exemple amb la opció -M want: `host2$ ping -c 2 -s 1460 -M want -i 1 172.16.1.3`
Ara directament veiem un missatge de R1 que indica **fragmentation needed**, i per tant el host2 fragmenta abans d'enviar els paquets, i així quan arriben a R1 ja no ho han de fer.

## -M do

`host2$ ping -c3 -s 1000 -M do -i 1 172.16.1.3`

NO funciona cap ping. A SN0 podem veure dos paquets ICMP echo-request però no n'hi ha cap de fragmentat a causa del -M do.
La diferència amb la opció -M want, és que el tercer paquet es fragmentava a l'origen perquè el host2 sabia que s'havia de fragmentar un cop rebut el Fragmentation Needed, però l'opció **-M do prohibeix la fragmentació**.

## -M dont

`host2$ ping -c3 -s 1000 -M dont -i1 172.16.1.3`

Veiem com TOTS 3 pings funcionen correctament, sense cap missatge d'error, ja que la opció **-M dont oblida a tenir el flag DF = Not Set**, per permetre la fragmentació.
La millor opció és fer ús de -M want ja que s'adapta a les capacitats del sistema. En canvi, l'opció -M do és massa restrictiva, i la -M dont massa poc, perquè aquesta segona pot portar problemes a la xarxa.


## PMTUDISC
El mode **pmtudisc** del tunnel provoca que l'encapsulador (R1) NO accepti fragmentació (DF=Set).

```
simctl iptunnel exec deltun
simctl iptunnel exec addtun_pmtu
ip tunnel show
```

## -M want
`host2$ ping -c 3 -s 1000 -M want -i 1 172.16.1.3`
+
Veiem com el ping no funciona, a SN0 veiem 3 echo-request i en rebre el DU-FN el host2 fragmenta el tercer paquet. Tot i això si ara mirem a SN1 veurem el primer paquet i el missatge DU-FN, però no veiem la fragmentació del 3r, perquè R1 no ha pogut fragmentar degut a la opció **pmtudisc**.

## -M do
`host2$ ping -c 3 -s 1000 -M do -i1 172.16.1.3`
Només veiem paquets a la SN0, ja que com que no podem fragmentar a l'origen degut a la opció -M do no passem ni del R1.

## -M dont
`host2$ ping -c3 -s 1000 -M dont -i1 172.16.1.3`
El ping segueix sense funcionar, ja que el tunnel no te permès fragmentar per la opció **pmtudisc**, i tot i tenir el flag DF=Not Set des del primer echo request, aquests no han passat a través del tunel, ja que R1 els hauria d'haver fragmentat.

## Testing tunnels: TCP and MSS

MSS is the **Maximum Size Segment**, és la mida màxima que pot tenir cada segment TCP quan s'envien streams de segments.

## Netcat and TCP
```
host3$ nc -l -p 12345
host2$ nc 172.16.1.3 12345
```
Podem veure el clàssic handshake (client-server: SYN, SYN-ACK, ACK) i com ens marca un MSS de 1460B.

## Netcat to transfer files
```
host3$ nc -l -p 12345 -q 0 < /etc/services
host2$ nc 172.16.1.3 12345
```

A SN3 veiem 2 paquets de DU-FN, ja que en aquell punt la MTU és de 976, i per tant R2 ha hagut d'avisar d'això a host2 per a tenir la fragmentació ben feta i poder retransmetre bé els paquets.

## TCP, tunnels and filtering

`R1$ iptables -A INPUT -p icmp -j DROP`

Tornem a executar els comandos de netcat anteriors. 
Hi ha handshake sense problemes. Veiem missatges ICMP, no deixem que entrin a R1, però si que surtin.
El fitxer no es transmet correctament, ja que R1 no s'entera que hi ha una MTU menor limitant.

Si a la regla de iptables hi posem FORWARD en comptes de INPUT, el fitxer es retransmetria correctament, ja que detectaria que ha d'enviar el missatge a l'altre borde del tunel, i per tant és com si NO li estiguessin enviant el missatge a ell (INPUT), sinó que ha de fer forwarding.

La MSS només conta els bytes de dades dels segments, les capçaleres no.