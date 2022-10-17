# TP4 : TCP, UDP et services réseau

# 0. Prérequis

# I. First steps

Faites-vous un petit top 5 des applications que vous utilisez sur votre PC souvent, des applications qui utilisent le réseau : un site que vous visitez souvent, un jeu en ligne, Spotify, j'sais po moi, n'importe.

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- Discord : TCP, 99.181.69.250:443, port local 51081
- Client RiotGames : TCP, 162.159.133.234:443, port local 60930
- Twitch : TCP, 52.223.198.2:443, port 52113
- Chrome : TCP, 66.22.199.20:443, port local 62624
- Blitz : TCP, 23.57.5.5:443, port local 60464

🌞 **Demandez l'avis à votre OS**

````
PS C:\Windows\system32> netstat -abfnt
````
- Problème avec les captures Wireshark

# II. Mise en place


## 1. SSH


🌞 **Examinez le trafic dans Wireshark**

- SSH : Utilise de TCP, Port Local : 22, Port distant : 60110

🌞 **Demandez aux OS**

````
PS C:\Windows\system32> netstat -abfnt
[...]
 Impossible d’obtenir les informations de propriétaire
    TCP    10.3.1.1:50138         10.3.1.11:22           ESTABLISHED
````
````
[marce@localhost ~]$ sudo ss -ltpn
State      Recv-Q     Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN     0          128                  0.0.0.0:22                0.0.0.0:*        users:(("sshd",pid=699,fd=3))
LISTEN     0          128                     [::]:22                   [::]:*        users:(("sshd",pid=699,fd=4))
````
- Problème avec les captures Wireshark

## 2. NFS

🌞 **Mettez en place un petit serveur NFS sur l'une des deux VMs**

````
[marce@localhost ~]$ sudo dnf -y install nfs-utils
Complete!
[marce@localhost ~]$ sudo nano /etc/idmapd.conf
[marce@localhost ~]$ sudo nano /etc/exports
[marce@localhost ~]$ cat /etc/exports
/srv/nfsshare 10.3.1.0/24(rw,no_root_squash)
[marce@localhost ~]$ sudo systemctl enable --now rpcbind nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[marce@localhost ~]$ sudo firewall-cmd --add-service=nfs
[marce@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
[marce@localhost ~]$ sudo mkdir /srv/nfsshare
[marce@localhost ~]$ sudo nano /srv/nfsshare/oui.txt
[marce@localhost ~]$ cat /srv/nfsshare/oui.txt
oui (lol)
````
````
[marce@localhost ~]$ sudo dnf -y install nfs-utils
Complete!
[marce@localhost ~]$ sudo nano /etc/idmapd.conf
[marce@localhost ~]$ sudo mount -t nfs 10.3.1.3:/srv/nfsshare /mnt
[marce@localhost ~]$ df -hT /mnt
Filesystem             Type  Size  Used Avail Use% Mounted on
10.3.1.3:/srv/nfsshare nfs4  6.2G  1.2G  5.1G  18% /mnt
[marce@localhost ~]$ cat /mnt/Thib.txt
BOOOOOOOM les gens
````

🌞 **Wireshark it !**

````
[marce@localhost ~]$ sudo tcpdump -i enp0s8 -c 10 -w nfs.pcapng not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
25 packets received by filter
0 packets dropped by kernel
````

🌞 **Demandez aux OS**

````
[marce@localhost srv]$ sudo ss -ltpn | grep 2049
LISTEN 0      64           0.0.0.0:2049       0.0.0.0:*
LISTEN 0      64              [::]:2049          [::]:*
````
````
[marce@localhost ~]$ sudo ss -tp
State         Recv-Q         Send-Q                 Local Address:Port                    Peer Address:Port          Process
ESTAB         0              0                          10.3.1.12:pop3s                      10.3.1.11:nfs
````

- Problème de Wireshark

## 3. DNS

🌞 Utilisez une commande pour effectuer une requête DNS depuis une des VMs

````
[marce@localhost ~]$ sudo tcpdump -i enp0s3 -c 10 -w dns.pcapng not port 22 &
[1] 1654
````
````
[marce@localhost ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37165
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               300     IN      A       172.67.74.226
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       104.26.10.233

;; Query time: 32 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Oct 15 17:32:33 CEST 2022
;; MSG SIZE  rcvd: 85
````
