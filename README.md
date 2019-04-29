# I. Manipulation de switches et de VLAN

Configuration des interfaces de l'infrastructure: <img src="Captures/Interfaces VLAN.png" />

Configuration du switch `SW1`:
```
SW1#show vlan br
VLAN    Name             Status    Ports
----    --------------   --------- -------------------------------
1       default          active    Et0/1, Et0/2, Et0/3, Et1/2
                                   Et2/3, Et3/0, Et3/1, Et3/2
                                   Et1/3, Et2/0, Et2/1, Et2/2
                                   Et3/3
10      lien-client1     active    Et0/0
20      lien-client2     active    Et1/0

```

Configuration du switch `SW2`:
```
SW2#show vlan br
VLAN    Name             Status    Ports
----    ---------------- --------- -------------------------------
1       default          active    Et0/1, Et0/2, Et0/3, Et1/0
                                   Et1/2, Et1/3, Et2/0, Et2/1
                                   Et2/2, Et2/3, Et3/0, Et3/1
                                   Et3/2, Et3/3
10      lien-client      active    Et0/0
```

On fait un `traceroute` de `client1` vers `client3` afin de voir par où passe notre paquet:
```
[dams@client1 ~]$sudo traceroute -I client3
traceroute to client3 (10.1.1.3), 30 hops max, 60 byte packets
1  client3 (10.1.1.3)  97.725 ms  103.865 ms  104.141 ms
```
Chaque temps affiché correspond à un appareil. Notre paquet a donc atteint `SW1` en 97.725 ms, puis `SW2` en 103.865 ms, pour enfin arriver à `client3` en 104.141 ms au total!


`client2`, étant dans un VLAN différent, n'arrive pas à joindre les autres machines...

```
[dams@client3 ~]$ ping -c 4 client1
PING client1 (10.1.1.1) 56(84) bytes of data.
From client2.lab1.tp3 (10.1.1.2) icmp_seq=1 Destination Host Unreachable
From client2.lab1.tp3 (10.1.1.2) icmp_seq=2 Destination Host Unreachable
From client2.lab1.tp3 (10.1.1.2) icmp_seq=3 Destination Host Unreachable
From client2.lab1.tp3 (10.1.1.2) icmp_seq=4 Destination Host Unreachable

--- client1 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3000ms
pipe 4
```

# II. Manipulation simple de routeurs

Configuration des interfaces et IP de l'infrastructure: <img src= "Captures/Interfaces Routeurs.png"  height= 300/>

`client1` n'ayant pas de passerelle, il ne pourra pas accéder au réseau... <img src="https://i.redd.it/255iuxdit0g21.jpg"/>

```
R1#ping 10.2.12.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.12.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 64/68/76 ms
```
- [x] Les routeurs peuvent communiquer entre eux.

```
[dams@server1 ~]$ ping -c 4 router2
PING router2 (10.2.2.254) 56(84) bytes of data.
64 bytes from router2 (10.2.2.254): icmp_seq=1 ttl=255 time=10.4 ms
64 bytes from router2 (10.2.2.254): icmp_seq=2 ttl=255 time=9.94 ms
64 bytes from router2 (10.2.2.254): icmp_seq=3 ttl=255 time=11.0 ms
64 bytes from router2 (10.2.2.254): icmp_seq=4 ttl=255 time=11.2 ms

--- router2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 9.945/10.672/11.274/0.538 ms
```

```
[dams@client2 ~]$ ping -c 4 router1
PING router1 (10.2.1.254) 56(84) bytes of data.
64 bytes from router1 (10.2.1.254): icmp_seq=1 ttl=255 time=20.5 ms
64 bytes from router1 (10.2.1.254): icmp_seq=2 ttl=255 time=2.09 ms
64 bytes from router1 (10.2.1.254): icmp_seq=3 ttl=255 time=7.35 ms
64 bytes from router1 (10.2.1.254): icmp_seq=4 ttl=255 time=4.07 ms

--- router1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 2.092/8.523/20.576/7.208 ms
```
- [x] Passerelles joignables!

__Afin que les deux clients puissent avoir la même passerelle, on pourrait les relier à un switch lui-même lié au routeur.__

# IV. Lab Final

Il ressemble à ça: <img src="Captures/Infra finale.png" />

### Les différentes adresses IP:

Hosts | `10.1.1.0/24` | `10.2.2.0/24` | `10.3.100.0/30` | `10.3.100.4/30` | `10.3.100.8/30` | `10.3.100.12/30`
--- | --- | --- | --- | --- | --- | ---
`server1.lab4.tp3` | `10.1.1.1/24` | X | X | X | X | X
`server2.lab4.tp3` | `10.1.1.2/24` | X | X | X | X | X
`client1.lab4.tp3` | X | `10.2.2.1/24` | X | X | X | X
`client2.lab4.tp3` | X | `10.2.2.2/24` | X | X | X | X
`router1.lab4.tp3` | X | X | X | `10.3.100.6/30` | `10.3.100.9/30` | X
`router2.lab4.tp3` | X | X | `10.3.100.2/30` | `10.3.100.5/30` | X | X
`router3.lab4.tp3` | X | X | X | X | `10.3.100.10/30` | `10.3.100.13/30`
`router4.lab4.tp3` | `10.1.1.254/24` | `10.2.2.254/24` | `10.3.100.1/30` | X | X | `10.3.100.14/30`
```
R1#show ip interface br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES NVRAM  administratively down down
FastEthernet1/0            192.168.122.26  YES DHCP   up                    up
FastEthernet2/0            unassigned      YES NVRAM  administratively down down
FastEthernet3/0            unassigned      YES NVRAM  administratively down down
NVI0                       unassigned      NO  unset  up                    up

R1#ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 24/32/40 ms
```
- [x] NAT et DHCP paramétré sur `R1`.

```
R1#show ip protocols
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1
  It is an autonomous system boundary router
  Redistributing External Routes from,
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.3.100.4 0.0.0.3 area 0
    10.3.100.8 0.0.0.3 area 0
 Reference bandwidth unit is 100 mbps
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)

R3#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           1   FULL/DR         00:00:32    10.3.100.14     FastEthernet3/0
1.1.1.1           1   FULL/DR         00:00:37    10.3.100.9      FastEthernet1/0

R4#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           1   FULL/BDR        00:00:30    10.3.100.13     FastEthernet3/0
2.2.2.2           1   FULL/BDR        00:00:39    10.3.100.2      FastEthernet2/0

```
- [x] OSPF paramétré sur les différents routeurs (ce sont les mêmes lignes sur les autres, avec juste les adresses et l'ID qui changent).

```

IOU1#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/1, Et0/2, Et0/3, Et1/3
                                                Et2/0, Et2/1, Et2/2, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
10   servers                          active    Et0/0, Et1/0
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup


IOU2#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/1, Et0/2, Et0/3, Et1/1
                                                Et1/2, Et1/3, Et2/0, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
20   clients                          active    Et0/0, Et1/0
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

- [x] VLANs paramétrés sur les deux switches.

```
R4#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES manual up                    up
FastEthernet0/0.10         10.1.1.254      YES manual up                    up
FastEthernet0/0.20         10.2.2.254      YES manual up                    up
FastEthernet1/0            unassigned      YES manual up                    up
FastEthernet2/0            10.3.100.1      YES manual up                    up
FastEthernet3/0            10.3.100.14     YES manual up                    up
```
Ici, on a paramétré **deux zones** pour que les deux VLANs puissent communiquer.
```
[dams@client2 ~]$ ping -c 4 server1
PING server1 (10.2.2.1) 56(84) bytes of data.
64 bytes from server1 (10.2.1.1): icmp_seq=1 ttl=64 time=15.3 ms
64 bytes from server1 (10.2.1.1): icmp_seq=2 ttl=64 time=9.33 ms
64 bytes from server1 (10.2.1.1): icmp_seq=3 ttl=64 time=14.6 ms
64 bytes from server1 (10.2.1.1): icmp_seq=4 ttl=64 time=13.5 ms

--- server1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 9.337/13.212/15.328/2.322 ms
```
- [x] Le Router On A Stick (ROAS) est paramétré, les clients ***peuvent joindre les serveurs et inversement!***

Plus qu'à paramétrer les routes sur les différentes machines et...
```
[dams@client2 ~]$ ping -c 4 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8 (8.8.8.8): icmp_seq=1 ttl=116 time=78.0 ms
64 bytes from 8.8.8.8 (8.8.8.8): icmp_seq=2 ttl=116 time=79.6 ms
64 bytes from 8.8.8.8 (8.8.8.8): icmp_seq=3 ttl=116 time=72.5 ms
64 bytes from 8.8.8.8 (8.8.8.8): icmp_seq=4 ttl=116 time=97.3 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 72.579/81.885/97.315/9.290 ms
```

**Bingo!** En revanche, ce n'est pas très sécurisé, on a:
- Des SPoF (Single Point of Failure) en `R4`, `R1` et `IOU1`.
- Aucune redondance des liaisons; une ne tient plus, et l'infrastructure est fichue!
- Aucune surveillance du trafic réseau.

[A suivre...](https://github.com/DamienOrl/Tp-reseau-4-menu-1) <img src="http://www.sclance.com/pngs/to-be-continued-meme-png/./to_be_continued_meme_png_1389102.png" width="350" alt="*Insérer une référence à Jojo*" />
