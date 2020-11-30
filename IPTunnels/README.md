# IP Tunnels Theory

- [IP Tunnels Theory](#ip-tunnels-theory)
- [The Interconnection Problem](#the-interconnection-problem)
- [The Tunneling Solution](#the-tunneling-solution)
- [IP Tunnels](#ip-tunnels)


---

# The Interconnection Problem
<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/problem.png"/>

Des d'una xarxa pública NO podem enrutar a les xarxes privades directament. Això produeix pèrdues de paquets als routers públics.

# The Tunneling Solution

<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/solution.png"/>

Els tunels s'utilitzen per transportar un altre protocol de xarxa encapsulant els seus paquets.

Es poden veure com "Source Routes" que no els hi importen els mecanismes de routing tradicionals.

Quan el tunel es construeix a la capa IP, s'anomena **IP tunnel**.

- L'adreça IP "exterior" (pública) del router, identifica els "endpoints" del tunel.


# IP Tunnels

Qualsevol paquet IP (incloent la informació de les adreces) és encapsulada a dins d'un altre paquet, el qual té el format natiu de la xarxa pública.

<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/tunnel.png"/>

**Mètode d'encapsulació de paquets**:
Quan un "border router" ha d'enviar paquets a través d'un tunel IP, aquest crea un nou "pseudo-dispositiu de xarxa", associat a la @IP exterior.

- Quan un paquet **s'envia** a través d'aquest "pseudo-dispositiu", és encapsulat en un paquet IP i enviat a l'altre "border-router".
- Quan un paquet és **rebut** a través d'aquest pseudo-dispositiu, el payload s'extreu del paquet IP, i es tracta com un paquet real.

*Obs: Els "pseudo-dispositius" només accepten paquets IP de altres "border-routers".*

La **taula de rutes** configurada per saber quins paquets ha d'enviar a través del tunel IP és tal que:

<img src="https://github.com/akaKush/Internet-Basics/blob/main/IPTunnels/Pictures/table.png"/>

Llavors, qualsevol paquet que s'envii a `192.168.2.0/24`, s'enviarà a través del "pseudo-dispositiu" `tunnel0`, encapsulat a dins d'un altre paquet IP, i aquest s'enviarà a l'altre "border-router".

