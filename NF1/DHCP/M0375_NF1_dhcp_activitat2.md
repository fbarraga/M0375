# Activitats d'ampliació DHCP i Ciberseguretat - M0375 ASIX

## Criteris d'Avaluació

- **Comprensió teòrica (30%):** Coneixement de vulnerabilitats i contramedides
- **Implementació pràctica (40%):** Correcta configuració i execució d'activitats
- **Documentació (20%):** Qualitat dels informes i documentació tècnica
- **Anàlisi crítica (10%):** Capacitat d'avaluar riscos i proposar millores

## Activitat Ampliació 2: Implementació d'un Atac DHCP Starvation

**Durada:** 3 hores  
**Objectiu:** Comprendre com funciona l'atac de DHCP Starvation i les seves conseqüències

### Escenari

Xarxa de laboratori amb:

- 1 servidor DHCP (pool de 20 IPs)
- 2 clients Windows/Linux
- 1 màquina atacant (Kali Linux)

### Tasques

1. **Configuració de l'entorn (45 min)**
   - Configura el servidor DHCP amb un pool limitat d'adreces
   - Verifica que els clients obtenen IP correctament
   - Documenta l'estat inicial de la xarxa

2. **Execució de l'atac (60 min)**
   - Utilitza eines com `yersinia` o `dhcpstarv` per exhaurir el pool DHCP
   - Monitoritza el comportament del servidor DHCP
   - Intenta connectar nous clients durant l'atac

3. **Anàlisi de l'impacte (45 min)**
   - Documenta com afecta l'atac als nous clients
   - Analitza els logs del servidor DHCP
   - Mesura el temps de recuperació després de l'atac

4. **Implementació de contramedides (30 min)**
   - Configura límits de concessió per MAC
   - Implementa monitoratge d'assignacions anòmales
   - Testa l'eficàcia de les mesures

**Entregable:** Informe pràctic amb captures, logs i avaluació de contramedides

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