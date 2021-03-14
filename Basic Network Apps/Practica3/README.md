# Pràctica 3 - Network Applications

Iniciem la simulació amb `simctl netapps-basic start`.

Al primer exercici simplement ens demanen trobar en quin port està el servei *daytime*.

`cat /etc/services | grep daytime`

Veiem com el port que utilitza és el **13**.

Assignem ara les IPs que ens dona l'enunciat a virt1 i virt2
```
virt1# ifconfig eth0 192.168.0.1
virt2# ifconfig eth0 192.168.0.2
```
Ara enviem 3 pings de virt2 a virt1:
`virt2# ping -c 3 192.168.0.1`

La interfície de **loopback** és una adreça interna de cada màquina, per serveis que s'executin dins la mateixa màquina.
No té @MAC perquè és una direcció interna i no està associada a cap interfície de hardware.

Al següent exercici ens demana primer de tot visualitzar els serveis TCP sota inetd amb el comando netstat:
`virt1# netstat -tnlp | grep inetd`
Per obrir el fitxer, `cat /etc/inetd.conf`

Ara ens demana activar el servei daytime a virt1. Per fer-ho hem de descomentar la línia del servei al fitxer inetd.conf:
`virt1# nano /etc/inetd.conf`

Un cop descomentat, reiniciem inetd:
`virt1# /etc/init.d/openbsd-inetd restart`

Ara utilitzant netcat desde virt2 podem connectar-nos a la IP de virt1 i indicar el port 13 per veure si ens retorna el servei daytime:

![Daytime](https://github.com/akaKush/Internet-Basics/blob/main/IPv4/images/Captura%20de%20Pantalla%202021-03-14%20a%20les%2015.49.45.png)

Ara hem de buscar el servei ssh:
`cat /etc/services | grep ssh`
Veiem com aquest està al port 22.

Si ara aturem el servei ssh `virt1# /etc/init.d/ssh stop` i tornem a buscar-lo amb el comando `netstat -tnlp | grep ssh` i veiem com ara no ens retorna res.

Canviem la configuració per tenir el servei ssh escoltant al port 2222.
Primer necessitem canviar el fitxer de configuració de ssh:
`nano /etc/ssh/sshd_config` i canviem el port 22 al 2222.

Passem al següent exercici, on haurem de configurar el nostre propi stand-alone server.

Iniciem una captura al wireshark a tap0, i executem el següent comando per enviar el fitxer /etc/services desde virt1 a virt2:
`virt1# cat /etc/services | nc -l -p 12345 -q 0` Per iniciar el servidor netcat i transferir el fitxer.

Ara des de virt2 intentem afegir-nos a la transmissió del fitxer:
`virt2# nc 192.168.0.1 12345 > file.txt`

**ex7 SSH i TELNET**
Volem iniciar una connexió TELNET, però al provar-ho no fa res, per tant primer hem d'activar el servei descomentant la línia de telnet amb `nano /etc/inetd.conf` i llavors reiniciem el inetd amb `etc/init.d/openbsd-inetd restart`.

Ara ja podem provar de connectar-nos desde virt2 amb `telnet 10.1.1.1`, però veiem que ens diu que el login no és correcte.

Per acceptar l'entrada al servei telnet com a root des d'un altre terminal, hem de modificar la configuració del fitxer **/etc/ftpusers** i eliminant o comentant la línia de l'usuari root.

Si ens transferim qualsevol cosa amb TELNET, des de wireshark podem veure la informació enviada, en canvi si utilitzem SSH, veurem com s'estan interncanviant datagrames entre els dos terminals, però no podrem veure el contingut d'aquests datagrames.

Per establir la sessió ftp simplement cal fer `virtX# ftp @IP_remota`

El següent que ens demana és transferir-nos tots els fitxers de /usr/bin que comencin amb "z" mitjançant FTP.

`ftp @IP_remota | dir /usr/bin/z*`

Si analitzem al wireshark, podem veure el contingut dels datagrames enviats.

D'altre banda, si fem exactament el mateix, però amb SFTP, llavors simplement podrem veure l'intercanvi de datagrames, però no el seu contingut.

L'últim exercici, ens demana assignar una nova @IP a SimNet0 (10.1.1.3/24), i utilitzar scp per enviar un fitxer que hem de crear a virt2, per enviar-lo al directori /root de virt1:
`virt2# sudo ifconfig SimNet0 10.1.1.3/24`
`virt2# touch file.txt` `nano file.txt` i afegim la línia "Hello World!" a dins el fitxer.

Ara ens transferim el fitxer a /root de virt1:
`virt2# scp root@10.1.1.1:file.txt`

Si analitzem amb wireshark, veiem com no podem veure el contingut de dins dels datagrames ja que SCP treballa amb SSH i per tant encripta els continguts.