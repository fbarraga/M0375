# Activitats d'ampliació DHCP i Ciberseguretat - M0375 ASIX

* *Autor:* Francesc Barragán
* *Darrera revisió:* 15.09.2025

## Criteris d'Avaluació

- **Comprensió teòrica (30%):** Coneixement de vulnerabilitats i contramedides
- **Implementació pràctica (40%):** Correcta configuració i execució d'activitats
- **Documentació (20%):** Qualitat dels informes i documentació tècnica
- **Anàlisi crítica (10%):** Capacitat d'avaluar riscos i proposar millores

## Activitat Ampliació 6: Monitoratge i Detecció d'Anomalies DHCP

**Durada:** 2.5 hores  
**Objectiu:** Implementar un sistema de monitoratge per detectar activitat maliciosa relacionada amb DHCP

### Tasques

1. **Configuració de logging (30 min)**
   - Activa logs detallats al servidor DHCP
   - Configura enviament de logs a un servidor centralitzat (syslog)
   - Estableix rotació i retenció de logs

2. **Creació de scripts de monitoratge (60 min)**
   - Desenvolupa scripts per detectar:
     - Múltiples sol·licituds des de la mateixa MAC
     - Assignacions anòmalament ràpides
     - Requests des de MACs desconegudes
   - Implementa alertes automàtiques

3. **Configuració de SIEM bàsic (45 min)**
   - Utilitza eines com ELK Stack o Splunk (versió gratuïta)
   - Crea dashboards per visualitzar activitat DHCP
   - Configura alertes per a patrons sospitosos

4. **Testing del sistema (35 min)**
   - Simula diferents tipus d'atacs
   - Verifica que les alertes funcionen correctament
   - Ajusta la sensibilitat per reduir falsos positius

**Entregable:** Sistema de monitoratge funcional amb documentació de configuració

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
