# Viikko 6 – Zabbix ja keskitetty verkonvalvonta

## 1. Johdanto

Mikä on Zabbix ja mihin sitä käytetään?

Zabbix on valvontaohjemlisto, jota käytetään palvelimien, verkkolaitteiden ja yms. seurantaan. Sillä voi seurata palveluiden tilaa, sekä suoritus kykyä. Zabbix kerää tietoja agenttien tai standardiprotokollien kautta ja pystyy hälyttämään ongelmisa välittömästi. Zabbix tarjoaa myös verkkokäyttöliittymän, josta saa graafisen näkymän kerätystä datasta.

### Zabbix käyttöliittymä

#### Hosts
Hosts-osiosta näkyy lisätyt laitteet (hostit) ja sieltä voi lisätä uusia sekä valita niiden nimen käytettävän templaten yms. Lisätyistä hosteista näkyy tieotoa koten applikaatioit, itemit triggerit, graafit, interface, proxy, templatet, status, availability ja agent encryption.

#### Templates
Tästä osioista löytyy kaikki valmiit ja itse loudut templatet eli mallit. Malleja voi hallinnoida, päivittää, poistaa kloonata. Nämä mallit sisältävät mittareita.

#### Monitoring
Monitiroing osioista voi tarkastella valvonnan tietoja. Esimerkiksi laitteiden tilan ja ongelmat.

#### Dashboards
Dasboards-osioissa voi itse muokata näkymää, sen mukaan, mitä tietoja haluaa nähdä ja miten. Esim laitteen suorituskyky, ongelmien määrä.

#### Reports
Tästä osioista löytyy Zabbixin tuottamia reportteja ja tapahtumatietoja. Sieltä näkee esim. Järjestelmän tiedot ja saatavuus raportit, sekä ilmoitukset.

#### Alerts
Tutkon läpi käyttöliittymää, eikä sieltä löytynyt tämän nimistä osioita. Mutta löytyi esim osio nimeltä Problems, josta näkyy hälytykset ongelmista.

## 2. Hostien lisääminen

Lisäsin web1, db1 ja branch-clietin hostiksi Zabbixiin.

![Hostien lisääminen 1](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week06-1.png "Hostien lisääminen 1")
![Hostien lisääminen 2](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week06-2.png "Hostien lisääminen 2")

Mutta ne valittavat ongelmasta Zabbix agent is not available, josta myös Zabbix serveri valittaa. WSL puolelta ainakin näyttää, että zabbix on päällä ja pyörii.

## 3. Dashboard

Ongelma näkyvissa dashboardissa
![Dashboard](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week06-3.png "Dashboard")

Yritin toisena päivänä uudestaan, mutta sama ongelma pysyi. Koitin ladata Zabbix agenttia web1 koneelle testi mielissä, jos se saisi ongelman ratkaistua ja vaihdoin Zabbixin configuraatio tiedostosta Hostnamen samaksi kuin mikä laitteelle on annettu Zabbixin verkkoliittymässä, mutta ongelma ei korjaantunut. Koitin kysyä tekoälyltä ratkaisua ongelmaan, sillä en saanut mitään toimimaan, mutta sekään ei saanut ratkaistua ongelmaa.     


## 4. Vertailu

SNMP vs Prometheus vs Zabbix.

| Ominaisuus | SNMP | Prometheus | Zabbix |
|------------|------|------------|--------|
| Tiedonkeruu | Verkkolaitteilla protokollan avulla | Kerää mittareita esim HTTP-rajapintojen kautta sovelluksista ja exportereista | Kerää tietoja agenteilta esim SNMP |
| Dashboardit | Ei omaa | Yksinkertainen kysely- ja kuvaajanäkymä. Voi käyttää Grafanan kanssa, jotta saa yksityiskohtaisemman ja monipuolisemman | Oma (Grafanan kaltainen) verkkokäyttöliittymä, jossa kuvaajat yms |
| Hälytykset | SNMP-trapeilla ilmoituksia | Voi määrittää hälytyssäännöt ja saada ilmoituksia ALertmanagerilla | Sisältää hätysjärjestelmän ja ongelmien seurannan |
| Käyttöönotto | Suhteellisen helppoa ja nopeaa | Vaatii asetusten määrittelyä | Vaatii palvelimen ja tietokannan käyttöönoton ja kohteiden ja mallien määrityksen |
| Skaalautuvuus | Riippuu järjestelmästä | Sopii suurille mittarimäärille | Hyvä suurissakin ympäristöissä |
| Yrityskäyttö | Verkkolaitteiden valvonnan standardi, osana isompaa kokonaisuutta | Pilvi- ja sovellusvalvonnassa | Sopii koko IT-infrastruktuurin valvontaan |
