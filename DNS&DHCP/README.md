# DNS & DHCP

Per una explicació extensiva de la teoria, podeu veure el següent enllaç: MEDIUM

Per a la pràctica partirem del següent escenari:
![Escenari DNS](https://github.com/akaKush/Internet-Basics/blob/main/Basic%20Network%20Apps/images/Captura%20de%20Pantalla%202021-03-15%20a%20les%200.16.42.png)

**Exercici 1**

Primer responem a on es trobarien els següents RRs dins el nostre escenari per ubicar-nos:
- A record for peter.example.com --> nsce
- A record for peter.left.example.net --> nsne
- A record for peter.right.example.net --> nsner
- SOA record of right.example.net --> nsner
- PTR record for peter.example.com --> nsce
- MX record for right.example.net --> nsner
- MX record for left.example.net --> nsne
- NS record for right.example.net --> nsne

Comencem amb els exercicis.
Iniciem la simulació amb `simctl dns-basic start` a la màquina virtual.

Per carregar la configuració de l'escenari, i no haver-la de configurar nosaltres mateixos executem `exec initial` dins la shell del simctl dns-basic.

(en cas que vulguessim restablir tots els processos dels name servers, netejar caches, i recarregar la configuració inicial ho podem fer amb `exec resetbind`).

1. Get a console at alice and looking at the configuration explain which is the name server used by this host.
   
   ![alice name servers](https://github.com/akaKush/Internet-Basics/blob/main/Basic%20Network%20Apps/images/Captura%20de%20Pantalla%202021-03-15%20a%20les%200.16.42.png)

   Veiem clarament com el nameserver de alice està ubicat a la IP 10.0.0.21, el qual pertany a **nsce**.

   Si executem també dig alice.example.com, veiem els RR que pertanyen tant a alice com a example.com, i ens donen la informació sobre qui és el name server de alice, així com on està ubicat, i també la IP de alice.

2. Identify which is the server of the zone example.com and describe the configuration of this zone.
   
   El servidor de example.com és **nsce**.
   Per veure la seva configuració, des de la terminal corresponent a nsce, executem `cat /etc/bind/named.conf`, i veiem el següent:
   ![configuracio nsce](https://github.com/akaKush/Internet-Basics/blob/main/Basic%20Network%20Apps/images/Captura%20de%20Pantalla%202021-03-15%20a%20les%200.16.42.png)

   On ens diu que la info per arribar al servidor ROOT està a `db.root`, i que la info per arribar a la zona de **example.com** està a `/etc/bind/db.com.example`

   Si ara ens fixem en la info que hi ha dins d'aquest últim fitxer (`cat /etc/bind/db.com.example`):

   ![info zona example.com](https://github.com/akaKush/Internet-Basics/blob/main/Basic%20Network%20Apps/images/Captura%20de%20Pantalla%202021-03-15%20a%20les%200.16.42.png)
   
   Veiem el següent:
   1. L'origen és example.com, per tant tot arreu on surti una "@", significa example.com
   2. el registre SOA amb el mail de l'administrador de la zona, i tots els seus paràmetres.
   3. Els records A i NS de nsce mateix.
   4. un record CNAME de **david**, que tradueix a david.example.net
   5. Els records MX que indiquen els mail servers (i a sota els seus A records, indicant on es troben aquests)
   6. Finalment un A record per saber on es troba alice.

3. In nsce using the command netstat, identify the name of the DNS server process.
   
   ????

4. In this exercise, we analyze a simple query from alice. In first place, reset the name servers processes and then, capture with wireshark tap0 and explain the output of the following command: `alice$ dig alice.example.com`
   
   L'output del comando anterior el podem veure a la resposta de l'exercici 1.

   Veiem la captura del wireshark:
   ![wireshark alice](https://github.com/akaKush/Internet-Basics/blob/main/Basic%20Network%20Apps/images/Captura%20de%20Pantalla%202021-03-15%20a%20les%200.16.42.png)

   
   