# Multicast

- [Multicast](#multicast)
- [Intro](#intro)
  - [What is Multicasting?](#what-is-multicasting)
  - [Multicast IP Service](#multicast-ip-service)
- [Multicast Addressing](#multicast-addressing)
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





# IGMP





# MBONE





# Multicast Routing