# Pràctica de IP Tunnels

# The Interconnection Problem
<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/escenari.png"/>

## Configuring IPIP tunnels

Per crear un tunnel entre R1 i R2:
```
R2#: ip tunnel add tunnel0 mode ipip local 198.51.100.2 remote 192.0.2.2 ttl 0 nopmtudisc dev eth2
R2#: ifconfig tunnel0 1.2.3.4 //s’ha d’assignar una @IP per a que funcioni
R2#: route add -net 192.168.0.0/24 dev tunnel0 //@privada
R1#: ip tunnel add tunnel0 mode ipip local 192.0.2.2 remote 198.51.100.2 ttl 0 nopmtudisc dev eth2
R1#: ifconfig tunnel0 1.2.3.4
R1#: route add -net 172.16.1.0/24 dev tunnel0
```

### Testing the tunnels (protocol and TTL)

Des del host2 fem un ping a 172.16.1.3 (host3). Veiem que arriba, per tant hem configurat bé el tunel. Si limitem al wireshark que no ens ensenyi els paquets fragmentats, llavors només veiem el echo-request i el echo-reply.

Si ens fixem en el TTL dels paquets, veiem com l'encapsulament IP fa que el paquet exterior si que vagi decrementant el seu TTL a mesura que passa pels diferents punts, però el paquet interior no decrementa el TTL.

## Comportament TTL
`host2$ ping -c1 -t1 172.16.1.3`
Veiem com el paquet es perd al primer salt, a R1. Només observem tràfic a SimNet0, i veiem un missatge ICMP de retorn que diu: TTL exceeded.
Source: 192.168.0.1 (R1)
Destination: 192.168.0.2 (host2)

`host2$ ping -c1 -t2 172.16.1.3`
El paquet no arriba a destí i es perd a RC amb IP 192.0.2.1. Ho podem comprovar a la SimNet1, on observem el missatge de TTL exceeded.

`host2$ ping -c1 -t3 172.168.1.3`
Finalment el paquet arriba al destí i rebem l'echo-reply.

## Testing the tunnels: nopmtudisc
El valor de l'MTU en els dos costats del tunel és de 1480B (1500 MTU eth - 20 Header)

- Primer borrem el tunel que haviem creat: `simctl iptunnel exec deltun`
- Configurem el MTU a SimNet2 de 996B: `R2$ ifconfig eth2 mtu 996` `RC$ ifconfig eth2 mtu 996`
- Restablim el tunel: `simctl iptunnel exec addtun_nopmtu`

Ara tenim la MTU de sortida del tunel = 996B.

## Fragmentation fields
`host2$ ping -c1 -s 500 -M want 172.16.1.3`
Podem veure 2 paquets ICMP (request i reply).
**L=542B a SimNet0** --> 500B (dades) + 20(IP Inner Header) + 14(Header Eth) + 8 (Header ICMP)
**L=562B a SimNet1** --> 500B (dades) + 20B (H.IP Inner) + 20B (**IP Outer Header** ) 14B + 8B

No hi ha fragmentació pq el tamany és inferior al "Path MTU".

`host2$ ping -c1 -s 500 -M dont 172.16.1.3` ==> **L'opció "-M dont" permet la fragmentació durant el camí**

`host2$ ping -c1 -s 500 -M do 172.16.1.3` ==> **L'opció "-M do" prohibeix la fragmentació, fins i tot la local**

## The -M want option
*Abans de cada ping nou, netagem la caché:* `HOST$ simctl iptunnel exec flushcache` 
Enviem ara el ping:
`host2$ ping -c1 -s 1000 -m want 172.16.1.3`

El ping no s’envia, podem veure un paquet ICMP echo-request amb DF=Set a SimNet0.
I a SimNet1 veiem 2 paquets ICMP (Request amb DF=Set tant en inner com outer i DU (Frag. Need.))
El dispositiu que envia aquest missatge és el RC i la destinació és l’entrada eth2 del R1.
El host2 hauria de ser el destí adequat del missatge anterior.
No s’està fent cap ICMP Relaying, però l’encapsulador fa Soft State.

`host2$ ping −c2 −s 1000 −M want −i 1 172.16.1.3` (Els dos pings separats per un interval d'1 segon, amb la opció **-i 1**)

El ping no arriba a destí. Podem veure tres missatges ICMP en SimNet0, dos echo-request (Tant un com l’altre amb DF=Set) i un missatge de DestinationUnreachable, fragmentation needed (Amb DF = Not Set)

En SimNet1 podem veure dos missatges ICMP, un echo-request (Amb DF=Set tant en capa inner com outer) i un missatge DU-FN (Amb DF = Set tant en capa inner com outer).

Podem veure un missatge de fragmentation-needed a SimNet0 que té @IPsrc= 192.168.0.1 i @IPdst=192.168.0.2 (De R1 a
host2). La MTU que notifica és = 976B.
L’R1 manté el soft state de la MTU del túnel ja que notifica a RC de que es necessita fragmentar.

`host2$ ping -c3 -s 1000 -M want -i1 172.16.1.3`
Ara sí que el ping funciona, ho veiem ja que rebem echo-reply a SimNet0.
A SN0: Primer echo-request DF=Set, segon echo-request DF=Set, tercer echo-request DF=Not Set i MF=Set. Deduïm que
el tercer paquet s’ha pogut fragmentar.
La fragmentació la fa el host2, l’emissor d’aquest paquet. La mida dels paquets és:
El primer fragment de 986B = 944B (dades) +20B(IP)+8B(ICMP)+14B(eth)
El fragment de 90B = 56B(Data)+20B(IP)+14B(eth)