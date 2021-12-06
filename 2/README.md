# TP2 : On va router des trucs





### 1. Echange ARP

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre

```
node1 -> node2 : ping 10.2.1.12
PING 10.2.1.12 (10.2.1.12) 56(84) bytes of data.
64 bytes from 10.2.1.12: icmp_seq=1 ttl=64 time=0.669 ms
64 bytes from 10.2.1.12: icmp_seq=2 ttl=64 time=0.515 ms
64 bytes from 10.2.1.12: icmp_seq=3 ttl=64 time=0.931 ms
```

```
node2 -> node1 : ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=64 time=0.294 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=64 time=0.990 ms
```

- observer les tables ARP des deux machines
- repérer l'adresse MAC de `node1` dans la table ARP de `node2` et vice-versa

```
node1 :  ip n s
10.2.1.12 dev enp0s8 lladdr 08:00:27:80:22:38 REACHABLE
```


```
node2 : ip n s
10.2.1.11 dev enp0s8 lladdr 08:00:27:a5:51:b2 STALE
```
- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)
  - une commande pour voir la table ARP de `node1`

```
node1 : ip n s
10.2.1.12 dev enp0s8 lladdr 08:00:27:80:22:38 STALE
```


 - et une commande pour voir la MAC de `node2`

```
ip a :
enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:80:22:38
```

### 2. Analyse de trames

🌞**Analyse de trames**

- utilisez la commande `tcpdump` pour réaliser une capture de trame

` sudo tcpdump -i enp0s8 -w tp2_arp.pcap not port 22`

- videz vos tables ARP, sur les deux machines, puis effectuez un `ping`

`sudo ip neigh flush all`

- stoppez la capture, et exportez-la sur votre hôte pour visualiser avec Wireshark
- mettez en évidence les trames ARP
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, je veux TOUTES les trames**

Par exemple (copiez-collez ce tableau ce sera le plus simple) :

| ordre | type trame  | source                     | destination                |       |             |                            |
|-------|-------------|----------------------------|---------------------------|
| 1     | Requête ARP | `PcsCompus_80:22:38`       | Broadcast  |
|
| 2     | Réponse ARP | `PcsCompus_05:51:b2`       | `PcsCompus_80:22:38`   
|
| 3     | Requête ARP | `PcsCompus_05:51:b2`       |  `PcsCompus_80:22:38`    |                 
| 4     | Réponse ARP | `PcsCompus_80:22:38`       |  `PcsCompus_05:51:b2`


📁 Capture `tp2_arp.pcap`




## II. Routage

### 1. Mise en place du routage



🌞**Activer le routage sur le noeud `router.2.tp2`**

```
[gaetan@router ~]$ sudo firewall-cmd --add-masquerade --zone=public
success
```

```
[gaetan@router ~]$  sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```


🌞**Ajouter les routes statiques nécessaires pour que `node1.net1.tp2` et `marcel.net2.tp2` puissent se `ping`**


`node1: 10.2.2.0/24 via 10.2.2.11`
`marcel : 10.2.1.0/24 via 10.2.1.11`
```
[gaetan@node1 ~]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=0.711 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=0.865 ms
```

### 2. Analyse de trames

🌞**Analyse des échanges ARP**


- videz les tables ARP des trois noeuds

`sudo ip neigh flush all`


- effectuez un `ping` de `node1.net1.tp2` vers `marcel.net2.tp2`

```
[gaetan@node1 ~]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=1.38 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=0.948 ms
```

- regardez les tables ARP des trois noeuds

```
[gaetan@node1 network-scripts]$ ip n s
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:3f REACHABLE
10.2.1.11 dev enp0s8 lladdr 08:00:27:ea:b3:d6 REACHABLE
```
```
[gaetan@router ~]$ ip n s
10.2.2.12 dev enp0s9 lladdr 08:00:27:da:7e:1d REACHABLE
10.2.1.12 dev enp0s8 lladdr 08:00:27:a5:51:b2 REACHABLE
```
```
[gaetan@marcel network-scripts]$ ip n s
10.2.2.1 dev enp0s8 lladdr 0a:00:27:00:00:44 DELAY
10.2.2.11 dev enp0s8 lladdr 08:00:27:c5:8b:3d REACHABLE
```




- essayez de déduire un peu les échanges ARP qui ont eu lieu

`sur node1 on peut voir que node1 (10.2.1.12) communique avec la passerelle qui est dans le lan de node1 (10.2.1.11).`

`Sur router on peut voir les ip des deux ports ethernet (un dans chaque lan).`

`Sur marcel on peut voir seulement l'IP de la passerelle de son reseau lan (10.2.2.11) qui lui a envoyé un packet.`

- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur les 3 noeuds, afin de capturer les échanges depuis les 3 points de vue




- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames**

| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
|-------|-------------|-----------|---------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `node1` `AA:BB:CC:DD:EE`  | x              | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | 10.2.2.1         | `marcel` `PcsCompu_da:7e1d` | 10.2.2.12             | `0a:00:27:00:00:44`    |
| ...   | ...         | ...       | ...                       |                |                            |
| ?     | Ping        | 10.2.2.1         | `0a:00:27:00:00:44`                         | 10.2.2.12              | `marcel` `PcsCompu_da:7e1d`                          |
| ?     | Pong        | 10.2.2.12        | `marcel` `PcsCompu_da:7e1d`                         | 10.2.2.1              | `0a:00:27:00:00:44`                          |


### 3. Accès internet

- 🌞**Donnez un accès internet à vos machines**

- le routeur a déjà un accès internet
- ajoutez une route par défaut à `node1.net1.tp2` et `marcel.net2.tp2`
```
node1 : default via 10.2.1.11
marcel : default via 10.2.2.11
```
- vérifiez que vous avez accès internet avec un `ping`

```
[gaetan@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=22.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=22.6 ms
```

```
[gaetan@marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=23.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=22.2 ms
```

  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`

```
[gaetan@marcel ~]$ dig diglab.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> diglab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54210
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;diglab.com.                    IN      A

;; ANSWER SECTION:
diglab.com.             10800   IN      CNAME   comingsoon.namebright.com.
comingsoon.namebright.com. 300  IN      CNAME   cdl-lb-1356093980.us-east-1.elb.amazonaws.com.
cdl-lb-1356093980.us-east-1.elb.amazonaws.com. 60 IN A 54.204.55.163
cdl-lb-1356093980.us-east-1.elb.amazonaws.com. 60 IN A 54.85.93.188

;; Query time: 410 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Sep 23 16:38:28 CEST 2021
;; MSG SIZE  rcvd: 163
```
```
[gaetan@node1 ~]$ dig gitlab.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 840
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;gitlab.com.                    IN      A

;; ANSWER SECTION:
gitlab.com.             149     IN      A       172.65.251.78

;; Query time: 21 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Sep 23 16:40:09 CEST 2021
;; MSG SIZE  rcvd: 55
```


  - puis avec un `ping` vers un nom de domaine

```
ping nike.com
PING nike.com (13.32.158.68) 56(84) bytes of data.
64 bytes from server-13-32-158-68.cdg50.r.cloudfront.net (13.32.158.68): icmp_seq=1 ttl=238 time=19.5 ms
64 bytes from server-13-32-158-68.cdg50.r.cloudfront.net (13.32.158.68): icmp_seq=2 ttl=238 time=21.0 ms
64 bytes from server-13-32-158-68.cdg50.r.cloudfront.net (13.32.158.68): icmp_seq=3 ttl=238 time=21.3 ms
64 bytes from server-13-32-158-68.cdg50.r.cloudfront.net (13.32.158.68): icmp_seq=4 ttl=238 time=19.10 ms
```

🌞Analyse de trames

- effectuez un `ping 8.8.8.8` depuis `node1.net1.tp2`

```
[gaetan@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=22.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=20.6 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=21.1 ms
```

- capturez le ping depuis `node1.net1.tp2` avec `tcpdump`

| ordre | type trame | IP source           | MAC source               | IP destination | MAC destination |     |
|-------|------------|---------------------|--------------------------|----------------|-----------------|-----|
| 1     | ping       | `node1` `10.2.1.12` | `node1` `PcsCompu_a5:51b2` | `8.8.8.8`      |  `PcsCompu_ea:b3d6`              |     |
| 2     | pong       | `8.8.8.8`                 | `PcsCompu_ea:b3d6`                      | `node1` `10.2.1.12`            | `node1` `PcsCompu_a5:51b2`             | ... |

## III. DHCP

- On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `node1.net1.tp2`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `node1.net1.tp2`

```
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.2.1.0 netmask 255.255.255.0 {
  range 10.2.1.2 10.2.1.10;
  option routers 10.2.1.11;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 1.1.1.1;

}
```

- créer une machine `node2.net1.tp2`

`fait ;)`

- faites lui récupérer une IP en DHCP à l'aide de votre serveur


```
[gaetan@node2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:cc:55:bf brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:80:22:38 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.2/24 brd 10.2.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 826sec preferred_lft 826sec
    inet6 fe80::a00:27ff:fe80:2238/64 scope link
       valid_lft forever preferred_lft forever
```

- 🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :

- une route par défaut

```
[gaetan@node2 ~]$ ip route show
default via 10.2.1.11 dev enp0s8 proto dhcp metric 102
10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.2 metric 102
```
- un serveur DNS à utiliser

`  option domain-name-servers 1.1.1.1;`

- récupérez de nouveau une IP en DHCP sur `node2.net1.tp2` pour tester :
- `node2.net1.tp2` doit avoir une IP

`sudo dhclient`


- vérifier avec une commande qu'il a récupéré son IP


```
[gaetan@node2 ~]$ ip a
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:80:22:38 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.3/24 brd 10.2.1.255 scope global dynamic enp0s8
       valid_lft 773sec preferred_lft 773sec
    inet6 fe80::a00:27ff:fe80:2238/64 scope link
       valid_lft forever preferred_lft forever
```

`avant j'étais en 10.2.1.2 ;)`

- vérifier qu'il peut `ping` sa passerelle

```
[gaetan@node2 ~]$ ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=64 time=0.828 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=64 time=0.518 ms
```


- il doit avoir une route par défaut

```
[gaetan@node2 ~]$ ip route show
default via 10.2.1.11 dev enp0s8 proto dhcp metric 102
10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.2 metric 102
```

- vérifier la présence de la route avec une commande

```
[gaetan@node2 ~]$ ip route show
default via 10.2.1.11 dev enp0s8 proto dhcp metric 102
10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.2 metric 102
```



- vérifier que la route fonctionne avec un `ping` vers une IP

```
[gaetan@node2 ~]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=0.895 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=0.981 ms
```

- il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms

```
[gaetan@node2 ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=62 time=4.21 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=62 time=3.84 ms
```


- vérifier avec la commande `dig` que ça fonctionne.

```
[gaetan@node2 ~]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16281
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               2739    IN      A       92.243.16.143

;; Query time: 63 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Sep 23 19:51:12 CEST 2021
;; MSG SIZE  rcvd: 53
```

- vérifier un `ping` vers un nom de domaine


```
[gaetan@node2 ~]$ ping nike.com
PING nike.com (143.204.225.14) 56(84) bytes of data.
64 bytes from server-143-204-225-14.cdg3.r.cloudfront.net (143.204.225.14): icmp_seq=1 ttl=243 time=116 ms
64 bytes from server-143-204-225-14.cdg3.r.cloudfront.net (143.204.225.14): icmp_seq=2 ttl=243 time=45.4 ms
```

### 2. Analyse de trames

🌞**Analyse de trames**

- videz les tables ARP des machines concernées

`sudo ip neigh flush all`

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP

` sudo tcpdump -i enp0s8 -w tp2_dhcp.pcap not port 22`

- choisissez vous-mêmes l'interface où lancer la capture
- répéter une opération de renouvellement de bail DHCP, et demander une nouvelle IP afin de générer un échange DHCP
`sudo dhclient`

- exportez le fichier `.pcap`
- mettez en évidence l'échange DHCP *DORA* (Disover, Offer, Request, Acknowledge)
- **écrivez, dans l'ordre, les échanges ARP + DHCP qui ont lieu, je veux TOUTES les trames**


`1   pcsCompu_a5:51:b2  ||   0a:00:27:00:00:3f  || ARP||42|| Who has 10.2.1.1?Tell 10.2.1.12`
`2   0a:00:27:00:00:3f || pcsCompu_a5:51:b2 || ARP || 60 || 10.2.1.1 is at 0a:00:27:00:00:3f`
`3   10.2.1.3 || 255.255.255.255 || DCHP || 342 DCHP - transaction ID 0xc2d1f91c`
`4   10.2.1.3 || 255.255.255.255 || DHCP || 342 DCHP - transaction ID 0xc2d1f91c`
