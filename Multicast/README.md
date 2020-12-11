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
- [IGMP](#igmp)
- [MBONE](#mbone)
- [Multicast Routing](#multicast-routing)

# Intro
## What is Multicasting?

Quan fem multicasting tenim almenys un host que envia, i múltiples que reben, anomenats **multicast group**.

Al **multicast routing** el router pot enviar el paquet rebut per diverses de les seves interfícies.

- Multicast NO ÉS Broadcast

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/1.png"/>


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

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/2.png"/>

- *w* entre 0 i 127  ==>  w == NetID,  x.y.x == HostID
- Existeixen 126 xarxes de tipus A, amb 16.777.214 hosts
- Xarxes gegants
- La xarxa *10* és privada: 10.0.0.0 - 10.255.255.255
- La xarxa *127* és interna (loopback): 127.0.0.0 - 127.255.255.255

### Class-B Addresses

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/3.png"/>

- *w* entre 128 i 191  ==>  w.x == NetID,  y.z == HostID
- 16.384 xarxes de classe B, amb 65.534 hosts
- Xarxes mitjanes/llargues
- Adreces privades: 172.16.0.0 - 172.32.255.255

### Class-C Addresses

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/4.png"/>

- *w* entre 192 i 223  ==>  w.x.y == NetID,  z = HostID
- 2.097.152 xarxes de classe C, 254 hosts a cada una
- Xarxes petites
- Adreces privades: 192.168.0.0 - 192.168.255.255


### Class-D & Class-E

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Multicast/Pictures/5.png"/>


# IGMP





# MBONE





# Multicast Routing