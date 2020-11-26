<h1>Firewalls i Traducció d'Adreces de Xarxa</h1>


- [Firewalls](#firewalls)
  - [Introducció](#introducció)
  - [Objectius dels Firewalls](#objectius-dels-firewalls)
  - [Packet Filtering Firewalls](#packet-filtering-firewalls)
  - [Stateful Filtering Firewalls](#stateful-filtering-firewalls)
  - [Proxy Firewalls](#proxy-firewalls)
  - [Next Generation Firewalls](#next-generation-firewalls)
- [NAT](#nat)
  - [Raons per utilitzar NAT](#raons-per-utilitzar-nat)
  - [Intro](#intro)
  - [Utilització de NAT](#utilització-de-nat)
  - [Tipus de NAT](#tipus-de-nat)
  - [Conseqüències d'utilitzar NAT](#conseqüències-dutilitzar-nat)
  - [Problemes i limitacions de NAT](#problemes-i-limitacions-de-nat)
  - [Exemples](#exemples)
  - [Conclusions](#conclusions)
- [Firewalls & NAT in Linux](#firewalls--nat-in-linux)
  - [Netfilter](#netfilter)
  - [Netfilter Operational](#netfilter-operational)
  - [Chain & Rule Configuration](#chain--rule-configuration)

---

# Firewalls
## Introducció

**Què és un Firewall?**<br><br>
Els routers distribueixen el tràfic  segons un conjunt de normes que simplement apunten a l'adreça de destí.<br>
Però i sí necessitem que el tràfic estigui controlat segons uns altres paràmetres?<br>
Exemples:
- Equipament a la xarxa externa només pot accedir el servidor web públic.
- El departament de Marketing pot accedir al server privat però no al públic.
- El tràfic d'un dispositiu en concret pot accedir a la xarxa de marketing.

Una primera solució proposada a aquest problema seria restringir el tràfic produit per cada un dels dispositius.
- Extramadement difícil de fer actualitzacions
- No és escalable
- De vegades no és ni possible

Per tant un **firewall** el podem entendre com un **controlador d'accés o PEATGE**, el qual ens divideix la xarxa en 2:
- Internal Network (Podria ser la xarxa interna d'una empresa) --> Considerada "segura i fiable"
- External Network (Internet) --> Considerada "insegura"

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/firewall.png"/>

## Objectius dels Firewalls

- **Punt d'estrangulament únic per el tràfic de xarxa.** Tot el tràfic tant entri o surti ha de passar pel firewall.
  - Escala degudament, i minimitza el risc de nous perills.

- **Normes de control d'accés**. Només el tràfic autoritzat està permès a passar el firewall.
  - Facilitat l'administració.

- **Seguretat al dispositiu**. El firewall en sí mateix s'assumeix que és immune a penetracions, ha de ser especialment segur.

## Packet Filtering Firewalls

Els **Packet Filters** inspeccionen paquets IP que arriben al firewall i apliquen un conjunt de normes per determinar si són acceptats o descartats.
- **ACCEPT/FORWARD**. El paquet s'accepta.
- **REJECT** El paquet és descartat i l'origen del paquet és informat a través d'un ICMP.
- **DROP**. El paquet és descartat silenciosament.

Les normes de filtreig estan basades en **informació** que es troba al mateix paquet de xarxa:
- L'adreça dorigen i de destí del paquet.
- Les adreces origen i destí del transport (per exemple els ports UDP/TCP)
- El protocol de la xarxa superior (UDP, TCP, etc)
- L'interfície de xarxa entrant i sortint.

<h3>Avantatges dels Packet Filtering Firewalls</h3>

- Simplicitat
- Evaluació ràpida
- No canvia el flow del tràfic o les seves característiques.

<h3>Desavantatges</h3>

- Auditoria d'informació limitada
- Vulnerable a alguns atacs (relativament simples)
  - Atacs de *Network Spoofing* (suplantació de xarxa)
  - *Source Routing Attacks* (Eludeixen les normes d'enrutament)

## Stateful Filtering Firewalls

També anomenats **Dynamic Packet Filtering**. Tenen en compte el context, i mantenen un historial de packets vistos anteriorment per prendre millors decisions futures.
- Fan un seguiment de les connexions obertes actuals.
- Mantenen una taula amb aquestes connexions.
- Associen les requests de noves connexions a connexions actuals legítimes.

Un **stateful firewall** pot mantenir diversos atributs de les conexions que està guardant, com:
- Adreces IP
- Ports
- Números de seqüència
- Estat de la connexió

<h3>Avantatges</h3>

- Més segurs que els primers
  - Tenen mecanismes de defensa per atacs de *spoofing* i *DoS*.
  - Prevenen atacs de número de seqüència TCP.

<h3>Desavantatges</h3>

- Les normes són més difícils d'escriure
- Continuen tinguent accés limitat a l'auditoría d'informació


## Proxy Firewalls

Els anomenats **Application Layer Gateway (ALG)**, Application Layer Firewall o Application Proxy, són **dispositius** que actuen com un relé per tràfic a nivell d'aplicació. 
<br><br>
Els ALG poden "entendre" alguns protocols d'aplicació. Són capaços de detectar:

- Si un protocol NO desitjat està intentant eludir el firewall en un port en concret.
- Si un protocol s'està utilitzant de manera perjudicial.
- Si les credencials dels usuaris són suficients per poder utilitzar algún port.

*Per exemple, un ALG amb suport de HTTP és capaç de bloquejar la descàrrega d'una pàgina web en un lloc, mentres permet la descàrrega d'altres pàgines web en el mateix lloc.*

<h3>Avantatges</h3>

- Protocol més segur.
- Capacitats d'auditoria d'informació plenes.
- Amaga l'esquema intern d'adreces.
- Les aplicacions perjudicials es poden bloquejar.

<h3>Desavantatges</h3>

- Requereix de molts més recursos, i normalment són més lents
- Requereix un coneixement complex del protocol
- No està disponible per a tots els protocols d'aplicació
- Els protocols d'aplicació es tendeixen a actualitzar més freqüentment que els de transport.
- Pot requerir configuració extra del client


## Next Generation Firewalls
Aquests firewalls combinen diversos tipus de firewalls, com per exemple: *packet inspection, stateful inspection, application proxies, etc.*

<br>

Introdueixen el concepte de **Deep Packet Inspection**:
- Els firewalls tradicionals només inspeccionen els HEADERS.
- La *Deep Packet Inspection* inspecciona els HEADERS i la DATA que el paquet porta
  - Poden detectar protocols/aplicacions ofuscades
  - Poden detectar documents HTML perjudicials
  - ...

---

# NAT
## Raons per utilitzar NAT
<h3>Problema</h3>

- L'adreçament IP públic és limitat (actualment s'ha exhaurit) i és molt car

<h3>Solució</h3>

- Utilitzar **adreçament IP privat** en **xarxes internes**
- Blocs d'adreces disponibles:
  - Classe A: 10.0.0.0/8
  - Classe B: 172.16.0.0/16
  - Classe C: 192.168.0.0/24
- El problema que té aquesta solució és que les **adreces privades NO es poden utilitzar en xarxes públiques** (INTERNET)


## Intro
NAT és un mecanisme de traducció d'adreces IP (i números de port), el qual permet connectar xarxes públiques i privades, de manera que s'estalvien adreces IP públiques.
<br>
El NAT suporta TCP, UDP i ICMP.
<br><br>

Quan utilitzem NAT, **es realitzen dues traduccions d'adreces**:
- Una traducció quan el paquet surt del router NAT
- Una traducció inversa quan el paquet retorna al router NAT.


## Utilització de NAT

<h2>MIRAR VIDEOS</h2>


## Tipus de NAT

Tenim **2 tipus principals de NAT**:
- **Source NAT (SNAT)**:
  - El router tradueix l'adreça d'origen
  - La traducció es fa just abans que el paquet surti del router
  - El mateix router farà l'operació inversa (traducció de l'adreça de destí) just quan els paquets arribin.

- **Destination NAT (DNAT)**:
  - El router tradueix l'adreça de destí
  - La traducció es fa abans de l'enrutament, just quan el paquet arriba al router
  - El router farà l'operació inversa (traducció de l'adreça d'origen) quan els paquets arribin.

## Conseqüències d'utilitzar NAT

Quan les @IP canviem, hem de tenir en compte algunes consideracions:
- Els **checksums s'han de tornar a calcular** (inclou els checksums de IP, TCP i UDP)
- Quan s'utilitza NAT, els paquets de resposta haurien de tornar al mateix router.
- Per aquesta raó, en la majoria de casos només hi ha un router NAT que controla l'accés a Internet.

## Problemes i limitacions de NAT

<h3>Problemes i limitacions (I)</h3>

- Quan utilitzem NAT, l'**Internet esdevé una xarxa pseudo-orientada-a-connexió**.
- El NAT **acaba amb la transparència de les capes de protocol**
- El NAT **esdevé un router BOTTLENECK**
  - Ha de manejar les conexions entrants i sortints, el que provoca problemes de CPU i memòria
  - El número de IP públiques és limitat, per tant això limita el número de possibles conexions externes


<h3>Problemes i limitacions (II)</h3>

- Alguns protocols i aplicacions inclouen les @IP als seus HEADERS i DATA fields.
- Aquestes @IP han d'estar correctament traduides --> **El router ha d'utilitzar** (a part de NAT) **ALG (Application-Level-Gateway)**

## Exemples


## Conclusions


# Firewalls & NAT in Linux
## Netfilter


## Netfilter Operational


## Chain & Rule Configuration
