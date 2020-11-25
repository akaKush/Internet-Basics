<h1>Firewalls i Traducció d'Adreces de Xarxa</h1>


- [Firewalls](#firewalls)
  - [Introducció](#introducció)
  - [Objectius dels Firewalls](#objectius-dels-firewalls)
  - [Packet Filtering Firewalls](#packet-filtering-firewalls)
  - [Stateful Filtering Firewalls](#stateful-filtering-firewalls)
  - [Proxy Firewalls](#proxy-firewalls)
  - [Next Generation Firewalls](#next-generation-firewalls)
- [NAT](#nat)
  - [Raons per utilitzar NAT](#raons-per-utilitzar-nat)
  - [Intro](#intro)
  - [Utilització de NAT](#utilització-de-nat)
  - [Tipus de NAT](#tipus-de-nat)
  - [Conseqüències d'utilitzar NAT](#conseqüències-dutilitzar-nat)
  - [Problemes i limitacions de NAT](#problemes-i-limitacions-de-nat)
  - [Exemples](#exemples)
  - [Conclusions](#conclusions)
- [Firewalls & NAT in Linux](#firewalls--nat-in-linux)
  - [Netfilter](#netfilter)
  - [Netfilter Operational](#netfilter-operational)
  - [Chain & Rule Configuration](#chain--rule-configuration)

---

# Firewalls
## Introducció

**Què és un Firewall?**<br><br>
Els routers distribueixen el tràfic  segons un conjunt de normes que simplement apunten a l'adreça de destí.<br>
Però i sí necessitem que el tràfic estigui controlat segons uns altres paràmetres?<br>
Exemples:
- Equipament a la xarxa externa només pot accedir el servidor web públic.
- El departament de Marketing pot accedir al server privat però no al públic.
- El tràfic d'un dispositiu en concret pot accedir a la xarxa de marketing.

Una primera solució proposada a aquest problema seria restringir el tràfic produit per cada un dels dispositius.
- Extramadement difícil de fer actualitzacions
- No és escalable
- De vegades no és ni possible

Per tant un **firewall** el podem entendre com un **controlador d'accés o PEATGE**, el qual ens divideix la xarxa en 2:
- Internal Network (Podria ser la xarxa interna d'una empresa) --> Considerada "segura i fiable"
- External Network (Internet) --> Considerada "insegura"

<img src="https://github.com/akaKush/Internet-Basics/blob/main/Firewalls%26NAT/Pictures/firewall.png"/>

## Objectius dels Firewalls

- **Punt d'estrangulament únic per el tràfic de xarxa.** Tot el tràfic tant entri o surti ha de passar pel firewall.
  - Escala degudament, i minimitza el risc de nous perills.

- **Normes de control d'accés**. Només el tràfic autoritzat està permès a passar el firewall.
  - Facilitat l'administració.

- **Seguretat al dispositiu**. El firewall en sí mateix s'assumeix que és immune a penetracions, ha de ser especialment segur.

## Packet Filtering Firewalls


## Stateful Filtering Firewalls


## Proxy Firewalls


## Next Generation Firewalls




# NAT
## Raons per utilitzar NAT


## Intro


## Utilització de NAT


## Tipus de NAT


## Conseqüències d'utilitzar NAT


## Problemes i limitacions de NAT


## Exemples


## Conclusions


# Firewalls & NAT in Linux
## Netfilter


## Netfilter Operational


## Chain & Rule Configuration
