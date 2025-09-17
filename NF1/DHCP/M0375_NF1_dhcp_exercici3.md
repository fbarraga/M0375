# Exercici 4: Configuració de DHCP en Switch Cisco Catalyst 2xxx

**Autor:** Francesc Barragán

**Darrera Actualització:** 15.09.2025

## Objectius de la pràctica

En finalitzar aquesta pràctica, l'estudiant serà capaç de:
- Establir connexió per consola amb un switch Cisco
- Configurar una adreça IP de gestió
- Crear i configurar VLANs
- Implementar un servidor DHCP al switch
- Verificar el funcionament del servei DHCP

## Escenari

Disposes d'un switch Cisco Catalyst 2xxx gestionable que necessites configurar per proporcionar adreces IP automàticament als dispositius de la xarxa. El switch ha d'actuar com a servidor DHCP per a una VLAN específica.

## Requisits previs

- Coneixements bàsics de CLI de Cisco IOS
- Comprensió de conceptes de VLAN i DHCP
- Cable de consola RJ45 a DB9 o USB
- Programari de terminal (PuTTY, Tera Term, etc.)

## Material necessari

- 1 Switch Cisco Catalyst 2xxx amb IOS que suporti DHCP
- 1 Cable de consola
- 1 PC amb emulador de terminal
- 2-3 PCs client per a proves

## Tasques a realitzar

### Fase 1: Connexió inicial
1. Connecta el cable de consola al switch i al PC
2. Configura el programari de terminal amb els paràmetres correctes
3. Accedeix al mode de configuració del switch
4. Etiqueta el switch amb una Dymo indicant el vostre grup
5. Emplena el document excel amb les dades de l'equip que estàs utilitzant

### Fase 2: Configuració bàsica
4. Configura un hostname descriptiu per al switch
5. Estableix contrasenyes per a l'accés per consola i enable
6. Configura una adreça IP de gestió a la VLAN per defecte

### Fase 3: Configuració de VLAN
7. Crea la VLAN 100 amb el nom "DHCP_VLAN"
8. Configura la interfície VLAN 100 amb l'adreça IP "Segons el vostre pool de ips"
9. Assigna almenys un port del switch a la VLAN 100

### Fase 4: Configuració del servidor DHCP
10. Crea un pool DHCP anomenat "VLAN100_POOL"
11. Defineix el rang d'adreces: "Segons el vostre pool de ips"
12. Configura la porta d'enllaç per defecte: x.x.x.1
13. Estableix els servidors DNS: 8.8.8.8 i 8.8.4.4
14. Configura un temps de concessió d'1 dia
15. Exclou les adreces x.x.x.1 - x.x.x.9

### Fase 5: Verificació
16. Connecta un PC client al port assignat a la VLAN 100
17. Verifica que el client obté una adreça IP automàticament
18. Comprova la connectivitat i configuració de xarxa del client
19. Visualitza les concessions DHCP actives al switch

## Lliurables

1. **Documentació de configuració**: Captures de pantalla o text de totes les ordres utilitzades
2. **Verificacions**: Evidències del funcionament correcte del DHCP (ipconfig, ping, etc.)
3. **Anàlisi**: Breu informe explicant possibles problemes trobats i com es van resoldre

## Criteris d'avaluació

- **Connexió per consola (10%)**: Establiment correcte de la sessió de terminal
- **Configuració bàsica (20%)**: Hostname, contrasenyes i IP de gestió
- **Configuració VLAN (25%)**: Creació i configuració correcta de la VLAN 100
- **Servidor DHCP (35%)**: Configuració completa i funcional del pool DHCP
- **Verificació (10%)**: Proves i documentació del funcionament

## Notes importants

⚠️ **Atenció**: No tots els switches Catalyst 2xxx suporten funció de servidor DHCP. Verifica que el teu model tingui aquesta característica habilitada.

⚠️ **Compatibilitat**: Alguns models antics poden requerir una versió específica d'IOS per suportar DHCP server.

## Temps estimat

- Preparació i connexió: 15 minuts
- Configuració bàsica: 20 minuts  
- Configuració VLAN i DHCP: 30 minuts
- Proves i verificació: 15 minuts
- **Total**: 80 minuts
