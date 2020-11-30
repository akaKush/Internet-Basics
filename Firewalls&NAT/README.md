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
    - [FTP + NAT](#ftp--nat)
  - [Conclusions](#conclusions)
- [Firewalls & NAT in Linux](#firewalls--nat-in-linux)
  - [Netfilter](#netfilter)
  - [Netfilter Operational](#netfilter-operational)
  - [Chain & Rule Configuration](#chain--rule-configuration)
    - [Exemples de iptables](#exemples-de-iptables)

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
  - El router tradueix l'adreça d'origen (Passa d'una IP privada a una IP pública, perquè des de Internet es pugui reconèixer aquella adreça)
  - La traducció es fa just abans que el paquet surti del router
  - El mateix router farà l'operació inversa (traducció de l'adreça de destí) just quan els paquets arribin.

- **Destination NAT (DNAT)**:
  - El router tradueix l'adreça de destí (l'adreça de destí normalment és una xarxa privada, per tant primer envia a la IP pública, llavors la tradueix a la privada corresponent i així el paquet pot arribar)
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
### FTP + NAT

A l'inici d'una sessió FTP, hi ha un intercanvi de comandos de control, i les **adreces IP** del host **s'inclouen** en aquests comandos.

Aquestes adreces es troben a la data, i s'han de traduir per fer funcionar FTP correctament. *Les adreces estan escrites en ASCII, NO estan codificades en 32 bits*.

L'objectiu és mantenir la longitud de les adreces originals, així el router no ha de canviar els números de seqüència de la connexió.

Si la nova adreça és més curta que la original, s'emplena amb 0's.
- Per exemple: Si NAT tradueix `206.245.160.2` a `192.168.1.2`, el router soluciona el problema afegint 0's fins que hi ha el mateix número de caràcters. Per tant l'adreça codificada serà: **`192.168.001.2`**.

Si la nova adreça és més llarga que la vella, afegim alguns bits nous.
- Exemple: traduim `192.168.1.2` a `206.245.160.2` el router ho soluciona afegint 2 bytes extres.

Un cop el router executa alguna d'aquestes accions, ha de canviar els ACK i números de seqüència per continuar mantenint la connexió coherent.

## Conclusions

- NAT ha extès la vida útil de IPv4, allargant el desplegament de IPv6.
- NAT no és una solució òptima
- Poden haver-hi diferents comportaments a diferents implementacions de NAT.
- Les noves aplicaciones i nous protocols necessitaran suportar NAT.
- Utilitza NAT només si és necessari.

# Firewalls & NAT in Linux
## Netfilter

Netfilter és el framework de filtrat de paquets utilitzat a Linux.
- Consisteix en unes *cadenes de normes* per fer el tractament de paquets.
- Cada cadena està associada a un "rol" diferent del host quan aquest està processant paquets.
- Els paquets són processats sequencialment travessant les cadenes de normes.
- Cada paquet de xarxa, arribant o abandonant un dispositiu travessa **com a mínim** una cadena.


Existeixen **5 tipus predeterminats de cadenes**:
- **PREROUTING**: És la primera cadena que un paquet que ve d'una xarxa troba. Es troba abans de la decisió de routing.
- **INPUT**: Aquesta cadena s'aplica quan un paquet s'ha d'entregar localment (el host és el destí).
- **FORWARD**: El paquest entrarà a aquesta cadena si ve d'una xarxa, però el host NO és el destí (el host actua com a router)
- **OUTPUT**: Aquesta cadena s'aplica quan els paquets s'envien des del host (el host és l'origen)
- **POSTROUTING**: Els paquets entraran a aquesta cadena després que es prengui la decisió d'enrutament.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/netfilter.png"/>

Com a últim comentari, cal destacar que si un paquet arriba al final de la cadena, aquest passarà per **DROP**.

## Netfilter Operational

Quan un paquet entra una cadena, el kernel (de Linux) verifica si les normes d'aquella cadena quadren amb el paquet.

Cada cadena conté un seguit d'especificacions per decidir si els paquets quadren amb aquella cadena. S'aplica un veredicte per determinar si aquella cadena s'ha d'aplicar al paquet.

Possibles veredictes de la cadena poden ser: "accept" o "drop for filtering", "change addresses (and ports) for NAT"...

**Metodologia:**
- Les normes s'examinen sequencialment.
- Si la norma NO es pot aplicar al paquet, s'examina la següent norma.
- Si la norma es pot aplicar, s'executa l'acció indicada al veredicte, i no s'examinen més normes.
- Si un paquet arriba al final de la cadena sense haver passat per cap norma, s'aplica la política per defecte en aquell paquet (drop).

## Chain & Rule Configuration

Si volem configurar les cadenes de netfilter al kernel de Linux fem servir el comando `iptables` tal que:

- `iptables <table> <op> <chain> <pkt-match-condition> <action>`
  - `<table>`: Seleccions la taula amb la que treballarem
    - `-t filter`: selecciona **packet filtering**
    - `-t nat`: selecciona la taula NAT
  - `<op>`: Normes d'operacions dins d'una cadena
    - APPEND: afegeix una nova norma a la cadena (`-A`)
    - INSERT: insereix una nova norma a una posició dins la cadena (`-I`)
    - DELETE: eliminar una norma d'una posició dins la cadena (`-D`)
    - MOVE: mou una norma a una altre posició (`-R`)
  - `<chain>` El nom de la cadena amb la que volem operar
    - Packet Fitering: INPUT, OUTPUT, FORWARD
    - NAT: PREROUTING, POSTROUTING, OUTPUT
  - `<pkt-match-condition>`: El conjunt de condicions que un paquet ha de cumplir
    - Physical/Link layer conditions: La interfaç de xarxa on han d'arribar (o transmetre) els paquets.
    - Network Layer Conditions: Camps al IP Header.
    - Transport Layer Conditions: Camps al transport Header.
  - `<action>`: El veredicte s'aplica al paquet si totes les condicions es cumpleixen.

Depenent de les condicions que afegim al comando, haurem d'afegir-ne algunes d'extres per especificar bé la condició que hem escollit.

Per exemple:

- Condicions per la **physical/link layer**: Hem d'indicar la interfície on el paquet s'ha d'enviar o de la que ha de ser transmesa
  - -i, --in-interface [!] name --> Nom de la interfície per la qual el paquet s'ha rebut (només per paquets entrant a les cadenes INPUT, FORWARD i PREROUTING). *Si afegim l'argument "!", el sentit s'inverteix.*
  - -o, --out-interface [!] name --> Nom de la interfície per on s'enviarà el paquet (només per paquets que entren a les cadenes FORWARD, OUTPUT i POSTROUTING).
- Condicions per la **network layer**:
  - -p, --protocol [!] protocol --> El protocol del paquet que s'ha de comprovar. Pot ser `tcp`, `udp`, `icmp` o `all`.
  - -s, --source [!] address[/mask] --> Especificació de l'origen.
  - -d, --destination [!] address[/mask] --> Especificació del destí.


### Exemples de iptables

1. Afegir una norma a la cadena de INPUT per fer un DROP de paquets l'origen dels quals és la IP 192.168.1.1, i el protocol  de transport és TCP:
   `iptables -t filter -A INPUT -s 192.168.1.1 -p tcp -j DROP`
2. Afegir una norma a FORWARD per fer un DROP de paquets que provinguin del rang d'adreces 192.168.1.0/24, el protocol ICMP, i icmp type echo-request, i rebut a la interfície eth0:
   `iptables -t filter -A FORWARD -i eth0 -s 192.168.1.0/24 -p ICMP --icmp-type-echo-request -j DROP`
3. Afegir una norma a OUTPUT per evitar conexions http cap al servidor www.google.com:
   `iptables -t filter -A OUTPUT -d www.google.com -p tcp --dport 80 --syn -j DROP`
4. Afegir una norma a SNAT per tots els paquets que abandonen el router utilitzant la interfície eth1, traduint les adreces a 172.16.1.1:
   `iptables -t nat -A POSTROUTING -o eth1 -j SNAT --to 172.16.1.1`
