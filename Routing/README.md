# Routing
- [Routing](#routing)
  - [SPA](#spa)
    - [Bellman-ford](#bellman-ford)
    - [Distributed BF Version](#distributed-bf-version)
    - [Dijkstra](#dijkstra)
  - [RIP](#rip)
  - [Linux/Quagga](#linuxquagga)
  - [OSPF](#ospf)

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



## Linux/Quagga

## OSPF