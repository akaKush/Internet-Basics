# Multicast

- [Multicast](#multicast)
- [Intro](#intro)
  - [What is Multicasting?](#what-is-multicasting)
  - [Multicast IP Service](#multicast-ip-service)
- [Multicast Addressing](#multicast-addressing)
  - [IP Address Structure (w.x.y.z)](#ip-address-structure-wxyz)
    - [Class-A Addresses](#class-a-addresses)
    - [Class-B Addresses](#class-b-addresses)
    - [Class-C Addresses](#class-c-addresses)
    - [Class-D & Class-E](#class-d--class-e)
  - [Mapping class D to Ethernet](#mapping-class-d-to-ethernet)
  - [Multicast Scoping](#multicast-scoping)
- [IGMP](#igmp)
  - [IGMP Message Types](#igmp-message-types)
- [MBONE](#mbone)
- [Multicast Routing](#multicast-routing)

# Intro
## What is Multicasting?

Quan fem multicasting tenim almenys un host que envia, i múltiples que reben, anomenats **multicast group**.

Al **multicast routing** el router pot enviar el paquet rebut per diverses de les seves interfícies.

- Multicast NO ÉS Broadcast

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/1.png" height=50% width=50%/>


## Multicast IP Service

- Enviament: Es fa de manera normal amb IP, especificant una **adreça IP de multicast** com a destí.
  - Indicant la interfície de sortida
  - Indicant un TTL en el paquet de sortida
  - Habilitant/Deshabilitant la interfície de loop-back si el host que envia el paquet pertany/no pertany al multicast group.
- Recepció: Rebem paquets multicast en grups de manera normal (IP-Receive operation)
  - Join-IP-Multicast-Group
  - Leave-IP-Multicast-Group



# Multicast Addressing

- Transmission:
  - Un paquet "IP multicast packet" es transmet com a capa d'enllaç multicast, en aquells links que es permet multicast.
  - El destí de l'adreça de la capa d'enllaç és determinat per un algoritme específic del tipus d'enllaç.

- Recepció:
  - Multiples passos són necessaris per rebre els paquets multicast en un enllaç en particular, com modificar els filtres de recepció d'interfícies LAN.
  - Els routers multicast són capaços de rebre tots els IP multicasts en un enllaç, sense saber per avançat quins grups es faran servir.


## IP Address Structure (w.x.y.z)

Les adreces que estudiarem son les v4, és a dir **32 bits**.

Aquests 32 bits estan dividits en 2 parts:
- **NetID**
- **HostID**

Per què tenim 2 parts? Perquè si veiem que la @IP de destí té la mateixa NetID que la meva, significa que el host de destí està en la meva mateixa xarxa d'enllaç.
Llavors, el HostID identifica el host específic a dins d'aquesta xarxa.

### Class-A Addresses

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/2.png" height=50% width=50%/>

- *w* entre 0 i 127  ==>  w == NetID,  x.y.x == HostID
- Existeixen 126 xarxes de tipus A, amb 16.777.214 hosts
- Xarxes gegants
- La xarxa *10* és privada: 10.0.0.0 - 10.255.255.255
- La xarxa *127* és interna (loopback): 127.0.0.0 - 127.255.255.255

### Class-B Addresses

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/3.png" height=50% width=50%/>

- *w* entre 128 i 191  ==>  w.x == NetID,  y.z == HostID
- 16.384 xarxes de classe B, amb 65.534 hosts
- Xarxes mitjanes/llargues
- Adreces privades: 172.16.0.0 - 172.32.255.255

### Class-C Addresses

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/4.png" height=50% width=50%/>

- *w* entre 192 i 223  ==>  w.x.y == NetID,  z = HostID
- 2.097.152 xarxes de classe C, 254 hosts a cada una
- Xarxes petites
- Adreces privades: 192.168.0.0 - 192.168.255.255


### Class-D & Class-E

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/5.png" height=50% width=50%/>

## Mapping class D to Ethernet

Tenim una adreça de classe D (multicast) que s'ha d'encapsular en un paquet de capa 2 (física, ethernet) per poder-lo enviar.

Per fer-ho simplement es mapeja exactament els 23 bits de l'adreça de multicast als últims 23 bits dels 48 que té una adreça Ethernet. (Els 9 bits restant corresponen a els 4 primers `1110` que indiquen que es tracta de multicast, i els 5 sobrants són bits que no s'utilitzen per res, veure imatge següent)

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/6.png" height=50% width=50%/>

*Nota: Com que els bits mapejats són els últims, poden haver-hi 32 adreces IP amb la mateixa adreça MAC (degut als primers bits que no són els mapejats), i per això el host ha de comprovar l'adreça IP multicast que li arriba i descartar els paquets si no està registrada.*

Finalment, cada adreça Multicast té la següent estructura:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/7.png" height=50% width=50%/>

I la capa MAC, a les interfícies Ethernet segueix el següent procediment per determinar on enviar els paquets:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/8.png" height=50% width=50%/>

## Multicast Scoping

Tenim 2 maneres de determinar l'abast de les transmissions:

- Abast basat en TTL
  - Els routers tenen un límit de TTL fins a descartar el paquet si TTL<= TTL_límit
- Abast Administratiu
  - S'utilitza una porció de l'espai reservat a adreces de classe D (239.0.0.0 - 239.255.255.255)
  - Realment local al domini administratiu. Es poden reutilitzar adreces.
    - No s'utilitza per tràfic global d'Internet
    - S'utilitza per limitar l'abast del tràfic multicast.
    - Les adreces reutilitzables poden estar a diferents llocs al mateix temps per diferents sessions de multicast.


# IGMP

IGMP o **Internet Group Management Protocol** és utilitzat pels hosts per registar la seva "dynamic multicast group membership", és a dir serveix per indicar quan un host pertany a un grup de multicast o no.

Els **routers** també utilitzen IGMP per descobrir aquests membres del grup de multicasting.

## IGMP Message Types

Perquè els hosts indiquin si pertanyen a un grup de multicast o no, utilitzen diversos missatges que intercanvien amb l'emissor dels missatges multicast:

- **Membership Report**: L'envia el host per indicar que vol unir-se a un grup de multicast
- **Leave Report**: El mateix per indicar que abandona el grup quan el host ja no està interessat en ser-hi. Mentres hi hagi algun host interessat en el procés del grup G, aquest no eliminarà la llista.
- **Query**: Quan un router vol assegurar-se de si els hosts continuen estan interessats en el grup o no, envia queries a la seva llista de hosts. Si cap respon ni envia cap membership report, aquest elimina la llista del grup.
  - **General**: Serveix per si per exemple un membre d'un grup s'apaga, aquest no podrà enviar el Leave Report, i es quedaria a dins el grup consumint recursos inútilment. Per això el router envia General Queries cada 125s per defecte a tots els grups. Si els hosts encara estan interessats en estar al grup, envien un altre Membership Report i el seu router, indica al router que ha fet la general query que encara tenen a hosts escoltant.
  - **Special**: El router envia una query a un grup en concret.

Per estalviar recursos, **IGMPv1 i IGMPv2** quan tenim una xarxa on més d'un host està connectat al grup, només cal que cada cop que tenim una query, 1 dels hosts respongui, ja que així ja continuarà el router de la xarxa connectat al grup.

Això a la versió **IGMPv3** s'elimina, i tots els hosts responen a les queries, però s'utilitza una **nova adreça**: 224.0.0.22 (on només escolten els routers IGMPv3), i tots els hosts envien els seus reports a aquesta adreça en comptes del grup que desitgen estar connectats.

# MBONE





# Multicast Routing