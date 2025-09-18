# Activitat: Gestió de DHCP amb PowerShell
## Mòdul: Serveis de xarxa i internet - 2n ASIX

### Objectius
- Aprendre a instal·lar el rol DHCP mitjançant PowerShell
- Gestionar àmbits DHCP amb comandes PowerShell
- Configurar opcions i reserves DHCP
- Monitoritzar i mantenir el servei DHCP

### Escenari
Sou l'administrador de sistemes d'una petita empresa que necessita implementar un servei DHCP per gestionar automàticament les adreces IP de la xarxa corporativa. Haureu d'utilitzar exclusivament PowerShell per completar totes les tasques.

**Dades de la xarxa:**
- Xarxa: 192.168.100.0/24
- Rang DHCP: 192.168.100.50 - 192.168.100.200
- Gateway: 192.168.100.1
- DNS primari: 8.8.8.8
- DNS secundari: 8.8.4.4
- Durada del lease: 8 dies

---

## PART 1: Instal·lació del servei DHCP

### Exercici 1.1
Utilitzeu PowerShell per verificar si el rol DHCP està instal·lat al servidor. Escriviu la comanda necessària.

**Comanda:** ________________________________

### Exercici 1.2
Instal·leu el rol DHCP Server utilitzant PowerShell. Incloeu les eines d'administració.

**Comanda:** ________________________________

### Exercici 1.3
Verificau que la instal·lació ha estat correcta consultant l'estat del servei DHCP.

**Comanda:** ________________________________

---

## PART 2: Configuració inicial del DHCP

### Exercici 2.1
Configureu el servidor DHCP perquè s'autoritzi automàticament en el domini. (Simularem que estem en un entorn de domini)

**Comanda:** ________________________________

### Exercici 2.2
Creeu un àmbit DHCP amb les següents característiques:
- Nom de l'àmbit: "Xarxa_Corporativa"
- Descripció: "Àmbit principal per a ordinadors de l'oficina"
- Xarxa: 192.168.100.0
- Màscara: 255.255.255.0
- Rang: 192.168.100.50 - 192.168.100.200

**Comanda:** ________________________________

### Exercici 2.3
Activeu l'àmbit que acabeu de crear.

**Comanda:** ________________________________

---

## PART 3: Configuració d'opcions DHCP

### Exercici 3.1
Configureu l'opció de gateway per defecte (router) per a l'àmbit creat.

**Comanda:** ________________________________

### Exercici 3.2
Configureu els servidors DNS per a l'àmbit.

**Comanda:** ________________________________

### Exercici 3.3
Establiu la durada del lease a 8 dies per a l'àmbit.

**Comanda:** ________________________________

---

## PART 4: Gestió avançada

### Exercici 4.1
Creeu una reserva DHCP per a un servidor d'impressió amb les següents dades:
- Nom: "PrintServer01"
- Adreça MAC: 00:15:5D:00:04:10
- Adreça IP: 192.168.100.10
- Descripció: "Servidor d'impressió principal"

**Comanda:** ________________________________

### Exercici 4.2
Creeu una exclusió per al rang 192.168.100.1 - 192.168.100.20 (reserves per a servidors).

**Comanda:** ________________________________

---

## PART 5: Monitoritzat i manteniment

### Exercici 5.1
Consulteu l'estat actual de l'àmbit DHCP i mostreu un resum de les estadístiques.

**Comanda:** ________________________________

### Exercici 5.2
Llisteu totes les concessions DHCP actives de l'àmbit.

**Comanda:** ________________________________

### Exercici 5.3
Exporteu la configuració completa del servidor DHCP a un fitxer XML per a còpia de seguretat.

**Comanda:** ________________________________

### Exercici 5.4
Reinicieu el servei DHCP utilitzant PowerShell.

**Comanda:** ________________________________

---

## PART 6: Resolució de problemes

### Exercici 6.1
Un usuari informa que no pot obtenir una adreça IP. Escriviu una comanda per verificar si hi ha adreces disponibles a l'àmbit.

**Comanda:** ________________________________

### Exercici 6.2
Forceu l'alliberament d'una concessió específica per a l'adreça IP 192.168.100.150.

**Comanda:** ________________________________

### Exercici 6.3
Comproveu els logs d'esdeveniments del servei DHCP per identificar possibles problemes.

**Comanda:** ________________________________

---

## Entrega
1. Documenteu totes les comandes utilitzades en cada exercici
2. Feu captures de pantalla dels resultats més rellevants
3. Expliqueu breument què fa cada comanda i per què és necessària
4. Proposeu millores adicionals per a la configuració DHCP

### Criteris d'avaluació
- **Correctesa tècnica (40%)**: Les comandes són correctes i funcionals
- **Documentació (30%)**: Explicacions clares i captures adequades  
- **Comprensió (20%)**: Demostració d'entendre el funcionament del DHCP
- **Millores proposades (10%)**: Creativitat en les propostes d'optimització

**Data límit d'entrega:** [Inserir data]