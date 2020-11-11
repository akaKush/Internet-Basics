<h2>TELNET (TELecommunication NETwork)</h2>

<h4>Qué es TELNET?</h4>
TELNET és un protocol estàndard d'internet que serveix per emular un terminal a un "remote host" mitjançant una @IP.

El client es connecta amb el protocol TCP, i el port _per defecte és el 23_.


--> Per utilitzar TELNET, només cal obrir un terminal i escriure `telnet` de manera que ja podrem utilitzar la **shell** de telnet.

*(**nota**: si no funciona, comprovem que estigui activat escrivint `netstat -tnlp | grep 23`, en cas de no rebre resposta per consola significa que no el tenim activat a la configuració de xarxa. 
Per tant activem el port 23 tal que: `nano /etc/inetd.conf` i un cop dins l'editor busquem **telnet** i el descomentem.
Un cop descomentat hem de fer restart de la configuració tal que: `/etc/init.d/openbsd-inetd restart`.
Si ara tornem a executar el comando netstat previ, veurem com ja tenim el port 23 escoltant a telnet).*

Un cop dins podem obrir el terminal al host desitjat mitjançant `open @IP_host 23`. (El 23 és opcional ja que és el seu port per defecte)
També podem connectar-nos al host de destí escrivint directament `telnet @IP_host`, ja que la shell de telnet executarà directament el "open @IP".

Quan haguem establert la connexió, TELNET ens ofereix una comunicació **orientada a text** bidireccional, i els comandos que executem localment, també seran executats al host remot.

*Per accedir com a root a l'altre terminal:*
Hem d'editar el següent fitxer: `nano /etc/securetty` busquem els diferents **pts** que tinguem disponibles i descomentem el que ens interessa accedir com a root.


Per sortir del terminal remot simplement executem `exit`.
