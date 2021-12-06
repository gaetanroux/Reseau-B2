# TP4 : Vers un réseau d'entreprise

# I. Dumb switch

## 1. Topologie 1

## 2. Adressage topologie 1

| Node  | IP            |
|-------|---------------|
| `pc1` | `10.1.1.1/24` |
| `pc2` | `10.1.1.2/24` |

## 3. Setup topologie 1

🌞 **Commençons simple**

- définissez les IPs statiques sur les deux VPCS
```
PC1> ip 10.1.1.1/24
PC2> ip 10.1.1.2/24
```

- `ping` un VPCS depuis l'autre
```
PC2> ping 10.1.1.1

84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=7.088 ms
84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=6.528 ms
```

# II. VLAN

## 2. Adressage topologie 2

| Node  | IP            | VLAN |
|-------|---------------|------|
| `pc1` | `10.1.1.1/24` | 10   |
| `pc2` | `10.1.1.2/24` | 10   |
| `pc3` | `10.1.1.3/24` | 20   |

### 3. Setup topologie 2

🌞 **Adressage**

- définissez les IPs statiques sur tous les VPCS
`PC3> ip 10.1.1.3/24`

- vérifiez avec des `ping` que tout le monde se ping
```
PC3> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=6.931 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=6.813 ms
PC3> ping 10.1.1.1

84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=7.145 ms
84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=6.940 ms

```
🌞 **Configuration des VLANs**

- référez-vous [à la section VLAN du mémo Cisco](../../cours/memo/memo_cisco.md#8-vlan)
- déclaration des VLANs sur le switch `sw1`

```
Switch#show vlan

VLAN Name                             Status    Ports
[...]
10   10                               active
20   20                               active
```


- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#2-adressage-topologie-2))
```
Switch(config)#interface gigabitEthernet0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#exit
```
```

Switch#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- 
[...]
10   10                               active    Gi0/1, Gi0/2
20   20                               active    Gi0/3

```

🌞 **Vérif**

- `pc1` et `pc2` doivent toujours pouvoir se ping
```
PC1> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=3.719 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=3.434 ms
```
- `pc3` ne ping plus personne
```
PC3> ping 10.1.1.1

host (10.1.1.1) not reachable

PC3> ping 10.1.1.2

host (10.1.1.2) not reachable
```

# III. Routing

## 2. Adressage topologie 3

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse       | VLAN associé |
|-----------|---------------|--------------|
| `clients` | `10.1.1.0/24` | 11           |
| `servers` | `10.2.2.0/24` | 12           |
| `routers` | `10.3.3.0/24` | 13           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`       | `admins`        | `servers`       |
|--------------------|-----------------|-----------------|-----------------|
| `pc1.clients.tp4`  | `10.1.1.1/24`   | x               | x               |
| `pc2.clients.tp4`  | `10.1.1.2/24`   | x               | x               |
| `adm1.admins.tp4`  | x               | `10.2.2.1/24`   | x               |
| `web1.servers.tp4` | x               | x               | `10.3.3.1/24`   |
| `r1`               | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.3.254/24` |

## 3. Setup topologie 3

🖥️ VM `web1.servers.tp4`, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞 **Adressage**
- définissez les IPs statiques sur toutes les machines **sauf le *routeur***
```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.1/24
```
```
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.2/24
```
```
PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.2.2.1/24
```

🌞 **Configuration des VLANs**

- déclaration des VLANs sur le switch `sw1`
```
Switch(config)#vlan 11
Switch(config-vlan)#name clients
Switch(config-vlan)#exit
Switch(config)#vlan 12
Switch(config-vlan)#name admins
Switch(config-vlan)#exit
Switch(config)#vlan 13
Switch(config-vlan)#name servers
Switch(config-vlan)#exit
```
- ajout des ports du switches dans le bon VLAN
```
Switch(config)#interface gigabitEthernet0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 11
Switch(config-if)#exit
```
```
Switch(config)#interface gigabitEthernet0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 11
Switch(config-if)#exit
```
```
Switch(config)#interface gigabitEthernet0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 12
Switch(config-if)#exit
```
```
Switch(config)#interface gigabitEthernet1/0
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 13
Switch(config-if)#exit
```
```
Switch#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
[...]
11   clients                          active    Gi0/1, Gi0/2
12   admins                           active    Gi0/3
13   servers                          active    Gi1/0
```

- il faudra ajouter le port qui pointe vers le *routeur* comme un *trunk*
```
Switch(config)#interface gigabitEthernet0/0
Switch(config-if)#switchport trunk encapsulation dot1q
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk allowed vlan add 11,12,13
Switch(config-if)#exit
Switch(config)#exit
Switch#show interface trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       1-4094

Port        Vlans allowed and active in management domain
Gi0/0       1,10-13,20

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       1,10-13,20
```

---

➜ **Pour le *routeur***
🌞 **Config du *routeur***

- attribuez ses IPs au *routeur*
  - 3 sous-interfaces, chacune avec son IP et un VLAN associé
```
R1(config)#interface fastEthernet 0/0.11
R1(config-subif)#encapsulation dot1Q 11
R1(config-subif)#ip addr 10.1.1.254 255.255.255.0
R1(config-subif)#exit
```
```
R1(config)#interface fastEthernet 0/0.12
R1(config-subif)#encapsulation dot1Q 12
R1(config-subif)#ip addr 10.2.2.254 255.255.255.0
R1(config-subif)#exit
```
```
R1(config)#interface fastEthernet 0/0.13
R1(config-subif)#encapsulation dot1Q 13
R1(config-subif)#ip addr 10.3.3.254 255.255.255.0
R1(config-subif)#exit
```
🌞 **Vérif**
- tout le monde doit pouvoir ping le routeur sur l'IP qui est dans son réseau
```
PC1> ping 10.1.1.254


10.1.1.254 icmp_seq=1 timeout
84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=11.924 ms
84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=7.493 ms
84 bytes from 10.1.1.254 icmp_seq=4 ttl=255 time=8.069 ms
```
```
PC2> ping 10.1.1.254

10.1.1.254 icmp_seq=1 timeout
84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=8.262 ms
84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=16.375 ms
84 bytes from 10.1.1.254 icmp_seq=4 ttl=255 time=11.776 ms
```
```
PC3> ping 10.2.2.254

84 bytes from 10.2.2.254 icmp_seq=1 ttl=255 time=10.382 ms
84 bytes from 10.2.2.254 icmp_seq=2 ttl=255 time=7.543 ms
84 bytes from 10.2.2.254 icmp_seq=3 ttl=255 time=13.843 ms
84 bytes from 10.2.2.254 icmp_seq=4 ttl=255 time=7.576 ms
```
```
web1> ping 10.3.3.254

84 bytes from 10.3.3.254 icmp_seq=3 ttl=255 time=7.493 ms
84 bytes from 10.3.3.254 icmp_seq=4 ttl=255 time=8.069 ms
84 bytes from 10.3.3.254 icmp_seq=4 ttl=255 time=7.576 ms
```





- en ajoutant une route vers les réseaux, ils peuvent se ping entre eux
```
R1(config)#ip route 10.1.1.0 255.255.255.0 10.1.0.254
R1(config)#ip route 10.2.2.0 255.255.255.0 10.2.0.254
R1(config)#ip route 10.3.3.0 255.255.255.0 10.3.0.254
R1(config)#exit
```
```
R1#show ip route
[♦....]

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 3 subnets
C       10.3.3.0 is directly connected, FastEthernet0/0.13
C       10.2.2.0 is directly connected, FastEthernet0/0.12
C       10.1.1.0 is directly connected, FastEthernet0/0.11
```

- ajoutez une route par défaut sur les VPCS
```
PC1> ip 10.1.1.1/24 10.1.1.254
Checking for duplicate address...
PC1 : 10.1.1.1 255.255.255.0 gateway 10.1.1.254
```
```
PC2> ip 10.1.1.2/24 10.1.1.254
Checking for duplicate address...
PC2 : 10.1.1.2 255.255.255.0 gateway 10.1.1.254
```
```
PC3> ip 10.2.2.1/24 10.2.2.254
Checking for duplicate address...
PC3 : 10.2.2.1 255.255.255.0 gateway 10.2.2.254
```

  - ajoutez une route par défaut sur la machine virtuelle
`[gaetan@web1 ~]$ ip route add default via 10.3.3.254 dev enp0s3`
  - testez des `ping` entre les réseaux
```
[gaetan@web1 ~]$ ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=63 time=33.8 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=63 time=22.2 ms
```
```
PC1> ping 10.2.2.1

84 bytes from 10.2.2.1 icmp_seq=1 ttl=63 time=37.011 ms
84 bytes from 10.2.2.1 icmp_seq=2 ttl=63 time=21.672 ms
```
```
PC3> ping 10.3.3.1

84 bytes from 10.3.3.1 icmp_seq=1 ttl=63 time=25.298 ms
84 bytes from 10.3.3.1 icmp_seq=2 ttl=63 time=25.638 ms
```

# IV. NAT

On va ajouter une fonctionnalité au routeur : le NAT.

On va le connecter à internet (simulation du fait d'avoir une IP publique) et il va faire du NAT pour permettre à toutes les machines du réseau d'avoir un accès internet.

![Yellow cable](./pics/yellow-cable.png)

## 1. Topologie 4

![Topologie 3](./pics/topo4.png)

## 2. Adressage topologie 4

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse       | VLAN associé |
|-----------|---------------|--------------|
| `clients` | `10.1.1.0/24` | 11           |
| `servers` | `10.2.2.0/24` | 12           |
| `routers` | `10.3.3.0/24` | 13           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`       | `admins`        | `servers`       |
|--------------------|-----------------|-----------------|-----------------|
| `pc1.clients.tp4`  | `10.1.1.1/24`   | x               | x               |
| `pc2.clients.tp4`  | `10.1.1.2/24`   | x               | x               |
| `adm1.admins.tp4`  | x               | `10.2.2.1/24`   | x               |
| `web1.servers.tp4` | x               | x               | `10.3.3.1/24`   |
| `r1`               | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.3.254/24` |

## 3. Setup topologie 4

🌞 **Ajoutez le noeud Cloud à la topo**

- côté routeur, il faudra récupérer un IP en DHCP
```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
[...]
FastEthernet1/0            10.0.3.16       YES DHCP   up                    up
```
- vous devriez pouvoir `ping 1.1.1.1`
```
R1#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/25/40
``` 

🌞 **Configurez le NAT**

```
R1(config)#interface fastEthernet 0/0
R1(config-if)#ip nat inside

*Mar  1 02:14:28.103: %LINEPROTO-5-UPDOWN: Line protocol on Interface NVI0, changed state to up
R1(config-if)#exit
```
```
R1(config)#interface fastEthernet 1/0
R1(config-if)#ip nat outside
R1(config-if)#exit
R1(config)#exit
```

🌞 **Test**

- ajoutez une route par défaut (si c'est pas déjà fait)

`C'est déjà fait !`

- configurez l'utilisation d'un DNS
- sur les VPCS
```
PC1> ip dns 1.1.1.1
PC2> ip dns 1.1.1.1
PC3> ip dns 1.1.1.1
```

- sur la machine Linux
```
[gaetan@web1 ~]$ sudo nano /etc/resolv.conf
DNS=1.1.1.1
```
```
[gaetan@web1 ~]$ ping google.com
PING google.com (216.58.213.78) 56(84) bytes of data.
64 bytes from 216.58.213.78 (216.58.213.78): icmp_seq=1 ttl=112 time=39.8 ms
64 bytes from 216.58.213.78 (216.58.213.78): icmp_seq=2 ttl=112 time=37.0 ms
```

- vérifiez un `ping` vers un nom de domaine
```
[gaetan@web1 ~]$ ping google.com
PING google.com (216.58.213.78) 56(84) bytes of data.
64 bytes from 216.58.213.78 (216.58.213.78): icmp_seq=1 ttl=112 time=39.8 ms
64 bytes from 216.58.213.78 (216.58.213.78): icmp_seq=2 ttl=112 time=37.0 ms
```
# V. Add a building

On a acheté un nouveau bâtiment, faut tirer et configurer un nouveau switch jusque là-bas.

On va en profiter pour setup un serveur DHCP pour les clients qui s'y trouvent.

## 1. Topologie 5

![Topo 5](./pics/topo5.png)

## 2. Adressage topologie 5

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse       | VLAN associé |
|-----------|---------------|--------------|
| `clients` | `10.1.1.0/24` | 11           |
| `servers` | `10.2.2.0/24` | 12           |
| `routers` | `10.3.3.0/24` | 13           |

L'adresse des machines au sein de ces réseaux :

| Node                | `clients`       | `admins`        | `servers`       |
|---------------------|-----------------|-----------------|-----------------|
| `pc1.clients.tp4`   | `10.1.1.1/24`   | x               | x               |
| `pc2.clients.tp4`   | `10.1.1.2/24`   | x               | x               |
| `pc3.clients.tp4`   | DHCP            | x               | x               |
| `pc4.clients.tp4`   | DHCP            | x               | x               |
| `pc5.clients.tp4`   | DHCP            | x               | x               |
| `dhcp1.clients.tp4` | `10.1.1.253/24` | x               | x               |
| `adm1.admins.tp4`   | x               | `10.2.2.1/24`   | x               |
| `web1.servers.tp4`  | x               | x               | `10.3.3.1/24`   |
| `r1`                | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.3.254/24` |

## 3. Setup topologie 5

Vous pouvez partir de la topologie 4. 

🌞  **Vous devez me rendre le `show running-config` de tous les équipements**

- de tous les équipements réseau
  - le routeur
  - les 3 switches
  
C'est par ici --> https://gitlab.com/gaetan33460/b2-reseau-2021/-/tree/main/4

🖥️ **VM `dhcp1.client1.tp4`**, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞  **Mettre en place un serveur DHCP dans le nouveau bâtiment**
```
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.1.1.0 netmask 255.255.255.0 {
  range 10.1.1.1 10.1.1.252;
  option routers 10.1.1.254;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 1.1.1.1;

}
```




🌞  **Vérification**

- un client récupère une IP en DHCP
- il peut ping le serveur Web
```
PC3> ping 10.3.3.1

10.3.3.1 icmp_seq=1 timeout
84 bytes from 10.3.3.1 icmp_seq=2 ttl=63 time=30.005 ms
84 bytes from 10.3.3.1 icmp_seq=3 ttl=63 time=40.602 ms
84 bytes from 10.3.3.1 icmp_seq=4 ttl=63 time=34.527 ms
```

- il peut ping `8.8.8.8`
```
PC3> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=112 time=37.822 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=112 time=49.848 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=114 time=42.763 ms
```

- il peut ping `google.com`
```
PC3> ping google.com
google.com resolved to 216.58.209.238

84 bytes from 216.58.209.238 icmp_seq=1 ttl=115 time=51.197 ms
84 bytes from 216.58.209.238 icmp_seq=2 ttl=115 time=45.982 ms
84 bytes from 216.58.209.238 icmp_seq=3 ttl=115 time=36.596 ms
```



