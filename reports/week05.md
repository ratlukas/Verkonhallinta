## Johdanto

Tietoturva on kriittinen osa verkonhallintaa ja verkon turvallisuutta. Tulee pystyä tunnistamaan esimerkiksi hyökkäyksiä ja poikkeavaa liikennettä.

## Kaappaus

Liikenne kerättii käyttämällä tcpdump:ia web1:sestä käsin. Pistettiin päälle liikenteen kaappaus komennolla
```bash
tcpdump -i eth1 -w web1.pcap
```
Tämä komento aloittaa kaappauksen ethernet 1:sen liikenteestä ja tallentaa sen tiedostoon web1.pcap, joka voidaan sitten avata Wireshark ohjelmalla.

## Porttiskannaus

Porttiskannaus suoritettiin attacker koneelta käyttäen komentoa:
```bash
nmap 10.10.20.101
```
Tämä komento lähtetää IP paketteja valittuun kohteeseen (tässä tapauksessa web1) ja yrittään löytää avoimia portteja.

![Nmap tulos](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week05-1.png "Nmap tulos")

Nmap ei saanut yhtään vastausta web1:seltä, eli ei löytänyt avoimia portteja. Web1 esti siis vieraasta kohteesta tulleen liikenteen.

## Wireshark

![Wireshark](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week05-2.png "Wireshark")

Kun kaappauksen avasi Wiresharkista käsin, huomasi, että web1 vastaanotti liikennettä osoitteesta 10.10.10.200, eli attackerilta. Wiresharkista myös huomaa, että web1 on lähettänyt vastaukset attackerin pyyntöihin, vaikka Nmap sanoikin, että kohde ei ole saatavilla.

Attacker on testannut yhteyttä portteihin 80 ja 443, jotka ovat HTTP ja HTTPS portit. Mutta kaikki portit mitä on testattu, ovat olleet suljettuja, jonka huomaa siitä, että web1 vastaa RST paketeilla, jotka kieltävät yhteyden portteihin. Tämän vuoksi Nmap palauttaakin vastauksen, ettei se löydä avoimia portteja.

Suodattamalla dns tai http ei löydy mitään:

![DNS suodatus](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week05-3.png "DNS suodatus")
![Nmap tulos](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week05-4.png "Nmap tulos")

## Loki-analyysi

Tutkitut lokit ja löydökset.

![Loki 1](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week05-5.png "Loki 1")

journalctl logista huomaa että kun kaappaus laitettiin päälle eth1 meni "promiscious modeen". Sieltä näkyy myös kun menin eri kontteihin ja kun tarkistin ympäristön statuksen. Lisäksi näkyy paljon dropattuja cacheja.

![Loki 2](https://github.com/ratlukas/Verkonhallinta/blob/main/reports/images/week05-6.png "Loki 2")

Tässä /var/log/syslog. Kuvassa näkyy wsl-pro palvelun deactivointi ja uudelleenkäynnistys. Lisäksi virhe tilanne, jossa se ei saa ynteyttä Windows agenttiin.

## Turvallisuusarvio

Turvallista ympäristössä on se, että verkkoliikennettä voi seurata ja tallentaa tiedostoon, jota voi tarkkailla tarkemmin Wirehsarkilla. Tämän helpottaa mahdollisen poikkeavan liikenteen seuraamista ja havaitsemista. Lisäksi porttiskannaus, jonka avulla voitaisiin huomata mahdolliset avoimet portit, joita hyökkääjät voisivat hyväksi käyttää. Tämän avulla voitiin todeta, ettei ole avoimia portteja, joista hyökkääjät voisivat käydä käsiksi.

Mahdollinen parannusehdotus ympäristöön olisi se, että lisättäisiin palomuuri, joka kieltäisi liikenteen kaikkialta paitsi tietyistä tarvittaviasta kohteista. Myös päältä pois kaikki käyttämättömät palvelut ja portit.

## Yhteenveto

Opin, miten tallentaa verkkoliikennettä tiedostoon, jota voin lukea ja analysoida Wiresharkin avulla. Lisäksi opin, että verkkoliikenteen analysoinnin avulla voi ylläpitää ympäristön turvallisuutta, sillä se auttaa havaitsemaan epäilyttävää liikennettä ja mistä se tulee. Esimerkiksi mahdollisen hyökkäyksen voisi tunnistaa Wiresharkin avulla, jos näkisi, että tietystä tuntemattomasta osoitteesta tulee jatkuvasti paketteja.
