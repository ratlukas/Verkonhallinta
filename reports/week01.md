# Viikko 1 - Verkon dokumentointi

## 1. Johdanto
Kyseinen ympäristö on harjoituskäyttöön tarkoitettu verkkorakenne. Verkon tarkoituksena on mahdollistaa eri käyttäjien, palvelinten ja hallintalaitteiden välinen liikenne sekä verkon hallinta ja tarkkailu.

## 2. Verkkokaavio
![Verkkokaavio](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/topology.png "Verkkokaavio")

## 3. Laiteluettelo

| Laite | Tarkoitus |
| --- | ----------- |
| r1 | Reititin User LAN:ille |
| r2 | Reititin Server ja Management LAN:ei lle |
| r3 | Reititin Branch Clientille |
| client1 | Clientti |
| attacker | Turvallisuus |
| web1 | Web palvelin |
| db1 | Database palvelin |
| branch-client | Clientti |
| ansible | Verkonhallinta ja tarkkailu |
| prometheus |  Verkonhallinta ja tarkkailu |
| grafana | Verkonhallinta ja tarkkailu |
| zabbix | Verkonhallinta ja tarkkailu |
| mgmt-bp | Management LAN kytkin |
| srv-bp | Server LAN kytkin |

## 4. IP-suunnitelma

| Verkko | Tarkoitus | Yhdyskäytävä |
| ---------- | ----------- | ----------- |
| 10.10.10.0/24 | User LAN | 10.10.10.1 |
| 10.10.20.0/24 | Server LAN | 10.10.20.1 |
| 10.10.30.0/24 | Branch Office | 10.10.30.1 |
| 10.10.99.0/24 | Management LAN | 10.10.99.1 |
| 10.255.12.0/30 | r1 ja r2 välinen yhteys | 10.255.12.1 |
| 10.255.23.0/30 | r2 ja r3 välinen yhteys | 10.255.23.1 |

User LAN:in yhdyskäytävänä toimii r1. Server ja Management LAN:ien yhdyskäytävänä toimii r2. Branch Office yhdyskäytävänä toimii r3.

### Verkon 10.10.10.0/24 laitteet
- r1 10.10.10.1
- client1 10.10.10.101
- attacker

### Verkon 10.10.20.0/24 laitteet
- r2 10.10.20.1
- srv-bp
- web1
- db1

### Verkon 10.10.30.0/24 laitteet
- r3 10.10.30.1
- branch-cleint 10.10.30.101

### Verkon 10.10.99.0/24 laitteet
- r2 10.10.99.1
- mgmt-bp
- ansible
- grafana
- prometheus
- zabbix

### Verkon 10.255.12.0/30 laitteet
- r1 10.255.12.1
- r2 10.255.12.2

### Verkon 10.255.23.0/30 laitteet
- r2 10.255.23.1
- r3 10.255.23.2

## 5. Reitityksen analyysi
### ip addr ja ip route
root@client1:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
91: eth0@if92: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:14:14:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.20.20.4/24 brd 172.20.20.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe14:1404/64 scope link
       valid_lft forever preferred_lft forever
122: eth1@if121: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP group default
    link/ether aa:c1:ab:56:f4:78 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    altname clab-o-05031180f95d8850
    inet 10.10.10.101/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:fe56:f478/64 scope link
       valid_lft forever preferred_lft forever
       
root@client1:/# ip route
default via 10.10.10.1 dev eth1
10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.101
172.20.20.0/24 dev eth0 proto kernel scope link src 172.20.20.4

### Pingaukset
root@client1:/# ping -c 4 10.10.20.101
PING 10.10.20.101 (10.10.20.101) 56(84) bytes of data.
64 bytes from 10.10.20.101: icmp_seq=1 ttl=62 time=0.870 ms
64 bytes from 10.10.20.101: icmp_seq=2 ttl=62 time=0.137 ms
64 bytes from 10.10.20.101: icmp_seq=3 ttl=62 time=0.177 ms
64 bytes from 10.10.20.101: icmp_seq=4 ttl=62 time=0.177 ms

--- 10.10.20.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3056ms
rtt min/avg/max/mdev = 0.137/0.340/0.870/0.306 ms
root@client1:/# ping -c 4 10.10.30.101
PING 10.10.30.101 (10.10.30.101) 56(84) bytes of data.
64 bytes from 10.10.30.101: icmp_seq=1 ttl=61 time=0.452 ms
64 bytes from 10.10.30.101: icmp_seq=2 ttl=61 time=0.146 ms
64 bytes from 10.10.30.101: icmp_seq=3 ttl=61 time=0.214 ms
64 bytes from 10.10.30.101: icmp_seq=4 ttl=61 time=0.103 ms

--- 10.10.30.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3080ms
rtt min/avg/max/mdev = 0.103/0.228/0.452/0.134 ms

Yhteys löytyi kaikkiin verkkoihin ongelmitta, sillä kaikki pingaukset onnistuivat.

### Traceroute
root@client1:/# traceroute 10.10.30.101
traceroute to 10.10.30.101 (10.10.30.101), 30 hops max, 60 byte packets
 1  10.10.10.1 (10.10.10.1)  2.426 ms  2.049 ms  1.986 ms
 2  10.255.12.2 (10.255.12.2)  1.963 ms  1.740 ms  1.590 ms
 3  10.255.23.2 (10.255.23.2)  1.519 ms  1.288 ms  1.190 ms
 4  10.10.30.101 (10.10.30.101)  1.121 ms  1.037 ms  0.938 ms

Liikenne kulki r1, r2 ja r3 reitittimien kautta ennen kuin se pääsi branch clientille. R1 ja r3 eivät ole suoraan ydistettyjä toisiinsa, joten liikenteen täytyy kulkea r2 reitittimen kautta, kun näihin kahteen reitittimeen yhdistetyt verkot viestivät. R2 reitittimeen on yhdistetty kaksi LAN:ia, joten näiden viestitellessä toisilleen täytyy liikenteen kulkea vain r2 reitittimen kautta.

## 6. Yhteenveto

Eniten aikaa dokumentaatiossa meni verkkokaavion tekemiseen, sillä sitä varten piti selvitellä eniten asioita ja sitten järjestellä niistä selkeä verkkokaavio. Aikaa kului myös aika paljon IP-osoitteiden ja eri verkkojen tarkoitusten selvittelyyn.

Dokumentaatio auttaa palvelusta vastaavaa it-asiantuntiaa, siten että esimerkiksi ongelmaa selvittäessä verkon yleiset tiedot löytyvät jo valmiiksi, eikä niitä tarvitse lähteä selvittelemään. Dokumentaatio helpottaa myös verkon myöhempää ylläpitoa ja laajentamista.
