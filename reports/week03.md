# Viikko 3 – Prometheus, Node Exporter ja Grafana

## 1. Johdanto

Prometheus on monitorointijärjestelmä, joka kerää ja tallentaa järjestelmien suorityskykyä kuvaavia tietoja. Prometheus hakee tietoja valvottavilta laitteilta tietyn aikamäräärän välein.

## 2. Node Exporterin käyttöönotto

Node Exporterin asennus onnistui kirjautumalla ensin web1-koneelle, jonne päivitettiin ensin paketit ja sitten ladattiin wget tar työkalu. Sitten Tarkistin GitHubista Node Exporterin uusimman version, joka oli 1.12.1 ja latasin tämän wgetillä sekä purin paketin. Sitten siirryin hakemistoon ja käynnistin sieltä Node Exporterin.

Tarkistin Node Exporterin vastaavuuden curl komentoa käyttäen ja sain tulokseksi suuren määrän mittareita, kuten toivottu.

![Curl tuloste](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-1.png "Curl tuloste")


## 3. Prometheus

Avasin selaimessa Prometheuksen. Sieltä havaitsin, että web1 näkyy kohteena ja sen tila on UP.

![Prometheus web1 UP](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-2.png "Prometheus web1 UP")

Kirjauduin Grafanaan, josta lisäsin tietolähteeksi Prometheuksen ja sain yhteyden toimimaan.

![Prometheus Data sourcena](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-4.png "Prometheus Data sourcena")
![Prometheus Grafana yhteys toimii](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-3.png "Prometheus Grana yhteys toimii")

## 4. Dashboard

Loin Grafanaan uuden dashboardin hyödyntäen Prometheus yhteyttä.

![Grafana dashboard 1](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-5.png "Grafana dashboard 1")
![Grafana dashboard 2](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-6.png "Grafana dashboard 2")

## 5. Kuormitustesti

Mittareiden käyttäytyminen kuormituksen aikana.

Levy kuormitus

![Levy kuormitus](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-7.png "Levy kuormitus")

Levytilan käyttö muuttui siis kuudesta prosentista 6.05%

CPU-kuormitus

![CPU-kuormitus](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-10.png "CPU-kuormitus")

CPU-käyrä lähti nousemaan hitaammin kuin levykäyrä ja jatkoi pidempään nousuaan kunnes se tasautui noin 8% kohdalla. Testays komentona käytin yes > /dev/null

Verkkoliikenteet pysyivät tasaisina.
![Verkkoliikenne käyrä 1](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-8.png "Verkkoliikenne käyrä 1")
![Verkkoliikenne käyrä 2](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week03-9.png "Verkkoliikenne käyrä 2")

## 6. SNMP vs Prometheus

Vertailutaulukko ja johtopäätökset.

| Ominaisuus | SNMP | Prometheus |
|------------|------|------------|
| Tiedonkeruu | Hyvä | Hyvä |
| Käyttöönotto | Monimutkaisempaa | Nopeaa |
| Mittarien määrä | Vähemmän | Enemmän |
| Visualisointi | Ei | Kyllä |
| Hälytysmahdollisuudet | Kyllä | Kyllä |
| Soveltuvuus pilviympäristöihin | Huono | Hyvä |

Prometheuksella voi kerätä hyvin tarkasti juuri haluamaansa dataa halutussa muodossa käyttäen queryja. Koen tämän hyödylliseksi verrattuna SNMP:hen. Lisäksi Prometheusta voi hyödyntää työkalujen, kuten Grafanan kanssa ja tehdä dashboardin, kuten tässä tehtävässä, josta voi tarkkailla tietoja helposti ja visuaalisesti.

## 7. Yhteenveto

Opin harjoituksen avulla Prometheuksen käyttöön otosta ja mittareiden hyödynnästä. Opin, että mittareita joita ylläpitäjän kanattaa seurata jatkuvasti on muunmuassa CPU:n, muistin ja levyn käyttö ja verkon sisään- ja ulostulo. Näiden avulla ongelma tilanteessa voi helposti nähdä, mistä päin ongelma tulee. Lisäksi voisi tarkailla esim CPU coreja.

Opin myös Prometheuksen ja SNMP:n eroavaisuuksia ja eri käytännöllisyyksiä.
