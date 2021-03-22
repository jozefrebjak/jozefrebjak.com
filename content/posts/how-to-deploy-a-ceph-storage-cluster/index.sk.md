---
title: 'Ako nasadiť úložisko Ceph'
resources:
  - name: 'featured-image'
    src: 'featured-image-sk.png'
categories: ['documentation']
tags: ['ceph', 'installation']
date: 2021-03-22T09:31:44+01:00
draft: false
---

## Čo je Ceph

**Ceph** je bezplatný klastrový softvér s otvoreným zdrojovým kódom, ktorý spája viac úložných serverov, z ktorých každý obsahuje veľké množstvo pevných diskov. Spoločnosť Ceph v podstate poskytuje ukladanie objektov, blokov a súborov v jednom horizontálne škálovateľnom klastri bez jediného bodu zlyhania. Úložisko Ceph je možné ľahko škálovať v priebehu času. Môže byť nakonfigurovaný na vysokú dostupnosť odstránením jednotlivých bodov zlyhania. Má tiež veľa podnikových funkcií vrátane snapshot-ov, thin provisioning, tiering a možnosti samoopravovania.

## Ako funguje Ceph

Klaster úložiska Ceph pozostáva z viacerých typov démonov:

1. Ceph Monitor
2. Ceph OSD Daemon
3. Ceph Manager
4. Ceph Metadata Server

**Ceph Monitor** (ceph-mon)

Ceph Monitor udržuje hlavnú kópiu mapy klastra. Klaster monitorov Ceph zaisťuje vysokú dostupnosť v prípade zlyhania démona monitora. Klienti úložného klastra načítajú kópiu mapy klastra z monitora Ceph. Mali by existovať minimálne tri inštancie, aby sa udržalo kvórum. Ak sú iba dve, je ľahké sa dostať do situácie s rozdeleným mozgom, z ktorej sa dá ťažko zotaviť. Ak existuje iba jeden, potom ak táto inštancia prejde do režimu offline, je offline aj celý klaster Ceph.

**Ceph OSD Daemon** (ceph-osd)

Ceph OSD Daemon kontroluje svoj vlastný stav a stav ostatných OSD a podáva správy monitorom. Každý OSD démon je priradený ku konkrétnej inštancii úložiska blokov. Takže ak máte tri servery Ceph a každý z nich má pripojené dva pevné disky na použitie s Ceph, potom bude mať každý server Ceph dva procesy OSD démonov, teda celkovo šesť klastrov OSD démonov.

{{< admonition type=info open=true >}}
`replicas=3` nastavenie (čo je predvolené a malo by sa považovať za minimálne) znamená, že každý blok bude aktívny v primárnom OSD a bude kopírovaný do dvoch ďalších.
{{< /admonition >}}

**Ceph Manager** (ceph-mgr)

A Ceph Manager slúži ako koncový bod pre monitorovanie, orchestráciu a zásuvné moduly. Zatiaľ čo monitory Ceph používajú na udržanie konzistentnosti kvóra, Ceph manažéri používajú mechanizmus aktívneho pohotovostného režimu. Aktívny je kedykoľvek iba jeden Manager démon. V prípade, že aktívny démon prejde do režimu offline, okamžite ho prevezme jedna z pohotovostných inštancií. Technicky stačia dve inštancie správcu, pretože kedykoľvek je aktívna iba jedna, je však stále dobré spustiť inštanciu na každom systéme, na ktorom je tiež spustený Ceph Monitor.

**Ceph Metadata Server** (ceph-mds)

A Ceph Metadata Server (MDS) spravuje súborové metadáta, keď sa na poskytovanie súborových služieb používa CephFS.

Klienti úložiska Ceph a každý démon Ceph OSD používajú algoritmus CRUSH na efektívne počítanie informácií o umiestnení údajov namiesto toho, aby boli závislí od centrálnej vyhľadávacej tabuľky.

{{< admonition type=info open=true >}}
Ak sa chcete dozvedieť viac, prosím prečítajte si oficiálnu {{< link href="https://docs.ceph.com/en/latest/architecture/" content=dokumentáciu >}}.
{{< /admonition >}}

Keď je nasadený jeden alebo viac monitorov a dva alebo viac objektových úložísk, je to známe ako **Ceph Storage Cluster**. Systém súborov, úložisko objektov a blokové zariadenia čítajú a zapisujú údaje do a a z úložného klastra. V predvolenom nastavení sú hranice hostiteľa najvyššou prioritou, pričom dve kópie sa uložia v OSD, ktoré sú na rôznych hostiteľoch.

Napríklad `block-1` bude existovať na hostiteľoch `A`, `B` a `C`. Dôvod, prečo je také dôležité mať aspoň troch hostiteľov, je taký, aby určil integritu údajov, ktoré musí Ceph dosiahnuť, aby to bolo pre každý blok uznášaniaschopné. Ak sa `block-1` na hostiteľovi `A` poškodí, Ceph to dokáže zistiť a opraviť, pretože `block-1` na hostiteľoch `B` a `C` sa zhoduje s tým na hostitelovi `A`.

Ak sú hostitelia iba dvaja, potom je podstatne ťažšie a niekedy nemožné určiť, ktorý blok je v prípade sporu správny. To tiež znamená, že ak chceme `5TB` použiteľného úložiska, budeme potrebovať `15TB` surového (RAW) úložiska distribuovaného ako `5TB` na každom zo serverov Ceph.

`OSD` maju vlastný formát disku, ktorý sa nazýva `Bluestore`, a priradenie OSD ID k bloku zariadenia sa dosahuje pomocou `LVM` a `Device Mapper`. To znamená, že ak nakoniec zamiešame disky v jednom systéme, OSD ID na blokoch zariadenia sa zachová. To je obzvlášť užitočné pri používaní diskov pripojených cez USB.

## Vytvorenie klastra Ceph

Pre začiatok a vyskúšanie si toho ako to vlastne celé funguje potrebujeme v podstate 3 uzly (nody). V našom prípade použijeme virtuálne servery v prostredí VMware.

Každý server bude mať minimálne virtuálne prostriedky:

- 2 vCPU
- 4GB RAM
- 2x vDisk (Prvý 30GB OS disk a druhý 20GB disk ako OSD pre Ceph)
- Ubuntu 20.04

|   Hostname   |  IP Adresa   |       Komponenty        |
| :----------: | :----------: | :---------------------: |
| ceph-node-01 | 10.99.107.81 | Ceph MON, MGR, OSD, MDS |
| ceph-node-02 | 10.99.107.82 | Ceph MON, MGR, OSD, MDS |
| ceph-node-03 | 10.99.107.83 | Ceph MON, MGR, OSD, MDS |

### Príprava

Pred spustením Ceph klastra sa uistime, že v každom systéme, ktorý bude serverom Ceph, máme nainštalované závislosti:

- Systemd
- Podman alebo Docker na prevádzkovanie kontajnerov
- Synchronizácia času (napríklad chrony alebo NTP)
- LVM2 na zabezpečenie úložných zariadení

Kedže žijeme v modernej dobe, máme k dispozícii nástroj **ansible**, ktorý nám celý proces zjednodušuje a dokáže pre nás všetko potrebne zabezpečiť.

Prihlasime sa do ceph-node-01, krory bude nas ADMIN uzol pomocou SSH:

```sh
ssh root@ceph-node-01
```

Aktualizujeme súbor `/etc/hosts` položkami pre všetky adresy IP a názvy hostiteľov.

```sh
10.99.107.81  ceph-node-01
10.99.107.82  ceph-node-02
10.99.107.83  ceph-node-03
```

Aktualizujeme OS:

```sh
apt update && apt -y upgrade
```

Nainštalujeme Ansible a ďalšie základné pomocné programy:

```sh
apt update
apt -y install software-properties-common git curl bash-completion ansible
```

Zaistime, aby bola cesta `/usr/local/bin` pridaná do **PATH**:

```sh
echo "PATH=\$PATH:/usr/local/bin" >>~/.bashrc
source ~/.bashrc
```

Vygenerujeme kľúč SSH:

```sh
ssh-keygen -t rsa -b 4096 -N '' -f ~/.ssh/id_rsa
```

Nahráme kľúč SSH na všetky uzly pomocou jednoducheéo BASH skriptu:

```bash
while read SERVER
do
    ssh-copy-id root@"${SERVER}"
done <<\EOF
ceph-node-01
ceph-node-02
ceph-node-03
EOF
```

Nainštalujeme `cephadm`:

```sh
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/octopus/src/cephadm/cephadm
chmod +x cephadm
sudo mv cephadm  /usr/local/bin/
```

Uistíme sa, že `cephadm` je k dispozícii na miestne použitie:

```sh
cephadm --help
```

S nakonfigurovaným prvým uzlom `ceph-node-01` vytvorime zodpovedajúcu ansible príručku na aktualizáciu všetkých uzlov a do všetkých uzlov nahrame verejný kľúč ssh a aktualizačný súbor `/etc/hosts`.

```sh
cd ~/
nano prepare-ceph-nodes.yml
```

Upravíme obsah nižšie, aby sme mali správne časové pásmo a pridali ho do súboru:

```yaml
---
- name: Prepare ceph nodes
  hosts: ceph_nodes
  become: yes
  become_method: sudo
  tasks:
    - name: Set timezone
      timezone:
        name: Europe/Bratislava

    - name: Update system
      apt:
        name: '*'
        state: latest
        update_cache: yes

    - name: Install common packages
      apt:
        name: [nano, git, bash-completion, wget, curl, chrony]
        state: present
        update_cache: yes

    - name: Install Docker
      shell: |
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" > /etc/apt/sources.list.d/docker-ce.list
        apt update
        apt install -qq -y docker-ce docker-ce-cli containerd.io
```

Vytvoríme súbor inventára:

```sh
nano hosts
```

Pripravíme inventár:

```sh
[ceph_nodes]
ceph-node-01
ceph-node-02
ceph-node-03
```

Nakonfigurejeme SSH:

```sh
tee -a ~/.ssh/config<<EOF
Host *
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    IdentitiesOnly yes
    ConnectTimeout 0
    ServerAliveInterval 300
EOF
```

Spustíme Ansible príručku ktorá nainštaluje na všetky uzly `Chrony` (TZ Europe/Bratislava) a `Docker`:

```sh
ansible-playbook -i hosts prepare-ceph-nodes.yml --private-key ~/.ssh/id_rsa
```

Aktualizujeme `/etc/hosts` na všetkých uzloch pomocou Ansible:

```sh
nano update-hosts.yml
```

Pripravíme príručku:

```yaml
---
- name: Prepare ceph nodes
  hosts: ceph_nodes
  become: yes
  become_method: sudo
  tasks:
    - name: Clean /etc/hosts file
      copy:
        content: ''
        dest: /etc/hosts

    - name: Update /etc/hosts file
      blockinfile:
        path: /etc/hosts
        block: |
          127.0.0.1     localhost
          10.99.107.81  ceph-node-01
          10.99.107.82  ceph-node-02
          10.99.107.83  ceph-node-03
```

Spustíme ansible príručku:

```sh
ansible-playbook -i hosts update-hosts.yml --private-key ~/.ssh/id_rsa
```

{{< admonition type=success open=true >}}
V tejto chvili mame pripravene uzly pre Ceph Cluster! Zvyšok už nebude takto náročný.
{{< /admonition >}}
