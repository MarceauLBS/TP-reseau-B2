# TP3 : On va router des trucs

## Sommaire

## 0. Prérequis

## I. ARP

### 1. Echange ARP

🌞**Générer des requêtes ARP**

````
[marce@john ~]$ ping 1 10.3.1.12

PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.623 ms

[marce@marcel ~]$ ping 1 10.3.1.11

PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.338 ms
````
````
[marce@john ~]$ ip neigh
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:0b REACHABLE
10.3.1.12 dev enp0s8 lladdr 08:00:27:28:e9:56 DELAY
[marce@marcel ~]$ ip neigh
10.3.1.11 dev enp0s8 lladdr 08:00:27:38:c3:24 STALE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:0b REACHABLE
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
````
- Adresse MAC de : 
    - John : ````08:00:27:38:c3:24````
    - Marcel : ````08:00:27:28:e9:56````

### 2. Analyse de trames

🌞**Analyse de trames**

````
[marce@john ~]$ sudo ip neigh flush all
[marce@john ~]$ sudo tcpdump -c 4 -i enp0s8 -w tp2_arp.pcapng arp
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
2 packets captured
3 packets received by filter
````

## II. Routage

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

````
[marce@router ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s3 enp0s8
[marce@router ~]$ sudo firewall-cmd --add-masquerade --zone=public
success
[marce@router ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
````

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut ajouter une seule route des deux côtés
````
[marce@john ~]$ sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s8
[marce@marcel ~]$ sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s8
````
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre
````
[marce@marcel ~]$ ip add show enp0s8

3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:28:e9:56 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s8
    valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe28:e956/64 scope link
    valid_lft forever preferred_lft forever
````
### 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds
````
[marce@john ~]$ sudo ip neigh flush all
[marce@marcel ~]$ sudo ip neigh flush all
[marce@router ~]$ sudo ip neigh flush all
````
- effectuez un `ping` de `john` vers `marcel`
````
[marce@john ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.94 ms
````
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange


| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `john` `08:00:27:38:c3:24` | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         | `routeur 08:00:27:d4:05:3e`                       | x              | `john` `08:00:27:38:c3:24`    |
| 3   | Requête ARP | x | `routeur 08:00:27:7a:0e:51` | x | Broadcast `FF:FF:FF:FF:FF` |
| 4 | Réponse ARP | x | `marcel` `08:00:27:28:e9:56` | x | `routeur 08:00:27:7a:0e:51` 
| 5 | Ping        | `10.3.1.11`         | `john` `08:00:27:38:c3:24` | 10.3.2.12 | `marcel` `08:00:27:28:e9:56` |
| 6     | Pong        | `10.3.2.12`         | `marcel` `08:00:27:28:e9:56` | 10.3.1.11 | `john` `08:00:27:38:c3:24` |

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet
````
[marce@router ~]$ ip a | grep enp0s9
    inet 10.0.4.15/24 brd 10.0.4.255 scope global dynamic noprefixroute enp0s9
````
- ajoutez une route par défaut à `john` et `marcel`
  - vérifiez que vous avez accès internet avec un `ping`
  - le `ping` doit être vers une IP, PAS un nom de domaine
````
[marce@john ~]$ sudo ip route add default via 10.3.1.254 dev enp0s8
[marce@john ~]$ ping 8.8.8.8
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=14.7 ms

(Même chose pour Marcel)
````
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine
````
[marce@john ~]$ sudo echo "DNS=1.1.1.1" >> /etc/sysconfig/network-scripts/ifcfg-enp0s3
[marce@john ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 | grep DNS
DNS=1.1.1.1
[marce@john ~]$ sudo systemctl restart NetworkManager

[marce@john ~]$ dig gitlab.com | head -n 14 | tail -n 1
gitlab.com.             236     IN      A       172.65.251.78

[marce@john ~]$ ping gitlab.com
PING gitlab.com (172.65.251.78) 56(84) bytes of data.
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=1 ttl=61 time=25.7 ms

(Même chose avec Marcel)
````

🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
````
[marce@john ~]$ dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
ping -c 1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=37.5 ms
````
- capturez le ping depuis `john` avec `tcpdump`

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination | 
|-------|------------|--------------------|-------------------------|----------------|-----------------|
| 1     | ping       | `john` `10.3.1.12` | `john` `AA:BB:CC:DD:EE` | `8.8.8.8`      | `08:00:27:d4:05:3e` |
| 2     | pong       | `8.8.8.8` | `08:00:27:d4:05:3e` | `john` `10.3.1.12` | `john` `08:00:27:38:c3:24` |

## III. DHCP

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`
````
[marce@john ~]$ sudo dnf install dhcp-server -y

Complete!
````
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur
````
[marce@bob ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3 
NAME=enp0s3
DEVICE=enp0s3

BOOTPROTO=dhcp
ONBOOT=yes

[marce@bob ~]$ sudo systemctl restart NetworkManager
[marce@bob ~]$ ip a | grep dynamic
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s3
````

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
 - une route par défaut
````
[marce@john ~]$ sudo cat /etc/dhcp/dhcpd.conf | grep routers
option routers 10.3.1.254;
````
- un serveur DNS à utiliser
````
[marce@john ~]$ sudo cat /etc/dhcp/dhcpd.conf | grep domain
option domain-name-servers 1.1.1.1;
````

- récupérez de nouveau une IP en DHCP sur `bob` pour tester :
 - `marcel` doit avoir une IP
````
[marce@bob ~]$ ip a | grep dynamic
    inet 10.3.1.4/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
*
[marce@bob ~]$ ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.642 ms

````
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
    - vérifier que la route fonctionne avec un `ping` vers une IP
````
[marce@bob ~]$ ip route
default via 10.3.1.254 dev enp0s8 proto dhcp src 10.3.1.4 metric 101

[marce@bob ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=58.8 ms
````
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
    - vérifier un `ping` vers un nom de domaine
````
[marce@bob ~]$ dig gitlab.com | head -n 14 | tail -n 1
gitlab.com.             290     IN      A       172.65.251.78

[marce@bob ~]$ ping gitlab.com
PING gitlab.com (172.65.251.78) 56(84) bytes of data.
64 bytes from 172.65.251.78 (172.65.251.78): icmp_seq=1 ttl=61 time=37.5 ms
````

### 2. Analyse de trames

🌞**Analyse de trames**

````
[marce@john ~]$ sudo tcpdump -i enp0s3 -c 10 -w test.pcap not port 22
````
````
[marce@bob ~]$ sudo dhclient -r enp0s3
[marce@bob ~]$ sudo dhclient enp0s3
````