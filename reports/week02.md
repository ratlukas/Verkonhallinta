# Viikko 2 – SNMP ja verkon perustason valvonta

## 1. Johdanto

SNMP on laitteiden ja järjestelmien etähallintaan sekä -valvontaan käytettävä protokolla. SNMP on lyhenne sanoista Simple Network Management Protocol. SNMP on tällä hetkellä yleisein verkkohallintaprotokolla. SNMP:n avulla voidaan toteuttaa etähallintaa tai -valvontaa esimerkiksi reitimmissä ja kytkimissä.

SNMP:ssä on kaksi pääkomponenttia. Nämä ovat SNMP Agent ja SNMP Manger. Agentti toimii valvottavassa laitteessa kuten reitittimessä, kytkimessä, Linux-palvelimessa yms. Se vastaa valvontapalvelimen kyselyihin. Manager taas kerää tietoa agentilta. Manager voi olla esimerkiksi Zabbix.

## 2. Asennus

SNMP-agentin asennus tapahtui kirjautumalla ensin web1 palvelimmelle käyttäen Dockeria. Sitten päivitettiin paketit ja asennettiin agentti komennolla:
```bash:
apt install snmp snmpd -y
```

Sitten tarkistettiin palvelun tila komennolla:
```bash:
service snmpd status
```
Tämä palautti tulosteen:
```bash:
* snmpd is not running
```
Lataus oli siis onnistunut, mutta palvelu oli pois päältä. Laitoin sen päälle komennolla:
```bash:
service snmpd start
```
Seuraavaksi avasin konfiguraatiotiedoston komennolla:
```bash:
nano /etc/snmp/snmdp.conf
```
Ennen tätä vaihetta jouduin lataamaan nanon koneelle, sillä sitä ei vielä ollut.

Konfiguraatiotiedoston sisällä vaihdoin yhteison nimeksi rocommunity public ja tallensin muutokset. Sitten uudelleenkäynnistin palvelun komennolla:
```bash:
service snmpd restart
```
Tämän jälkeen tarkistin, että prosessi on käynnissä komennolla ps aux | grep snmpd ja sain tulosteen:
```bash:
Debian-+    5387  0.0  0.1  20240  8116 ?        S    11:29   0:00 /usr/sbin/snmpd -LSwd -Lf /dev/null -u Debian-snmp -g Debian-snmp -I -smux mteTrigger mteTriggerConf -p /run/snmpd.pid
root        5396  0.0  0.0   3536  1868 pts/3    S+   11:29   0:00 grep --color=auto snmpd
```
Nyt agentti oli asennettu onnistuneesti.

## 3. Kerätyt tiedot

Kirjauduin ansible palvelimelle ja latasin SNMP:n sille, mutta kun yritin testata yhteyttä, se palautti virheen:
```bash:
system: Unknown Object Identifier (Sub-id not found: (top) -> system)
```
En tiennyt mitä tämä virhe tarkoittaa, joten kysyin AI:lta mitä kannattaisi yrittää ja se se ehdotti kokeilla system OID:n numeerista versiota 1.3.6.1.2.1.1, mutta tämä taas palautti virheen:
```bash:
Timeout: No Response from web1
```
Kokeilin myös komentoa, siten että vaihdoin web1 tekstin sen IP-osoitteeksi, mutta yhteys ei siltikään toiminut.
```bash:
root@ansible:/# snmpwalk -v2c -c public 10.10.20.101 1.3.6.1.2.1.1
Timeout: No Response from 10.10.20.101
```
Kysyin vielä tekoälyltä mikä ongelma on, ja sain homman toimimaan kun vaihdoin konfiguraatiotiedostossa agentaddress riville tekstin udp:161.
Sitten sain viimein tulosteen:

![snmpwalk komennon tuloste](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/SNMP%201.png "snmpwalk kommennon tuloste")

### Järjestelmän nimi:
iso.3.6.1.2.1.1.5.0 = STRING: "web1"

### Järjestelmän kuvaus: 
iso.3.6.1.2.1.1.1.0 = STRING: "Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64"

### Käyttöaika
iso.3.6.1.2.1.1.3.0 = Timeticks: (39404) 0:06:34.04

Latasin SNMP-agentin myös laitteille db1 ja branch-client ja hain samat tiedot.

| Laite | Nimi | Käyttöjärjestelmä | Uptime |
|---------|---------|---------|---------|
| web1 | web1 | Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 0:06:34.04 |
| db1 | db1 | Linux db1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 | 0:04:40.60 |
| branch-client | branch-client | Linux branch-client 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64" | 0:03:52.17 |

## 4. Verkkorajapinnat

SNMP:n avulla kerätyt rajapintatiedot.

Komennolla
```bash:
snmpwalk -v2c -c public web1 ifDescr
```
tuli tuloste:

iso.3.6.1.2.1.2.2.1.2.1 = STRING: "lo"

iso.3.6.1.2.1.2.2.1.2.177 = STRING: "eth0"

iso.3.6.1.2.1.2.2.1.2.204 = STRING: "eth1"

Rajapintoja löytyi siis 3 kappaletta. lo on loopback, joka lähettää tiedon vain takaisin samaan laitteeseen eikä ulospäin verkkoon. eth0 ja eth1 ovat yhteydessä verkkoon. Laitteen yhdistää siis kaksi eri rajapintaa verkkoon.

Käytin komentoa snmpwalk -v2c -c public web1 ifOperStatus selvittääkseni, ovatko molemmat eth0 ja eth1 oikeasti yhteydessä verkkoon ja totesin sen avulla, että ne ovat.

## 5. OID-analyysi

OID-objektien käyttötarkoitus.

| OID | Tarkoitus |
|------|------|
| sysName.0 | Laitteen nimi |
| sysDescr.0 | Laitteen kuvaus |
| sysUpTime.0 | Laitteen käynnissäolo aika |
| ifDescr | Verkkorajapintojen listaus |
| ifOperStatus | Verkkorajapintojen toimintatila |

## 6. Pohdinta

Omat havainnot SNMP:n hyödyistä ja rajoituksista

SNMP:n saa tietoa verkonlaitteiden toiminnasta ja yhteyksistä. Sen avulla voi saada selville ongelmia ja kerätä tietoa verkon suorituskyvystä. SNMP ei kerro suoraan ongelmia, mutta se tarjoaa työkaluja, joilla niitä voi havaita, sillä sen avulla laitteista voi saada paljon erilaista tietoa.

SNMP:n avulla voi kerätä paljon tietoa esimerkiksi laitteen nimen, kuvauksen ja käyttöajan, kuten tässä tehtävässä tehtiin kolmelle laitteelle. Lisäksi saa listattua verkkorajapinnat ja niiden tilat. Voi myös tarkkailla esimerkiksi CPU:n käyttlä tai laitteen lämpötiloja.

Yhteisöpohjaisen SNMPv2:n ongelmia on, että sen liikennettä ei salata, joten mahdolliset haitantekijät voisivat siepata sen liikennettä. Se ei myöskään tarjoa vahvaa käyttäjän tunnistamista.

SNMPv3 taas tarjoaa enemmän turvaa, jonka vuoksi sitä käytetää ympäristöissä, joissa tietoturva on tärkeää. SMPv3:ssa on mahdollisuus käyttäjän tunnistukseen ja liikenteen salaamiseen.
