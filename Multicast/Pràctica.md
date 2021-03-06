# Pràctica de Routing Multicast

- [Pràctica de Routing Multicast](#pràctica-de-routing-multicast)
  - [0.3 Inicialitzem escenari](#03-inicialitzem-escenari)
  - [0.4 Adreces IP multicast i DNS](#04-adreces-ip-multicast-i-dns)
  - [0.5 Adreces MAC multicast](#05-adreces-mac-multicast)
  - [0.6 Configuració Multicast](#06-configuració-multicast)
    - [UDP-SENDER](#udp-sender)
    - [UDP-RECEIVER](#udp-receiver)
    - [Enviant el video a la subxarxa](#enviant-el-video-a-la-subxarxa)
  - [0.7 Routing i Multicast](#07-routing-i-multicast)
    - [Configurant Tunels](#configurant-tunels)
    - [Routing Multicast Estàtic (enviament de ping mcast)](#routing-multicast-estàtic-enviament-de-ping-mcast)
    - [Testejant la xarxa](#testejant-la-xarxa)
    - [Arbres amb Múltiples branques](#arbres-amb-múltiples-branques)
    - [Enviant el video (unidireccionalment)](#enviant-el-video-unidireccionalment)
    - [Arbre BIDIRECCIONAL](#arbre-bidireccional)

## 0.3 Inicialitzem escenari
Iniciem la simulació amb `simctl ipmulticast start` i obtenim el següent escenari:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/escenari.png" height=75% width=75%/>

## 0.4 Adreces IP multicast i DNS
Un cop dins la simulació i comprovat que l'escenari s'hagi carregat bé, executem el següent comando:
`host all-stystems.mcast.net` i veiem com ens retorna el següent output: **all-systems.mcast.net has address 244.0.0.1**.

Si executem `host 224.0.0.22` veiem com ens retorna: **22.0.0.224.in-addr-arpa domain name pointer igmp.mcast.net**, el qual ens indica el nom igmp.mcast.net, que és el que s'utilitza per indentificar els routers que utilitzen el protocol **IGMPv3**.

## 0.5 Adreces MAC multicast
En aquest apartat veurem paquets multicast, els quals simplement són paquets IP amb la "Destination Address" sent la de multicast.

El fet és que a part de la IP de destí necessitem indicar una MAC per indicar al *Ethernet frame*. Per crear aquesta MAC s'utilitza **MAPEJAT DIRECTE**, el qual inclou els 24 bits de menys pes de l'adreça IP a la MAC directament.

Comencem enviant un ping:
`server# ping -c 1 224.0.0.2`
i escoltant a SimNet3 amb Wireshark veiem:
- només tenim 1 missatge ICMP de echo request i cap de tornada *<-- Això és degut a que el tràfic multicast és per enviar flow de data i no pings que responguin*
- El TTL d'aquest paquet és de 1.
- A la IP Header veiem:
  - Source: 172.16.1.3
  - Destination: 224.0.0.1
- A la Ethernet Header veiem:
  - Source: fe:fd:00:99:03:01 (fe:fd:00:99:03:01)
  - Destination: **IPv4mcast_01** (01:00:5e:00:00:01)  *<-- Veiem com utilitza el prefix 01:00:5E, els quals corresponen a les adreces IP mcast. Llavors fa el mapeig directe a partir de la IP 224.**0.0.1** i afegeix els 24 bits menys significants a la MAC directament tal que: 01:00:5E:**00:00:01**.*


Ara enviem el següent ping `server# ping -c 1 232.0.0.1` i observem el següent:
- L'adreça MAC de destí **és la mateixa**. Veiem que es pot produir ambiguetat amb totes les adreces IP dins del rang de les de classe D (multicast) que tinguin els 24 bits menys significatius iguals.

Per resoldre aquesta ambiguetat el sistema utilitza les @IP per enrutar correctament.

---

**Important**: Com ja hem comentat el tràfic multicast no està fet per enviar respostes als pings, però en aquesta pràctica volem rebre alguna resposta per comprovar el bon funcionament.

Per **DESACTIVAR EL FLAG al Linux que DEIXI D'IGNORAR PAQUETS BROADCAST i MULTICAST ICMP**:
- `server# echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts`

---

Ara tornem a fer `server# ping -c3 224.0.0.1`, però tot i així encara **no veiem** els paquets de **reply**.

**Per veure'ls hem de desactivar el mateix flag al router multicast** igual que abans pero des de la seva terminal.

Finalment, si tornem a executar el ping des del server cap a 224.0.0.1 veiem com ara:
- Ja tenim paquets de reply per a cada ping.
- Els paquets de reply provenen (source) del R2, desde la seva interfície eth1 (172.16.1.1)


---

**Conclusió:** Si estem interessats en tràfic multicast, per fer-ho primer hem de deshabilitar el flag que ignora paquets broadcast i multicast, tant del host on volem accedir al tràfic multicast com des del router que permet enviar-ne, i evidentment també de l'emissor d'aquest tràfic.


## 0.6 Configuració Multicast

Si des de qualsevol màquina volem veure els grups multicast als que s'ha connectat ho podem fer des de **/proc/net/igmp**
- `#server cat /proc/net/igmp` veiem el següent:

    <img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/grups.png" height=50% width=50%/>

    Veiem com ens dóna informació de les interfícies que estan conectades a certs grups. Per exemple aquí veiem com tant la de loopback com la eth1 estan connectades al grup: **010000E0**, el qual traduit a decimal, és igual a **1.0.0.224** (multicast).

    Podem veure la mateixa info executant `netstat -gn`:

    <img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/netstatgn.png" height=50% width=50%/>

- `server# cat /proc/net/ip_mr_vif`:
    
    <img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/ip_mr_vif.png" height=50% width=50%/>

    Aquí veiem les interfícies que estan involucrades en operacions multicast al router multicast, i estatístiques d'ús (tot i que ara no es veuen pq hi ha d'haver tràfic multicast fluint).

- `server# cat /proc/net/ip_mr_cache` o millor utilitzar **`ip mroute show`**
    El qual ensenya els continguts de la **cache del Multicast Forwarding**.

    Important saber que només ensenya les rutes actives en el moment d'executar el comando.

---

Per enviar tràfic multicast, utilitzarem protocol UDP a la capa de transport, ja que el TCP està orientat a control d'errors, i no ens interessa, és més adequat UDP que permet enviar paquets a multicast i no és tan estricte a l'hora de transmetre.

Per fer-ho utilitzarem els comandos `udp-sender` i `udp-receiver`.

El nostre software utilitza 2 grups per habilitar la transmissió:
- Grup Multicast per enviar dades: **232.43.211.234**
- Grup Multicast per control i fiabilitat: **225.1.2.3**

### UDP-SENDER

Obrim una altre consola de server: `simctl ipmulticast get server 1`.

Des de la consola 0, executem 
```
server# udp-sender --file=./big_664.mpg --min-clients 1 --portbase 22345 \
--nopointopoint --interface eth1 --ttl 1 --mcast-addr 232.43.211.234 \
--mcast-all-addr 225.1.2.3
```
I veiem el següent: 
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/udp-sender.png" height=50% width=50%/>

Això ens **prepara el server per transmetre** l'arxiu *big_664.mpg* guardat localment, **via multicast**.

Si ens fixem al wireshark, veiem 3 paquets:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/paquets.png" height=50% width=50%/>

- Paquet **UDP**
  - Src: 172.16.1.3
  - Dst: 225.1.2.3
  - TTL: 1
  - Port number: 17
- Paquets **IGMPv3**
  - 2 Membership report packets
  - Src: 172.16.1.3
  - Dst: 224.0.0.22

Com que no tenim cap router multicast, no tenim cap missatge de resposta als membership reports.

Però si ara anem a la consola 1 de server, i tornem a executar `cat /proc/net/igmp` veiem el següent:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/nou-grup.png" height=50% width=50%/>

Veiem com a eth1 tenim un nou grup, **`030201E1`**, el qual passat a notació decimal és **3.2.1.225**.

Però quan tanquem el server des de la consola 0, si tornem a visualitzar el fitxer igmp veurem que ja no estem afegits al grup (pq aquest s'ha tancat).

### UDP-RECEIVER

El comendo `udp-receiver` treballa al seu corresponent client.

En el nostre cas anem a R2 (consola 1) i executem cat /proc/net/igmp i veiem que no estem a cap grup a part del default (224.0.0.1), per les interfícies de loopback, eth1 i eth2.

Des de la consola 0 de R2 executem: 
```
udp-receiver --file=big_664.mpg --mcast-all-addr 225.1.2.3 --ttl 1 \
--portbase 22345
```
Aquest comando **prepara R2 per rebre un video via multicast**, el qual es guardaria localment amb el nom big_664.mpg

---
Tornem a adjuntar l'escenari de la pràctica per una millor comprensió
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/escenari.png" height=75% width=75%/>

---

Ara si ens fixem als Wiresharks:
- **SimNet2** (xarxa de R2 a Internet): 2 paquets UDP i 2 IGMPv3
- **SimNet3** (xarxa del servidor a R2): No veiem cap paquet

Veiem que **no rebem els paquets a la xarxa que desitgem**, i això es deu pq si no especifiquem la interfície per la que volem tenir el trànsit de tràfic multicast, el sistema operatiu tria la que té per defecte al routing unicast, i no sempre és la que volem.

Per tant: <h3>S'HA D'ESPECIFICAR LA INTERFÍCIE PER LA QUE VOLEM ENVIAR PAQUETS</h3>

De tota manera, si ens fixem en els paquets enviats podem veure el següent:
- Als paquets UDP, l'@IP de destí és `225.1.2.3` i el TTL=1.
- El port que utilitzen és el 17
- Els paquets IGMPv3 són 2 Membership Reports (**Join Group**) que envien però no reben resposta pq no hi ha router multicast.

Si ara tornem a veure els grups que tenim a R2 des de la consola 1, veiem com a la interfície eth2 tenim el nou grup al que estem afegits, i si sortim del `udp-receiver` a la consola 0, veurem com aquest grup desapareix, i així mateix com a la SimNet2 arriben **2 paquets IGMPv3** de Membership Report/**Leave Group** (els quals tampoc obtindran resposta)

### Enviant el video a la subxarxa

Ara que ja sabem com funciona `udp-sender` i `udp-receiver` ja podem fer una **distribució real del fitxer de video a través de multicast**:

- Obrim un Wireshark a **SimNet3**
- Anem a consola 0 de **server** i executem el següent comando: `server# udp-sender --file=./big_664.mpg --min-clients 1 --portbase 22345 \ --nopointopoint --interface eth1 --ttl 1 --mcast-addr 232.43.211.234 \ --mcast-all-addr 225.1.2.3`
- A consola 0 de **R2** executem: `R2# udp-receiver --file=big_664.mpg --mcast-all-addr 225.1.2.3 --ttl 1 \ --interface eth1 --portbase 22345`

1. La transmissió s'ha fet bé però analitzem alguns aspectes dels paquets enviats:
   1. Al principi de tot tenim **4 paquets petits UDP**, els quals serveixen perquè l'emissor i el receptor es coneguin mútuament i puguin intercanviar informació clau per la transmissió.
   2. Ara l'emissor sap quin és el grup d'enviament multicast, però no sap qui és el receptor concret. L'últim dels 4 paquets li indica al receptor quin és el grup al que s'ha d'unir. Veiem com és un missatge Unicast enviat directament al receptor.
   3. Als següents paquets es transmet la DATA del paquet. Si convertim `232.43.211.234` a **hex** obtenim: `E8:2B:D3:EA` <-- Els últims 24 bits de l'@IP, són els que mapegem directament a la @MAC de destí multicast: `01:00:5E:2B:D3:EA`.
   4. Els dos paquets que s'envien després del Membership Report suposo que són per veure si tenim establerta la comunicació.
   5. El quart paquet enviat pel server, el qual és UNICAST, deu ser per indicar que s'ha establert la comunicació, i ja es pot unir el host interessat al grup indicat per la recepció de paquets.
   6. Llavors veiem finalment com s'envia un últim Membership Report / Join Group definitiu.
2. Si ara ens fixem en els paquets que contenen la DATA:
   1. L'@IP de destí d'aquests paquets és la 232.43.211.234, la qual pertany a una adreça multicast, del host en concret proporcionat pels 24 bits menys significatius.
   2. El tamany de la majoria d'aquests paquets és de **1514** bits.
   3. De tant en tant tenim missatges UNICAST des de R2 al server. Aquests deuen ser missatges generals de MR, per continuar indicant que el host està interessat en aquell grup multicast.
3. Finalment tant l'emissor com el receptor acaben la transmissió i abandonen els corresponents grups enviant un **IGMP Leave Group**.


## 0.7 Routing i Multicast

Ara que ja hem transmès un paquet en una subnet, anem a fer un pas més i intentant **comunicar diferents routers multicast** durant la transmissió, per poder enviar els paquets més lluny.
   
Com sabem tenim 3 "illes" de xarxes Multicast diferents (SimNet0, Simnet3 i SimNet5) connectades a través d'una xarxa pública.
Tenim 3 routers multicast (R1, R2 i R3). Aquestes illes multicast estan connectades via GRE tunnels, els quals ara crearem.
1. `tunnel0` entre R1 i R2
2. `tunnel1` entre R2 i R3

### Configurant Tunels

- `R2# ip tunnel add tunnel0 mode gre local 198.51.100.2 remote 192.0.2.2 dev eth2` --> Configura el tunnel a la interfície que li toca i a quina interfície s'ha de connectar
- `R2# ifconfig tunnel0 192.168.110.1` --> Activem una **nova interfície virtual del tipus tunnel**, i li afegim una adreça IP perquè funcioni bé.
- `R2# route add -net 192.168.0.0/24 dev tunnel0` --> Actualitzem taula de rutes per enrutar degudament els paquets que han d'anar pel tunel.

---

De totes maneres per la pràctica creem els tunnels automàticament amb `simctl ipmulticast exec addtun`, i per comprovar que funcionin, obrim **SimNet2** a Wireshark i **enviem un ping** des del **server** a **host2**.
- Veiem com el ping funciona correctament, s'envia bé el request i el reply.
- Si ens fixem en **Src** i **Dst** del **paquet IP exterior**, veiem com aquest s'envia directament des de R2 fins a R1, i de tornada igual, sense passar per internet.
- Al **paquet IP interior** veiem com les adreces de destí i origen són les del host2 i server respectivament.

Si enviem un ping de server a host4, comprovem com també funciona el tunel ja que el paquet s'envia directament entre routers.

---
*Observació:* Cal tenir en compte que ICMP s'utilitza normalment per detectar errors, però en el cas de multicast no funciona tant bé, i per tant és possible que tinguem errors a la xarxa i no rebem missatges de control com "Host Unreachable", "Fragmentation Needed", "Time Exceeded", etc.

---
### Routing Multicast Estàtic (enviament de ping mcast)

Ara **crearem l'arbre de routing multicast des de l'emissor fins a tots els receptors**.
Utilitzarem el comando `smcroute`, amb el qual es pot manipular les rutes estàtiques del kernel de Linux.

Primer començarem configurant l'arbre de routing multicast estàtic per comunicar les dues illes **SimNet0** i **SimNet3**, les quals estan connectades pel **tunnel0**. Llavors connectarem l'altre illa **SimNet5**.

Obrim 4 wiresharks de SimNet0 a Simnet3:
- Al R2 (EL QUAL ÉS EL **ROUTER EMISSOR DE MULTICAST**) **executem el daemon: `smcroute -d`**
  - Si ara executem `cat /proc/net/ip_mr_vif` Veiem totes les interfícies que tenim ara per les que podem enviar tràfic multicast.
  - **AFEGIM NOVA RUTA MULTICAST ESTÀTICA a R2**: `R2# smcroute -a eth1 0.0.0.0 232.43.211.234 tunnel0` --> *Això enruta a través del tunnel0 els datagrames IP que entrin a través de eth1, de qualsevol @IP d'origen (0.0.0.0), i amb @IP de destí el grup multicast 232.43.211.234*.
    - Ens **ajuntem a aquesta ruta**: `smcroute -j eth1 232.43.211.234` i si ara ens fixem en /proc/net/igmp, veiem com la interfície **eth1** s'ha afegit a un grup (EAD32BE8 --> El qual passat a binari és 234.211.43.232)
    - Si mirem SimNet3, veiem com han aparegut un parell de **missatges IGMP** Membership Reports / Join the group 232.43.211.234
  - **Enviem un ping des del server al grup multicast 232.43.211.234** amb un ttl de 3: `ping -t3 -c1 232.43.211.234`
    - Veiem com el ping funciona. Aquest es veu tant a SimNet2 com a SimNet3, *però hi ha una diferència*: A SimNet3 veiem tant el echo request com el echo reply, pero a SimNet2 només veiem el echo request i cap reply. Això es deu a que el grup multicast, no retorna missatges de confirmació del ping, però en canvi el router si que el retorna al host.
    - Només veiem encapsulació IP a SimNet2 --> pq a SimNet3 ho envia amb enrutament unicast, mentres que a SimNet2 al enviar el paquet del R2 al grup, aquest anirà encapsulat.
    - Mida de la GRE Header ???????????????
    - Si ara mirem el fitxer /proc/net/ip_mr_vif veiem com a la interfície eth1 se li han afegit dades a les seves estadístiques, i si executem `ip mroute show` veurem com tenim una ruta creada, amb @origen, @destí, Interfície d'entrada i interfícies de sortida (veure foto)
    <img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/rutes.png" height=50% width=50%/>
    - Tot i així a SimNet1 no veiem res pq no tenim R1 configurat per fer forwarding de paquets IP amb destí al grup 232.43.211.234.
  - **Afegim Ruta Estàtica Multicast a R1**: `R1# smcroute -a tunnel0 0.0.0.0 232.43.211.234 eth1` --> *Com que volem arribar a la SimNet0, hem d'enviar els paquets que ens arribin pel tunnel0, sigui quina sigui la @origen, i amb destí al grup 232.43.211.234, a través de la interfície eth1.*
    - Ara si tornem a fer un ping des de server al grup, veurem com a SimNet0 tindrem un missatge Echo Request (el reply no pq no el sap enviar el tràfic multicast).
    - El **TTL** a SimNet0 **és de 1**, i si l'enviem a menys de 3, el paquet no arriba. En cas que posessim el TTL a 4 o més, el paquet arribaria bé però potser també arribaria a altres xarxes no destijades i gastar més recursos dels necessaris.

### Testejant la xarxa

Fins ara hem utilitzat pings per comprovar com flueix el tràfic mcast, però el ping no és una aplicació que envii dades reals.
A partir d'ara utilitzarem 2 eines per testejar que el multicast funcioni correctament:
- **ssmping** --> Verifica unincast i multicast routing. Determina si un host por rebre SSM (Source Specific Multicast) d'un altre host, mitjanç¡ant enviament de paquets UDP.
- **mcsender** and **emcast** --> Ens permeten verificar el correcte funcionament de Mcast.

1. `ssmping`
   - Iniciem el daemon amb `ssmpingd` des de la consola 0 del server, i comprovem que aquest s'hagi ajuntat al nou grup des de la consola 1 amb **`netstat -gn`**
   - A host2 executem `ssmping 172.16.1.3`

A la consola del server veiem com aquest rep les requests dels pings mcast enviats des de 192.168.0.2 (host2), envia seguidament un altre missatge unicast de tornada cap a host2, i finalment envia també el missatge mcast amb destí el grup (232.43.211.234).

Si provem de fer el mateix però invertint el sentit, és a dir, host2 treballant com a daemon, i enviem un ssmping des de server cap a host2, veiem com ara es transmeten correctament els pings UNUCAST, però els MULTICAST SÓN DESCARTATS.

### Arbres amb Múltiples branques

Ara hem pogut fer transmissions de paquets entre SimNet0 i SimNet3, però també hem de connectar SimNet5.

L'objectiu és crear un arbre de routing multicast estàtic que pugui transmetre multicast des del server als receptors que es troben a SimNet0 i SimNet5.

1. Eliminem el daemon anterior (amb les seves configuracions) `R2# smcroute -k` i l'iniciem de nou `R2# smcroute -d`
2. Ara hem de connectar noves rutes multicast estàtiques, però la **diferència** és que ara posarem **una llista d'interfícies de sortida** que s'utilitzaran com a output.
Ho fem amb el següent comando: `R2# smcroute -a eth1 0.0.0.0 232.43.211.234 tunnel0 tunnel1`.
Ens unim a la interfície entrant eth1 per unir-nos al grup: `R2# smcroute -j eth1 232.43.211.234`

Ara comprovem que funcioni la configuració amb **ssmping**:
```
server# ssmpingd
host2# ssmping 172.16.1.3
host4# ssmping 172.16.1.3
```
- La comunicació amb host2 funciona, però la de **host4** no --> R1 està ben configurat per fer forwarding de paquets multicast però **R3** no, li **hem d'afegir la ruta** per fer **forwarding de paquets amb destí el grup** 232.43.211.234.

**Configurem R3:**
- Iniciem daemon `smcroute -d` --> *sempre que vulguem fer que un router pugui fer forwarding de multicast, hem d'iniciar el daemon en aquest router*
- Afegim ruta per fer forwarding de paquets mcast: `R3# smcroute -a tunnel1 0.0.0.0 232.43.211.234 eth1` --> Tots els paquets que entrin per la interfície tunnel1 amb destí 232.43.211.234 sortiran per defecte per la interfície eth1 (per arribar a SimNet5).
- Ens unim a la nova ruta: `R3# smcroute -j tunnel1 232.43.211.234`

Si ara tornem a enviar els ssmping des de host4, veiem com **ja ens arriben**.

### Enviant el video (unidireccionalment)

Ara afegim un nou grup 225.1.2.3 destinat al control de la transmissió de missatges.
Configurem els 3 routers multicast pel grup 225.1.2.3:
`RX# smcroute -a "if d'entrada" 0.0.0.0 225.1.2.3 "ifs de sortida"`

I executem netstat -gn a cada un per comprovar que ens haguem afegit als grups.

- Iniciem `udp-receiver` a host2 i host4:
    `host2# udp-receiver --file=big_664_in_host2.mpg --mcast-all-addr 225.1.2.3 --ttl 3 --interface eth1 --portbase 22345`
    `host4# udp-receiver --file=big_664_in_host4.mpg --mcast-all-addr 225.1.2.3 --ttl 3 --interface eth1 --portbase 22345`

- Iniciem `upd-sender` al server:
    `server# udp-sender --file=./big_664.mpg --portbase 22345 --nopointopoint --interface eth1 --ttl 3 --mcast-addr 232.43.211.234 --mcast-all-addr 225.1.2.3 --nokbd --async --max-bitrate 900K --autostart 1 --blocksize 1400`

Si ho tenim tot ben configurat, veiem com se'ns transmet el fitxer als dos hosts. 
Si volem comprovar que sigui el mateix arxiu, executem `md5sum big_664.mpg` al server, host2 (`md5sum big_664_in_host2.mpg`) i host4 (`md5sum big_664_in_host4.mpg`), i si ens retorna el mateix Hash, es tracta del mateix fitxer.


### Arbre BIDIRECCIONAL

Per crear un flow bidireccional, a part de tenir l'arbre que fa forwarding de paquets mcast des de **server** cap a host2 i host4, hem de configurar aquests tal que també tinguem forwarding de **host2** i **host4** cap a server.

Per fer-ho hem de configurar els routers multicast que arriben a cada un d'aquests hosts tal que:

- `R2# smcroute -a tunnel0 0.0.0.0 232.43.211.234 eth1` --> Tot el que ens arribi per tunnel0 a R2, amb destí 232.43.211.234, que faci forwarding a través de eth1. 
- `R2# smcroute -a tunnel1 0.0.0.0 232.43.211.234 eth1` --> Tot el que ens arribi per tunnel1 a R2, amb destí 232.43.211.234, que faci forwarding a través de eth1.
- `R1# smcroute -a eth1 0.0.0.0 232.43.211.234 tunnel0` --> fem forwarding de paquets que entrin per eth1, de qualsevol src, amb destí 232.43.211.234 cap a la interfície tunnel0
- `R3# smcroute -a eth1 0.0.0.0 232.43.211.234 tunnel1` --> fem forwarding de paquets que entrin per eth1, de qualsevol src, amb destí 232.43.211.234 cap a la interfície tunnel1

Ara des de host2 i host4 exeutem:
- `host2# udp-receiver --file=big_664_in_host2.mpg --mcast-all-addr 225.1.2.3 --ttl 3 --interface eth1 --portbase 22345`
- `host4# udp-receiver --file=big_664_in_host4.mpg --mcast-all-addr 225.1.2.3 --ttl 3 --interface eth1 --portbase 22345`

I desde server:
- `server# udp-sender --file=./big_664.mpg --portbase 22345 --nopointopoint --interface eth1 --ttl 3 --mcast-addr 232.43.211.234 --mcast-all-addr 225.1.2.3 --nokbd --autostart 1 --blocksize 1400`

I observem com es produeix la transmissió del fitxer. En aquest cas com que no hem limitat el bitrate la transmissió ha estat molt més ràpida.

El wireshark veiem com a totes les xarxes pròpies dels hosts, tenim un missatge inicial i un final indicant que s'uneixen/abandonen el grup un cop s'ha iniciat/acabat la transmissió de dades.