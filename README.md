# NetBox Ansible Lab

## Demo

🎬 [Se demo på Streamable](https://streamable.com/pcja5z)

Det här är första gången jag jobbar med NetBox. Jag hade aldrig använt det innan det här projektet.
Jag byggde det här labbet med hjälp av [Claude (claude.ai)](https://claude.ai) — en AI-assistent som hjälpte mig förstå felmeddelanden, fixa playbooks och lära mig hur allt hänger ihop.

---

## Vad projektet gör

Det här projektet använder Ansible för att automatiskt:
1. Installera NetBox på en Ubuntu VM med Docker
2. Skapa en labmiljö med en switch och tre servrar i NetBox
3. Lägga till nätverksinterface och kabelkopplingar mellan servrarna och switchen

Allt är automatiserat — ingen manuell klickning i gränssnittet behövs.

---

## Labbtopologi

```
SVR-01  eth0  ──►  SW-01  GigabitEthernet0/1
SVR-01  eth1  ──►  SW-01  GigabitEthernet0/2
SVR-01  ipmi0 ──►  SW-01  GigabitEthernet0/3

SVR-02  eth0  ──►  SW-01  GigabitEthernet0/4
SVR-02  eth1  ──►  SW-01  GigabitEthernet0/5
SVR-02  ipmi0 ──►  SW-01  GigabitEthernet0/6

SVR-03  eth0  ──►  SW-01  GigabitEthernet0/7
SVR-03  eth1  ──►  SW-01  GigabitEthernet0/8
SVR-03  ipmi0 ──►  SW-01  GigabitEthernet0/9
```

---

## Filer

| Fil | Beskrivning |
|-----|-------------|
| `01_install.yml` | Installerar Docker och startar NetBox på VM:en |
| `02_bootstrap.yml` | Skapar site, rack, switch och servrar i NetBox |
| `connections.yml` | Definierar vilken serverport som går till vilken switchport |
| `03_cables.yml` | Skapar interface och kablar mellan servrar och switch |
| `inventory` | Ansible inventory-fil med anslutningsuppgifter till VM:en |

---

## Så här kör du det

### Steg 1 — Installera NetBox
```bash
ansible-playbook -i inventory 01_install.yml
```
Vänta 5–10 minuter. Du ska se:
```
✓ NetBox is running at http://192.168.90.160:8000
```

### Steg 2 — Lägg in labbdata
```bash
ansible-playbook -i inventory 02_bootstrap.yml
```
Du ska se:
```
✓ NetBox bootstrap complete
Token: <din-api-token>
```
Spara token — du behöver den i nästa steg.

### Steg 3 — Skapa connections.yml
Skapa filen som beskriver kabeldragningen:
```bash
nano connections.yml
```
Klistra in:
```yaml
connections:
  - server: SVR-01
    interfaces:
      - { name: eth0,  switch_port: "GigabitEthernet0/1" }
      - { name: eth1,  switch_port: "GigabitEthernet0/2" }
      - { name: ipmi0, switch_port: "GigabitEthernet0/3", type: ipmi }
  - server: SVR-02
    interfaces:
      - { name: eth0,  switch_port: "GigabitEthernet0/4" }
      - { name: eth1,  switch_port: "GigabitEthernet0/5" }
      - { name: ipmi0, switch_port: "GigabitEthernet0/6", type: ipmi }
  - server: SVR-03
    interfaces:
      - { name: eth0,  switch_port: "GigabitEthernet0/7" }
      - { name: eth1,  switch_port: "GigabitEthernet0/8" }
      - { name: ipmi0, switch_port: "GigabitEthernet0/9", type: ipmi }
```
Spara med `Ctrl+O` → Enter → `Ctrl+X`.

### Steg 4 — Lägg till interface och kablar
Klistra in din token i `03_cables.yml` under `nb_token`, kör sedan:
```bash
ansible-playbook -i inventory 03_cables.yml
```

Om du får det här felet:
```
InvalidVersion: Invalid version: '4.2.8-Docker-3.2.0'
```
Lös det genom att uppgradera Ansible-kollektionen:
```bash
ansible-galaxy collection install netbox.netbox --upgrade
```
Kör sedan playbooken igen.

---

## Anpassa kabeldragningen

Redigera `connections.yml` för att matcha din verkliga kabeldragning. Det är den enda filen du behöver ändra när du lägger till nya servrar eller byter portar.

---

## Vad jag lärde mig

- Vad NetBox är och hur det fungerar som ett verktyg för nätverksdokumentation
- Hur Ansible playbooks är uppbyggda och hur man kör dem
- Hur man använder `netbox.netbox` Ansible-kollektionen för att hantera enheter, interface och kablar via API
- Hur man läser och åtgärdar felmeddelanden från Ansible
- Hur Docker används för att köra NetBox i containers

---

## Verktyg som användes

- [NetBox](https://github.com/netbox-community/netbox) — Verktyg för nätverksdokumentation och DCIM
- [Ansible](https://www.ansible.com/) — Automatiseringsverktyg
- [Docker](https://www.docker.com/) — Container-runtime
- [Claude (claude.ai)](https://claude.ai) — AI-assistent som hjälpte mig genom installationen
