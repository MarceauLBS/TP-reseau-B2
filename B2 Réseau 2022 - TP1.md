# B2 Réseau 2022 - TP1

# TP1 - Mise en jambes

# I. Exploration locale en solo

## 1. Affichage d'informations sur la pile TCP/IP locale

### En ligne de commande

**🌞 Affichez les infos des cartes réseau de votre PC**

- commande :
````ìpconfig /all````
- Carte réseau sans fil Wi-Fi, E4-42-A6-A3-75-78 et 10.33.18.120
- Carte Ethernet Ethernet, 10-7B-44-2A-2B-25 et pas d'adresse ip
**🌞 Affichez votre gateway**

- Passerelle par défaut. . . . . . . . . : 10.33.19.254
  
### En graphique (GUI : Graphical User Interface)

**🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

- Paramètres -> Réseau et internet -> État --> Afficher les propriétés du matériel et de la connexion

### Questions

**🌞 à quoi sert la [gateway](../../cours/lexique/README.md#passerelle-ou-gateway) dans le réseau d'YNOV ?**
- Elle sert au routage des paquets, mais elle peut servir de pare-feu ou de proxy.

## 2. Modifications des informations

### A. Modification d'adresse IP (part 1)  

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

- changez l'adresse IP de votre carte WiFi pour une autre
- ne changez que le dernier octet
  - par exemple pour `10.33.1.10`, ne changez que le `10`
  - valeur entre 1 et 254 compris
  
````Adresse IPv4. . . . . . . . . . . . . .: 10.33.18.122(en double)````

🌞 **Il est possible que vous perdiez l'accès internet.** Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.
- Car il se peut qu'on choisisse la même ip qu'un autre déjà connecté.

---

# II. Exploration locale en duo

## 1. Prérequis

## 2. Câblage

## Création du réseau (oupa)

## 3. Modification d'adresse IP

🌞Si vos PCs ont un port RJ45 alors y'a une carte réseau Ethernet associée :

````
Carte Ethernet Ethernet :
Adresse IPv4. . . . . . . . . . . . . .: 192.168.0.1(préféré)
````
````
C:\Users\marce>ping 192.168.0.2

Envoi d’une requête 'Ping'  192.168.0.2 avec 32 octets de données :
Réponse de 192.168.0.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.0.2 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.0.2 : octets=32 temps=2 ms TTL=128
````
````
C:\Users\marce>arp -a

Interface : 192.168.0.1 --- 0x5
  Adresse Internet      Adresse physique      Type
  192.168.0.2           8c-8c-aa-be-62-5a     dynamique
  192.168.0.3           ff-ff-ff-ff-ff-ff     statique
  ````

## 4. Utilisation d'un des deux comme gateway
---

- 🌞 pour tester la connectivité à internet on fait souvent des requêtes simples vers un serveur internet connu
````
PS C:\Users\justi> ping 8.8.8.8

Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
Réponse de 8.8.8.8 : octets=32 temps=22 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=23 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=22 ms TTL=113
````
- 🌞 utiliser un `traceroute` ou `tracert` pour bien voir que les requêtes passent par la passerelle choisie (l'autre le PC)
````
PS C:\Users\justi> tracert 8.8.8.8

Détermination de l’itinéraire vers dns.google [8.8.8.8]
avec un maximum de 30 sauts :

  1     1 ms     1 ms    <1 ms  192.168.137.1
  2     *        *        *     Délai d’attente de la demande dépassé.
  3    18 ms     4 ms     2 ms  10.33.19.254
  4     4 ms     4 ms     5 ms  137.149.196.77.rev.sfr.net [77.196.149.137]
  5    10 ms    10 ms     9 ms  108.97.30.212.rev.sfr.net [212.30.97.108]
  6    20 ms    20 ms    21 ms  222.172.136.77.rev.sfr.net [77.136.172.222]
  7    21 ms    21 ms    24 ms  221.172.136.77.rev.sfr.net [77.136.172.221]
  8    24 ms    22 ms    23 ms  186.144.6.194.rev.sfr.net [194.6.144.186]
  9    23 ms    23 ms    23 ms  186.144.6.194.rev.sfr.net [194.6.144.186]
 10    22 ms    22 ms    21 ms  72.14.194.30
 11    24 ms    23 ms    22 ms  172.253.69.49
 12    24 ms    23 ms    22 ms  108.170.238.107
 13    26 ms    23 ms    23 ms  dns.google [8.8.8.8]

Itinéraire déterminé.
````

## 5. Petit chat privé

Sur un Windows, ça donne un truc comme ça :

```schema
C:\Users\It4\Desktop\netcat-win32-1.11>nc.exe -h
[v1.11 NT www.vulnwatch.org/netcat/]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [options] [hostname] [port]
options:
        -d              detach from console, background mode

        -e prog         inbound program to exec [dangerous!!]
        -g gateway      source-routing hop point[s], up to 8
        -G num          source-routing pointer: 4, 8, 12, ...
        -h              this cruft
        -i secs         delay interval for lines sent, ports scanned
        -l              listen mode, for inbound connects
        -L              listen harder, re-listen on socket close
        -n              numeric-only IP addresses, no DNS
        -o file         hex dump of traffic
        -p port         local port number
        -r              randomize local and remote ports
        -s addr         local source address
        -t              answer TELNET negotiation
        -u              UDP mode
        -v              verbose [use twice to be more verbose]
        -w secs         timeout for connects and final net reads
        -z              zero-I/O mode [used for scanning]
port numbers can be individual or ranges: m-n [inclusive]
```

L'idée ici est la suivante :

- l'un de vous jouera le rôle d'un *serveur*
- l'autre sera le *client* qui se connecte au *serveur*

Précisément, on va dire à `netcat` d'*écouter sur un port*. Des ports, y'en a un nombre fixe (65536, on verra ça plus tard), et c'est juste le numéro de la porte à laquelle taper si on veut communiquer avec le serveur.

Si le serveur écoute à la porte 20000, alors le client doit demander une connexion en tapant à la porte numéro 20000, simple non ?  

Here we go :

- 🌞 **sur le PC *serveur*** avec par exemple l'IP 192.168.1.1
````
PS C:\Users\marce\Downloads> ncat.exe -l -p 8888
ccccccccccccaaacc
caccaca

léo le plus bo
````
- 🌞 **sur le PC *client*** avec par exemple l'IP 192.168.1.2
````
PS C:\Users\justi\Downloads> ncat.exe 192.168.137.1 8888
libnsock ssl_init_helper(): OpenSSL legacy provider failed to load.

ccccccccccccaaacc
caca
cac
léo le plus bo
````
---
- 🌞 pour aller un peu plus loin
````
PS C:\Users\marce\Downloads> .\nc.exe -l -p 8888 -s 192.168.137.1
oui

PS C:\Users\justi\Downloads> .\nc64.exe 192.168.137.1 8888
oui 
ouiiiiiiiiii
````

## 6. Firewall

- 🌞 Autoriser les `ping`
````
PS C:\WINDOWS\system32> netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=out action=allow
Ok.

PS C:\WINDOWS\system32> netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
Ok.
````
- 🌞 Autoriser le traffic sur le port qu'utilise `nc`
  - on parle bien d'ouverture de **port** TCP et/ou UDP
  - on ne parle **PAS** d'autoriser le programme `nc`
  - choisissez arbitrairement un port entre 1024 et 20000
  - vous utiliserez ce port pour [communiquer avec `netcat`](#5-petit-chat-privé-) par groupe de 2 toujours
  - le firewall du *PC serveur* devra avoir un firewall activé et un `netcat` qui fonctionne
  
# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

🌞Exploration du DHCP, depuis votre PC

```` 
PS C:\Users\marce> ipconfig /all
Carte réseau sans fil Wi-Fi :

Bail obtenu. . . . . . . . . . . . . . : mercredi 28 septembre 2022 22:16:28
Bail expirant. . . . . . . . . . . . . : jeudi 29 septembre 2022 22:16:28
Passerelle par défaut. . . . . . . . . : 192.168.0.1
Serveur DHCP . . . . . . . . . . . . . : 192.168.0.1
````

## 2. DNS

Le protocole DNS permet la résolution de noms de domaine vers des adresses IP. Ce protocole permet d'aller sur `google.com` plutôt que de devoir connaître et utiliser l'adresse IP du serveur de Google.  

Un **serveur DNS** est un serveur à qui l'on peut poser des questions (= effectuer des requêtes) sur un nom de domaine comme `google.com`, afin d'obtenir les adresses IP liées au nom de domaine.  

Si votre navigateur fonctionne "normalement" (il vous permet d'aller sur `google.com` par exemple) alors votre ordinateur connaît forcément l'adresse d'un serveur DNS. Et quand vous naviguez sur internet, il effectue toutes les requêtes DNS à votre place, de façon automatique.

- 🌞 trouver l'adresse IP du serveur DNS que connaît votre ordinateur
````
PS C:\Users\marce> ipconfig /all
Carte réseau sans fil Wi-Fi :

   Serveurs DNS. . .  . . . . . . . . . . : 89.2.0.1
                                       89.2.0.2
````

- 🌞 utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main
````
PS C:\Users\marce> nslookup
Serveur par dÚfaut :   ns1.numericable.net
Address:  89.2.0.1

> google.com
Serveur :   ns1.numericable.net
Address:  89.2.0.1

Réponse ne faisant pas autorité :
Nom :    nc-ass-vip.sdv.fr
Address:  212.95.74.75
Aliases:  google.com.numericable.fr

> ynov.com
Serveur :   ns1.numericable.net
Address:  89.2.0.1

Réponse ne faisant pas autorité :
Nom :    nc-ass-vip.sdv.fr
Address:  212.95.74.75
Aliases:  ynov.com.numericable.fr
````
  - déterminer l'adresse IP du serveur à qui vous venez d'effectuer ces requêtes
````
Address:  212.95.74.75
````
  - faites un *reverse lookup* (= "dis moi si tu connais un nom de domaine pour telle IP")
````
PS C:\Users\marce> nslookup
Serveur par dÚfaut :   ns1.numericable.net
Address:  89.2.0.1

> 78.74.21.21
Serveur :   ns1.numericable.net
Address:  89.2.0.1

Nom :    host-78-74-21-21.homerun.telia.com
Address:  78.74.21.21

> 92.146.54.88
Serveur :   ns1.numericable.net
Address:  89.2.0.1

*** ns1.numericable.net ne parvient pas à trouver 92.146.54.88 : Non-existent domain
````
- L'ip 78.74.21.21 es associé à un nom de domaine (homerun.telia.com) tandis que l'ip 92.146.54.88 n'est associé à aucun non de domaine

# IV. Wireshark

- 🌞 utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :
  - un `ping` entre vous et la passerelle
  - un `netcat` entre vous et votre mate, branché en RJ45
  - une requête DNS. Identifiez dans la capture le serveur DNS à qui vous posez la question.
  - prenez moi des screens des trames en question
  - on va prendre l'habitude d'utiliser Wireshark souvent dans les cours, pour visualiser ce qu'il se passe