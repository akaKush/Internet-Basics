# Teoria
- [Teoria](#teoria)
  - [SPA](#spa)
    - [Bellman-ford](#bellman-ford)
    - [Distributed BF Version](#distributed-bf-version)
    - [Dijkstra](#dijkstra)
  - [RIP](#rip)
    - [Routing Database](#routing-database)
    - [Update Algorithm](#update-algorithm)
      - [Exemples](#exemples)
    - [Possibles millores](#possibles-millores)
    - [RIP Protocol](#rip-protocol)
      - [RIP-1](#rip-1)
      - [RIP-2](#rip-2)
      - [RIPng](#ripng)
    - [RIP Limitations](#rip-limitations)
  - [Linux/Quagga](#linuxquagga)
- [Pràctica](#pràctica)

En aquest document escriure insights de tot el que crec important a saber sobre els diferents temes que componen Routing.

## SPA
El shortest path algorithm és el problema de trobar el camí més curt entre dos nodes d'un graf, tal que la suma de tots els seus límits sigui minimitzada.

### Bellman-ford
Aquest algoritme treballa en topologies que podrien tenir límits amb pesos negatius, però no pot contenir **cicles** negatius.

**Observacions del BF**:
- Un camí MAI pot ser més llarg que el número de vèrtexs de la topologia.
- Per definició, el camí més curt està format per els camins més curts més petits. Per això, en l'algoritme si busquem el camí més curt entre **(s, v1)** trobarem el camí més curt en la **primera iteració**, perquè BF "relaxa"* a la seva primera iteració.
- **Relaxar** significa intentar trobar diferents límits del camí per trobar una estimació del camí més curt.
- Per la mateixa raó, el camí **(s, v1, v2)** també serà el SP. Aquest cop trobarem el camí com a màxim a la **2a iteració**.
- Finalment si executem una iteració del BF, i **no hi ha cap relaxació**, s'acaba l'algoritme, ja que no és capaç de produir cap SP nou.

### Distributed BF Version

En un BF centralitzat, el node d'origen sempre coneix tota la topologia. Però no sempre és així, de vegades **no sabem la topologia exacta, només les distàncies proveides pels nodes veins.** Dit d'una altre manera, els **nodes veins intercanvien els seus vectors de distàncies dinàmicament**, és a dir cada cop que s'actualitza l'algoritme per si de cas la topologia ha canviat.

Quan un node rep el vector de distàncies dels seus veins, pot utilitzar-les per relaxar el seu límit cap al veí més proper.

Veiem un exemple:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/dBF.png" height=50% width=50%/>

Passos per trobar el **SP entre a i h**:

1. **b relaxa** (a,b) i **envia la update** als seus veins (c i d)
2. **d rep la prèvia update de b**, r**elaxa** i troba que **(a,b,d) és més curt** i envia aquesta update als seus veins (e i g).
3. e i g reben la update, relaxen i envien la update als seus veins. En aquest cas, com que g directament és veí amb h, la update ja arriba al destí del camí.
4. h rep la update de g i relaxa, i com que el camí que vé de g és més curt que el que ve de f, ja hem trobat el SP.

Un cop tots els nodes han enviat i rebut els seus missages, vindria a ser com una iteració completa de l'algoritme centralitzat.

### Dijkstra

Aquest algoritme s'utilitza només en topologies que **NO tinguin límits amb pesos negatius**. En general és més ràpid que el BF, per la raó dita prèviament.

Comprovem el funcionament d'aquest algoritme amb una hipòtesis inductiva. Això significa que hem de provar que el pas inicial de l'algoritme és òptim i llavors cada pas següent sigui també òptim.

## RIP
El **Routing Information Protocol** és un protocol que s'utilitza(va) en **xarxes IP petites/mitjanes**. El RIP es basa en un intercanvi de vectors de distàncies i una versió distribuida de BF algorithm.
Actualment està obsolet deixant pas a nous protocols com OSPF i el OSI.

Per poder utilitzar RIP, necessitem:
- BBDD de routing
- Protocol per intercanviar informació sobre les rutes.
- Un algoritme per actualitzar aquesta informació.

### Routing Database
Cada entitat RIP (normalment routers), manté informació de totes les seves xarxes (i possiblement dels hosts individuals), del seu **RIP routing domain**.

Cada entrada a la bbdd, conté el següent router intermediari (**next hop**) a on els datagrames s'han d'enviar per poder arribar a la destinació final.

Cada entrada a la bdd conté la següent info:
- **Address**: @ de la xarxa o host de destí (@IP/MASK)
- **Metric**: La mètrica o el cost des d'un node fins al destí.
- **Router**: La @IP del **next hop**
- **Interface**: La interfície que s'ha d'utilitzar per arribar al router següent.
- **Timers**: Utilitzats per controlar les dinàmiques la informació de routing.

Normalment *s'utilitza el número de hops com a mètrica*.

Si un RIP està connectat directament a la destinació, llavors la distància entre aquestes dues entitats és **1 hop**. Si hi ha un router entremig, tenim distància 2, i així anar fent.

El **número màxim de hops** per arribar al destí és **15**.
La **distància 16** significa "infinit", que no estan connectats.

### Update Algorithm

Els algoritmes de vector de distàncies reben el seu nom del fet que és possible computar rutes òptimes periòdicament intercanviant els vectors de distàncies de diferents destinacions que cada node té a la xarxa.

La **bbdd de cada node RIP** s'inicialitza amb una **descripció de les entitats RIP** que estan **directament connectades** (amb mètrica=1), i llavors actualitzar-la amb la versió distribuida de BF.

Veiem 2 exemples:

- **Topologia Estàtica**: Quan una entitat RIP rep el vector de distàncies amb les estimacions dels seus veins.
Llavors per cada destinació 'n', el node 'i' compara la mètrica proveida pel veí, amb la seva entrada actual per aquesta destinació. Si aquesta és més petita, el node canvia la mètrica.
D'aquesta manera, desrpés de rebre estimacions de tots els nodes de la xarxa, 'i' tindrà el camí més curt fins a totes les destinacions.

- **Topologia Dinàmica**:
    Si la mètrica inicial és massa petita, l'algoritme de topologia estàtica no funciona bé. Per tant, afegim un mètode per incrementar-la a part de decrementar-la.
    En el cas d'aquest algoritme, és segur executar-lo asíncronament, el que vol dir que cada entitat RIP pot enviar updates dels seus vectors de distància segons el seu propi clock.
    Originalment cada entitat transmetia updates cada 30 segons, però a mesura que les xarxes van anar creixent de mida, ha quedat en evidència que no podia haver-hi una rebentada del tràfic cada 30 segons.

#### Exemples
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/RIP-update.png" height=50% width=50%/>

*Com podem veure a la imatge la xarxa N1 està connectada a 2 routers, RA i RB. Assumim que tots els routers del domini corren RIP. Veurem com la informació que està a N1 es pot distribuir amb RIP. Com que l'ordre de les updates que farà el RIP està distribuit aleatòriament, l'exemple que veurem és només una de les possibles realitzacions de procés d'actualització de RIP.*

1. Com a condició inicial assumim que els únics routers que tenen la **info de N1** són **RA** i **RB**. Llavors, que **RA** és el **primer** router en **enviar info** de N1 en un missatge RIP. A aquesta info li direm **{N1, 1}**, que significa que el missatge RIP enviat per RA inclou una **entrada per N**1, ensenyant que el router només necessita **1 hop**. N1 envia el missatge als seus **one-hop-neighbours** RB i RC. Un cop rebuda aquesta info, **RC actualitza la seva routing database**, ja que la info rebuda desde RA informa d'una nova xarxa a l'abast. RB **no actualitza res**, ja que ja coneix N1 amb un **shorter path**.
2. Ara **RC** decideix que ha d'enviar un missatge RIP i **inclou la nova info** que coneix sobre R1, la qual és {N1, 2}. RC envia aquesta info als seus veins propers: RA i RD. En aquest cas, RD és el primer cop que veu info sobre N1, i per tant actualitza la seva bbdd. RA òbviament no fa res.
3. Ara **RB** envia un missatge RIP incloent la seva info ({N1, 1}) als seus veins: RA i RD. En aquest cas, RA òbviament no fa res, però **RD** sí que a**ctualitzarà** la seva bbdd ja que ha trobat un c**amí més curt** per arribar a N1.
4. Aquest cop, serà **RD** el que enviarà un missatge RIP, però ningú farà res ja que la info que passa RD és pitjor que la que tenen tots els seus veins per arribar a N1.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/RIP-update-2.png" height=50% width=50%/>

Exemple2: Assumim que **un dels enllaços està trencat**.
*Ara veurem com alguns nodes tenen estimacions massa baixes per arribar a N1, i haurem d'incrementar la mètrica.*

1. Condició inicial: Tots els routers ja tenen tota la info que hem extret al primer exemple. En aquest moment, com que **RB detecta que el seu enllaç està trencat**, i per tant actualitza la seva bbdd posant la seva mètrica a **16**.
2. **RA** envia un missatge RIP a un dels seus **one-hop** neighbours, RB i RC. Quan rep la info, **RB actualitza la seva bbdd** pq la info rebuda per RA informa d'un **nou camí** (més curt) per **arribar de nou a N1**. RC no fa res pq ja té la mateixa mètrica.
3. En aquest moment, **RB** envia un missatge RIP als seus veins propers, RA i RD. RA no fa res, però **RD** actualitza  la seva bbdd perquè **tot i obtenir una mètrica pitjor**, prové del seu **next-hop router**.

### Possibles millores

Existeixen millores per les dinàmiques de xarxa en un domini RIP. Aquestes millores són l'utilització de:
- **Timers**: Si un cert Router X s'inclou a la millor ruta per un destí en concret d'un altre router Y, i el router X **ja no està disponible**, l'algoritme possiblement mai reflexaria els canvis al router Y.
Per aquest motiu existeixen **dos timers** associats a cada ruta:
  - **Timeout**: Utilitzat per limitar el temps que una ruta pot quedarse en una bbdd d'un router en concret sense ser actualitzada. El timout normalment són **180 segons**. Si no es rep cap actualització en aquest temps, el **hop count** s'actualiltza a **16** i no es pot arribar més a aquella xarxa.
  - **Garbage-Collection Timer**: S'utilitza per fer-li saber als routers quan una **ruta ja no és valida**. Per default són **120 segon**s, i si durant tot aquest temps la ruta **no canvia** de **hop-count=16**, aquesta ruta **s'elimina de la bbdd**.

- **Count to Infinity**: Ens reflecteix el problema de si tots els links que enllacen amb una certa xarxa són trencats, els Routers enviarien la nova info als seus veins, la qual faria que l'últim router que estava connectat amb la xarxa posi la seva mètrica a 16. El problema vé quan un altre router li envia la seva info. Com que qualsevol dels altres routers tenen una mètrica més petita que 16 per arribar-hi (ja que abans havien fet RIP i tenien les mètriques de just abans de perdre la connexió amb la xarxa), quan aquests li enviin la seva info a RA, aquest s'actualitzarà i inclourà la mètrica del seu veí. El problema és que això és fals, ja que ni RA ni RB poden arribar a la xarxa, i per tant es produirà un bucle on els routers aniran augmentant en 1 la mètrica cada cop fins arribar a infinit. Com que cada cop les bbdd es van actualitzant, els timers no detecten cap problema. És per això que s'ha definit un número que indica "infinit" (16).

- **Split Horizon Rules**: El problema anterior és que RA i RC entren en un bucle de mútua decepció, ja que cada un diu que pot arribar a N1 a través de l'altre. Això es pot prevenir en molts casos, sent cuidadós en quins routers envies la info.
En RIP, la **split horizon rule** serveix per omitir rutes apreses d'un veí en actualitzacions enviades a aquell mateix veí.
Bàsicament mai és útil enviar info sobre una certa ruta al veí que estàs utilitzant com a next-hop per la ruta en qüestió.

- **Triggered Updates**: Split horizon rules with poisoned reverse prevenen que s'entri a un bucle entre 2 routers. Però si en tenim 3 ja no funciona bé. Les triggered updates el que fan és per prevenir això, afegeixen una nova norma que digui que qua un **router canvia la mètrica** d'una ruta, s'ha d'**enviar un missatge de update** als seus veins **immediatament**, encara que no sigui el moment d'enviar-lo.
Això provoca que si quan s'envia un missatge, els routers que el reben actualitzen la bbdd, aquests també hauran d'enviar un missatge immediatament per indicar que la han canviat, i per tant es produirà un **efecte cascada de triggered updates**.
El problema és que al mateix moment que es podrien estar enviant aquestes triggered updates, també podria ser que s'enviessin updates normals.
L'últim que cal tenir en compte sobre les triggered updates és que poden saturar la xarxa de càrrega. Perquè això no passi, s'han utilitzat diversos mecanismes per evitar-ho:
  - Limitar la freqüència de triggered updates.
  - Indicar que les triggered updates no cal que incloguin la taula de rutes sencera.

- **Hold Down Timer**: Cada router comença el **hold-down timer** quan aquest rep informació sobre una xarxa que ja no s'hi pot arribar (16). Fins que el hold-down timer expira, el router descarta qualsevol missatge d'update que indica que la ruta és accessible de nou.
Això fa que els routers no acabin confosos amb les diferents informacions que li arribin.
L'únic problema d'aquest hold-down timer és que força un retard al router a l'hora de respondre a una ruta.

### RIP Protocol
Els missatges RIP s'envien amb **UDP** al **port 520** per **RIP-1** i R**IP-2**, i al **port 521** per **RIPng**.

Els missatges RIP es poden enviar unicast o broadcast/multicast.

Tenim 2 tipus de missatges bàsics:
- **RIP requests**: Requests són missatges enviats des d'una entitat RIP a una altre demanant que li envii tota, o una part de la seva taula de rutes.
- **RIP Responses**: Responses són els missatges enviats d'una entitat RIP a una altre que contenen tota o una part de la taula de rutes. Tot i que es diguin respostes, moltes vegades aquests missatges són enviats sense necessitat d'una request.

#### RIP-1
Utilitza **classful network addresses** perquè no considera l'enviament de màscares. Per aquest problema no suporta subnetting ni supernetting. A part no suporta autentificació de routers, per tant és vulnerable a atacs.

**Paràmetres UDP/IP**
- Les RIP Requests s'envien amb UDP al port **destí** 520. De **source port** poden tenir **qualsevol port** (no té pq ser 520).
- Les RIP Responses, que **responen a una request** s'envien amb un **source port 520** i com a **port de destí** el que ens hagi **indicat la request** com a source.
- Les RIP responses que s'envien sense request, tenen tant el **port destí i origen al 520**.

#### RIP-2
Inclou algunes features noves, anomenades "extensions":
- **Classess Addressing Support i Subnet Mask Specification**
- **Use of multicassting** --> a l'adreça 224.0.0.9
- **Next Hop Specification** --> Indica el next hop que hauria de fer el paquet per arribar a la seva destinació final.
- **Authentication** --> Permet validar la identitat d'un router abans d'acceptar el missatge.
- **Route Tag** --> S'hi pot posar info addicional d'una ruta.

#### RIPng
És la versió RIP compatible amb IPv6, la quan també s'anomena **RIPv6**. Està dissenyat per ser el màxim semblant a RIP-2 (el qual està fet per IPv4).

Les principals diferències entre aquests protocols són:
- Els missatges tenen **128 bits** en comptes de 32 (IPv6)
- El **nº màxim de RTEs** **NO** està restringit a **25**. Està **restringit segons la MTU** de la xarxa on s'enviin els missatges.
- **No suporta autentificació**, ja que aquesta es fa a la capa IP per fer-ho més eficient.
- **No** es poden posar **route tags**.
- **No inclou el next-hop** per defecte. Si aquest és necessari, s'especifica a una entrada de ruta separada.

### RIP Limitations

El protocol RIP no soluciona tots els possibles problemes de Routing. Les principals limitacions són:

- El protocol està limitat a xarxes les quals el **longest path** (network diameter) és de **15 hops**, i això suposant que cada hop té un pes de 1, en cas de ser major, encara portaria més problemes.
- RIP depen del mecanisme "**counting to infinity**" per resoldre algunes situacions. Si el sistema de xarxes té diversos centenars de xarxes, i un **bucle de routing** es forma que les **involucra a totes elles**, es trigaria **massa temps a resoldre** el bucle, o **consumiria** tota la **bandwidth**.
- RIP **utilitza mètriques fixes** per comparar rutes alternatives. **No és apropiat** per situacions on les rutes necessiten ser escollides **basades en paràmetres en temps-real**, com retards, fiabilitat o càrrega.

---
## Linux/Quagga
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/quagga.png" height=50% width=50%/>

En sistemes Linux, el protocol RIP treballa amb 3 espais de memòria (bbdd), les quals mantenen un registre de les millors rutes per optimitzar el temps empleat en triar el SP per arribar al destí, cada cop que tenim un paquet a enviar.

Aquestes 3 taules són:

- **RIB**: BBDD on tots els protocols envien les seves rutes més curtes, i es va actualitzant cada cop que un router troba una nova ruta més curta sigui amb el protocol que sigui.
- **FIB**: És la BBDD que guarda el registre únicament amb les millors rutes de totes les que hi ha a la RIB. De tots els protocols que tinguem, s'escullen les millors que tenim a la RIB i s'envien a la FIB per mantenir un registre de les millors rutes.
- **Kernel Forwarding Cache**: És una memòria del kernel que ens permet guardar dinàmicament les rutes utilitzades prèviament per anar més ràpid a enviar els paquets que tenen destins que s'han fet servir fa poc.

**Observacions**
A la RIB pot ser que hi hagi 2 entrades cap al mateix destí que indiquin camins diferents, segons quin sigui el protocol que ha trobat aquell camí.
Però això a la FIB no passarà, ja que per escullir quina de les dues rutes és millor, utilitza una taula que es diu **Administrative Distances**, la qual ens indica quin protocol té prioritat sobre els altres, i llavors decideix la millor ruta per afegir a la FIB.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/admin_dist.png" height=50% width=50%/>


(Per millor comprensió dels comandos consultar la pràctica)
Per obrir una consola de Quagga:
- `vtysh`


---

# Pràctica

L'objectiu de la pràctica és entendre el **RIP-1** amb el següent escenari:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-21%20a%20les%2019.12.20.png"/>

Les interfícies estan configurades amb les IPs corresponents al hostID de cada màquina. Per exemple, la interfície **eth1 de r222** és **192.168.3.222**.

Per tant tenim les següents IPs a cada host:

- **h223**
  - eth1 --> 192.168.0.223
- **r222**
  - eth1 --> 192.168.3.222
  - eth2 --> 192.168.5.222
  - eth3 --> 192.168.0.222(/25)
- **r1**
  - eth1 --> 172.16.0.1 (/16)
  - eth2 --> 192.168.2.1
  - eth3 --> 192.168.3.1
  - eth4 --> 192.168.4.1
- **h11**
  - eth1 --> 192.168.4.11
- **r3**
  - eth1 --> 192.168.5.3
  - eth2 --> 192.168.2.3
  - eth3 --> 192.168.1.3
  - eth4 --> 192.168.0.3
- **h33**
  - eth1 --> 192.168.0.33
- **r4**
  - eth1 --> 192.168.1.4
  - eth2 --> 172.16.0.4 (/16)
- **r5**
  - eth1 --> 172.16.0.5 (/16)

Comencem la simulació amb `simctl rip start` i executem `simctl rip exec initial` per la config basica mencionada a dalt.

1. Execute a ping from router r3 to 192.168.2.1, 192.168.3.1 and 192.168.4.11. Discuss the results.

```
root@r3:~# ping -c 1 192.168.2.1                         
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
64 bytes from 192.168.2.1: icmp_req=1 ttl=64 time=0.276 ms

--- 192.168.2.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.276/0.276/0.276/0.000 ms

root@r3:~# ping -c 1 192.168.3.1             
connect: Network is unreachable

root@r3:~# ping -c 1 192.168.4.11
connect: Network is unreachable
```

El primer ping arriba correctament però els altres dos no. A wireshark veiem com només tenim els paquets corresponents al primer ping, ja que aquest s'envia a un host que pertany a la mateixa xarxa a la qual pertany r3 (estan connectats directament).

Els dos darrers pings no arriben perquè r3 no sap per on enviar els paquets corresponents a aquelles xarxes, només té entrades a la seva taula de rutes per arribar a les xarxes amb les que està directament connectat:
```
root@r3:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth4
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth3
192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 eth2
192.168.5.0     0.0.0.0         255.255.255.0   U     0      0        0 eth1
```

2. Start a capture on SimNet2. In r3, open the Quagga command tool and add the networks 192.168.1.0/24 and 192.168.2.0/24 to RIP:
```
root@r3:~# vtysh
r3# configure terminal
r3(config)# router rip
r3(config-router)# version 1 r3(config-router)# network 192.168.1.0/24 r3(config-router)# network 192.168.2.0/24
```
Explain the RIP response messages that you observe in SimNet2. In your explanation include the MAC addresses (L2), IP addresses (L3), ports (L4) and the RIP information.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-21%20a%20les%2020.15.24.png"/>

Veiem com només tenim request i response de la xarxa 192.168.2.0, els quals utilitzen RIPv1.
Les adreces de L2 són desde fe:fd:00:00:03:02 (r3) fins a ff:ff:ff:ff:ff:ff (broadcast).
Les adreces de L3 són desde 192.168.2.3 (r3) fins a 192.168.2.255 (broadcast).
Els ports utilitzats en L4 són el UDP 520 tant de source com destination.

3. Capture on SimNet2 and type the following RIP command in r3:
   `r3(config-router)# neighbor 192.168.2.1`
  Describe what the command does and explain why you receive an error message (ICMP) from 192.168.2.1. To finish this exercise disable the neighbor with:
  `r3(config-router)# no neighbor 192.168.2.1`

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-21%20a%20les%2020.24.19.png"/>

Veiem com rebem missatges ICMP d'error que ens indiquen "PORT UNREACHABLE".
Si ens fixem en les característiques del missatge, veiem les adreces d'origen i destí, sent 192.168.2.1 (r1) el que envia el missatge d'error cap a r3, indicant que no pot accedir al port.

4. Capture in SimNet2 and then, in r3, set eth3 down: 
`root@r3:~# ifconfig eth3 down`
Describe the RIP response messages captured waiting at least for 2 minutes.

Sortim de quagga mitjançant `exit` i executem el comando indicat. Veiem els següents missatges al cap d'un parell de minuts:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2011.44.27.png"/>

Només tenim RIP responses, enviades desde r3 cap a broadcast periòdicament.

5. In r3, set eth3 up. Describe and explain the RIP response messages that you capture on SimNet2 at least during 30 seconds.

També veig només RIPv1 responses...

6. In r3, remove the network 192.168.1.0/24 from RIP.
Describe the RIP response messages that you observe in SimNet2. Wait at least for 2 minutes to end the capture.
Finally, in r3, remove also the network 192.168.2.0/24 from RIP.

Veiem missatges IGMP Multicast per unir-nos al grup 224.0.0.9, i seguidament RIP responses.

Si ara deixem la xarxa 192.168.2.0/24 (`no network 192.168.2.0/24`) veiem 2 missatges IGMP per abandonar el mateix grup d'abans, i **cap missatge RIP més**.

7. *initial* Capture in all the networks to which r1 is connected and, in this router, type the following RIP command: `r1(config-router)# network 192.168.0.0/16`
In which networks do yo see a RIP response packet? Why?
Describe the RIP messages including L2 MAC addresses, L3 IP addresses, L4 ports and the RIP information.

Veiem response paquets a totes les xarxes menys la SN6, ja que aquesta correspon a la xarxa 172.16.0.0/16, i per tant no entra dins el rang anterior.

Els missatges capturats a les altres xarxes són així:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2012.40.57.png"/>

Veiem la info típica dels missatges RIP, enviats desde r1 cap a broadcast, tant en L2 com L3, i utilitzant els ports 520 de destí i origen.


8. Do you think that it makes sense to send RIP response messages through the 192.168.4.0/24 network? For example, it makes sense to send RIP response messages to h11?

No, en realitat cap a la xarxa SN4 no faria falta enviar missatges RIP, ja que no hi ha cap dispositiu que funcioni com a router, i per tant crec que allà no faria falta enviar tràfic RIP.

9. Capture on SimNet2 and type the following command in r1: `r1(config-router)# no network 192.168.0.0/16`
Wait for a few seconds, do yo see any RIP message? why?
Then, in r1, in less than 2 minutes, activate RIP for the networks 192.168.2.0/24 and 192.168.3.0/24. Explain the RIP messages captured on SimNet2.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2013.01.29.png"/>

Al principi en el moment d'abandonar la xarxa 192.168.0.0/16, veiem simplement dos missatges d'abandonar el grup 224.0.0.9, i ja cap missatge més RIP. Això és perquè abandonem el grup multicast per on enviem la info de RIP.

Seguidament activem les altres dues xarxes, i veiem els missatges RIP request i response corresponents, així com els IGMP per unir-se l grup multicast.


10. Capture on SimNet2 and SimNet3. In r3, start RIPv1 for the network 192.168.5.0/24.
Do you observe RIP messages on SimNet2 from r3? why?
Now, in r3, start RIPv1 for the network 192.168.2.0/24.
For the captured traffic, explain the networks that you observe in the RIP response messages. In addition, explain if triggered updates, split horizon and poison reverse are activated in r1 and how can you know this.

Després d'executar `network 192.168.5.0/24` a r3, ens retorna el següent missatge per terminal:
`Warning: closing connection to ripd because of an I/O error!`

... (s'havia reiniciat l'escenari?)


1.  To set this configuration you can use the labels of simctl `initial` and then `ripv1-a`. 
Try a ping from r3 to 192.168.3.1.
Does it work? why?
Try a ping from r3 to 192.168.4.11. Does it work? why?

Explain the RIB and FIB of the router r3.
• The RIB entries for RIP can be shown entering in Quagga and typing:
     `router# show ip rip`
• The FIB/RIB entries for all the protocols can be shown in Quagga with the command:
     `router# show ip route`
You can also see the FIB on a linux command-line typing:
`root@r3:~# route -n`

Analitzem la configuració `ripv1-a`, i les taules RIB i FIB.

A la **taula RIB corresponent només al mecanisme RIP** veiem el següent: (`r3# show ip rip`)
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2013.39.16.png"/>

Podem veure com tenim 2 xarxes directament connectades (.5.0 i .2.0), i que podem accedir a 192.168.3.0/24 a través de 192.168.2.1.

Per veure les **rutes de TOTS els protocols a les taules RIB i FIB** fem `r3# show ip route` i veiem el següent:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2013.43.10.png"/>

A més de les rutes anteriors, aquí veiem també la interfície de loopback i totes les que faltaven i estan directament connectades.

Finalment si executem desde terminal (fora de quagga) `$ route -n` també veiem les entrades de la taula FIB:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2013.44.17.png"/>



12. Capture on SimNet4 and set the interface eth4 in r1 as passive. Then, add the network 192.168.4.0/24 to RIP and try again a ping from r3 to 192.168.4.11.
Does it work now? why?
Do you see RIP messages on SimNet4? Why? Explain the RIB and FIB of the router r3.

No ha funcionat el ping. A les taules FIB/RIB de r3, no apareix SN4, tot i haver-la afegit amb `r3(config-router)# network 192.168.4.0/24`.

A r1, al haver activat la interfície eth4 com a passiva, fa que els missatges RIP no es transmetin cap allà.

13. `initial`, `ripv1-b` Capture on SimNet3 and SimNet5.
In r3, add 192.168.5.0/24 to RIP and also add 192.168.0.0/24 to RIP but as passive.
In r222, add 192.168.3.0/24 and 192.168.5.0/24 to RIP.

Procés per afegir les xarxes a RIP a r3, i la segona com a passiva:
```
r3@root# vtysh
r3# configure terminal
r3(config)# router rip
r3(config-router)# version 1
r3(config-router)# network 192.168.5.0/24
r3(config-router)# network 192.168.0.0/24
r3(config-router)# passive-interface eth4
```

A r222 fem el mateix però amb les xarxes que ens indica.

14. Another way of including routes to RIP is to use “redistribution”. Redistribute the connected networks of r1 and r222 using the following Quagga command:
`router(config-router)# redistribute connected`

Describe the entries (if any) of the networks 172.16.0.0/16 and 192.168.0.128/25 present on the RIB of r1 and r222. 
Observing the RIB of the different routers, explain if you notice any differences between the networks distributed with redistribution and the networks distributed with the network command.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/Captura%20de%20Pantalla%202021-05-22%20a%20les%2014.28.50.png"/>

Tenim entrades per la xarxa 172.16.0.0 als dos routers, en el cas del r222 ens indica que hi accedim a través de r1, i a r1 ens diu que la tenim directament connectada.

En el cas de la xarxa 192.168.0.128/25, tenim una entrada a la taula de r222, directament connectades, però a r1 cap.
Es deu a que tenim entrada per la xarxa 192.168.0.0/24, a través de r3, i dins d'aquest rang inclou totes les adreces de 192.168.0.128/25.