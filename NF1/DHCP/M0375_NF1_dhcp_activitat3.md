# Activitats d'ampliació DHCP (M0375 ASIX2)

## Criteris d'Avaluació

- **Comprensió teòrica (30%):** Coneixement de vulnerabilitats i contramedides
- **Implementació pràctica (40%):** Correcta configuració i execució d'activitats
- **Documentació (20%):** Qualitat dels informes i documentació tècnica
- **Anàlisi crítica (10%):** Capacitat d'avaluar riscos i proposar millores

## Activitat Ampliació 3: Configuració de DHCP Snooping

**Durada:** 2.5 hores  
**Objectiu:** Implementar i configurar DHCP Snooping com a mesura de seguretat

### Equipament

- Switch gestionable (Cisco o similar)
- Servidor DHCP legítim
- Client de test
- Servidor DHCP maliciós (simulat)

### Tasques

1. **Escenari sense protecció (30 min)**
   - Configura dos servidors DHCP a la mateixa xarxa
   - Observa el comportament dels clients
   - Documenta els problemes que sorgeixen

2. **Configuració de DHCP Snooping (60 min)**
   - Activa DHCP Snooping al switch
   - Configura ports trusted i untrusted
   - Estableix límits de velocitat per als missatges DHCP

3. **Testing i validació (45 min)**
   - Verifica que només el servidor legítim pot respondre
   - Intenta atacs des de ports untrusted
   - Comprova els logs de seguretat del switch

4. **Configuració avançada (35 min)**
   - Implementa Option 82 (DHCP Relay Agent Information)
   - Configura binding database per a persistència
   - Estableix polítiques de recuperació

**Entregable:** Configuració documentada del switch i informe de proves

---

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
