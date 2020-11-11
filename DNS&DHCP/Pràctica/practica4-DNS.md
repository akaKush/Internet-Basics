# Pràctica 4 - DNS

---
<h2>Intro</h2>
<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/1.png" height="75%" width="75%">

Abans de començar veiem quin servidor delega a cada (sub)domini.

<b>. (root) delega a:</b>
- .com --> nsc.com
- .net --> nsn.net

<b>nsc.com delega a:</b>
- example.com --> nsce.example.com

<b>nsce.example.com delega a:</b>
- (No delega a cap server) alice.example.com, david.example.com

<b>nsn.net delega a:</b>
- example.net --> nsne.example.net

<b>nsne.example.net delega a:</b>
- (sense delegar a cap server) directament a david.example.net
- (sense delegar a cap server) directament a bob.left.example.net
- right.example.net --> nsner.right.example.net

<b>nsner.right.example.net delega a:</b>
- (No delega a cap server) directament a carla.right.example.net

<br>
<h3>Resum dels RR més típics</h3>

- <b>Type "A":</b> (mapeig del nom a direcció IP)
Conté l'@IP associada amb un nom.
Ex: `alice.example.com. 30 IN A 10.0.0.22`   <i>És bo veure com aquest és un <b>FQDN</b>.</i>

- <b>Type "SOA":</b> (Source of Authority, Each zone must have a != SOA)
Origin: name of the zone's primary server.
Person: e-mail of the zone's administrator.
Serial: integer (YYYY/MM/DD/XX) that must be increased after any modification of the zone data.
Refresh: time between zone transfer requests by secondary servers (usually days).
Retry: time between requests whenever a zone transfer request fails (usually hours).
Expire: time a secondary server keeps the data if the connection to the master fails (usually months).
Negative caché: the time that an inexistent translation may be cached.

---

<h3>Ex. 1</h3> According to the previous considerations about our DNS tree, explain in which server we should find the following
resource records (RR):

• An A record for peter.example.com --> nsce.example.com
• An A record for peter.left.example.net --> nsne.example.net
• An A record for peter.right.example.net --> nsner.right.example.net
• The SOA record of right.example.net --> nsner.right.example.net
• A PTR record for peter.example.com --> nsce.example.com
• A MX record for right.example.net --> nsner.right.example.com
• A MX record for left.example.net --> nsne.example.net
• A NS record for right.example.net --> nsner.right.example.net

Ara ja comencem la pràctica amb `simctl dns-basic start`

<b>1.1</b> Get a console at alice and looking at the configuration explain which is the name server used by this host.

<u>Per trobar el nameserver fem un cat al fitxer /etc/resolv.conf</u>:
<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/2.png">

Com podem veure el nameserver que ens indica és el .21, el qual correspon al <b>nsce</b>


<b>1.2</b> Identify which is the server of the zone example.com and describe the configuration of this zone.
<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/3.png">
Com podem veure a la AUTHORITY SECTION (SOA RR) el server de example.com és nsce.example.com

<b>1.3</b> In nsce using the command netstat, identify the name of the DNS server process.

??


<b>1.4 </b>In this exercise, we analyze a simple query from alice. In first place, reset the name servers processes and then,
capture with wireshark tap0 and explain the output of the following command:
`alice$ dig alice.example.com`
Explain also the DNS messages captured with wireshark.

<b>Note. Recall that with the command dig you can do DNS queries and show the corresponding responses.</b>

<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/4.png">

Veiem com Alice fa un missatge ARP per poder-se guardar a la seva caché on està el servidor, però per poder resoldre el nom s'envia una standard query DNS, la qual el nsce respon, i llavors aquest també guarda a Alice a la seva caché.



<b>1.5</b> Using dig, try to resolve the IP address of joker.example.com. Did you find any resolution for this name?
Discuss the results.
<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/5.png">

Com podem veure a wireshark, no podem trobar joker.example.com perquè nsce no té informació d'aquest nom. Si despleguem la informació del la <b>dns response</b> veurem com ens indica que no hi ha cap "Answer RRs", i "No Such Name", tot i que si que ens indica 1 registre de Authority, per tant ens està indicant que un authority nameserver ha contestat però que no ha pogut trobar a joker.


<b>1.6</b> Add the adequate RR in the appropriate server to map the name joker.example.com to the IP address 10.0.0.201.
Reset the name server to load the configuration and explain how you test your configuration with dig and ping.

Per arribar desde alice a joker, li entrem el RR del tipus A corresponent al joker, al nameserver <b>nsce</b>: <br>(<u>Per entrar nous RRs a un nameserver, ho hem d'escriure al fitxer /etc/bind/db.com.example</u>)

<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/6.png">

Un cop ja hem mapejat al nom de joker (a dins el server que ja contempla example.com), fem un `exec resetbind` al pyhost i tornem a fer un `dig joker.example.com` desde alice.

Ara al wireshark ja podem veure com rebem una DNS response desde joker:
<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/7.png">



<b>1.7</b> Which IP address will be contacted by a mail server if it has to send an e-mail to john@example.com.



<b>1.8</b> Try the following command:
`alice:~# dig -t MX example.com`
Explain the output of the command.

<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/8.png">
El comando anterior ens dóna informació detallada dels mailservers.



<h3>Ex 2. DNS-basic (caching strategy)</h3>

**1.1** In this exercise, we analyze a recursive query from alice. To do so, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line:
`alice:~# dig bob.com`

Passos que segueix alice per saber on està bob.com:
1. (query) Alice contacta nsce
2. (query) nsce no sap on està bob.com directament per tant contacta amb root (autoritat superior) perquè li digui
3. (response) root li contesta que bob.com està a nsc
4. (query) nsce contacta a nsc per saber on esta bob.com
5. (response) nsc contesta a nsce on està bob.com
6. (response) nsce contesta a alice on està bob.com



**2.2** We analyze DNS caching in this exercise. To do so, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line:
`alice:~# dig bob.com ; sleep 5 ; dig bob.com`

<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/9.png">

Com podem veure a la primera part, tenim la mateixa resposta que a l'apartat anterior, però un cop s'ha esperat 5s i tornem a fer una query, ara alice té a la seva caché que bob està situat a nsc, per tant no cal tornar a fer tot el recorregut fins a root i tornar.
Com que ara ja es coneixen tots, només fan falta 2 missatges DNS (query + response).



**2.3** Continuing with the analysis of DNS caching, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line:
`alice:~# dig bob.com; sleep 5 ; dig bob.com ; sleep 30 ; dig bob.com`

La primera part del comando farà exactament el que ha fet a l'exercici anterior. Primer buscarà bob.com fins al root anar i tornar, llavors el tindrà guardat en caché i per tant només farà falta query + response, però a l'últim dig, si ens fixem al wireshark, veiem com el recorregut per descobrir on està bob.com ha canviat, <i>com si no el tingués guardat a la caché</i>.

Efectivament, <u>si mirem al fitxer /etc/bind/db.com</u> al server nsc, veurem com el TTL per bob és de 30, i per això s'ha eliminat bob.com de la caché de nsc. Tot i així, veiem com el procés sencer NO es repeteix. Aquest cop no passem per root, perquè si ens fixem en el TTL de root, veiem com és de 60000 per tothom. Per tant aquest cop ens podrem estalviar el viatge fins a root, pq tindrem guardat a la caché de nsce que bob.com està a nsc.com (encara tindrem la caché del que ens havia dit root), i llavors només fa la query de nsc per trobar a bob.com (la que havia expirat per TTL), i finalment ja ens torna les respostes fins a alice de on està bob.com.

Podem veure-ho més fàcilment al dibuix següent:
<img src="https://github.com/akaKush/DNS-DHCP/blob/main/Pr%C3%A0ctica/10.png">


**2.4** Continuing with the analysis of DNS caching, reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line:
`alice:~# dig alice.com ; sleep 5 ; dig alice.com`
Note. Take into account that alice.com is not a FQDN under our DNS tree.

En aquest cas, fem un dig per Alice igual que hem fet per bob, però com que alice.com NO és un FQDN, el nsce respòn que alice.com no existeix.
Llavors s'esperarà 5 segons i al tornar a fer dig ens diu exactament el mateix, però ara com que ho te a la caché ja no cal que vagi fins a root, simplement li pregunta a nsc, i aquest li diu que no existeix (passant per nsce entremig és clar).


**2.5** Set the negative cache TTL to 10 in the SOA of nsc. Reset the name servers processes of the scenario, capture with wireshark tap0 and explain the flow of DNS messages captured when executing the following command line:
`alice:~# dig alice.com ; sleep 5 ; dig alice.com ; sleep 10 ; dig alice.com`

Desde nsc executem `nano /etc/bind/db.com` i canviem el negative caché TTL a 10.
Llavors fem reset als nameservers, i executem el comando indicat.

Primer farà tota la ruta per trobar a alice.com, seguint els passos de l'ex 2.1, tot i que finalment digui que no existeix. 
Llavors fa sleep 5s i seguidament torna a fer el mateix, però aquest cop no va fins a root pq el seu TTL és elevat i encara tenim a la caché que contacti directament a nsc per resoldre els .com. Així ho fa i tornem a rebre la mateixa resposta però amb menys queries aquest cop.

