# Pràctica Firewalls & NAT

- [Pràctica Firewalls & NAT](#pràctica-firewalls--nat)
  - [Exercici 1](#exercici-1)
    - [1.](#1)
    - [2.](#2)
    - [3.](#3)
  - [Exercici 2](#exercici-2)
  - [1.](#1-1)
  - [2.](#2-1)
  - [3.](#3-1)

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/escenari.png"/>

## Exercici 1

- Iniciem l'escenari de fwnat: `host$ simctl fwnat start`
- Carreguem la configuració inicial de les interfícies: `host$ simctl fwnat exec ifcfg`
- Carreguem la configuració inical de les rutes: `host$ simctl fwnat exec routecfg`

### 1.

**Bloquejar tot tipus de tràfic ICMP entrant**

`host1# iptables -t filter -A INPUT -d 192.168.1.7 -p icmp -j DROP`
Host1 - Rint ==> Un echo-request des de Rint es transmetrà per la xarxa, ja que el firewall no atura l'enviament de pings, però el host1 NO acceptarà missatges d'aquest tipus degut a la configuració anterior, per tant rep el echo-request, però no envia cap echo-reply.

Rint - Host1 ==> Quan enviem un echo-request des de Host1 a Rint, s'envia correctament, ja que només hem bloquejat els missatges entrants, i no els sortints. L'echo-reply des de Rint s'enviarà i es podrà capturar a la SimNet2, però no arribarà al host1.

### 2.

**Host1 ha de poder realitzar un ping a tothom, però no n'ha de respondre a cap**

Primer de tot eliminem la norma anterior:
`host1# iptables -t filter -D INPUT -d 192.168.1.7 -p icmp -j DROP`

Llavors configurem la nova norma:
`host1# iptables -t filter -A INPUT -d 192.168.1.7 -p icmp --icmp-type echo-request -j DROP` --> Veiem que només cal descartar els echo-request ja que per sí mateix ja arriba a tots els altres al fer un ping.


### 3.

**Inconvenient --> Configurar taules de filtrat a cada una de les màquines fa que l'administració de la xarxa sigui complexa.**
**Solució --> "confiar" en la seguretat al router de la xarxa, ja que totes les comunicacions amb l'exterior passaran a través d'ell.**

Per fer això, reiniciem tot l'escenari i configurem el router Rint com a bastió de la xarxa, per protegir els hosts interns (host1 en aquest cas).

El comportament que volem obtenir ha de ser tal que:
"Un ping des d'una màquina externa a Net2, cap a una màquina dins de Net2 NO s'ha de respondre. En sentit contrari ha de funcionar correctament."

`Rint# iptables -A FORWARD -d 192.128.1.0/24 -p tcp -tcp-flags ALL SYN -j DROP`

(*Amb el ALL SYN especifiquem els intents d'inici de connexió TCP*)

Per provar el funcionament, intentem iniciar sessió telnet:
`www#: telnet 192.168.1.7`

Observem com NO es pot connectar, i només la Net1 veu els paquets.

En canvi, fer telnet des de host1 fins a www si que es pot.

**Filtrar tot el tràfic UDP que entri o surti de Net2, excepte el tràfic UDP que vagi dirigit a un servidor DNS (se suposa que és extern a Net2)**

Sabem que TOTES LES COMUNICACIONS AMB EL DNS UTILITZEN UDP PEL PORT 53. Per tant, hem d'impedir que entri tot tràfic UDP, menys el que surti d'algun host des del port 53.

`Rint#: iptables -A FORWARD -p udp --sport 53 -d 192.168.1.0/24 -j ACCEPT`
`Rint#: iptables -A FORWARD -p udp --dport 53 -s 192.168.1.0/24 -j ACCEPT`
`Rint#: iptables -A FORWARD -p udp -j DROP`

*Obs: Les comandes amb `iptable` es llegeixen en ordre, per tant si un paquet compleix la primera norma, les altres ja no es miren*

## Exercici 2

```
host$ simctl fwnat start
host$ simctl fwnat exec ifcfg
host$ simctl fwnat exec routecfg
host$ simctl fwnat exec fwcfg
```

## 1.
**Des del host de www de Net1, realitzem un ping a 10.0.4.2 (test)**

No funciona ja que la IP de www és privada.

## 2.
**Per solucionar els problemes anteriors configurar el router extern Rbcn per que realitzi SNAT per les seves xarxes internes. Un cop configurat, provar a realitzar el ping a 10.0.4.2 i comprovar si funciona**

`Rbcn#: iptables -t nat -A POSTROUTING -o eth2 -j SNAT --to 10.0.2.2`
- -o eth2: output interace
- -j SNAT: t'indica l'acció que realitzarà
- --to: @IP de destí
- POSTROUTING: la traducció de la IP d'origen (la qual les xarxes públiques no entenen) es fa un cop el paquet ja s'ha encaminat.

## 3.
**Configurem el bastió extern (Rbcn) per donar accés al servidor web de www des d'internet**
`Rbcn#: iptables -t nat -A PREROUTING -i eth2 -d 10.0.2.2 -j DNAT --to 172.16.1.2`
- -i: input, ens indica que el destí del paquet és el host des d'on fem la comanda.
- -j DNAT: el router tradueix l'adreça de destí abans del routing (PREROUTING), ja que des d'internet no podem posar IPs privades.