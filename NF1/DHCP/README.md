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

- Adreça IP fixa assignada a una MAC específica
- Sempre la mateixa IP per al mateix dispositiu
- Ideal per a servidors i equips crítics

### 3. **Automàtica**

- Assignació permanent la primera vegada
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

* [Exercici 4: Configuració d'un DHCP en un dispositiu CISCO](./M0375_NF1_dhcp_exercici4.md)

## Activitats DHCP Avançades

* [Activitat Ampliació 1: Anàlisi de Vulnerabilitats DHCP](./M0375_NF1_dhcp_activitat1.md)
* [Activitat Ampliació 2: Implementació d'un Atac DHCP Starvation](./M0375_NF1_dhcp_activitat2.md)
* [Activitat Ampliació 3: Configuració de DHCP Snooping](./M0375_NF1_dhcp_activitat3.md)
* [Activitat Ampliació 4: Auditoria de Seguretat DHCP](./M0375_NF1_dhcp_activitat4.md)
* [Activitat Ampliació 5: Implementació de DHCPv6 Segur](./M0375_NF1_dhcp_activitat5.md)
* [Activitat Ampliació 6: Monitoratge i Detecció d'Anomalies DHCP](./M0375_NF1_dhcp_activitat6.md)

## Informació rellevant

* [Evolució del DHCP a Linux amb la discontinuitat de ISC](./M0375_NF1_dhcp_linux_futur.md)
  
