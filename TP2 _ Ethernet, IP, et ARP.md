# TP2 : Ethernet, IP, et ARP

---

**Toutes les manipulations devront être effectuées depuis la ligne de commande.** Donc normalement, plus de screens.

**Pour Wireshark, c'est pareil,** NO SCREENS. La marche à suivre :

- vous capturez le trafic que vous avez à capturer
- vous stoppez la capture (bouton carré rouge en haut à gauche)
- vous sélectionner les paquets/trames intéressants (CTRL + clic)
- File > Export Specified Packets...
- dans le menu qui s'ouvre, cochez en bas "Selected packets only"
- sauvegardez, ça produit un fichier `.pcapng` (qu'on appelle communément "un ptit PCAP frer") que vous livrerez dans le dépôt git

**Si vous voyez le p'tit pote 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu.**

# I. Setup IP

Le lab, il vous faut deux machine : 

- les deux machines doivent être connectées physiquement
- vous devez choisir vous-mêmes les IPs à attribuer sur les interfaces réseau, les contraintes :
  - IPs privées (évidemment n_n)
  - dans un réseau qui peut contenir au moins 38 adresses IP (il faut donc choisir un masque adapté)
  - oui c'est random, on s'exerce c'est tout, p'tit jog en se levant
  - le masque choisi doit être le plus grand possible (le plus proche de 32 possible) afin que le réseau soit le plus petit possible

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

````
PS C:\WINDOWS\system32> New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.137.1 -PrefixLength 30


IPAddress         : 192.168.137.1
InterfaceIndex    : 5
InterfaceAlias    : Ethernet
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 30
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Tentative
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : ActiveStore

IPAddress         : 192.168.137.1
InterfaceIndex    : 5
InterfaceAlias    : Ethernet
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 30
PrefixOrigin      : Manual
SuffixOrigin      : Manual
AddressState      : Invalid
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : PersistentStore
````
````
PS C:\WINDOWS\system32> ipconfig /all

   Adresse IPv4. . . . . . . . . . . . . .: 192.168.137.1(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.252
````
🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

````
PS C:\WINDOWS\system32> ping 192.168.137.2

Envoi d’une requête 'Ping'  192.168.137.2 avec 32 octets de données :
Réponse de 192.168.137.2 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=3 ms TTL=128
Réponse de 192.168.137.2 : octets=32 temps=3 ms TTL=128
````

🌞 **Wireshark it**

- **Sur le GIT**
- Echo reply : type 0
- Echo Request : 8

# II. ARP my bro

🌞 **Check the ARP table**

- utilisez une commande pour afficher votre table ARP
````
PS C:\Users\marce> arp -a

Interface : 10.5.1.1 --- 0x8
  Adresse Internet      Adresse physique      Type
  10.5.1.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.0.20 --- 0xd
  Adresse Internet      Adresse physique      Type
  192.168.0.1           b8-d9-4d-0d-0b-34     dynamique
  192.168.0.255         ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.200.1.1 --- 0x11
  Adresse Internet      Adresse physique      Type
  10.200.1.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
````
- Gateway :
````
192.168.0.1     b8-d9-4d-0d-0b-34
````

🌞 **Manipuler la table ARP**

````
PS C:\WINDOWS\system32> arp -d
PS C:\WINDOWS\system32> arp -a

Interface : 10.5.1.1 --- 0x8
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.0.20 --- 0xd
  Adresse Internet      Adresse physique      Type
  192.168.0.1           b8-d9-4d-0d-0b-34     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.200.1.1 --- 0x11
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
PS C:\WINDOWS\system32> ping 8.8.8.8

Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
Réponse de 8.8.8.8 : octets=32 temps=15 ms TTL=116
Réponse de 8.8.8.8 : octets=32 temps=35 ms TTL=116
Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=116

PS C:\WINDOWS\system32> arp -a

Interface : 10.5.1.1 --- 0x8
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.0.20 --- 0xd
  Adresse Internet      Adresse physique      Type
  192.168.0.1           b8-d9-4d-0d-0b-34     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.200.1.1 --- 0x11
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
````

🌞 **Wireshark it**

# II.5 Interlude hackerzz

# III. DHCP you too my brooo

🌞 **Wireshark it**

- **Sur le site**
- Dynamic Host Configuration Protocol (Offer) : 
    - **1** Bootp Flags : 
        - You (client) IP address : 192.168.1.26
    - **2** Option (3) Router : 
        - Router : 192.168.1.1
    - **3** Option (1) Subnet Mask : 
        - Subnet Mask : 255.255.255.0

# IV. Avant-goût TCP et UDP

🌞 **Wireshark it**

- IP : 142.250.201.174
- Port SRC : 443
- Port DST : 49714