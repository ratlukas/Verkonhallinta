# Viikko 4 – Ansible ja Infrastructure as Code

## Johdanto

Infrastructure as Code on periaate, jonka mukaan verkot, palvelimet jne. määritellään koodina ja tiedostoina, eikä käsin konfiguroida. Ansible on yksi tapa toteuttaa tätä periaatetta.

## Inventory

Ympäristön rakenne.
Tässä Ansiblen antama inventory.
```bash
root@ansible:/# cat /ansible/inventory.ini
# ============================================================
# Ansible Inventory for HAMK Network Management Lab
# ============================================================
# Generated for containerlab topology: hamk-verkonhallinta-golden
#
# Management network: 172.20.20.0/24 (clab-mgmt)
# All nodes are accessible via their containerlab short names
# ============================================================

# ============================================================
# ROUTERS
# ============================================================
[routers]
r1 ansible_host=clab-hamk-verkonhallinta-golden-r1
r2 ansible_host=clab-hamk-verkonhallinta-golden-r2
r3 ansible_host=clab-hamk-verkonhallinta-golden-r3

[routers:vars]
ansible_network_os=frr
ansible_connection=network_cli
ansible_user=admin
ansible_become=yes
ansible_become_method=enable

# ============================================================
# CLIENT WORKSTATIONS
# ============================================================
[clients]
client1 ansible_host=clab-hamk-verkonhallinta-golden-client1 data_ip=10.10.10.101
attacker ansible_host=clab-hamk-verkonhallinta-golden-attacker data_ip=10.10.10.200
branch-client ansible_host=clab-hamk-verkonhallinta-golden-branch-client data_ip=10.10.30.101

[clients:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=Hamk2024!
ansible_python_interpreter=/usr/bin/python3

# ============================================================
# SERVERS
# ============================================================
[servers]
web1 ansible_host=clab-hamk-verkonhallinta-golden-web1 data_ip=10.10.20.101 role=webserver
db1 ansible_host=clab-hamk-verkonhallinta-golden-db1 data_ip=10.10.20.102 role=database

[servers:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=Hamk2024!
ansible_python_interpreter=/usr/bin/python3

# ============================================================
# MONITORING INFRASTRUCTURE
# ============================================================
[monitoring]
prometheus ansible_host=clab-hamk-verkonhallinta-golden-prometheus service=prometheus port=9090
grafana ansible_host=clab-hamk-verkonhallinta-golden-grafana service=grafana port=3000
zabbix ansible_host=clab-hamk-verkonhallinta-golden-zabbix service=zabbix port=8080
cadvisor ansible_host=clab-hamk-verkonhallinta-golden-cadvisor service=cadvisor port=8080

[monitoring:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=Hamk2024!

# ============================================================
# MANAGEMENT TOOLS
# ============================================================
[management]
ansible ansible_host=clab-hamk-verkonhallinta-golden-ansible

[management:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=Hamk2024!

# ============================================================
# NETWORK SEGMENTS (Logical Groups)
# ============================================================
[user_network]
client1
attacker

[server_network]
web1
db1

[branch_office]
branch-client

# ============================================================
# INFRASTRUCTURE GROUPS
# ============================================================
[network_devices:children]
routers

[linux_hosts:children]
clients
servers
monitoring
management

[ubuntu_hosts]
client1
web1
db1
branch-client

[ubuntu_hosts:vars]
ansible_distribution=Ubuntu

# ============================================================
# NODE_EXPORTER TARGETS (for Prometheus monitoring)
# ============================================================
[node_exporter:children]
routers
clients
servers

# ============================================================
# ALL NODES
# ============================================================
[all:vars]
# Default connection settings
ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
# Containerlab uses root by default
ansible_become=no
```
Inventorysta löytyy verkon laitteet kuten reitittimet, clientit ja serverit. Lisäksi tarkkailu työkalut kuten Prometheus ja Grafana. Sitten löytyy hallintatyökaluja. Lisäksi on on ryhmät eri aliverkoille. On myös ryhmittelyä sen mukaan ovatko ne hosteja vai childreneitä. Myös Prometheus node exporttereilla on oma ryhmä.

Näistä ryhmistä voi helposti nähdä, mitä laitteita verkossa on, mitä ne tekevät, mikä niiden rooli on tai sitten missä aliverkossa ne ovat. Lisäksi näkee mitä työkaluja löytyy verkon tarkkailuun ja hallintaan. Ylipäätään selkeä ja hyvä overview verkosta.

### Tehtävä 4.5 Järjestelmätiedot

Tuloksia komennosta

```bash
    ansible all -i ../inventory.ini -m setup
```

| Käyttöjärjestelmä | IP-osoite | Prosessoreiden määrä | Muistin määrä |
|------------|---------|------------|------------|
| Ubuntu 24.04.4 LTS | 172.20.20.14 | 16 | 7591 mb|

## Esimerkkiplaybookit

Havainnot SNMP- ja Node Exporter -playbookeista.

Playbookin inventory.ini ping yml. käyttö palautti tämän:

```bash
root@ansible:/ansible/playbooks# ansible-playbook -i ../inventory.ini ping.yml

PLAY [Test Linux hosts] ************************************************************************************************************************************

TASK [Ping via Ansible] ************************************************************************************************************************************
ok: [attacker]
ok: [branch-client]
ok: [client1]
fatal: [prometheus]: UNREACHABLE! => {"changed": false, "msg": "Data could not be sent to remote host \"clab-hamk-verkonhallinta-golden-prometheus\". Make sure this host can be reached over ssh: ssh: connect to host clab-hamk-verkonhallinta-golden-prometheus port 22: Connection refused\r\n", "unreachable": true}
ok: [db1]
fatal: [grafana]: UNREACHABLE! => {"changed": false, "msg": "Data could not be sent to remote host \"clab-hamk-verkonhallinta-golden-grafana\". Make sure this host can be reached over ssh: ssh: connect to host clab-hamk-verkonhallinta-golden-grafana port 22: Connection refused\r\n", "unreachable": true}
fatal: [zabbix]: UNREACHABLE! => {"changed": false, "msg": "Data could not be sent to remote host \"clab-hamk-verkonhallinta-golden-zabbix\". Make sure this host can be reached over ssh: ssh: connect to host clab-hamk-verkonhallinta-golden-zabbix port 22: Connection refused\r\n", "unreachable": true}
ok: [web1]
fatal: [cadvisor]: UNREACHABLE! => {"changed": false, "msg": "Data could not be sent to remote host \"clab-hamk-verkonhallinta-golden-cadvisor\". Make sure this host can be reached over ssh: ssh: connect to host clab-hamk-verkonhallinta-golden-cadvisor port 22: Connection refused\r\n", "unreachable": true}
fatal: [ansible]: UNREACHABLE! => {"changed": false, "msg": "Invalid/incorrect password: Warning: Permanently added 'clab-hamk-verkonhallinta-golden-ansible' (ED25519) to the list of known hosts.\r\nPermission denied, please try again.", "unreachable": true}

PLAY [Test routers] ****************************************************************************************************************************************

TASK [Run show version] ************************************************************************************************************************************
[WARNING]: ansible-pylibssh not installed, falling back to paramiko
fatal: [r2]: FAILED! => {"changed": false, "msg": "paramiko is not installed: No module named 'paramiko'"}
fatal: [r1]: FAILED! => {"changed": false, "msg": "paramiko is not installed: No module named 'paramiko'"}
fatal: [r3]: FAILED! => {"changed": false, "msg": "paramiko is not installed: No module named 'paramiko'"}

PLAY RECAP *************************************************************************************************************************************************
ansible                    : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
attacker                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
branch-client              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
cadvisor                   : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
client1                    : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
db1                        : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
grafana                    : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
prometheus                 : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
r1                         : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
r2                         : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
r3                         : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
web1                       : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
zabbix                     : ok=0    changed=0    unreachable=1    failed=0    skipped=0    rescued=0    ignored=0
```
Playbookin tarkoitus oli testata yhteys kohteisiin pingamalla niitä. Sen palauttamista tuloksista huomataan, ettei yhteyttä saa ainakaan Prometheukseen, Grafanaan, Zabbixiin tai cadvisoriin. Yhteys toimi web1, db1, attackeriin ja client1:seen sekä branch-clienttiin, mutta myöskään routtereita ei saatu pingattua. Osa yhteyksistä ei toiminut sillä yhteydenotto estettiin ja osa taas sanoo, ettei toimivuus pelaa puuttuvien tiedostojen vuoksi.

### install-snmp.yml

Tässä playbookissa käytetään ainakin tämän tyyppisiä moduuleja: service, debug, copy, notify ja apt. Playbookin tarkoitus on nimensä mukaan asentaa SNMP. Lisäksi se laittaa palvelun pyörimään.

Ensimmäinen moduuli päivittää apt paketit, sitten seuraava lataa SNMP paketit. Nämä kaksi ovat apt moduuleja. Seuraavaksi copy moduuli konfiguroi SNMP communityn vaihtamalla configuraatio tiedostotsta rocommunity kohdan ja sitten käyttäen notify:ta kutsuu handler serviceä, joka uudelleen käynnistää SNMPD:n

Sitten service moduuli enablaa SNMP palvelun ja shell moduuli käyttää pgrep komentoa varmistaakseen, että SNMP pyörii. Lopuksi debug moduuli näyttää varmennuksen ja millä hostilla SNMP agentti pyörii.

Playbookissa muuttujia on käytetty vain kerran vastaavasti:
```bash
vars:
    snmp_community: public
```
Muuttujalle snmp_community on asetettu arvo public.

Handlers lohkossa on yksi palvelu nimeltä restart snmpd, joka nimensä mukaan uudelleen käynnistää snmpd:n.

### install-node-exporter.yml.

Tämän playbookin tarkoitus on asentaa node exporter.

Tässä playbookissä käytetään ainakin apt, file, get_url, unarchive, copy shell, args, uri. register ja debug moduuleja. Muuttujia tässäkin playbookissa on vain yksi, joka asettaa node exporter versioksi 1.9.1. Handlers lohkoa tässä playbookissa ei ole.

Ensimmäiseksi playbook lataa apt:tä käyttäen paketit joita tarvitaan node exportteria varten. Sitten se käyttää file moduulia luodakseen directoryn latausta varten. Seuraavaksi get_url:lää hyödyntäen node exporter ladataan GitHub linkistä. Tämän jälkeen unarchviea käytetään exporttaukseen ja copya exporter binaryn lataukseen. Tässä moduulissa hyödynnetään aiemmin asetettua muuttujaa.

Seuraavaksi node exporter käynnistetään shell moduulilla, joka syöttää tarvittavan komennon käynnistykseen. Sitten uri moduulia käytetään tarkistamaan, että exporter vastaa ja lopuksi debug moduuli näyttää varmennuksen, että Node exportter pyörii hostilla.

Tämä playbook ei tarvitse handleria, sillä ei ole käytössä moduulia, joka hyödyntää notify:ta. Eli ei ole mitään tehtävää jonka tarvitsisi kutsua toista.

## Oma playbook

Koodi ja suorituksen tulokset. Tein playbook vaihtoehdon A eli playbookin joka asentaa web-palvelimen web1-koneelle käyttäen Nginxiä. Nginxin valitsin sen vuoksi, että sen käytöstä minulla on aiempaa kokemusta.

Tässä on koodi, jonka kirjoitin:

```bash
---
- name: Install Web-server to web1
  hosts: web1
  become: true

  vars:
    server_name: "{{ ansible_hostname }}"

  tasks:

    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Create index.html
      copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html>
          <head>
            <title>Web server</title>
          </head>
          <body>
            <h1>Nginx web server</h1>
            <p>Server: {{ server_name }}</p>
          </body>
          </html>
        owner: root
        group: root
        mode: "0644"

    - name: Start Nginx
      shell: systemctl start nginx

    - name: Enable Nginx
      service:
        name: nginx
        enabled: yes

    - name: Verify web server responds
      uri:
        url: http://localhost
        status_code: 200

    - name: Show verification result
      debug:
        msg: "Web server running on {{ inventory_hostname }}"
```
Aluksi, kun yritin pyörittää sain virheitä siitä, ettei sisennykseni olleet oikein ja välilyöntejä puuttui (yllä näkyvästä versiosta nämä jo korjattu), mutta kun sain syntaxin oikein pystyin yrittämään pyörittää playbookkia. Ensimmäinen pyöritys yritys näytti tältä:

```bash
root@ansible:/ansible/playbooks# ansible-playbook -i ../inventory.ini install-webserver.yml

PLAY [Install Web-server to web1] **********************************************************************************************************************************************************

TASK [Update package cache] ****************************************************************************************************************************************************************
ok: [web1]

TASK [Install nginx] ***********************************************************************************************************************************************************************
changed: [web1]

TASK [Create index.html] *******************************************************************************************************************************************************************
changed: [web1]

TASK [Start Nginx] *************************************************************************************************************************************************************************
changed: [web1]

TASK [Enable Nginx] ************************************************************************************************************************************************************************
ok: [web1]

TASK [Verify web server responds] **********************************************************************************************************************************************************
fatal: [web1]: FAILED! => {"changed": false, "elapsed": 0, "msg": "Status code was -1 and not [200]: Request failed: <urlopen error [Errno 111] Connection refused>", "redirected": false, "status": -1, "url": "http://localhost"}

PLAY RECAP *********************************************************************************************************************************************************************************
web1                       : ok=5    changed=3    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```
Itse asennus onnistui, mutta lopussa tapahtuva tarkistus epäonnistui totesin, että tämä johtui siitä, ettei Nginx mennyt varmaan oikein päälle, sillä palvelin ei vastannut. Muokkasin täten osuutta koodista, joka pistää serverin päälle tähän muotoon, jota käytettin SNMP esimerkissä:

```bash
service:
    name: nginx
    state: started
    enabled: yes
```
Tämä ei kuitenkaan korjannut ongelmaani vaan seuraavaksi, kun yritin pyörittää koodin sain tälläisen tulosteen:

```bash
root@ansible:/ansible/playbooks# nano install-webserver.yml
root@ansible:/ansible/playbooks# ansible-playbook -i ../inventory.ini install-webserver.yml

PLAY [Install Web-server to web1] **********************************************************************************************************************************************************

TASK [Update package cache] ****************************************************************************************************************************************************************
ok: [web1]

TASK [Install nginx] ***********************************************************************************************************************************************************************
ok: [web1]

TASK [Create index.html] *******************************************************************************************************************************************************************
ok: [web1]

TASK [Start and enable Nginx] **************************************************************************************************************************************************************
fatal: [web1]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}

PLAY RECAP *********************************************************************************************************************************************************************************
web1                       : ok=3    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```
Ongelmani on siis edelleen, ettei Nginx pyöri tai käynnisty oikein. Sain tämän saman errorin kun yritin käyttää esimerkki playbookkia SNMP:n lataukseen. Tässä tuloste SNMP playbookista:

```bash
TASK [Enable SNMP service] *****************************************************************************************************************************************************************
fatal: [web1]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}
fatal: [branch-client]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}
fatal: [db1]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}
fatal: [client1]: FAILED! => {"changed": false, "msg": "Service is in unknown state", "status": {}}
```
Sitten menin web1 palvelimelle sisälle ja kokeilin käyttää sieltä manuaalisesti systemctl komentoa, jota olin playbookissa yrittänyt käyttää Nginxin käynnistykseen alunperin. Tämä palautti minulle virheen:

```bash
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```
Tästä tajusin miksei komennon käyttäminen ollut sitten pistänytkään käyntiin palvelua. En vieläkään ymmärtäny miksei esimerkissä käytetty versiokaan toiminut, mutta päätin testata käyttää jotain toista komentoa Nginxin käynnistykseen. Päädyin tähän:

```bash
    shell: service nginx start
```
Sitten yritin pyörittää playbookkia uudestaan ja tällä kertaa en saanut yhtään erroria, tässä tuloste:

```bash

PLAY [Install Web-server to web1] **********************************************************************************************************************************************************

TASK [Update package cache] ****************************************************************************************************************************************************************
ok: [web1]

TASK [Install nginx] ***********************************************************************************************************************************************************************
ok: [web1]

TASK [Create index.html] *******************************************************************************************************************************************************************
ok: [web1]

TASK [Start Nginx] *************************************************************************************************************************************************************************
changed: [web1]

TASK [Enable Nginx] ************************************************************************************************************************************************************************
ok: [web1]

TASK [Verify web server responds] **********************************************************************************************************************************************************
ok: [web1]

TASK [Show verification result] ************************************************************************************************************************************************************
ok: [web1] => {
    "msg": "Web server running on web1"
}

PLAY RECAP *********************************************************************************************************************************************************************************
web1                       : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Nyt playbook viimein asentaa ja käynnistää onnistuneesti Nginx web palvelimen. Tässä on lopullinen koodi:

```bash
---
- name: Install Web-server to web1
  hosts: web1
  become: true

  vars:
    server_name: "{{ ansible_hostname }}"

  tasks:

    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Create index.html
      copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html>
          <head>
            <title>Web server</title>
          </head>
          <body>
            <h1>Nginx web server</h1>
            <p>Server: {{ server_name }}</p>
          </body>
          </html>
        owner: root
        group: root
        mode: "0644"

    - name: Start Nginx
      shell: service nginx start

    - name: Enable Nginx
      service:
        name: nginx
        enabled: yes

    - name: Verify web server responds
      uri:
        url: http://localhost
        status_code: 200

    - name: Show verification result
      debug:
        msg: "Web server running on {{ inventory_hostname }}"
```
Tässä myös vielä tulosta web1:sen puolelta index.html:stä
```bash
root@web1:/# curl http://localhost
<!DOCTYPE html>
<html>
<head>
  <title>Web server</title>
</head>
<body>
  <h1>Nginx web server</h1>
  <p>Server: web1</p>
</body>
</html>
```

## Vertailu

Ansiblella latausten tekeminen tuntui aluksi paljon monimukteisemmalta, kuin latausten tekeminen käsin, mutta kun pääsi itse kirjoittamaan playbookkia se alkoi tuntua paljon selkeämmältä. Yksittäistä latausta varten käsin lataus on ihan riittävä keino, mutta automatisoitu lataus on hyvä tilanteissa, joissa tehtävä pitää suorittaa monta kertaa tai lataus pitää tehdä usualle eri laitteelle.

Yaml tiedoston kirjoittamisessa menee jonkin aikaa ja varsinkin jos ei ole aiempaa kokemusta paljoa, se voi olla haastavaa. Tämän vuoksi yksittäisissä latauksissa on helpompii tehdä se käsin. Mutta taas sitten kun lataus tarvitsee tehdä vaikkapa 5 eri koneelle, on paljon helpompaa kirjoittaa kaikki vaiheet kerran YAML tiedostoon, kuin kirjoittaa ne joka koneelle 5 kertaa.

5 ei vielä ole kuitenkaan kovin suuri luku. Voisi olla tilanne, jossa on hyvin suuri verkko, jossa on vaikkapa yli 100 laitetta ja kaikille laitteille pitää tehdä useita asennuksia. Tällöin käsin asennus on niin aikaa vievä tehtävä, että se on vähintään erittäin järkevää jollei jopa välttämätöntä automatisoida.

## Yhteenveto

Tässä tehtävässä opin käyttämään Ansible inventorya ja playbookkeja. Opin luomaan oman playbookin ja suorittamaan sellaisen onnistuneesti. Tämän tiedon avulla osaisin nyt tehdä hallitsemassani verkossa latauksen johonkin laitteeseen etää käsin. Lisäksi opin Infrastructure as code periaattesta, jota pääsin tehtävässä hyödyntämään.
