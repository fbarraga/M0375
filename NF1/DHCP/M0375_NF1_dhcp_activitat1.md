# Activitats d'ampliació DHCP i Ciberseguretat - M0375 ASIX

## Criteris d'Avaluació

- **Comprensió teòrica (30%):** Coneixement de vulnerabilitats i contramedides
- **Implementació pràctica (40%):** Correcta configuració i execució d'activitats
- **Documentació (20%):** Qualitat dels informes i documentació tècnica
- **Anàlisi crítica (10%):** Capacitat d'avaluar riscos i proposar millores

## Activitat Ampliació 1: Anàlisi de Vulnerabilitats DHCP

**Durada:** 2 hores  
**Objectiu:** Identificar i comprendre les principals vulnerabilitats del protocol DHCP

### Tasques

1. **Investigació teòrica (30 min)**
   - Investiga els següents atacs DHCP:
     - DHCP Starvation
     - DHCP Spoofing
     - Rogue DHCP Server
     - DHCP Snooping Bypass

2. **Anàlisi de captures de tràfic (45 min)**
   - Utilitza Wireshark per analitzar captures de tràfic DHCP normal
   - Identifica els diferents tipus de missatges DHCP (DISCOVER, OFFER, REQUEST, ACK)
   - Documenta quina informació sensible es transmet sense xifrar

3. **Documentació de riscos (45 min)**
   - Crea una taula amb vulnerabilitats, impacte i probabilitat
   - Proposa mesures de mitigació per a cada vulnerabilitat identificada

**Entregable:** Informe de vulnerabilitats amb captures de pantalla i propostes de millora

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
