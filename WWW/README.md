# WWW - Pràctica TCGI

For a theoretical explanation of this topic, read this article: <a href="medium.com">WWW</a>

Content:




---

## Introduction

L'escenari que utilitzarem per les pràctiques és el següent:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2012.49.34.png)

*In this exercise, we are going to practice with the WEB service. To do so, start the scenario www shown in the picture above on your physical host (phyhost) by typing the following command:*
 `phyhost$ simctl www-new start`
 
*This scenario starts four virtual machines host, www, server and dns. Each virtual machine has also two consoles (0 and 1) enabled.*

*First of all, we’re going to have a look into the nginx configuration files. Remember that the nginx configuration files are stored in the `/etc/nginx folder`.*

*Answer the following questions:*

## 1.1
1. *Capture the traffic on tap0 and use a lynx browser in the host virtual machine to connect to www.example.com on port 8080.*
*Which is the IP address associated to www.example.com?*
*Why the browser is not able to establish a TCP connection with the server? Describe the DNS and TCP traffic captured.*

Veiem com ens indica que no s'ha pogut connectar al host remot.
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2012.53.45.png)

Ens indica que primer ha buscat el nom www.example.com, després el port, però que no s'hi ha pogut connectar.

Al wireshark veiem:
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2012.55.31.png)

Primer es fa la resolució IPv6 de www.example.com (AAAA record), i llavors es retorna el A record indicant que www.example.com està a **10.1.1.1**, i qui és el seu nameserver.

Després veiem com s'ha intentat establir una connexió TCP però no s'ha pogut.


2. *Capture the traffic on tap0 and repeat the previous experiment, but this time execute a netcat in the www machine listening on port 8080. Which version of HTTP is using the browser? Is the connection closed? Describe the DNS and HTTP traffic and kill the netcat to finish.*

Veiem com al terminal ja ens indica que està fent servir **HTTP 1.0**:
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2012.59.50.png)

Host ens indica que està esperant una resposta del servidor.
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.00.46.png)

Analitzem els missatges del wireshark:
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.02.15.png)

Veiem com primer tenim la resolució típica DNS on ens acaba indicant que www.example.com està a 10.1.1.1, i llavors inicia la connexió TCP correctament (**SYN, SYN-ACK, ACK**), i també veiem com hi ha una connexió HTTP 1.0. Si apretem "enter" veiem missatges de "Continuation" i finalment quan matem el procés de netcat veiem com s'acaba la connexió TCP (**FIN, FIN-ACK, ACK**).

3. *Let’s take a look to the nginx web server in the www machine. How many websites are available? Is there any website enabled?*
   
Per veure la info del servidor nginx anem al directori **/etc/nginx/**.
Si volem veure les websites available, ho mirem al següent fitxer:
```
www:~# cd /etc/nginx/sites-available/
www:/etc/nginx/sites-available# ls
balancer  default  non-existing-sites
```

Per veure els llocs web available pel nostre servidor ho fem en el fitxer **default**:
```
www:/etc/nginx/sites-available# cat default
server {
     listen       80;
     # .example.com could be used for both example.com and *.example.com
     # "" is used to attend requests with no "host" header
     server_name  www.example.com example.com "";

     location / {
         root   /var/www;
         index  index.html index.htm;
     }

     location ~ ^/cgi/ {
         # Disable gzip (it makes scripts feel slower since they have to comple$
         # before getting gzipped)
         gzip off;

         # Required to forbid default content browsing
         autoindex off;

         # Default document root
         root /var/www/cgi-bin;

         # Changing the url according to what fastcgi expects
         rewrite ^/cgi/(.*) /$1 break;

         # Fastcgi parameters, include the standard ones
         include /etc/nginx/fastcgi_params;

         # Fastcgi socket for library communication
         fastcgi_pass  unix:/tmp/cgi.sock;

         # Setting the script filename
         fastcgi_param  SCRIPT_FILENAME /var/www/cgi-bin$fastcgi_script_name;
     }

}
```
Veiem que ens indica que només hi ha www.example.com disponible, escoltant al port 80 (per defecte HTTP), i que els arxius d'aquest lloc web es troben a /var/www.

Si ara mirem el contingut de "sites-enabled" veiem com tenim pràcticament el mateix i per tant www.example.com està enabled.

4. *Open the default configuration. What is the virtual host name(s) for this site? Where is the website content placed?*
   
```
www:/etc/nginx/sites-enabled# cat default
server {
     listen       80;
     # .example.com could be used for both example.com and *.example.com
     # "" is used to attend requests with no "host" header
     server_name  www.example.com example.com "";

     location / {
         root   /var/www;
         index  index.html index.htm;
     }

     location ~ ^/cgi/ {
         # Disable gzip (it makes scripts feel slower since they have to comple$
         # before getting gzipped)
         gzip off;

         # Required to forbid default content browsing
         autoindex off;

         # Default document root
         root /var/www/cgi-bin;

         # Changing the url according to what fastcgi expects
         rewrite ^/cgi/(.*) /$1 break;

         # Fastcgi parameters, include the standard ones
         include /etc/nginx/fastcgi_params;

         # Fastcgi socket for library communication
         fastcgi_pass  unix:/tmp/cgi.sock;

         # Setting the script filename
         fastcgi_param  SCRIPT_FILENAME /var/www/cgi-bin$fastcgi_script_name;
     }

}

```

5. *Open the non-existing-sites configuration. What do you think the purpose of this site configuration is?*

```
www:/etc/nginx/sites-available# cat non-existing-sites 
server {
    listen      80 default_server;
    server_name _ ;
    return 503  "No server is currently configured for the requested host." ;
}
```

Aquest fitxer serveix per saber què retornar en les peticions que no són adequades.


6. *Enable the non-existing-sites by typing, from the sites-enabled folder, the following command: `ln -s ../sites-available/non-existing-sites`.*

7. *Capture the traffic on tap0 and start the nginx Web server in the www machine.*
*`www# /etc/init.d/nginx start`. On the host machine, execute a netcat to connect to the nginx server that you have just started. Over the connection established with netcat and using HTTP 1.0, send an HTTP GET request for the resource “/”. Which response do you obtain? Describe the HTTP traffic captured for the GET request.*

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.25.43.png)

Després d'iniciar el servidor nginx i establir una connexió amb netcat a www.example.com, demanem un **GET /** i obtenim l'anterior resposta.

Al wireshark veiem el següent tràfic:
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.27.29.png)

On tenim la resolució DNS, la connexió TCP (SYN), la petició de GET (PSH-ACK i ACK) i llavors la petició GET / amb HTTP1.0, però la resposta utilitza HTTP1.1.

8. *Now edit the default configuration and remove the "" from the server_name section. Send again the HTTP GET request for the resource “/”. Which response do you obtain now? Why?*
*Hint: How many sites are enabled in the nginx server? What is the purpose of each of them?*
*Note: Restore the "" in the server_name section of the default configuration if you want to keep accessing the server through the IP instead of the sites ́ name.*

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.31.39.png)

Si **eliminem la part "" de l'arxiu default al directori sites-enabled** i tornem a demanar el mateix GET, ara obtenim una resposta de **503 Service Temporarily Unavailable**.

A wireshark simplement veiem els missatges corresponents a aquesta connexió, i l'últim missatge HTTP de resposta des del server cap al host ens indica el mateix que el terminal:
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.33.51.png)

Tornem a escriure les "" a l'arxiu default per continuar accedint utilitzant el nom.

9. *Again, over the connection established with netcat and using HTTP 1.1, send an HTTP GET request for the resource “/” targeting the host www.home.com (or any other hostname you want) in www Which response do you obtain now? Why? Describe the HTTP traffic captured for the GET request.*

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.36.21.png)
Òbviament obtenim un error 503 ja que aquest host no està a cap arxiu de la configuració.

10.  *Send a GET request for the resource “/doc.html”. Which response do you obtain for each request? Is there a resource called doc.html in the www server? Describe the HTTP traffic captured for the GET request.*

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2013.39.37.png)

L'arxiu doc.html NO existeix, per tant no el troba i ens retorna **404 not found**.

11.  *Configure the tap0 interface of the physical host (phyhost) with the IP address 10.1.1.4/24. After that, ask for “/” and “/doc.html” from the phyhost using a firefox browser and the IP address 10.1.1.1. Describe the HTTP traffic captured. Can you use the name www.example.com from the phyhost? why? Propose a way to reach the www machine when typing www.example.com.*

PERQUE NO VA?????????? --> Tinc mal escrita la configuració de /etc/bind9/db.example.com segur, sino no sé què és.


---


## Exercici 2

1. *Start a capture on the physical NIC, i.e. enp8s0. In the phyhost open a firefox browser and request for the index of a complex webpage, such as ieeexplore.ieee.org.*
*Describe the HTTP traffic captured. In particular, discuss the GET requests that you observe and the number of connections. To do this analysis easier, you can take one or combine both of the following approaches:*
*• In wireshark, you can use the option statistics ◃ conversations, go to the label for TCP and then, use the option follow  stream for analyzing the data transmitted through each TCP connection.*
*• Open the developer tools (e.g. push control+shift+i). A new section will show up in the bottom of the browser. There, one can analyze any matter related with the browser performance. Select the tab named Network, which is responsible for displaying the traffic being exchanged (e.g. when HTTP requests are commited).*
*Now ask a second time for the index of the previously requested page (e.g. ieeexplore.ieee.org). Describe how this time HTTP caching works.*

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2014.04.33.png)

Veiem moltissims GETs per a cada fitxer de la pàgina, els quals la majoria són js, html o gifs.

Si tornem a refrescar la pàgina veiem com molts dels GETs ara tenen el codi 304, i ens indica "cached", ja que no s'ha canviat la petició des de l'últim cop i ja tenim aquell fitxer a la caché.

*Note. To reproduce the experiment you have to remove the cache of firefox. You can do this clicking in ”clear your recent history“ in the menu Edit◃Preferences◃Privacy of firefox*

2. *Remove the cache of firefox and decrease its maximum number of persistent connections per server from 6 to 2. For this purpose, use the about:config string in the URL of firefox and then, search and modify the parameter network.http.max-persistent-connections-per-server.*
*From firefox request again for the index page of the complex webpage used above (e.g. ieeexplore.ieee.org). Describe the HTTP traffic captured. In particular, comment the number of persistent connections that you observe now and the GET requests through each connection.*
*Note. When you finish the exercise, do not forget to set to 6 again the maximum number of persistent connections per server.*

Les connexions ara duren més, ja que hem limitat les connexions d'aquest server a 2.

3. *Under the directory /tmp of the virtual machine www you will find files with images. In www, copy these images to a directory called “images” relative to the DocumentRoot of the NGINX’s default site (named “default”).*
*Note. You must create the “images” directory.*
*Modify the HTML index of the server and create local hyperlinks to these images. Describe how you do it.*

Creem el directori images: `mkdir /var/www/images`
Copiem les imatges al directori: `cp /tmp/upc1.gif /tmp/upc2.gif /tmp/upc3.png /var/www/images`
Editem l'arxiu index.html: `nano /var/www/index.html` i li afegim les imatges dins el body del html.
Fem restart de nginx: `/etc/init.d/nginx restart`

Accedim al navegador per visualitzar les imatges a 10.1.1.1/

4. *Start a capture on the tap0 interface. Execute a netcat in the phyhost to connect to 10.1.1.1 port 80 redirecting the output of this command to a file called response.http. Through the established connection type an HTTP request for the resource upc1.gif using HTTP 1.0. Explain how you do it.*
*Edit appropriately the file response.http with mousepad or vi to obtain the original image upc1.gif.*
*Save the image with the name image.gif and test that it is correct using the command display:*
`phyhost# display image.gif`
*You should be able to see an UPC logo. After that, repeat the process using HTTP 1.1.*

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2014.32.00.png)

Al wireshark veiem els següents missatges:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2014.32.36.png)

On ens diu que ens ha retornat un codi 200 OK, per tant hauriem de tenir la imatge.

Per editar la resposta utilitzem vi...???



Si volem repetir el procés però utilitzant HTTP 1.1 necessiteme specificar un HOST en el moment del GET tal que:


1. *Repeat the process to obtain upc1.gif but this time use the command wget. Explain how you do it (consult the manual page of wget if necessary). Which version of HTTP is used by wget?*

Per fer servir **wget** ho fem tal que: `pyhost# wget 10.1.1.1/images/upc1.gif` i ens retorna la imatge indicada. **WGET suporta HTTP, HTTPS i FTP**. Per tant entenem que suporta tant 1.0 com 1.1.

Si mirem el wireshark veiem com **wget ha utilitzat HTTP 1.1**.



------

**Exercici 1.3**

1. 
Creem l'arxiu datecgi.sh a /var/www/cgi-bin: `nano datecgi.sh`
L'editem amb el codi 1.6 que ens donen al pdf de l'enunciat (mirar més amunt a la teoria per trobar-lo).
Ho guardem i modifiquem permisos:
```
www:/var/www/cgi-bin# chown www-data:www-data datecgi.sh
www:/var/www/cgi-bin# chmod 700 datecgi.sh
```

2. 
obrim el browser al pyhost amb la url "http://www.example.com/cgi/datecgi.sh" i veiem el següent:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2015.10.33.png)

3. 
El browser intenta descarregarse el cgi perquè a l'arxiu `/etc/nginx/sites-enabled/default` tenim que les default locations són "/" i "_ ^/cgi/", el qual significa que buscarà els cgis dins el directori cgi i ara mateix no hi es ja que cgi-bin és diferent a cgi.

4. 
Perquè ho faci correctament, hem d'editar el fitxer anterior per indicar-li on buscar i executar bé el CGI:
`nano /etc/nginx/sites-enabled/default`

I a dins l'arxiu a sota de la "location /" escribim el següent:
```
location /cgi-bin/ {
    root /var/www/cgi-bin;
    return 403;
}
```

5. 
???


**Exercici 1.4** Multiple domains and multiple IP addresses (Modificació del server DNS)

1. 
Per comprovar que nginx està executantse properly fem:
```
www# netstat -tnlp | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1244/nginx 
```

Veiem que està correcte.

Ara afegim el nom www.example.net al DNS server perquè ho tradueixi a la @IP 10.1.1.1
Obrim la màquina DNS, afegim el A record de www.example.net al fitxer `/etc/bind/db.example.net` i llavors fem restart de bind perquè funcioni, `/etc/init.d/bind9 restart`.

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2016.31.27.png)

Comprovem desde pyhost que estigui www.example.com i .net ben configurats enviant pings i veien si els reben bé:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2016.33.08.png)

**(MATEIX PROCÉS PER .com)**

Veiem com tots dos arriben bé.

1. 
Executem els comandos indicats a l'enunciat per CREAR i ACTIVAR DOS LLOCS WEB
```
www# cd /etc/nginx/sites-available/
www# cp default www.example.com
www# cp default www.example.net
```

Ara executem els comandos de l'enunciat per canviar la configuració de root i els server name keys per la nova configuració.

Un cop fets obrim els dos fitxers per comprovar que estiguin ben configurats:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2016.44.47.png)

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2016.45.01.png)

Un cop comprovat necessitem ACTIVAR els dos llocs web. Per fer-ho anem a sites-enabled i executem el següent:
```
www# cd /etc/nginx/sites-enabled/
www# rm default
```

I un cop eliminat el fitxer actual de default, hem d'activar els virtual hosts dels dos noms:
```
www# ln -s ../sites-available/www.example.com
www# ln -s ../sites-available/www.example.net
```

Fem un reload de nginx, i ja podem generar els continguts que tindrà cada web.
Per fer-ho afegim un fitxer index.html a les carpetes corresponents (S'HAN DE CREAR PRÈVIAMENT ELS DIRECTORIS):

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2016.50.45.png)

Per .com tenim un altre fitxer index.html igual, però on el text indica que es tracta del .com.

Per comprovar que ara funcioni, obrim un wireshark a SimNet0 i intentem accedir als dos dominis des del navegador:

Veiem que accedeixen bé al navegador, però si escrivim 10.1.1.1 accedeix per default a www.example.com. Si volem accedir al .net ho podem fer amb netcat indicant el host que ens interessa amb HTTP 1.1


3. 
Afegim registres A a la configuració de www.example.org per redirigir les adreces 10.1.1.1 i 10.1.1.2 a www.example.org.

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2016.57.13.png)

Llavors activem aquest domini a www i server amb els següents comandos:
```
# cd /etc/nginx/sites-available/
# cp default www.example.org
# sed -i "s/\/var\/www/\/var\/www\/org/g" www.example.org
# sed -i "s/server_name.*/server_name www.example.org;/" www.example.org # cd /etc/nginx/sites-enabled/
# rm default 2> /dev/null
# ln -s ../sites-available/www.example.org
# /etc/init.d/nginx reload
# mkdir /var/www/org
```

Creem un index.html als dos servidors (www i server) a /var/www/org i comprovem la configuració connectantnos amb netcat i demanant per el fitxer / (index.html).

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2017.13.02.png)

Ens connectem per nc i demanem "/":

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2017.14.43.png)

Veiem com ens retorna correctament el fitxer, codi 200 OK, d'aquesta manera hem configurat el servidor DNS pq distribueixi la càrrega entre diferents IPs.


**Exercici 1.5** (Reverse Proxy per distribuir el traffic)

1. 
Obrim l'arxiu `balancer` de /etc/nginx/sites-available per veure què hi posa:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2017.44.32.png)

Veiem com el server escolta al port 80, i que el server name és www.example.com.
Seguidament ens indica la localització on va a buscar els arxius (/). Quan un usuari carrega la pàgina principal (/), el servidor posa els Headers escrits i carrega la url http://tcgi-app$"requested_uri".

A més veiem com tenim 2 servers per distribuir la càrrega, que es contacten en ordre, primer el www1 i després el www2.

2. 
Per fer que només s'executi la configuració del balancer, hem d'eliminar les altres configuracions:
`www/etc/nginx/sites-enabled# rm wwww.example.com www.example.net www.example.org`
I afegir la del balancer:
`www/etc/nginx/sites-enabled# ln -s ../sites-available/balancer`

Reiniciem el servidor nginx i introduim la url www.example.com i veiem com encara NO funciona.

3. 
Iniciem el servidor nginx als www1 i www2 i tornem a provar.
`www1# /etc/init.d/nginx start`
`www2# /etc/init.d/nginx start`

Si ara tornem a introduir www.example.com al browser veiem com ens retorna el missatge "It works!".

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2018.11.59.png)

Si mirem al wireshark veiem que l@IP que ha rebut la petició és 10.1.1.12 el qual correspon a **www2**, a més veiem com havia cachejat la request anteriorment, i veiem un missatge de 304 Not Modified.

4. 
Si recarreguem la pàgina X vegades, veiem que cada cop és un servidor diferent el que processa la petició (alterna entre www1 i www2):

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2018.34.19.png)


5. 
Aturem el servidor www2 amb `www2# /etc/init.d/nginx stop` i recarreguem la pàgina diverses vegades, i mirem ara al wireshark què passsa:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2018.41.17.png)

Ara cada cop és www1 (10.1.1.11) el que processa la petició, però també veiem com hi ha algún cop que s'intenta fer una connexió TCP amb www2 (10.1.1.12) i aquesta no es pot dur a terme, i llavors es torna a connectar amb www1.

Si restablim el servei nginx a www2 tot torna a funcionar com toca.
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2018.44.11.png)


6. WEIGHTS

Modifiquem la configuració "balancer" per afegir pesos als servidors, i que aquest atengui un 75% més de peticions que l'altre:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2018.47.02.png)

Ara accedim 4 cops com a mínim a www.example.com per veure qui processa les sol·licituds:

![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2018.50.44.png)

Com podem veure la majoria de vegades la petició és processada per www1, i una de cada 4 per www2. He recarregat la pàgina 8 vegades i d'aquestes 8 només 2 ha estat servida per www2 (25%).


**Exercici 1.6 HTTPS**


1.  CREACIÓ DE CERTIFICATS I CLAUS

Per poder testejar una connexió encriptada primer hem d'establir un certificat.
Utilitzarem el certificat proveït pel paquet *ssl* que es troba a:
`certificate: /etc/ssl/certs/ssl-cert-snakeoil.pem`
`key: /etc/ssl/private/ssl-cert-snakeoil.key`

Passos:
   1. Creem un **parell de claus** pel CA:
   `www# openssl genrsa -des3 -out mycakey.pem 2048` (protegida amb DES3)
   ![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2019.10.17.png)

   2. Creem el **certificat** pel CA:
   `www# openssl req -new -x509 -days 2000 -key mycakey.pem -out mycacert.pem`
   ![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2019.13.35.png) (emplenem amb les dades que creiem, no és important en aquest exercici)

   3. Si volem **comprovar les dades del certificat** ho podem fer amb:
   `www# openssl x509 -in mycacert.pem -text -noout`

   4. Ara ja podem **crear la "parella de la clau del servidor"**, "server key pair":
   `www#openssl genrsa -out myserverkey.pem 2048` (notar que NO fem servir la opció -des3 aquí, ja que necessitem la clau privada en clar, no protegida)
   ![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2019.16.13.png)

   5. Com que no la hem protegit, li hem de **restringir l'accés**, per tand modifiquem permisos pq no tothom hi pugui accedir:
   `www# chmod 400 myserverkey.pem`

   6. Ara creem la **request del certificat** (.csr):
   `www# openssl req -new -key myserverkey.pem -out myservercert.csr` --> Ens demanarà que emplenem uns camps, aquí hem d'utilizar un FQDN del server, en aquest cas www.example.com

   7. Utilitzem la clau del CA per **signar el certificat**, i creem el fitxer "myserver.crt":
   `www# openssl x509 -req -in myservercert.csr -CA mycacert.pem -CAkey mycakey.pem -CAcreateserial \-days 360 -out myservercert.pem` (important utilitzar www.example.com com a Domain Name)
   
   8. Finalment **eliminem el request** del certificat:
   `www# rm myservercert.csr`


1. Canviem la configuració per utilitzar HTTPS

Primer utilitzem la configuració default per tenir una plantilla i la editem:
`www/etc/nginx/sites-available# cp default www.example.com`

Llavors l'editem amb nano i posem les configuracions necessaries per utilitzar el certificat ssl i la clau:
![WWW scenario](https://github.com/akaKush/Internet-Basics/blob/main/WWW/Practica_Images/Captura%20de%20Pantalla%202021-04-24%20a%20les%2019.31.48.png)

Habil·litem la nova configuració i restablim el servei nginx i comprovem que funcioni:





