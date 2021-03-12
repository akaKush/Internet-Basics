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
