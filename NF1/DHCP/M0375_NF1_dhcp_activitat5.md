# Activitats d'ampliació DHCP i Ciberseguretat - M0375 ASIX

* *Autor:* Francesc Barragán
* *Darrera revisió:* 15.09.2025

## Criteris d'Avaluació

- **Comprensió teòrica (30%):** Coneixement de vulnerabilitats i contramedides
- **Implementació pràctica (40%):** Correcta configuració i execució d'activitats
- **Documentació (20%):** Qualitat dels informes i documentació tècnica
- **Anàlisi crítica (10%):** Capacitat d'avaluar riscos i proposar millores

## Activitat Ampliació 5: Implementació de DHCPv6 Segur

**Durada:** 3 hores  
**Objectiu:** Configurar i securitzar DHCPv6 considerant les seves particularitats de seguretat

### Tasques

1. **Configuració bàsica DHCPv6 (45 min)**
   - Configura un servidor DHCPv6 (ISC DHCP o Windows Server)
   - Estableix pools d'adreces IPv6
   - Configura opcions bàsiques (DNS, domini)

2. **Anàlisi de diferències de seguretat (60 min)**
   - Compara vulnerabilitats DHCPv4 vs DHCPv6
   - Investiga atacs específics de DHCPv6 (RA flooding, fake DHCPv6)
   - Documenta les diferències en el model de seguretat

3. **Configuració de seguretat avançada (60 min)**
   - Implementa autenticació DHCP (si disponible)
   - Configura DHCPv6 Guard
   - Estableix polítiques de RA Guard

4. **Testing i validació (75 min)**
   - Prova el funcionament amb clients IPv6
   - Intenta atacs específics de DHCPv6
   - Verifica l'eficàcia de les mesures de seguretat

**Entregable:** Configuració segura de DHCPv6 i comparativa de seguretat

---

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
