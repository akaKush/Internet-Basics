<h2>FTP</h2>

FTP (File Transfer Protocol) és un protocol d'internet per transferir arxius entre hosts.
Està basat en en model client-servidor i utilitza IP.

FTP té 2 tipus de connexió separats, un per **control** i l'altre per les **dades**.
<ul>
<li> El port 20 és el que su'utilitza per intercanviar dades. </li>
<li> El port 21 és el que s'utilitza per la connexió (i el control d'aquesta) </li>
</ul>

El control de la connexió es mante obert durant tota la sessió i es fa servir per el manteniment de la sessió (comandos, negociació de paràmetres, passwords...).

<h4>Modes actiu i passiu</h4>
El mode determina com s'estableix la connexió de les dades.

<ul>
  <li> En <b>mode actiu</b>, el client envia al server l'adreça IP i el nº del port a on el client estarà escolant, i el servidor inicia la connexió de dades.
(En cas que el client tingui un firewall o no pugui acceptar connexions TCP, millor utilitzar mode passiu) </li>
  <li> En <b>mode passiu</b> el client envia un comando PASV al servidor i rep una @IP i nº de port de retorn. Llavors el client utilitza aquests paràmetres per obrir la connexió de dades amb els servidor </li>
</ul>

<h4>Representació de dades</h4>
Per poder enviar dades per la xarxa, existeixen 4 tipus != de representació de dades (tot i que a la pràctica només s'utilitzen ASCII i binari).
<ul>
  <li> <b>ASCII mode</b>: per TEXT. Les dades abans de ser enviades es converteixen a "8-bit ASCII" </li>
  <li> <b>Binari mode</b>(també anomenat mode imatge): L'emissor, envia cada fitxer byte per byte, i el receptor el va guardant el flux de bytes a mesura que li arriba</li>
  <li> <b>EBCDIC mode</b>: S'utilitza per "plain text" entre hosts que utilitzen el "EBCDIC character set" </li>
  <li> <b>Local mode</b>: Permet que dos ordinadors amb configuracions idèntiques enviin dades en un format especific sense haver de convertir a ASCII </li>
</ul>

<h4>Modes de Transmissió de dades</h4>
El mode determina com es transfereixen es dades. A la pràctica es fa servir el Steam transfer mode.

<ul>
  <li> <b>Stream mode</b> (mode de fluxe): Les dades s'envien en un flux continu, sense que FTP hagi de processar res. En aquest mode, tot el processat el farà el TCP.</li>
  <li> <b>Block mode</b>: el FTP divideix les dades en != blocs (block header, byte count i data field), i llavors li passa a TCP </li>
  <li> <b>Compressed mode</b>: Es comprimeixen les dades amb un algoritme (normalment run-lenght encoding) i llavors s'envien. </li>
</ul>

---

<h3>Com utilitzar FTP en Linux</h3>

El client **ftp** de linux suporta modes actiu i passiu, trasmissions ASCII i binari i només el Stream mode.

Per connectar-nos a un **ftpd** server podem utilitzar qualsevol de les següents opcions:

`$ ftp name`
`$ ftp 192.168.0.1`
`$ ftp user@192.168.0.1`

--> Per establir una sessió FTP, necessitem saber el **ftp username and password**. 
Si la sessió és anònima, normalment, podem utilitzar "anonymous" com a user & password.

Un cop haguem entrat el usrname i pwd, ens retorna el prompt "ftp>", el qual es la subshell de ftp.
Podem veure a continuació els principals comandos que hi podem executar:

<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ftp_commands.png" 
alt="daytime port 13" width="50%" height="50%"/>

El mode 'default' es el actiu, si volem passar a passiu executem `ftp -p` o `pftp`.

Com a última observació, és bo saber que podem utilitzar ftp als navegadors web, i amb altres aplicacions gràfiques.
Navegadors com el firefox, ens permeten executar ftp tal que:

`ftp://ftp.upc.edu`
`ftp://ftpusername@ftp.upc.edu`
`ftp://ftpusername:password@ftp.upc.edu`

---

<h5>Observacions dels videos</h5>
<ul>
  <li> Si volem <b>comprovar si el servei FTP està activat</b>, fem `$ netstat -tnlp | grep 21` i si ens retorna per pantalla el port vol dir que si que està activat.
En cas de que no ho estigui, si volem que el usuari root pugui fer servir FTP al terminal, fem:      <ol>
    <li> `$ cd /etc` </li>
    <li> `$ nano ftpusers` i <b>comentem</b> a l'usuari que vulguem donar-li accés al servei ftp </li>
   </ol> 
  </li>
  <li> Un cop tenim permisos per fer servir ftp amb el nostre usr, simplement amb `ftp @IP` se'ns conectarà per ftp al usuari de la @IP que li haguem passat. Llavors entrem el usr i pwd i ja serem <b>dins la subshell ftp de la màquina remota</b>. Ara podem pujar i descarregar arxius en aquesta màquina remota.
    <ol><li>p. ex: <b>get hola.txt</b> <== Això ens agafarà el fitxer hola.txt del host i el portarà a la nostra màquina local </li></ol>
  </li> 


