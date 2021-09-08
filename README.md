# Hands-on pro Ansible

## Instalace Ansible
Nainstalujte si na svoje PC Ansible verze 4 z [PPA pro Ubuntu](https://launchpad.net/~ansible/+archive/ubuntu/ansible-4):

```bash
sudo add-apt-repository ppa:ansible/ansible-4
sudo apt-get update
sudo apt install ansible
```

Pokud máte jiný Linux než Mint nebo Ubuntu, postupujte podle návodu [Installing Ansible on specific operating systems](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-specific-operating-systems).

Pokud máte MS Windows, nainstalujte si Ubuntu do WSL podle návodu [How to install Windows Subsystem for Linux (WSL) on Windows 10](https://www.windowscentral.com/install-windows-subsystem-linux-windows-10).

Dokumentace:
- [Ansible Documentation](https://docs.ansible.com/ansible/latest/)
- [Index of all modules](https://docs.ansible.com/ansible/latest/collections/index_module.html#ansible-builtin)

## Ssh klíč

Zjistěte si, jaký používáte ssh klíč. Nejspíš stačí
```bash
cat ~/.ssh/id_*.pub
```
nebo
```bash
ssh-add -L
```

## Vytvoření virtuálního stroje na pokusy v OpenStacku

- přihlaste se na [https://cloud.muni.cz/](https://cloud.muni.cz/)
- zobrazte stránku [Compute - Key Pairs](https://dashboard.cloud.muni.cz/project/key_pairs)
- pokud tam žádný Key Pair nemáte, klikněte na **Import Public Key** a přidejte tam svůj ssh klíč pojmenovaný po sobě, tj. do **Key Pair Name** zadejte třeba "Pepa" (bude to jediný zobrazovaný údaj, podle kterého půjde poznat vlastník VM), zvolte **Key Type** SSH Key, a do **Public Key** zkopírujte svůj ssh klíč zjištěný v předchozím bodu
- zobrazte stránku [Compute - Instances](https://dashboard.cloud.muni.cz/project/instances/)
- zkontrolujte, že vlevo nahoře vedle loga OpenStack máte zvolený projekt **perun-cesnet-muni**
- klikněte na tlačítko **Launch Instance** vpravo nahoře, zobrazí se wizard pro vytvoření nového virtuálného stroje
  - v položce **Detail** zvolte hostname, např. "pepa-1"
  - v položce **Source** v seznamu **Select Boot Source** zvolte "Image" a v seznamu **Available** klikněte vpravo na šipku u image **debian-10-x86_64**
  - v položce **Flavor** klikněte vpravo na šipku u **standard.small** (1 CPU, 2GB RAM, 80GB HDD)
  - v položce **Networks** vyberte kliknutém na šipku vpravo síť **group-project-network**
  - klikněte na tlačítko **Launch instance** ve wizardu vpravo dole
- instance VM v tomto okamžiku má jen neveřejnou IP adresu, je potřeba přiřadit veřejnou "floating IP" 
  - navštivte stránku [Network - Floating IPs](https://dashboard.cloud.muni.cz/project/floating_ips/) a zkontrolujte, že je volná nějaká IP, pokud ne, klikněte na tlačítko "Allocate IP To Project" a jednu alokujte z poolu public-muni-147-251-21-GROUP
  - vraťte se na stránku [Compute - Instances](https://dashboard.cloud.muni.cz/project/instances/)
  - klikněte v seznamu instancí VM v řádku této instance ve sloupci Actions na seletion list a vyberte v něm **Associate floating IP**
  - vyberte volnou floating IP adresu a klikněte na tlačítko **Associate**
  - zkopírujte si do clipboardu tu druhou zobrazenou IP adresu, to je ta veřejná
- zjistěte, na jaké DNS jméno je IP mapována a přihlaste na stroj pomocí ssh na uživatele "debian", např:
```bash
$ host 147.251.21.230
230.21.251.147.in-addr.arpa domain name pointer ip-147-251-21-230.flt.cloud.muni.cz.

$ ssh debian@ip-147-251-21-230.flt.cloud.muni.cz

```
## Příprava Ansible

Oklonujte si tuto gitovou repo: 
```bash
git clone https://github.com/martin-kuba/ansible-hands-on.git --recurse-submodules
cd ansible-hands-on
```

Vytvořte tzv. inventory - soubor `hosts` obsahující jméno ovládaného stroje, např.:
```bash
echo ip-147-251-21-230.flt.cloud.muni.cz >hosts
```

## První playbook

Prohlédněte si soubor playbook_1.yml a spusťte ho:
```bash
ansible-playbook playbook_1.yml
```

Ansible provede playbook ze souboru pro všechny stroje uvedené v inventory (soubor hosts specifikovaný v souboru ansible.cfg).
Výstup by měl vypadat nějak takto:

```
PLAY [Moje první hra] ***************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************************************************************************************
ok: [ip-147-251-21-230.flt.cloud.muni.cz]

TASK [zobraz nějaká fakta o stroji] *************************************************************************************************************************************************************************************
ok: [ip-147-251-21-230.flt.cloud.muni.cz] => {
    "msg": "operační systém je Debian 10 buster"
}

TASK [zobraz vybranou proměnnou] ****************************************************************************************************************************************************************************************
ok: [ip-147-251-21-230.flt.cloud.muni.cz] => {
    "ansible_facts.default_ipv4": {
        "address": "172.16.2.175",
        "alias": "eth0",
        "broadcast": "172.16.3.255",
        "gateway": "172.16.0.1",
        "interface": "eth0",
        "macaddress": "fa:16:3e:7a:83:c2",
        "mtu": 1442,
        "netmask": "255.255.252.0",
        "network": "172.16.0.0",
        "type": "ether"
    }
}

PLAY RECAP **************************************************************************************************************************************************************************************************************
ip-147-251-21-230.flt.cloud.muni.cz : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

## Úpravy stroje pro přímý přístup na roota

Prohlédněte si soubor playbook_2.yml a spusťte ho:  
```bash
ansible-playbook playbook_2.yml
```

Playbook obsahuje dvě hry, první povolí přímý přístup na roota, aby všechny tasky nemusely obsahovat "become: yes, become_user: root",
a pak druhá hra nastaví prostředí opračního systému do použitelného stavu.

## Instalace Apache

Prohlédněte si soubor playbook_3.yml a spusťte ho:
```bash
ansible-playbook playbook_3.yml
```

Nainstaluje Apache s TLS certifikáty od Let's Encrypt.

## Bonus - instalace Peruna

Na stroj lze nainstalovat Peruna pomocí playbooku v repo [https://github.com/CESNET/perun-ansible](https://github.com/CESNET/perun-ansible).

Jen je potřeba zase odinstalovat Apache jako službu, aby se nepral o TCP porty s Apachem v kontejneru:
```bash
apt purge apache2
```
