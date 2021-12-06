# TP3 : Progressons vers le réseau d'infrastructure


# I. (mini)Architecture réseau

- 🌞 Vous pouvez d'ores-et-déjà créer le routeur. Pour celui-ci, vous me prouverez que :

```
[gaetan@router ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f7:b1:95 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86217sec preferred_lft 86217sec
    inet6 fe80::a00:27ff:fef7:b195/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:c8:e3:89 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.2/26 brd 10.3.0.63 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fec8:e389/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e6:87:66 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.66/25 brd 10.3.0.127 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4c:c7:f8 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.194/28 brd 10.3.0.207 scope global noprefixroute enp0s10
       valid_lft forever preferred_lft forever
```

```
[gaetan@router ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=37.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=18.8 ms
```


```
[gaetan@router ~]$ ping ynov.com
PING ynov.com (92.243.16.143) 56(84) bytes of data.
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=1 ttl=53 time=17.7 ms
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=2 ttl=53 time=17.7 ms
```

# 1. Serveur DHCP

- 🌞 Mettre en place une machine qui fera office de serveur DHCP dans le réseau client1. Elle devra 

```
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.0.0 netmask 255.255.255.192 {
  range 10.3.0.4 10.3.0.62;
  option routers 10.3.0.2;
  option subnet-mask 255.255.255.192;
  option domain-name-servers 1.1.1.1;

}
```

- 🌞 **Mettre en place un client dans le réseau `client1`**
- 🌞 Depuis marcel.client1.tp3

```
[gaetan@marcel ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ec:66:5e brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3c:e7:40 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.4/26 brd 10.3.0.63 scope global dynamic noprefixroute enp0s8
       valid_lft 677sec preferred_lft 677sec
    inet6 fe80::a00:27ff:fe3c:e740/64 scope link
       valid_lft forever preferred_lft forever
```

```
[gaetan@marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=19.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=19.2 ms
```


```
[gaetan@marcel ~]$ ping ynov.com
PING ynov.com (92.243.16.143) 56(84) bytes of data.
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=1 ttl=52 time=19.7 ms
64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=2 ttl=52 time=19.5 ms
```

```
[gaetan@marcel ~]$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.3.0.2)  2.890 ms  2.763 ms  2.582 ms
```

## 2. Serveur DNS


### B. SETUP copain

- 🌞 **Mettre en place une machine qui fera office de serveur DNS**

`sudo nano /etc/named.conf`
```
zone "client1.tp3" IN {
        type master;
        file "/var/named/client1.tp3.forward";
        allow-update { none; };

};

zone "server1.tp3" IN {
        type master;
        file "/var/named/server1.tp3.forward";
        allow-update { none; };
};

zone "server2.tp3" IN {
        type master;
        file "/var/named/server2.tp3.forward";
        allow-update { none; };

};
```
`sudo nano /var/named/client1.tp3.forward`
```
$TTL 86400

@   IN  SOA     dns1.server1.tp3. root.server1.tp3. (
         2021080804  ;Serial
         3600        ;Refresh
         1800        ;Retry
         604800      ;Expire
         86400       ;Minimum TTL
)
        ; Set your Name Servers here
@         IN  NS      dns1.server1.tp3.
         ; define Name Server's IP address
@         IN  A       10.3.0.67

; Set each IP address of a hostname. Sample A records.
dhcp           IN A        10.3.0.3
```
`sudo nano /var/named/server1.tp3.forward`
```
$TTL 86400
@   IN  SOA     dns1.server1.tp3. root.server1.tp3. (
         2021080804  ;Serial
         3600        ;Refresh
         1800        ;Retry
         604800      ;Expire
         86400       ;Minimum TTL
)
        ; Set your Name Servers here
@         IN  NS      dns1.server1.tp3.
         ; define Name Server's IP address
@         IN  A       10.3.0.67

; Set each IP address of a hostname. Sample A records.
dns1           IN  A       10.3.0.67
```
`sudo nano /var/named/server2.tp3.forward`
```
$TTL 86400
@   IN  SOA     dns1.server1.tp3. root.server1.tp3. (
         2021080804  ;Serial
         3600        ;Refresh
         1800        ;Retry
         604800      ;Expire
         86400       ;Minimum TTL
)
        ; Set your Name Servers here
@         IN  NS      dns1.server1.tp3.
         ; define Name Server's IP address
@         IN  A       10.3.0.67

; Set each IP address of a hostname. Sample A records.
```

- 🌞 **Tester le DNS depuis `marcel.client1.tp3`**
```
[gaetan@marcel ~]$ dig dhcp.client1.tp3

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> dhcp.client1.tp3
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22178
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c7e0859f0a3f9e54d174ceac61548c4c6b651a6471115d9d (good)
;; QUESTION SECTION:
;dhcp.client1.tp3.              IN      A

;; ANSWER SECTION:
dhcp.client1.tp3.       86400   IN      A       10.3.0.3

;; AUTHORITY SECTION:
client1.tp3.            86400   IN      NS      dns1.server1.tp3.

;; ADDITIONAL SECTION:
dns1.server1.tp3.       86400   IN      A       10.3.0.67

;; Query time: 2 msec
;; SERVER: 10.3.0.67#53(10.3.0.67)
;; WHEN: Wed Sep 29 17:54:51 CEST 2021
;; MSG SIZE  rcvd: 132
```

- 🌞 Configurez l'utilisation du serveur DNS sur TOUS vos noeuds

```
[gaetan@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[sudo] password for gaetan:
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.0.0 netmask 255.255.255.192 {
  range 10.3.0.4 10.3.0.62;
  option routers 10.3.0.2;
  option subnet-mask 255.255.255.192;
  option domain-name-servers 10.3.0.67;

}
```


`On a pas de serveur pour l'instant`


## 3. Get deeper

### A. DNS forwarder

- 🌞 **Affiner la configuration du DNS**

```
sudo cat /etc/named.conf
[sudo] password for gaetan:
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { any; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;
```

- 🌞 Test !

```
[gaetan@marcel ~]$ dig google.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6611
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 14077ff055633578ba3ec2366154a2e9990af988d8d39a4a (good)
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       142.250.179.78

;; AUTHORITY SECTION:
google.com.             172799  IN      NS      ns2.google.com.
google.com.             172799  IN      NS      ns1.google.com.
google.com.             172799  IN      NS      ns3.google.com.
google.com.             172799  IN      NS      ns4.google.com.

;; ADDITIONAL SECTION:
ns2.google.com.         172799  IN      A       216.239.34.10
ns1.google.com.         172799  IN      A       216.239.32.10
ns3.google.com.         172799  IN      A       216.239.36.10
ns4.google.com.         172799  IN      A       216.239.38.10
ns2.google.com.         172799  IN      AAAA    2001:4860:4802:34::a
ns1.google.com.         172799  IN      AAAA    2001:4860:4802:32::a
ns3.google.com.         172799  IN      AAAA    2001:4860:4802:36::a
ns4.google.com.         172799  IN      AAAA    2001:4860:4802:38::a

;; Query time: 532 msec
;; SERVER: 10.3.0.67#53(10.3.0.67)
;; WHEN: Wed Sep 29 19:31:21 CEST 2021
;; MSG SIZE  rcvd: 331
```



### B. On revient sur la conf du DHCP

 🖥️ VM johnny.client1.tp3

- 🌞 **Affiner la configuration du DHCP**
```
[gaetan@dhcp ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
nameserveur 1.1.1.1
[gaetan@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[sudo] password for gaetan:
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.0.0 netmask 255.255.255.192 {
  range 10.3.0.4 10.3.0.62;
  option routers 10.3.0.2;
  option subnet-mask 255.255.255.192;
  option domain-name-servers 10.3.0.67;

}
```


```
[gaetan@johnny ~]$ ip a
[...]
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4f:cf:e1 brd ff:ff:ff:ff:ff:ff
    inet 10.3.0.4/26 brd 10.3.0.63 scope global dynamic noprefixroute enp0s8
       valid_lft 857sec preferred_lft 857sec
    inet6 fe80::a00:27ff:fe4f:cfe1/64 scope link
       valid_lft forever preferred_lft forever
```
```
[gaetan@johnny ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search client1.tp3
nameserver 10.3.0.67
```

# III. Services métier

## 1. Serveur Web


🖥️ VM web1.server2.tp3

- 🌞 **Setup d'une nouvelle machine, qui sera un serveur Web, une belle appli pour nos clients**


```
[gaetan@web1 ~]$ sudo systemctl status web
● web.service - Very simple web service
   Loaded: loaded (/etc/systemd/system/web.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-09-29 22:27:37 CEST; 11min ago
```
`[gaetan@web1 ~]$ sudo dnf install nginx`

`[gaetan@web1 ~]$ systemctl start nginx`

🌞 **Test test test et re-test**

- testez que votre serveur web est accessible depuis `marcel.client1.tp3`
- utilisez la commande `curl` pour effectuer des requêtes HTTP
```
[gaetan@marcel ~]$ curl 10.3.0.195:8888
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="test.txt">test.txt</a></li>
</ul>
<hr>
</body>
</html>
```

### B. Le setup wola

🖥️ VM nfs1.server2.tp3

- 🌞 **Setup d'une nouvelle machine, qui sera un serveur NFS**

`[gaetan@nfs1 ~]$ dnf -y install nfs-utils`
`sudo nano /etc/idmapd.conf`
`Domain = server2.tp3`
`[gaetan@nfs1 ~]$ mkdir /srv/nfs_share/`
```
[gaetan@nfs1 ~]$ sudo systemctl enable --now rpcbind nfs-server
[sudo] password for gaetan:
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```
```
[gaetan@nfs1 ~]$ sudo !!
sudo firewall-cmd --add-service=nfs
success
```
```
[gaetan@nfs1 ~]$ sudo firewall-cmd --add-service={nfs3,mountd,rpc-bind}
success
```
```
[gaetan@nfs1 ~]$ sudo firewall-cmd --runtime-to-permanent
success
```

- 🌞 **Configuration du client NFS**

- effectuez de la configuration sur `web1.server2.tp3` pour qu'il accède au partage NFS

`[gaetan@web1 ~]$ sudo mount -t nfs nfs1.server2.tp3:/home/srv/nfs_share /srv/nfs/`


- le partage NFS devra être monté dans `/srv/nfs/`
```
[gaetan@web1 ~]$ df -hT /srv/nfs/
Filesystem                           Type  Size  Used Avail Use% Mounted on
nfs1.server2.tp3:/home/srv/nfs_share nfs4  6.2G  2.1G  4.2G  34% /srv/nfs
```

- 🌞 **TEEEEST**

- tester que vous pouvez lire et écrire dans le dossier `/srv/nfs` depuis `web1.server2.tp3`
`[gaetan@web1 nfs]$ sudo nano kahoot.txt`


- vous devriez voir les modifications du côté de  `nfs1.server2.tp3` dans le dossier `/srv/nfs_share/`
```
[gaetan@web1 nfs]$ cat kahoot.txt
je suis dernier sur kahoot!
```

# IV. Un peu de théorie : TCP et UDP


- 🌞 **Déterminer, pour chacun de ces protocoles, s'ils sont encapsulés dans du TCP ou de l'UDP :**

    `- SSH / TCP`
    `- HTTP / TCP`
    `- DNS / UDP par default et aussi TCP`
    `- NFS / TCP par default et aussi UDP`

📁 **Captures réseau `tp3_ssh.pcap`, `tp3_http.pcap`, `tp3_dns.pcap` et `tp3_nfs.pcap`**

- 🌞 **Expliquez-moi pourquoi je ne pose pas la question pour DHCP.**



- 🌞 **Capturez et mettez en évidence un *3-way handshake***

    `c'est fait :)`

📁 **Capture réseau `tp3_3way.pcap`**

- 🌞 **Bah j'veux un schéma.**


    `ohhhh nonnn je suis nul en schéma...`

https://drive.google.com/file/d/1Ek9q6Q8CYoSUqrAWCcE116qMLREktQ_P/view?usp=sharing

- 🌞🗃️ tableau des réseaux 🗃️


| Nom du réseau | Adresse du réseau | Masque        | Nombre de clients possibles | Adresse passerelle | [Adresse broadcast](../../cours/lexique/README.md#adresse-de-diffusion-ou-broadcast-address) |
|---------------|-------------------|---------------|-----------------------------|--------------------|----------------------------------------------------------------------------------------------|
| `client1`     | `10.3.0.0`        | `255.255.255.192` | `62`                           | `10.3.0.2`         | `10.3.0.63`                                                                                   |
| `server1`     | `10.3.0.64`        | `255.255.255.128` | `126`                           | `10.3.0.66`         | `10.3.0.190`                                                                                   |
| `server2`     | `10.3.0.192`        | `255.255.255.240` | `14`                           | `10.3.0.194`         | `10.3.0.207`                                                                                   |



- 🌞 🗃️ tableau d'adressage 🗃️ 

| Nom machine  | Adresse IP `client1` | Adresse IP `server1` | Adresse IP `server2` | Adresse de passerelle |
|--------------|----------------------|----------------------|----------------------|-----------------------|
| `router.tp3` | `10.3.0.2/26`         | `10.3.0.66/25`         | `10.3.0.194/28`         | Carte NAT             |
| `dhcp.client.tp3`         | `10.3.0.3`                 | `no`                  | `no`                  | `10.3.0.2/26`          |
| `marcel.client.tp3` | `dhcp`         | `no`         | `no`         | `10.3.0.2/26`             |    
| `dns1.serveur1.tp3`          | `no`                  | `10.3.0.67/25`                  | `no`                  | `10.3.0.66/25`          |
| `johnny.client.tp3` | `dhcp`         | `no`         | `no`         | `10.3.0.2/26`             |
| `web1.serveur2.tp3`          | `no`                  | `no`                  | `10.3.0.195/28`                  | `10.3.0.194/28`          |
| `nfs1.serveur2.tp3` | `no`         | `no`         | `10.3.0.196/28`         | `10.3.0.194/28`             |




- 🌞 **Et j'veux des fichiers aussi, tous les fichiers de conf du DNS**

    `C'est fait ;)`
    
- 📁 Fichiers de zone
- 📁 Fichier de conf principal DNS `named.conf`
