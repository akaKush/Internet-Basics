# Routing
- [Routing](#routing)
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
    - [Comandos](#comandos)
  - [OSPF](#ospf)
    - [Intro](#intro)
    - [Broadcast Segments](#broadcast-segments)
    - [States and Packets](#states-and-packets)
      - [DOWN](#down)
      - [INIT](#init)
      - [2-WAY](#2-way)

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

### Comandos

(Per millor comprensió dels comandos consultar la pràctica)
Per obrir una consola de Quagga:
- `vtysh`












---
## OSPF

### Intro

OSPF (**Open Shortest Path First**) és un protocol de routing dinàmic que igual que RIP té en compte quines rutes estan disponinbles i quines no, i va actualitzant la seva bbdd segons aquesta info.

La diferència entre RIP i OSPF és que RIP suposa que cada pes dels enllaços és de 1, i en canvi OSPF té en compte la **velocitat dels enllaços**, i **com estan connectades les xarxes** entre elles.

Cada **Router OSPF** guarda les següents **taules**:
- **Neighbour table** amb els routers connectats directament.
- **Topology table**, la qual guarda un mapa de la xarxa (és una bbdd composta per els **estats dels links**).
- **Routing table**, és la RIB, guarda els **shortest paths** i el protocol que utilitza per trobarlos és el de **Dijkstra**.

El temps de convergència és millor que en RIP perquè si l'estat d'un link canvia, aquest canvi s'exten ràpidament per tota la xarxa.

Els routers OSPF poden separar els dominis en diferents **àrees**.
Tots els **routers de dins de cada àrea** saben tota la **topologia** d'aquella àrea, i les altres només **reben vectors de distàncies**.

Normalment el disseny de les Àrees OSPF és tal que:
<img src="https://github.com/akaKush/Internet-Basics/blob/main/Routing/Pictures/arees.png" height=50% width=50%/>

On tenim una **area central**, o **area 0** per on passarà gairebé **tota la informació** que vulguem enviar d'un lloc a l'altre. Seria com un hub que distribueix la informació que li arriba. 
Totes les altres àrees han d'estar connectades a la 0.

**Àrea 1** i **Àrea 2**, i totes les àrees que estiguin connectades amb la Backbone, enviaran els seus **vectors de distàncies** a la àrea 0, a través dels seus **ABR**.

A més a més, els ABR poden agregar rutes, de manera que es minimitzi el trafic que s'intercanvia entre àrees, i el número d'entrades a la taula de rutes.
*Per exemple: El ABR d'una àrea que tingui les xarxes **10.0.0.0/24** i **10.0.1.0/24** pot agregar-les entre elles i fer una sola ruta **10.0.0.0/23***.

Cada cop que es produeix un canvi en la topologia, s'actualitzen les taules de rutes per poder fer la xarxa estable.

**AS i ASBRs**

- **Autonomous System:** El conjunt de xarxes IP sota el control d'un o varios operadors de xarxa que presenten una política de rutes comuna i clarament definida s'anomenen Autonomous Systems.

- **Autonomous System Border Routers**: Redistribueixen xarxes externes a OSPF (static, kernel, RIP, etc.), i poden agregar rutes però només si són externes.

### Broadcast Segments
- Els routers OSPF intercanvien peces d'info de la topologia als **LSAs (Link State Advertisements)**.
- Cada LSA té un **número de serie**.
- Si un router **rep un nou LSA**, aquest **actualitza la seva bbdd dels estats dels links ==> LSDB**
- Els routers **envien** els nous **LSA als seus veins**, per **sincronitzar totes les LSDB**.

Tot i així, per **evitar un excés de LSAs**, OSPF utilitza **Routers Designats** (DR), i **Backup Designated Routers** (DBR) i s'envien els LSAs que s'han d'actualitzar a través de multicast a l'adreça **224.0.0.6** --> (AllDRouters) (en comptes de bcast).
Llavors quan un DR vol contestar amb la LSA a la resta dels veins, utilitza l'adreça **224.0.0.5** --> (AllSPFRouters).

És a dir:
*Quan un router actualitza la seva LSDB, envia un LSA a tots els DR, perquè així aquests l'enviin a tots els altres veins de la xarxa. D'aquesta manera s'evita flooding de LSAs*.

Amb el comando `show ip ospf neighbor` veiem els rols dels veins per comprovar quins són els DR i quins no.

**Com triem el DR i el BDR?**

- Comparem els **Priority ID (Hello packet)** i escollim com a DR el que el tingui més alt.
- Per defecte totes les interfícies tenen **priority ID = 1**.
- Si tots els routers tenen PriorityID=1, llavors comparem **router-id**, el qual també es troba als Hello Packets.
- També escollirem com a DR el router amb router-id més alt.

### States and Packets

**OSPF NO UTILITZA CAP CAPA DE TRANSPORT (UPD/TCP)**.

- Encapsula els datagrames IP directament.
- Té les seves pròpies funcions de **detecció/correcció d'errors**, amb els seus estats i format de paquets:
  - **Les màquines OSPF tenen 6 estats**:
    - DOWN
    - INIT
    - 2-WAY
    - EXSTART
    - LOADING
    - FULL
  - **OSPF utilitza 5 tipus de paquets**:
    - Hello
    - Database Description (DD)
    - Link State Request (LSR)
    - Link State Update (LSU)
    - Link State Acknowledgement (LSACK)
- En qualsevol moment, cada router OSPF estarà en algun dels estats amb cada un dels seus veins.

Quan afegim una xarxa a OSPF, es comencen a enviar packets "Hello" i el router es posa en l'estat `DOWN`.
#### DOWN
Els **Hello** packets s'envien cada 10 segons i contenen:
- router-id
- hello and dead timers
- màscara de red
- area-id
- authentication info
- Llista de active neighbors (router-ids)
- Router Priority
- DR i BDR (link interface addresses)

Quan un router **rep un hello packet** passa a estar al **`INIT` state**.
#### INIT
En aquest estat, quan un router rep un Hello packet d'un dels seus veins, apunta el ruoter-id de l'emissor al seu **hello packet**. D'aquesta manera fem un reconeixement de que els dos routers estan connectats.

Quan el router emissor rep un Hello Packet vàlid d'un dels seus veins, amb el seu router-id inclòs, passa al **`2-WAY` state**
#### 2-WAY
Aquest estat defineix la comunicació bidireccional establerta entre els 2 routers.

En aquest estat **triem els DR/BDR**.