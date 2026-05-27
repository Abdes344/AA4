 # Guia d'activitat: Destination NAT i VPN amb IPFire

**Autor:** Abdeslam Khfif

---

## Part 1 – Configuració inicial de les MVs

1. **IPFire – eth0 en xarxa NAT** (Adaptador 1)
2. **IPFire – eth1 en xarxa interna** `intnet` (Adaptador 2)
3. **Zorin – en xarxa interna** `intnet`
4. **Client – en xarxa NAT**

> La meva xarxa interna és `192.168.13.0/24`. El Zorin té `192.168.13.2` i l’IPFire a la GREEN té `192.168.13.254`.

---

## Part 2 – Preparació del servidor Zorin

### 2.1 Instal·lar SSH i Apache2

```bash
sudo apt update && sudo apt install openssh-server apache2 -y
```

![Instal·lació de SSH i Apache2](/AA4/1.png)

Execució de l'ordre d'instal·lació des del terminal.

### 2.2 Modificar la pàgina web

```bash
sudo nano /var/www/html/index.html
```

![Edició de l’index.html](/AA4/2.png)

Contingut HTML personalitzat amb el nom `Abdeslam Khfif`.

### 2.3 Prova local de la web

Obrim navegador a `http://localhost`.

![Pàgina web local al Zorin](/AA4/3.png)

Es veu el missatge "Activitat DNAT + VPN" i el nom de l'alumne.

---

## Part 3 – Destination NAT a l’IPFire (port forwarding)

Accedim a la interfície web de l’IPFire: `https://192.168.13.254:444`.

Creem una regla DNAT per al port 80 (HTTP) a **Firewall > Regles del tallafocs**.

![Configuració DNAT - part 1](/AA4/4.png)

Origen "Qualsevol", NAT de destinació, destinació `192.168.13.2`.

![Configuració DNAT - part 2 (protocol i port)](/AA4/5.png)

Protocol TCP, port de destinació 80, port extern buit.

Després de guardar, hem de prémer **Aplicar els canvis**.

---

## Part 4 – Comprovació des del client exterior

Des del client (en xarxa NAT) obrim el navegador i anem a la IP pública de l’IPFire.

![Accés web des del client](/AA4/6.png)

Es veu la mateixa pàgina web del Zorin, cosa que confirma que el DNAT funciona.

---

## Part 5 – Configuració del VPN OpenVPN a l’IPFire

### 5.1 Generar certificats PKI

A **IPFire > Serveis > OpenVPN** > **Autoritats certificadores** > **Generar certificats root/host**.

En intentar generar un certificat d'usuari pot sortir un error si el nom no és vàlid.

![Error en generar certificat](/AA4/7.png)

Missatge "Entrada no válida para el nombre completo del usuario".

### 5.2 Crear connexió Roadwarrior

Anem a **Control i estat de connexió** > **Afegeix**. Triem **Host-to-Net (Roadwarrior)**. Omplim:

- Nom: `Serveis13`
- Xarxa: `10.239.192.0/24` (grup dinàmic)
- Autenticació: **Genera un certificat** amb usuari `usuari` i contrasenya PKCS12.

A la llista de connexions veurem l’estat:

![Llista de connexions OpenVPN](/AA4/8.png)

La connexió `Serveis13` apareix com a DESCONECTADA (encara no està activa).

### 5.3 Opcions avançades

Marquem que el client tingui accés a la xarxa **Green**.

### 5.4 Descarregar el paquet del client

Fem clic a la icona de descàrrega (zip) per obtenir el perfil `.ovpn` i el certificat `.p12`.

---

## Part 6 – Configuració del client VPN (al Zorin)

### 6.1 Editar el fitxer `hosts`

Afegim una entrada per resoldre el nom `ipfire.firewall13.foodlogistic` amb la IP pública de l’IPFire.

![Fitxer /etc/hosts](/AA4/9.png)

Línia `192.168.13.254    ipfire.firewall13.foodlogistic` afegida al final.

### 6.2 Instal·lar OpenVPN i el connector del Network Manager

```bash
sudo apt install openvpn network-manager-openvpn network-manager-openvpn-gnome
```

![Instal·lació de paquets OpenVPN](/AA4/10.png)

Terminal mostrant la instal·lació dels tres paquets.

### 6.3 Descomprimir el fitxer descarregat

Descarreguem el zip des de l’IPFire (a la carpeta Downloads).

![Carpeta de descàrregues](/AA4/11.png)

Visualització de la carpeta `Downloads` on es troba el fitxer zip.

### 6.4 Importar la configuració al Network Manager

Anem a **Configuració > Xarxa**, afegim una VPN, importem des del `.ovpn`.

Omplim:

- Nom: `Serveis13`
- Passarel·la: `1194`
- Autenticació: **Password with Certificates (TLS)**
- Usuari: `usuari`
- Certificats: apuntem al mateix fitxer `.p12` per als tres camps
- Contrasenya: la del PKCS12

![Configuració VPN al Network Manager](/AA4/12.png)

*Descripció:* Formulari d'importació de la VPN amb els camps omplerts.

### 6.5 Connectar la VPN

Al menú de xarxa (dreta superior), seleccionem la VPN i connectem.

![Menú de xarxa amb VPN connectada](/AA4/13.png)

Opció "Serveis13 VPN" disponible i connectada.

### 6.6 Comprovar connectivitat a Internet

Fem `ping google.com` per veure que la VPN no bloqueja la sortida.

![Prova de ping a Google](/AA4/14.png)

Resposta correcta dels paquets, 0% de pèrdua.

---

## Part 7 – Comprovacions finals a través de la VPN

Un cop connectats per VPN, el client (Zorin) pot accedir als serveis del mateix Zorin (o a qualsevol equip de la xarxa interna) usant la IP privada `192.168.13.2`.

- **Web:** Obrir navegador a `http://192.168.13.2` → ha de mostrar la mateixa pàgina.
- **SSH:** `ssh usuari@192.168.13.2`

Aquestes proves confirmen que la VPN funciona i que som dins de la xarxa interna.
