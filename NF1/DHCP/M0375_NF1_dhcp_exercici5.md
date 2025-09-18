# Exercici 5: DHCP Relay i Snooping amb Cisco 2960

## Objectius adaptats

- Configurar DHCP Snooping per seguretat
- Implementar DHCP Relay cap a servidor extern
- Crear infraestructura VLAN per DHCP
- Entendre limitacions dels switches de capa 2

## Escenari adaptat

Com que el Cisco 2960 no suporta servidor DHCP natiu, configurarem:

1. **Switch 2960**: DHCP Relay i Snooping
2. **Servidor extern**: Ubuntu amb Kea DHCP (de l'exercici anterior)

## Fase 1: Configuració bàsica (igual que abans)

### Connexió per consola i configuració inicial

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-RELAY-LAB
SW-RELAY-LAB(config)# enable secret cisco123
SW-RELAY-LAB(config)# line console 0
SW-RELAY-LAB(config-line)# password console123
SW-RELAY-LAB(config-line)# login
SW-RELAY-LAB(config-line)# exit
```

## Fase 2: Configuració de VLANs

### Crear VLANs per clients i servidor

```cisco
# VLAN per clients
SW-RELAY-LAB(config)# vlan 100
SW-RELAY-LAB(config-vlan)# name CLIENTS_VLAN
SW-RELAY-LAB(config-vlan)# exit

# VLAN per servidor DHCP
SW-RELAY-LAB(config)# vlan 200
SW-RELAY-LAB(config-vlan)# name SERVER_VLAN
SW-RELAY-LAB(config-vlan)# exit

# Configurar interfícies VLAN
SW-RELAY-LAB(config)# interface vlan 100
SW-RELAY-LAB(config-if)# ip address 192.168.100.1 255.255.255.0
SW-RELAY-LAB(config-if)# no shutdown
SW-RELAY-LAB(config-if)# exit

SW-RELAY-LAB(config)# interface vlan 200
SW-RELAY-LAB(config-if)# ip address 192.168.200.1 255.255.255.0
SW-RELAY-LAB(config-if)# no shutdown
SW-RELAY-LAB(config-if)# exit
```

## Fase 3: Configuració DHCP Relay

### Configurar IP Helper Address
```cisco
# A la VLAN dels clients, redirigir peticions DHCP
SW-RELAY-LAB(config)# interface vlan 100
SW-RELAY-LAB(config-if)# ip helper-address 192.168.200.10
SW-RELAY-LAB(config-if)# exit

# Habilitar routing entre VLANs (si el model ho suporta)
SW-RELAY-LAB(config)# ip routing
```

## Fase 4: Configuració DHCP Snooping

### Habilitar DHCP Snooping globalment
```cisco
SW-RELAY-LAB(config)# ip dhcp snooping
SW-RELAY-LAB(config)# ip dhcp snooping vlan 100

# Configurar ports trusted (on està el servidor DHCP)
SW-RELAY-LAB(config)# interface fastethernet 0/24
SW-RELAY-LAB(config-if)# ip dhcp snooping trust
SW-RELAY-LAB(config-if)# switchport access vlan 200
SW-RELAY-LAB(config-if)# exit

# Configurar ports d'accés per clients (untrusted per defecte)
SW-RELAY-LAB(config)# interface range fastethernet 0/1-12
SW-RELAY-LAB(config-if-range)# switchport mode access
SW-RELAY-LAB(config-if-range)# switchport access vlan 100
SW-RELAY-LAB(config-if-range)# exit
```

## Fase 5: Verificació

### Comandos de verificació
```cisco
# Verificar DHCP Snooping
SW-RELAY-LAB# show ip dhcp snooping
SW-RELAY-LAB# show ip dhcp snooping binding

# Verificar configuració de VLANs
SW-RELAY-LAB# show vlan brief
SW-RELAY-LAB# show interface vlan 100

# Verificar routing (si està habilitat)
SW-RELAY-LAB# show ip route
```

## Avantatges d'aquesta pràctica

✅ **Realista**: Reflecteix l'ús real dels 2960 en xarxes empresarials
✅ **Seguretat**: Aprenen DHCP Snooping (molt important)
✅ **Infraestructura**: Entenen com separar clients i servidors
✅ **Limitacions**: Comprenen les capacitats dels diferents equips

## Conclusió

Aquesta pràctica és **més realista** que intentar fer servidor DHCP al 2960, ja que:
- Els 2960 s'usen com a switches d'accés, no com a servidors
- DHCP Snooping és una funcionalitat essencial en entorns empresarials
- DHCP Relay és la manera habitual de gestionar DHCP en xarxes segmentades