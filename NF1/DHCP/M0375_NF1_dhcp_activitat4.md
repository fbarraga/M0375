# Activitats d'ampliació DHCP i Ciberseguretat - M0375 ASIX

* *Autor:* Francesc Barragán
* *Darrera revisió:* 15.09.2025

## Criteris d'Avaluació

- **Comprensió teòrica (30%):** Coneixement de vulnerabilitats i contramedides
- **Implementació pràctica (40%):** Correcta configuració i execució d'activitats
- **Documentació (20%):** Qualitat dels informes i documentació tècnica
- **Anàlisi crítica (10%):** Capacitat d'avaluar riscos i proposar millores

## Activitat Ampliació 4: Auditoria de Seguretat DHCP

**Durada:** 4 hores  
**Objectiu:** Realitzar una auditoria completa de la seguretat DHCP en una xarxa simulada

### Escenari

Xarxa corporativa simulada amb:
- Múltiples VLANs
- Servidors DHCP per VLAN
- Equips de diferents tipus (PCs, portàtils, dispositius IoT)

### Tasques

1. **Fase de reconeixement (60 min)**
   - Mapeja la topologia de xarxa
   - Identifica tots els servidors DHCP actius
   - Documenta els pools d'adreces utilitzats

2. **Testing de vulnerabilitats (90 min)**
   - Executa eines com `nmap`, `dhcpig`, `DHCPig`
   - Prova atacs de spoofing i starvation
   - Verifica la configuració de seguretat existent

3. **Anàlisi de configuracions (60 min)**
   - Revisa configuracions de servidors DHCP
   - Comprova logs i esdeveniments de seguretat
   - Identifica configuracions insegures

4. **Elaboració d'informe (90 min)**
   - Classifica vulnerabilitats per criticitat
   - Proposa un pla de remediació prioritzat
   - Inclou recomanacions de millors pràctiques

**Entregable:** Informe d'auditoria professional amb pla de remediació

## Recursos Addicionals

### Eines Recomanades

- **Wireshark:** Anàlisi de tràfic DHCP
- **Nmap:** Descobriment de serveis DHCP
- **Yersinia:** Testing de vulnerabilitats
- **DHCPig:** Auditoria específica de DHCP
- **Scapy:** Creació de paquets personalitzats

### Bibliografia

- RFC 2131 - Dynamic Host Configuration Protocol
- RFC 3315 - Dynamic Host Configuration Protocol for IPv6
- NIST SP 800-117 - Guide to Adopting and Using the Security Content Automation Protocol



---

*Nota: Aquestes activitats s'han de realitzar en un entorn controlat de laboratori. Mai s'han d'executar atacs en xarxes de producció sense autorització explícita.*
