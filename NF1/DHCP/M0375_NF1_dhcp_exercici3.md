## EXERCICI 3 -- DHCP a Ubuntu Server amb Kea DHCP

* **Autor:** Francesc Barragán

* **Darrera Actualització:** 15.09.2025

### OBJECTIU

**Instal·lar i gestionar per consola el servei de DHCP amb Kea DHCP a Ubuntu Server i associar-ho als conceptes treballats prèviament.**

### INSTRUCCIONS

**Utilitzarem la màquina virtual Ubuntu Server 24 des de 0.**

Tal com hem fet a l'exercici 1, en aquest segon exercici utilitzarem un tutorial introductori per a la instal·lació i configuració bàsica del servei DHCP, en aquest cas, sota Ubuntu Server 24 LTS utilitzant Kea DHCP.

El servei de DHCP el donarem amb el software Kea DHCP, que és la nova solució de codi obert desenvolupada per Internet Systems Consortium (ISC) per implementar servidors DHCP moderns. Kea DHCP és el successor d'ISC DHCP, oferint una arquitectura més modular, millor rendiment i suport natiu per a funcions modernes com APIs REST, hooks dinàmics i configuració JSON.

Kea suporta tant IPv4 com IPv6 de manera nativa, és adequat per al seu ús en aplicacions de gran volum i alta fiabilitat, i està activament desenvolupat i mantingut, a diferència d'ISC DHCP que va anunciar el final del seu manteniment a finals del 2022.

### Preparació de la xarxa

El primer que caldrà fer és associar una IP fixa a la targeta de xarxa, utilitzant el rang preassignat que teniu. Recordeu que des de Ubuntu 21 la configuració de xarxa es gestiona amb **netplan**.

Teniu diferents exemples a: https://netplan.io/examples/

Exemple de configuració `/etc/netplan/00-installer-config.yaml`:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Apliqueu la configuració:
```bash
sudo netplan apply
```

### Instal·lació de Kea DHCP

Considerant que la xarxa està configurada correctament, ja podem passar a instal·lar el servei de DHCP amb Kea:

```bash
sudo apt update
sudo apt install kea-dhcp4-server kea-admin kea-common
```

**Nota**: Si necessiteu IPv6, també podeu instal·lar `kea-dhcp6-server`.

### Configuració de Kea DHCP

Un cop instal·lat, el fitxer de configuració principal de Kea DHCP4 el trobareu a `/etc/kea/kea-dhcp4.conf`.

A diferència d'ISC DHCP, Kea utilitza format JSON per a la seva configuració, el que fa més estructurada i fàcil de validar.

#### Estructura del fitxer de configuració

El fitxer de configuració té aquesta estructura bàsica:

```json
{
    "Dhcp4": {
        "interfaces-config": {
            "interfaces": [ "enp0s3" ]
        },
        "lease-database": {
            "type": "memfile",
            "persist": true,
            "name": "/var/lib/kea/dhcp4.leases"
        },
        "valid-lifetime": 86400,
        "subnet4": [
            {
                "subnet": "192.168.1.0/24",
                "pools": [
                    {
                        "pool": "192.168.1.20 - 192.168.1.100"
                    }
                ],
                "option-data": [
                    {
                        "name": "routers",
                        "data": "192.168.1.1"
                    },
                    {
                        "name": "domain-name-servers",
                        "data": "8.8.8.8, 8.8.4.4"
                    }
                ]
            }
        ],
        "loggers": [
            {
                "name": "kea-dhcp4",
                "output_options": [
                    {
                        "output": "/var/log/kea-dhcp4.log"
                    }
                ],
                "severity": "INFO"
            }
        ]
    }
}
```

#### Paràmetres principals de configuració

**Configuració d'interfícies:**
- `interfaces-config`: Especifica les interfícies on Kea escoltarà peticions DHCP

**Base de dades de concessions:**
- `lease-database`: Configuració de l'emmagatzematge de concessions (memfile, mysql, postgresql)

**Configuració de subxarxa:**
- `subnet4`: Array de subxarxes IPv4 que Kea gestionarà
- `pools`: Rangs d'IPs disponibles per assignar
- `option-data`: Opcions DHCP com gateway, DNS, etc.

**Logging:**
- `loggers`: Configuració dels logs del servei

### Configuració detallada

Editeu el fitxer de configuració:

```bash
sudo nano /etc/kea/kea-dhcp4.conf
```

Exemple de configuració completa:

```json
{
    "Dhcp4": {
        "interfaces-config": {
            "interfaces": [ "enp0s3" ]
        },
        "lease-database": {
            "type": "memfile",
            "persist": true,
            "name": "/var/lib/kea/dhcp4.leases"
        },
        "valid-lifetime": 86400,
        "renew-timer": 43200,
        "rebind-timer": 75600,
        "subnet4": [
            {
                "subnet": "192.168.1.0/24",
                "pools": [
                    {
                        "pool": "192.168.1.20 - 192.168.1.100"
                    }
                ],
                "option-data": [
                    {
                        "name": "routers",
                        "data": "192.168.1.1"
                    },
                    {
                        "name": "domain-name-servers",
                        "data": "8.8.8.8, 8.8.4.4"
                    },
                    {
                        "name": "domain-name",
                        "data": "lab.local"
                    }
                ],
                "reservations": [
                    {
                        "hw-address": "aa:bb:cc:dd:ee:ff",
                        "ip-address": "192.168.1.10",
                        "hostname": "servidor-lab"
                    }
                ]
            }
        ],
        "loggers": [
            {
                "name": "kea-dhcp4",
                "output_options": [
                    {
                        "output": "/var/log/kea-dhcp4.log"
                    }
                ],
                "severity": "INFO"
            }
        ]
    }
}
```

### Gestió del servei

Kea DHCP s'instal·la com a servei systemd, de manera que quan el configureu correctament i reinicieu ja quedarà activat:

```bash
# Verificar configuració
sudo kea-dhcp4 -t

# Iniciar el servei
sudo systemctl start kea-dhcp4-server

# Habilitar inici automàtic
sudo systemctl enable kea-dhcp4-server

# Verificar estat del servei
sudo systemctl status kea-dhcp4-server

# Reiniciar després de canvis
sudo systemctl restart kea-dhcp4-server
```

### Verificació i monitoritzat

#### Verificar que el servei estigui escoltant:

```bash
sudo netstat -putan | grep kea
```

#### Consultar logs:

```bash
# Logs del sistema
sudo journalctl -u kea-dhcp4-server -f

# Log específic de Kea
sudo tail -f /var/log/kea-dhcp4.log
```

#### Verificar concessions actives:

```bash
# Veure fitxer de concessions
sudo cat /var/lib/kea/dhcp4.leases

# O amb format JSON més llegible
sudo python3 -m json.tool /var/lib/kea/dhcp4.leases
```

### Avantatges de Kea respecte ISC DHCP

1. **Configuració JSON**: Format més estructurat i fàcil de validar
2. **API REST**: Permet gestió remota i integració amb altres sistemes
3. **Hooks dinàmics**: Extensibilitat mitjançant plugins
4. **Millor rendiment**: Arquitectura més eficient
5. **Suport actiu**: Desenvolupament i manteniment continu
6. **Flexibilitat**: Configuració més granular i opcions avançades

### Recursos addicionals

- Documentació oficial de Kea: https://kea.readthedocs.io/
- Manual d'administrador: https://kea.readthedocs.io/en/latest/arm/admin.html
- Exemples de configuració: https://kea.readthedocs.io/en/latest/examples.html
- API REST: https://kea.readthedocs.io/en/latest/arm/agent.html

### Exercicis pràctics

1. **Configuració bàsica**: Implementeu la configuració mostrada i verifiqueu que un client obté IP automàticament

2. **Reservacions**: Afegiu una reservació per a un client específic utilitzant la seva MAC

3. **Múltiples subxarxes**: Configureu Kea per gestionar dues subxarxes diferents

4. **Logging avançat**: Configureu diferents nivells de log i analitzeu els missatges

5. **Validació**: Utilitzeu l'opció `-t` per validar diferents configuracions errònies

### Notes importants

⚠️ **Validació de configuració**: Sempre utilitzeu `sudo kea-dhcp4 -t` abans de reiniciar el servei per evitar errors

⚠️ **Format JSON**: La configuració ha de ser JSON vàlid. Utilitzeu eines com `jsonlint` per validar-la

⚠️ **Permisos**: Assegureu-vos que Kea tingui permisos d'escriptura al directori de concessions

⚠️ **Firewall**: Si teniu firewall activat, assegureu-vos que els ports 67/68 UDP estiguin oberts