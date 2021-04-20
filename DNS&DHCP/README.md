# DNS & DHCP

Per una explicació extensiva de la teoria, podeu veure els següents enllaços:
- <a href="">DNS</a>
- <a href="">DHCP</a> 

## DNS

Per a la pràctica partirem del següent escenari:
![Escenari DNS](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-18%20a%20les%2010.34.13.png)

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
   
   ![alice name servers](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-18%20a%20les%2011.46.25.png)

   Veiem clarament com el nameserver de alice està ubicat a la IP 10.0.0.21, el qual pertany a **nsce**.

   Si executem també dig alice.example.com, veiem els RR que pertanyen tant a alice com a example.com, i ens donen la informació sobre qui és el name server de alice, així com on està ubicat, i també la IP de alice.

2. Identify which is the server of the zone example.com and describe the configuration of this zone.
   
   El servidor de example.com és **nsce**.
   Per veure la seva configuració, des de la terminal corresponent a nsce, executem `cat /etc/bind/named.conf`, i veiem el següent:

   ![configuracio nsce](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-18%20a%20les%2011.49.40.png)

   On ens diu que la info per arribar al servidor ROOT està a `db.root`, i que la info per arribar a la zona de **example.com** està a `/etc/bind/db.com.example`

   Si ara ens fixem en la info que hi ha dins d'aquest últim fitxer (`cat /etc/bind/db.com.example`):

   ![info zona example.com](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-18%20a%20les%2011.52.18.png)
   
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
   ![wireshark alice](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.16.07.png)


   Com es pot veure, primer tenim un missatge ARP desde alice a broadcast per trobar on està nsce, ja que és el name server de alice.
   nsce respon amb un altre ARP la seva adreça MAC i llavors ja veiem els missatges DNS.
   
   En aquests missatges, primer veiem una query del A record per saber en quina IP està alice.example.com, enviada desde alice a nsce.

   En el segon missatge DNS veiem una resposta de nsce cap a alice, amb tots els RRs necessaris; A record de alice.example.com, NS de nsce i A de nsce.

   Finalment tornem a veure els dos últims missatges ARP de nsce cap a alice.


5. Using dig, try to resolve the IP address of joker.example.com. Did you find any resolution for this name? Discuss the results.
   
   Provem un dig desde alice per veure si trobem joker.example.com:

   ![dig alice -> joker](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.20.07.png)

   Ens retorna un A record, però sense cap adreça IP, i també ens retorna un SOA indicant que l'autoritat d'aquella zona és nsce.example.com, amb els seus paràmetres.

   Ara provem un dig desde nsce:

   ![dig nsce -> joker](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.21.39.png)

   I ens retorna exactament el mateix, on ens diu que NO tenim cap ANSWER.

6. Add the adequate RR in the appropriate server to map the name joker.example.com to the IP address 10.0.0.201. Reset the name server to load the configuration and explain how you test your configuration with dig and ping.
   
   Per afegir el nom joker.example.com ho hem de fer al fitxer `/etc/bind/db.com.example` de nsce (utilitzem nano per editar el fitxer):

   ![afegim joker](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.27.04.png)

   Si ara enviem un ping desde nsce o desde alice a 10.0.0.201 veiem com aquest arriba sense problemes, i fins i tot el podem veure al wireshark com es transmet tant l'echo request com l'echo reply.

   En canvi si fem `dig joker.example.com`no ens retorna res, ja que no hem resetejat el servei de bind.

   Un cop afegida la línia corresponent a joker.example.com, fem un restart del servei de bind: `/etc/init.d/bind9 restart`.

   Llavors fem el dig i ja ens retorna tot correctament:

   ![dig a joker correcte](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.33.34.png)

    Veiem com ara ja ens retorna el A record corresponent a joker, el seu NS (nsce) i el A record de nsce.

7. Which IP address will be contacted by a mail server if it has to send an e-mail to john@example.com.
   
   Si mirem l'arxiu de la zona example.com desde nsce, veiem com ens indica el següent:

   ![info de la zona example.com](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.39.03.png)

   On ens indica que els mailservers de la zona son el mailserver1 i mailserver2.example.com, els quals estan a 10.0.0.25 i 10.0.0.26 respectivament. Es contactaran en ordre.

8. Try the following command: `alice:~# dig -t MX example.com` Explain the output of the command.
   
    ![dig de alice a MX de example.com](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.41.31.png)

    Aquest comando ens retorna la info dels diferents mailservers (tots els que tenen el RR **MX**), amb la seva info corresponent, els seus A records, i el record NS de la zona example.com.



**Exercici 2**

En aquest exercici analitzem queries recursives i la estratègia de cache del DNS.

1. In this exercise, we analyze a recursive query from alice. To do so, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line: `alice:~# dig bob.com`
   
   ![wireshark dig bob.com](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2021.47.37.png)

   Veiem com ara tenim varis missatges DNS, anem a analitzar-los:
   - Primer tenim un missatge de alice a nsce.example.com preguntant per bob.com
   - Llavors nsce.example.com veu que no hi sap arribar, i per tant contacta amb el seu servidor ROOT (10.0.0.1).
   - Des de root li comunica altre cop a nsce.example.com que el NS de bob.com és nsc.com, el qual està a 10.0.0.11
   - Els següents dos missatges són una query de nsce.example.com del RR NS de ROOT, i la response de root que el NS està a 10.0.0.1
   - Un cop resolt, nsce ja pot enviarli una query a nsc preguntant-li pel A record de bob.com
   - nsc li respon indicant-li que bob.com està a 10.0.0.12, i que el seu NS està a 10.0.0.11
   - Finalment nsce li envia l'últim missatge a alice indicant el A record de bob.com, el qual es troba a 10.0.0.12 i qui és el seu NS.
  
  Després dels missatges DNS veiem uns quants ARP simplement per resoldre les adreces MAC de cada un dels hosts. Primer entre alice i nsce, llavors entre nsce i root i finalment entre nsce i nsc (entre alice i bob no n'hi ha cap pq no alice només vol saber la resolució del nom, però no on està ubicat físicament).

2. We analyze DNS caching in this exercise. To do so reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line: `alice:~# dig bob.com ; sleep 5 ; dig bob.com` **Note. The sleep command delays for a specified amount of seconds.**
   
   ![wireshark dig bob.com amb sleep 5](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2022.03.00.png)

   Veiem com el primer dig s'executa de la mateixa manera que en l'exercici anterior, però que el segon no passa per root, ja que alice té la resolució de bob guardada a la cache (**fixar-se en els últims 2 missatges**).

3. Continuing with the analysis of DNS caching, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line: `alice:~# dig bob.com ; sleep 5 ; dig bob.com ; sleep 30 ; dig bob.com`
   
   Veiem el següent al wireshark:

   ![wireshark dig bob.com amb sleep 5 i 30](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2022.09.05.png)

   Primer veiem els mateixos outputs que en l'exercici anterior, però analitzem els últims quatre missatges DNS:
   - El primer és desde alice a nsce demanant per la resolució de bob.com
   - nsce ara hauria de contactar ROOT, PERÒ com que la caché de nsce té una duració més alta que la de alice aquest sap que la resolució de bob.com està a nsc, i per tant contacta amb aquest directament.
   - nsc li respon a nsce on està bob.com (A i NS RRs)
   - finalment nsce li diu a alice on està bob.com

4. Continuing with the analysis of DNS caching, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line: `alice:~# dig alice.com ; sleep 5 ; dig alice.com` **Note. Take into account that alice.com is not a FQDN under our DNS tree.**
   
   Veiem els següents missatges a wireshark:

   ![wireshark alice sleep alice](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2022.26.45.png)

   - query de alice.example.com preguntant a nsce on està alice.com
   - nsce contactant a root directament per resoldre alice.com
   - root indicantli a nsce que l'administrador de la zona .com és nsc i els NS
   - nsce(.21) contactant nsc(.11) per resoldre alice.com
   - nsc indicantli a nsce que alice.com NO existeix i qui és la SOA de la zona
   - nsce indicantli a alice.example.com(.22) que alice.com NO existeix, i qui és la SOA de la zona.
  
  Després dels ARP i el temps de sleep, veiem 4 missatges més DNS:
  - El primer de alice a nsce preguntant per alice.com
  - Segon com que nsce encara té la resolució de nsc (el resolver de la zona .com) guardat a la seva cache el contacta directament per saber on està alice.com
  - nsc li respon que alice.com no existeix a nsce
  - nsce li respon a alice que alice.com no existeix.
  
5. Set the negative cache TTL to 10 in the SOA of nsc. Reset the name servers processes of the scenario, cap- ture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line: `alice:~# dig alice.com ; sleep 5 ; dig alice.com ; sleep 10 ; dig alice.com`
   
   Per posar la negative cache TTL a 10, hem d'editar l'arxiu **`/etc/bind/db.com`** de **nsc**:

   ![/etc/bind/db.com](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2022.40.50.png)

   Guardem l'arxiu, reiniciem el servei de bind amb `/etc/init.d/bind9 restart` i enviem el comando de l'enunciat després de posar el wireshark a capturar. Veiem els següents missatges:

   ![wireshark despres TTL 10](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-19%20a%20les%2022.45.04.png)

   Analitzem els missatges capturats:
   - Primer veiem fins al missatge 14 la resolució que ja hem vist en exercicis anteriors: **alice -> nsce -> root -> nsce -> nsc -> nsce -> alice**
   - Llavors els missatges **19 i 20**, són els corresponents al 2n comando `dig alice.com`, el qual només ha tingut 5 de sleep. Com que tots els TTL són més elevats simplement fem el següent recorregut, ja que tenim la info guardada a la cache **alice -> nsce -> alice**.
   - Finalment del missatge **21 al 24** tenim 4 missatges que corresponen a **alice -> nsce -> nsc -> nsce -> alice**, on veiem com nsce ja no tenia la resolució de alice.com a la seva cache, PERÒ la que si que té és la de com arribar a la zona .com directament, i per tant *no cal que aquest cop contacti a root.*


**Exercicis opcionals**
**1.3** Configuració dels NS i zones de .net.

1. In your configuration consider that **nsne** must be configured with a single zone for example.net (single configuration file) and that it must delegate right.example.net to **nsner**. Modify the configuration files of **nsn**, **nsne**, and **nsner** appropriately and describe and test your configuration.
   
Configuració de **nsn** (`/etc/bind/db.net`)

![config de la zona .net](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.58.05.png)

Configuració de **nsne** (`/etc/bind/db.net.example`)

![config zona example.net](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.48.05.png)

Configuració de **nsner** (`/etc/bind/db.net.example.right`)

![config right.example.net](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2012.05.49.png)


2. Notice that the server nsn has a mistake in its initial configuration file, describe this mistake.
   
   L'error d'aquest fitxer és que al principi utilitza la notació de "@", però no ha indicat quin és l'"ORIGIN", i per tant falta afegir la línia `$ORIGIN .net`, o canviar cada "@" per "net."

   Inicialment l'arxiu està així:

   ![config right.example.net](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.55.42.png)

   En el meu cas opto per la segona opció ja que crec que s'entén millor. El resultat després d'editar l'arxiu és el següent: (mateixa foto que la primera de l'exercici anterior)

   ![config right.example.net](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.58.05.png)

3. After you have implemented the configuration, reset bind in all the name servers of the scenario, capture with wireshark tap0 and comment the traffic and the results observed when executing: `alice:~# dig david.example.com`
   
   Resetegem els serveis de bind de cada un dels servidors modificats, i executem el comando.
   
   *NOTA: He fet el dig a david.example.net, i per tant la resolució la fa directament cap al .net, sense passar el CNAME de david.example.com, però afegeixo la foto de la captura del wireshark de quan he fet primer el dig a .net, i desrpés com es veuria el missatge del CNAME de example.com tot i que ja tenim resolucions a la cache*

   ![output dig alice -> david.example.net](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.58.05.png)

   ![captura wireshark](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.58.05.png)

   ![captura wireshark CNAME](https://github.com/akaKush/Internet-Basics/blob/main/DNS%26DHCP/DNS_images/Captura%20de%20Pantalla%202021-04-20%20a%20les%2011.58.05.png)

    Veiem com amb la configuració anterior la resolució de david.example.com es fa correctament, on se'ns indiquen tots els NS corresponents a cada pas de la resolució fins a arribar a **nsne** i resoldre que david.example.net està a 10.0.0.122, i que david.example.com es tradueix a la mateixa IP gràcies al registre CNAME.




