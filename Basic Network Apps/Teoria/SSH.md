<h2> SSH - Secure SHell </h2>

SSH és el protocol que substitueix TELNET i FTP, ja que és encriptat. En els dos anteriors, tant l'usuari com la pwd estan obertes, per tant qualsevol que ens intercepti el traffic de red ens pot robar les dades.

La idea de SSH és també la d'utilitzar un terminal remot desde la pròpia màquina.

Per iniciar el servei ssh de la forma més bàsica, executem `ssh 192.168.0.1`. Aquest comando intentarà establir una sessió SSH sobre un socket TCP entre el **client ssh** i el **servidor sshd**.

Per defecte, el servei assumeix que vols autentificar-te amb l'usuari que estàs fent servir actualment.
Si volem fer servir un altre usuari per entrar a la màquina remota, ho fem tal que:
`user1$ ssh user2@192.168.0.1` <-- *després ens demanarà la pwd del user2.*

La **configuracio** del sshd server està a **/etc/ssh/sshd_config**. Sempre que es canvii s'ha de reiniciar per aplicar els canvis.

Per iniciar/aturar el servei *sshd*:
`/etc/init.d/ssh stop`
`/etc/init.d/ssh start`

---

SSH també ofereix la opció de fer transferència d'arxius per ssh. Les dues principals utilitats de transferència de dades són **SCP** i **SFTP**.

---

<h3> Secure Copy (SCP) </h3>
El comando `scp` és un *programa client* que utilitza el protocol SSH per enviar i rebre arxius mitjançant una connexió encriptada.

--> Com fem servir SCP?
*Exemple: Copiar un arxiu "file.txt" del client al servidor*
`scp file.txt username@remotehost:`

*Si volem afegir un **nou nom** al fitxer:*
`scp file.txt username@remotehost:anothername.txt`

*Si volem **canviar el directori** a on guardarem el fitxer:*
`scp file.txt username@remotehost:mydirectory/anothername.txt`

*Si volem **copiar un Directori** (Documents en aquest cas) al home directory del remote host*
`scp -r Documents username@remotehost:`

**PER COPIAR DEL SERVIDOR AL CLIENT** només cal canviar el fitxer d'ordre, i indicar-lo que ara el volem rebre i a on el volem enviar.
`scp username@remostehost:file.txt .` **fixem-nos** en l'ultim punt del comando anterior. Aquest indica que volem copiar el fitxer al directori actual.

---

<h3> Secure File Transfer Protocol (SFTP) </h3>
SFTP és la implementació segura del protocol FTP, mitjançant una sessió SSH.

--> Com fem servir SFTP?
`sftp user@hostname`
Exemple:
<img src="https://github.com/akaKush/Basic-Network-Applications/blob/main/Practica3/sftp.png" 
alt="daytime port 13" width="100%" height="100%"/>

Comandos del **stfp shell**
<ul>
  <li> cd --> Changes the current directory on the remote machine </li>
  <li> lcd --> Changes the current directory on localhost </li>
  <li> ls --> lists the remote directory contents</li>
  <li> lls --> Lists the local directory contents</li>
  <li> put --> Send/Upload files to the remote machine from the working directory of the localhost</li>
  <li> get --> Receive/Download files from the remote machine to the current working directory of the localhost</li>
</ul>

