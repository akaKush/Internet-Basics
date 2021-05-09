# Pràctica Multicast

Amb aquesta pràctica entendrem els següents problemes:
- Com identificar receivers que estan interessats en un flow de dades multicast. Veurem com s'utilitzen IPs de classe D (adreces multicast) per identificar grups de receivers.
- Com la capa ethernet mapeja les adreces IP a dins de les MAC. Construcció del direct mapping.
- Com els receivers informen als routers sobre el seu interès de tenir un flow multicast. Protocol IGMP.
- Com els routers multicast arriben a tots els destinataeis amb un sol flux de dades per link. Veurem l'estructura d'arbre que formen els routers per enrutar paquets multicast.

---

Escenari de les pràctiques:
![escenari](https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/escenari.png)

Fixemnos que tenim 3 illes multicast (SN0, SN3 i SN5) connectades a través d'una xarxa pública.

**R1, R2 i R3** poden gestionar paquets mutlicast, però **RC NO POT**.
Els tunels multicast estan connectats mitjançant GRE tunnels.

- Entre **R1** i **R2** tenim el `tunnel0`
- Entre **R2** i **R3** tenim el `tunnel1`


Iniciem escenari amb `simctl ipmulticast start`

## 0.5 Multicast addresses

- `all-systems.mcast.net` correspòn a l'adreça IP `224.0.0.1`, la qual indica tots els HOSTS multicast del mateix segment de xarxa (TTL=1).
- `224.0.0.22` identifica l'adreça `igmp.mcast.net`, la qual ens indica els ROUTERS que utilitzen IGMP (multicast).

Per identificar qualsevol adreça multicast només necessitem 28 bits, ja que el prefix de 4 bits està fixat --> **1110** (224)
Totes les adreces MAC multicast comencen per **01:00:5e**:..., i els següents 28 bits són el MAPEIG DIRECTE de l'adreça IP a la MAC.

Fem un ping desde server a 224.0.0.1 `server$ ping -c 1 224.0.0.1`:

Només veiem un paquet ICMP a SN3 de echo-request. Si mirem la IP de destí veiem com és una adreça multicast. El TTL d'aquest paquet és 1, això significa que envia multicast només a un salt.
Si mirem la capçalera MAC veiem que la MAC de destí és **01:00:5e:00:00:01** (multicast).

`ping -c 1 232.0.0.1`

Veiem com és la mateixa MAC de destí que en l'anterior apartat, tot i que la IP multicast sigui diferent. Això és el que provoca ambigüitat.
Per resoldre-ho, el destinatari d'aquest ping en la capa IP serà el que enrutarà cap on toca. Si el host de destí està unit al grup multicast indicat acceptarà el paquet, sinó no.

Observem que no hi ha cap missatge de resposta. Això és degut a que les aplicacions multicast no necessiten resposta. Tot i així per veure els missatges ICMP desactivem el flag que hi ha al sistema Linux pq no ignori els paquets ICMP multicast i broadcast:

`server$ echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts`

`server$ ping -c3 224.0.0.1`

El ping funciona correctament, rebem resposta desde el nostre propi host ja que som l'únic que té activat rebre icmps.
A SimNet3 no veiem cap missatge.

Però si ara desactivem el flag també a R2:

`R2$ echo 0 0 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts`

`server$ ping -c3 224.0.0.1`

Veiem com tenim resposta de echo-reply a SN3, provinent de R2.

## 0.6 Multicast transmission in a subnet

- `/proc/net/igmp` --> fitxer que conté informació dels grups que s'ha inscrit el host.
- `netstat -gn` --> ens dóna la mateixa info que el fitxer anterior (llistat de grups)
- `/proc/net/ip_mr_vif` --> fitxer amb les interfícies involucrades en operacions multicast en el router multicast, i algunes estadístiques d'ús. Només té info quan hi ha alguna sessió multicast oberta al moment.
- `/proc/net/ip_mr_cache` --> Multicast Forwarding Caché. Només es veuen les rutes actives.

Per a enviar-nos un fitxer, utilitzarem `udp-sender` i `udp-receiver`, per veure el significat de les instruccions cal executar `man udp-sender`.

Executem el següent desde server 0:

`server$ udp-sender --file=./big_664.mpg --min-client 1 --portbase 22345 --nopointopoint --interface eth1 --ttl 1 --mcast-addr 232.43.211.234 --mcast-all-addr 225.1.2.3`

D'aquesta manera indiquem el següent:
- Un grup multicast per enviar data: **232.43.211.234**
- Un grup multicast de control: **225.1.2.3**
- **--file** el fitxer que volem enviar
- **--min-client** nº mínim de clients per iniciar la transmissió
- **--nopointopoint** indica que NO utilitzi punt-a-punt encara que hi hagi només un receptor
- **--ttl** ttl dels paquets multicast
- **--mcast-addr** adreça per transferir dades
- **--mcast-all-addr** adreça per on sender i receiver es troben

Obrim una altre consola de server (1), i mirem el fitxer `/proc/net/igmp`, on veiem que ara hi ha una nova connexió amb el grup `030201E1`, el qual correspòn a 225.1.2.3

Si observem el wireshark a **SimNet3**:
- 3 paquets, 1 UDP i 2 IGMPv3
- El paquet UDP té IPd=225.1.2.3 i TTL=1. Si ens fixem en la part de dades veiem com undica la IP de les dades.
- Sport: 22346, Dport: 22345
- PAQUETS IGMP: **Membership Report Join group 225.1.2.3**

Si ara **aturem** el udp-sender de server 0, observem com s'envien **dos paquets IGMP**, del tipus **Membership Report: Leave group 225.1.2.3**.

Al fitxer /proc/net/igmp ja no veurem la connexió al grup.

**udp-receiver**

Obrim **R2 1** i mirem el fitxer /proc/net/igmp.

Veiem com estem connectats al grup multicast 224.0.0.1 a les interfícies lo, eth1 i eth2.

`R2$ udp-receiver --file=big_664.mpg --mcast-all-addr 225.1.2.3 --ttl 1 --portbase 22345`

Com que no hem específicat cap interfície, fa servir la eth2 per la transmissió, i per això no veiem res a la SimNet3.

En canvi a la SimNet2 veiem 2 paquets UDP i 2 IGMPv3. Els paquets UDP tenen **IP Source = 198.51.100.2** i **IP Destí = 225.1.2.3**, TTL=1.

Els paquets UDP no obtenen cap resposta pq el servidor no està actiu, i els IGMP tampoc perquè no hi ha cap router multicast.

Si tanquem la connexió del receptor, veiem **2 missatges IGMP Membership Report Leave group 225.1.2.3**.

*TORNEM A EXECUTAR UDP-SEND i RECEIVE:*

`server 0$ udp-sender --file=./big_664.mpg --min-clients 1 --portbase 22345 --nopointopoint --interface eth1 --ttl 1 --mcast-addr 232.43.211.234 --mcast-all-addr 225.1.2.3`

`R2 0$ udp-receiver --file=big_664.mpg --mcast-all-addr 225.1.2.3 --ttl 1 --interface eth1 --portbase 22345`

**Observem com ara sí que especifiquem INTERFÍCIE eth1, i veurem coses a la SimNet3.**

Analitzem els paquets que han passat per SimNet3:
- **4 paquets UDP inicials** (Per establir connexió inicial i intercanviar informació sobre la transmissió)
  - **Paquet 1 UDP**: IPdesti= 225.1.2.3 (grup de control).
  - **Paquets 2 i 3 UDP**: Els envia R2 per *avisar que també es vol connectar al grup de control*, tot i que no sap quin és encara.
  - **Paquet 4 UDP**: L'envia server mitjançant UNICAST per indicar quin és el grup al qual s'ha d'unir R2 per a rebre el video.
- Després d'aquest últim missatge UNICAST, R2 es connecta als dos grups multicast.
- Paquets conformes a la transmissió del video (de server a R2, i algun de R2 al server per verificar que segueixi actiu).
- Fi de la connexió mitjançant missatges IGMP Membership Report Leave Group "grup".

*IMPORTANT: El Server MAI s'uneix al grup de dades multicast, ja que aquest és només de RECEPCIÓ, i ell és qui l'està enviant.*

## 0.7 Multicast and Routing

Ara utilitzarem l'escenari complet de la pràctica, no només R2 i server.

### 0.7.1 GRE TUNNELS

Per crear els tunels no fa falta que ho fem manualment, ja que en aquest escenari podem executar `HOST$ simctl ipmulticast exec addtun` i ja es configuren tots:
- tunnel0: Desde R1 a R2
- tunnel1: desde R2 a R3

---
En cas de voler-ho fer manualment, els comandos que s'haurien d'executar en cada router són els següents (p.ex. en el R2):
```
R2$ ip tunnel add tunnel0 mode gre local 198.51.100.2 remote 192.0.2.2 dev eth2
R2$ ifconfig tunnel0 192.168.110.1
R2$ route add -net 192.168.0.0/24 dev tunnel0
```
---

Si ara fem un ping de **server** a **host2** veiem com funciona bé, i analitzant la SN2 veiem 2 ICMP echo request i reply, amb les capçaleres corresponents a l'encapsulament i al paquet intern.

Fent el ping de **server** a **host4** també funciona correctament.

### smcroute

**smcroute** serveix per manipular les rutes multicast estàtiques del kernel de Linux.

`R2$ smcroute -d` --> iniciem el daemon smcroute.
`R2$ smcroute -a eth1 0.0.0.0 232.43.211.234 tunnel0` --> Enruta datagrames IP que entrin per eth1 des de qualsevol adreça d'origen (0.0.0.0, default) cap a la direcció multicast 232.43.211.234 a través del tunnel0.
`R2$ smcroute -j eth1 232.43.211.234` --> Fem que R2 s'uneixi al grup multicast 232.43.211.234 a través de la interfície eth1.

Si ara mirem el fitxer `/proc/net/igmp` veiem com apareix el grup EAD32BE8 a la eth1, el qual pertany a la IP=232.43.211.234.

A la SimNet3 podem veure 2 missatges IGMP, amb adreça de destí 224.0.0.22 (tots els routers multicast), i amb un missatge de join al grup 232.43.211.234

---

Fem un ping de **server** al **grup multicast** amb un màxim de 3 salts:
`server$ ping -c1 -t3 232.43.211.234`

Veiem com el ping funciona, es veuen paquets ICMP a SimNet3 (req i reply), SimNet2 (req.) i SimNet1 (req.) (als tunels multicast no es rep un reply perquè el destinatari era el grup multicast i no es responen els missatges a multicast).

Per SimNet0 NO ES VEU RES.

---

Configurem R1 perquè enruti datagrames IP que vinguin del tunel amb qualsevol adreça origen i com a adreça destí la multicast 232.43.211.234 a través de eth1:
```
R1$ smcroute -d
R1$ smcroute -a tunnel0 0.0.0.0 232.43.211.234 eth1
R1$ smcroute -j tunnel0 232.43.211.234
```

Si ara tornem a enviar un ping des de server cap a multicast, per SimNet0 ja veurem el missatge echo-request.

### Testing Tools
### ssmping

**ssmping** serveix per verificar rutes unicast i multicast.

Per comprovar les connexions multicast fem el següent:
`server$ ssmpingd`
`host2$ ssmping 172.16.1.3`

Analitzem les xarxes i veiem:

1. 1 paquet UDP UNICAST de host2 a server.
2. 2 paquets UDP unicast de tornada.
3. 1 paquet de server al grup 232.43.211.234.

---

### 0.7.5 Trees with multiple branches

Anem a configurar finalment l'escenari per enviar-nos el video.

1r eliminem el daemon anterior i el reiniciem:
```
R2$ smcroute -k
R2$ smcroute -d
```

Ara indiquem les DUES INTERFÍCIES DEL TUNEL:
```
R2$ smcroute -a eth1 0.0.0.0 232.43.211.234 tunnel0 tunnel1
R2$ smcroute -j eth1 232.43.211.234
```

Utilitzem ssmping per comprovar la configuració:
```
server$ ssmpingd
host2$ ssmping 172.16.1.3
host4$ ssmping 172.16.1.3
```

Veiem com al host4 NO arriben missatges multicast, només unicast. Ens falta fer que R3 sigui un router multicast.
```
R3$ smcroute -d
R3$ smcroute -a tunnel1 0.0.0.0 232.43.211.234 eth1
R3$ smcroute -j eth1 232.43.211.234
```
Ho podriem entendre com: "tots els missatges que entrin a R3 a través del tunnel1, provinents de QUALSEVOL adreça IP, amb direcció a la 232.43.211.234 han de sortir per la eth1. Seguidament ens ajuntem a la interfície eth1 per veure els missatges que hi passen".

Ara tornem a intentar el ssmping desde els dos hosts, i veiem com ja funciona correctament, rebem missatges IGMP a les dues SimNets.