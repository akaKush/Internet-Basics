# HTTPS

Fins ara hem estat veient els aspectes relacionats amb el protocol HTTP, però hem vist que més d'un cop ens hem trobat amb el mateix problema; transmetre dades públicament pot portar problemes de seguretat.

Per solucionar això, s'implementa el protocol **HTTPS**

Per implementar aquesta capa de seguretat sobre el protocol HTTP s'han utilitzat diversos algoritmes de Criptografia al llarg dels anys.
Els principals algoritmes que s'han fet servir són els següents:

1. **Criptografia SIMÈTRICA**.

    Què vol dir tenir Criptografia Simètrica? ==> Significa que tant l'emissor com el receptor utilitzen la mateixa clau tant per xifrar com per desxifrar el missatge.
    NO significa que s'utilitzin els mateixos algoritmes ni que hi hagi una simetria en la manera com es generen els textos xifrats, simplement que es comparteix la mateixa clau.

    És per això que també se li diu **Criptografia de CLAU simètrica**, o de **secret compartit**.

    L'_algoritme que més s'ha utilitzat en criptografia simètrica és el DES_ (i triple-DES), fins que es va inventar el AES, els quals poden tenir claus de més longituds que el DES.

    Utilitzant aquests sistemes de criptografia pretenem guanyar confidencialitat en els nostres intercanvis de dades a través d'internet, però apareix un problema en la criptografia simètrica, i és el fet de com compartir les claus entre els participants (com poden tenir tant emissor com receptor la mateixa clau? Per això s'havien de conèixer i acordar-ho prèviament).
<br><br>

2. **Criptografia Assimètrica**
   Precisament per aquest problema apareix un nou concepte de Criptografia, **Criptografia de CLAU PÚBLICA**, on l'emissor i receptor *NO necessiten compartir cap secret ni cap clau.*

    En aquest tipus de criptografia, cada usuari té **un parell de claus**:
    - Una clau **pública** (que tothom pot veure).
    - Una clau **privada** (només coneix l'usuari).

    Una de les claus s'utilitzarà per encriptar i l'altre per desencriptar el missatge.

    **Exemple Algoritme Assimètric:**

    Alice vol enviar un missatge xifrat a Bob. Per fer-ho amb criptografia assimètrica ho fa tal que:
    - Alice envia el missatge xifrat, utilitzant la clau PÚBLICA de Bob al seu algoritme de xifrat.
    - El missatge s'encripta amb la clau PÚBLICA de Bob i s'envia cap a Bob (només la clau privada de Bob podrà ara desxifrar el missatge).
    - Bob rep el missatge xifrat i utiliza la seva clau PRIVADA per desencriptar-lo, i així ja el pot llegir.
    
    Els *algoritmes més utilitzats en clau assimètrica són el RSA, DSA (Digital Signature)...*

<br><br>

3. **Funció de HASH**
   La funció de Hash va aparèixer degut al problema de *performance* que teniem a l'hora de buscar dades dins de bases de dades, les quals solien estar ordenades alfabèticament.

   Es va veure que era molt més eficient utilitzar la funció de hash
   - S'assigna un número a cada caràcter del missatge.
   - Es sumen tots aquests números.
   - Es fa mòdul 100 al resultat de la suma
   - Ja tenim el valor de HASH.

   Aquest valor de HASH és molt més fàcil de trobar dins d'una BD.

   Una de les característiques de la funció de HASH és que pot tenir els **INPUTS VARIABLES**, però el **OUTPUT** sempre és **FIXE**.
   El problema que causa això és que poden haver-hi molts inputs diferents que tinguin el mateix output, i per tant es produeixen **col·lisions**.

   Com podem reduir la probabilitat de col·lisions?
   1. Incrementar el mòdul (mod10000)
   2. Utilitzar una codificació diferent (p.ex. ASCII)
   
   Però com aconseguim tenir una probabilitat de col·lissió virtualment impossible? ==> **HASH CRIPTOGRÀFIC**

   Aquest algoritma utilitza el **SHA256**, el qual funciona de la següent manera:
   - Introduim una entrada (input varibale)
   - Retorna 256 bits (output)

   El fet és que des de l'output és pràcticament IMPOSSIBLE (l'únic que es pot fer són atacs de força bruta per buscar combinacions de 256 bits... com veiem és molt poc probable) tornar al missatge original.

   Per tant, com que no podem tornar al missatge original, és possible que es produeixin col·lisions, però no podrem saber d'on venen aquestes col·lisions.
<br><br>

4. **Digital Signatures**
   
   Generació d'una Firma Digital:
   1.  Alice calcula el hash del missatge (HASH FUNCTION)
   2.  Alice encripta el hash amb la seva clau privada (RSA)
   3.  Alice envia el missatge, i el hash signat cap a Bob.
   
   Per entendre aquest procés, veiem com el hash de 256 bits identifica univocament al missatge, i la clau privada identifica univocament a Alice.

   4. Bob rep el missatge xifrat --> Calcula el hash del missatge
   5. Desencripta el hash signat, utilitzant la clau pública de Alice.
   6. Si les dues claus coincideixen, la firma és vàlida i Bob pot accedir al missatge.
<br><br>

5. **Criptografia HÍBRIDA**
   
   Podem dividir els missatges que volem enviar en blocs, encriptant bloc a bloc amb la clau pública del destí.

   El fet és que la criptografia **Assimètrica és més segura**, però la **Simètrica és més ràpida**.
   Combinant les dues, podem aconseguir molt millors rendiments.

   Per utilitzar-les juntes, el que es fa és:
   - Utilitzar Criptografia Assimètrica per intercanviar una **clau de sessió**
   - Utilitzar Criptografia Simètrica per intercanviar les dades encriptades.
<br><br>

6. **Certificats Digitals**
   
   Els Certificats Digitals contenen:
   - Camps per **descriure la identitat**
   - Número de sèrie
   - Període de Validació
   - L'algoritme de la clau pública
   - La clau pública del CA
   - L'algoritme utilitzat per firmar el certificat
   - Camps amb l'objectiu del certificat, i altre informació
<br><br>

7. **OpenSSL**
   
   OpenSSL és l'implementació més àmpliament utilitzada per **crear** i **gestionar** claus i certificats.
   Disponible a la majoria de SO.


<br><br>

8. **HTTPS**
   
   Finalment podem veure com s'implementa el protocol HTTP amb seguretat de que no ens puguin interceptar les dades mentres aquestes viatgen per internet.

   Les WebApps utilitzen crptografia híbrida i certificats digitals per fer les transaccions segures.
   <img src="https://github.com/akaKush/Internet-Basics/blob/main/WWW/Teoria/Pictures/https.png"/>

   Llavors el navegador fa diversos checks de seguretat, per comprovar que estem contactant al servidor adequat:

   1. Està ben firmat el certificat digital?
   2. Coneixo la clau pública del CA?
   3. El certificat ha expirat?
   4. La URL és la mateixa que la del certificat?
   
   Nota: Els navegadors tenen llistes de les claus públiques de CA, per poder cumplir el punt 2.

   **TLS Tunnel:**

   Per comprovar que estem contactant al servidor adequat s'han de cumplir els següents passos a part dels checks del navegador:
   
   **Des del Browser**
   - Es genera una clau de sessió simètrica (Ks) i s'envia al servidor.
   - La clau de sessió és encriptada amb la clau pública (Ku) que apareix al certificat.

   **Des d'un servidor HONEST**
   - Tindrem la clau privada (Kp) corresponent a la pública (Ku) que se'ns ha enviat (si el servidor no fos el corresponent, no tindria la clau privada (Kp) que correspon a la pública (Ku) que figurava al certificat, la qual ens ha enviat el browser per comprovar que siguem el server que toca)
   - Desencriptem el missatge amb la clau privada corresponent i enviem la clau de sessió de tornada al navegador per demostrar que som el servidor que toca. (un server deshonest no podria accedir a la clau de sessió Ks i llavors no podrem demostrar que siguem el server bo)



