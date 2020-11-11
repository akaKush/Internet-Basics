<h2>Pràctica 3</h2>

**ex1** 
*1. Open all the consoles: virt1.0 (console 0 in virt1), virt1.1, virt2.0 and virt2.1. Then, figure out which is the port
number of the service daytime.
Tip. Take a look at the file /etc/services.*

```cat /etc/services | grep daytime``` 
<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/daytime.png" 
alt="daytime port 13" width="50%" height="50%"/>

*2. In a windows-like OS, which port number you expect for the daytime service?*
In a windows machine we supose that it should be running on the same port due to the universal nature of Layer4 protocols


*3.Annotate the MAC and IP addresses of each interface on virt1 and virt2. Which command have you used?*
`root@virt1$ ifconfig | grep HWaddr` same for virt2




*4. Assign the IP addresses 192.168.0.1 and 192.168.0.2 to the Ethernet interfaces of virt1 to virt2, respectively.*
`root@virt1$ ifconfig eth0 192.168.0.1`
`root@virt2$ ifconfig eth0 192.168.0.2`


*5. Send 3 ICMP echo-requests from virt2 to virt1 with the ping command.*
`root@virt2$ ping -c 3 192.168.0.1`


*6. Find information and explain what is and for what can be used the loopback (lo) interface. Why lo does not have a MAC address?*
**lo** is used for **loopback interface** and it does not have MAC address cause it is not a phisical address, it belong to the inner machine network configuration, and it is used to access the network services that are running on the host via the loopback network interface. Using the loopback interface bypasses any local network interface hardware.

*7. Configure the original IP addresses on each virtual machine.*
Si no sabem les IP's originals, fem un restart de la simulació i fem ifconfig per saber quines son.
Un cop les sabem:
`root@virt1$ ifconfig eth0 10.1.1.1`
`root@virt2$ ifconfig eth0 10.1.1.2`

**ex2**
*In this exercise you will start, stop and configure some network daemons (services).*
*1. Using the netstat command, list the TCP services that are active in virt1 under inetd. Describe the service names and ports used and check that the results are consistent with the configuration of inetd (/etc/inetd.conf).*
**`netstat -tnlp`** <-- Comando per llistar els serveis actius
`cat /etc/inetd.conf` <-- Comando per veure per pantalla el fitxer inetd.conf


*2. In virt1, edit the configuration of inetd and activate the service daytime. Restart the super-daemon and check that the port of daytime is being listened by inetd. Use nc to connect to this service from virt2 and describe what you observe.*
`nano /etc/inetd.conf` i descomentem el proces 'daytime' per activarlo.
`/etc/init.d/openbsd-inetd restart` per reiniciar el super-daemon.

Tornem a executar `netstat -tnlp | grep 13` i veiem com ara està escoltant.

Anem a virt2, ens connectem amb `root@virt2$ nc 10.1.1.1 13` i veiem com ens retorna el Daytime actual. 
<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ex2.2.png" 
alt="daytime port 13" width="50%" height="50%"/>

*3. In virt1, check that the SSH daemon is listening to TCP port 22. Stop the SSH daemon and check that now the TCP port 22 is not being listened.*
`netstat -tnlp | grep ssh`
<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ex2.3.png" 
alt="daytime port 13" width="75%" height="75%"/>
`service ssh stop`


*4. In virt1, change the configuration of the SSH daemon to listen to port 2222 instead of 22. Check your configuration. To finish this exercise, in virt1 change the configuration again of the SSH daemon to listen to its default port.*
1r modifiquem el fitxer /etc/ssh/sshd_config `nano /etc/ssh/sshd_config`
2n `service ssh start`
3r `netstat -tnlp | grep ssh` i veiem com ara estem escoltant al port 2222


**ex3**
*In this exercise, you have to use netcat to create your own stand-alone servers.

*1. Start a new capture on tap0 with wireshark in the phyhost and try the following command:*
virt1.0$ nc -l -p 12345
*Describe what the previous command does. It is a client or a server? Describe how can you know the ports andopen files related to this netcat process. Now try:*
virt2.0$ nc 10.1.1.1 12345
*Describe the ports and the open files used by each netcat process. Send some text from each machine and terminate the connection. Describe the network parameters of the captured traffic. Use also the follow TCP stream option and describe how it works.*

Quan executem `nc -l -p 12345` iniciem la màquina com a servidor (la opció -l fa que estigui "listening")
Després executant `nc 10.1.1.1 12345` al virt2 iniciem el procés com a client, enviant especificament el nostre stdinput al port 12345.


*2. Start a new capture on tap0 with wireshark in the phyhost. In virt1, create a server with netcat that listens on port 23456 and transfers the file /etc/services.
Tip. Use the option -q 0 to quit after the transmission of the file.
For captured traffic, describe what you observe using the option follow TCP stream.*

`root@virt1$ nc -l -p 23456`
`root@virt2$ cat /etc/services | nc 10.1.1.1 23456 -q0`

Si ara mirem al wireshark veiem com la connexió s'estableix tal que:
<ol>
  <li>virt2 envia un TCP segment de SYN</li>
  <li>virt1 envia el reply amb un SYN i un ACK, i ja tenim la comunicació establerta</li>
  <li>Ara ja veiem tots els paquets TCP del fitxer /etc/services que hem transferit desde virt2       a virt1</li>
</ol>

<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ex3.2.png" 
alt="daytime port 13" width="75%" height="75%"/>

*Connect to the previous server with a nc from virt2 and store the file sent by the server with the name “file.txt”.*

Executem els mateixos comandos que abans, però ara a virt2 li afegim `root@virt2$ cat /etc/services | nc 10.1.1.1 23456 -q0 > file.txt` <-- d'aquesta manera guardem el que contingut que hem enviat a virt1 a un file.txt (localment a virt2).

Si vulguessim guardar el fitxer a virt1, hauriem d'iniciar el servidor i dir-li que el que li entri per stdin ho envii a file.txt tal que: `root@virt1$ nc -l -p 23456 > file.txt`

*3. Start a new capture on tap0 with wireshark in the phyhost. Repeat the steps of the previous exercise but this time using UDP.
Note. In the client you have to type two times “ENTER“ to start the data reception.
For captured traffic, describe what you observe and the differences between the previous capture. Notice that in this case you can use the option follow UDP stream.*

Principals diferències amb UDP:
<ul>
  <li>Fent servir UDP no enviem cap ACK.</li>
  <li>UDP utilitza la màxima Bandwidth tota l'estona</li>
</ul>

*4. In the phyhost, create a server with netcat that listens on port 12345 and emulates the daytime service.
Tip. Use the date command.
After several seconds, try the service from phyhost with nc. Which interface you should use to capture the traffic of the previous network exchange?
Which is the actual date (minutes and seconds) that you observe as output in the client nc? It is correct? Which do you think that is the cause of this behavior?*

<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ex3.4.1.png" 
alt="daytime port 13" width="75%" height="75%"/>
<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ex3.4.2.png" 
alt="daytime port 13" width="50%" height="50%"/>


*5. In virt1, create a server with netcat that listens on port 22333 and provides to the client the amount of free disk in the machine.
Tip. Use the df -h command.
Try the service from virt2 with nc.*

`root@virt1$ df -h | nc -l -p 22333` <-- inicia el servidor al port 22333 i envia el diskfree
`root@virt2$ nc 10.1.1.1 22333` <-- inicia el client i veu el que li arriba desde el servidor

<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/ex2.2.png" 
alt="daytime port 13" width="50%" height="50%"/>


**ex4** *In this exercise, you must create a service under inetd.*

*1. In virt1, implement a service using inetd (without using a netcat as server) that listens on port 22333 and that provides the amount of free disk in the system to the client. Explain the configuration that you have to make.
In this case, can you connect to this service several times? To this respect, explain how this implementation works and the differences with respect to using a simple server with netcat.
Tip. If you have problems, check /var/log/daemon.log which is the inetd’s log file.*

Primer creem l'script i li donem permisos d'escriptura i execució:
```
#!/bin/bash 
df -h
chmod u+x freedisk.sh
```
Llavors editem `/etc/inetd.conf` i li afegim la linia: `freedisk stream tcp nowait root /root/freedisk.sh`

Ara, editant el fitxer /etc/services li assignem el servei al port 22333 (`freedisk 22333/tcp`) 

Fem un restart del servei: `/etc/init.d/openbsd-inetd restart` 

Tornem a executar el client `nc 10.1.1.1 22233` i veiem el disk ocupation.


**ex5**

**ex6***In this exercise, we are going to analyze the remote terminal service which TELNET and SSH.*

*1. Start a new capture on tap0 with wireshark in the phyhost. Try to establish a remote terminal on virt1 using telnet from virt2. Did it work? Explain the output of the telnet command and the captured traffic.
Which do you think that is cause of the behavior observed.*

*2. Activate the TELNET service under inetd in virt1.
Note. Be careful to not to leave any space at the beginning of inetd configuration lines.
Check your configuration with netstat and start a capture on tap0 with wireshark in the phyhost. Try to establish a remote terminal from virt2 to virt1 using telnet. Did it work? explain what you observe. Take a look at the /var/log/daemon.log and describe the messages in that file.*


*3. In general, the root user cannot access with TELNET to a remote machine. To enable to this possibility, you have to enable one or more TTYs in the file /etc/securetty. Each line with pts/X enables a TTY or possible TELNET connection. Start a new capture on tap0 with wireshark in the phyhost, enable a TTY for the root user in virt1 (uncommenting one this lines at the beginning of /etc/securetty) and try again to establish a remote terminal from virt2 to virt1 using telnet from virt2.0. Using virt2.1 and the TELNET session in virt2.0, type a netstat command with the appropriate parameters to check the ports used by the TELNET connection and the processes that have registered these ports. Explain the differences of what you see in virt2.0 and virt2.1. Check the file descriptors used by the telnet client and the telnet server.*

*4. Under the TELNET session, create an empty file called file.txt in the home directory of the root user in virt1 and exit. With virt1.0 check the creation of the file.*

*5. Use the follow TCP stream option of wireshark and comment the TELNET protocol. What do you think about the security of this protocol?*

*6. Capture in tap0 and try an SSH session from virt2 to virt1. Use the follow TCP stream option of wireshark and comment the differences between SSH with TELNET about ports used and security.*





