# Pràctica de Routing Multicast

- [Pràctica de Routing Multicast](#pràctica-de-routing-multicast)
  - [0.3 Inicialitzem escenari](#03-inicialitzem-escenari)
  - [0.4 Adreces IP multicast i DNS](#04-adreces-ip-multicast-i-dns)
  - [0.5 Adreces MAC multicast](#05-adreces-mac-multicast)
  - [0.6 Configuració Multicast](#06-configuració-multicast)

## 0.3 Inicialitzem escenari
Iniciem la simulació amb `simctl ipmulticast start` i obtenim el següent escenari:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/escenari.png" height=50% width=50%/>

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
- Grup Multicast per control i fiabilitat: **255.1.2.3**

Obrim una altre consola de server: `simctl ipmulticast get server 1`.

Des de la consola 0, executem 
```server# udp-sender --file=./big_664.mpg --min-clients 1 --portbase 22345 \
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


