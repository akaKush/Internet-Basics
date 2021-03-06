# VLAN, Logical Link Control, Switching

## Pràctica 1 TCGI

### Exs 1 i 2

Iniciem l'escenari amb: 
```
simctl switching-vlan sh
start
```

Si volem veure les màquines virtuals disponibles:
`vms`

Escollim les màquines que ens interessen, en aquest cas volem les dues màquines disponibles de Alice (0 i 1) i una de Bob i una de Carla:
```
get alice 0
get alice 1
get bob
get carla
```

En cada una d'elles escribim les credencials per accedir:
`root`(usuari) i `xxxx`(pw)

Ara ja podem crear els xats corresponents entre ells.
Primer executem `ifconfig` als diferents usuaris per veure la seva adreça MAC corresponent, i així poder enviar missatges al xat que volem.

Evidentment els dos terminals de Alice estan a la mateixa adreça **fe:fd:00:00:01:00**

Per tant des de alice 0 i 1 executem `server-chat-LLC1.py` i desde bob i carla executem `client-chat-LLC1.py -d fe:fd:00:00:01:00` i ja tindrem el xat creat entre alice-bob i alice-carla.

Si executem wireshark, a SimNet0 i enviem un missatge per exemple entre alice i carla, veiem el següent:

![LLC alice - carla](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.05.46.png)

Finalment tanquem cada terminal amb `exit` i acabem l'exercici.

### Ex 3
En aquest exercici crearem un xat utilitzant **BRIDGES** i veurem l'algoritme de **MAC LEARNING** dels switches, per aprendre com estan situats els hosts en l'escenari de la pràctica.

![Escenari](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.23.11.png)

Obrim els terminals de L1, L2 i L3 (`get LX`) i entrem les credencials de cada un.

A L1 executem el següent comando per a **veure la taula MAC del switch L1**, actualitzada cada 5 segons:
`watch -n 5 brctl showmacs br1`

![MAC table](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.25.42.png)

Obrim alice i bob i enviem un missatge entre ells utilitzant `send-frame-LLC1.py` i visualitzem SimNet1 i SimNet2 a Wireshark per veure què passa:

![send-frame](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-12%20a%20les%2017.41.50.png)

A les dues xarxes podem veure missatges similars, on ens indica la MAC de destí, la d'origen i també els SAPS de source i destination (0x88).

Si ara tornem a mirar la taula de MAC, podem veure com ha après l'adreça de bob, i la ha guardat a la taula.


### Ex 4
En aquest exercici crearem 2 VLANs, una per connectar Alice, Bob i Carla entre ells, i l'altre per connectar David, Eric i Frank, de forma separada a la primera VLAN. 

Per aixecar VLANs i connectar-les entre elles mitjançant Bridges, necessitem aixecar les interfícies primer per les quals volem crear aquestes VLANs. Ho fem de la següent manera:

![creant VLANs a L3](https://github.com/akaKush/Internet-Basics/blob/main/VLAN/images_p1/Captura%20de%20Pantalla%202021-03-13%20a%20les%2014.20.04.png)

Aquesta foto ensenya els comandos executats a L3, per a crear les dues VLANs 10 i 20, però aquests s'han de repetir tant a L1 com a L2, per indicar a cada router per quines interfícies està connectada aquella VLAN, quins hosts pertanyen als bridges, i per finalment assignar una adreça IP a cada una d'aquestes VLANs.