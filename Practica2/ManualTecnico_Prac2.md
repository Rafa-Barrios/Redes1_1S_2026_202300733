# PRÁCTICA 2 — Red del Aeropuerto Internacional La Aurora

---

**Institución:** Universidad San Carlos de Guatemala  
**Facultad:** Ingeniería en Ciencias y Sistemas  
**Curso:** Redes de Computadoras 1  
**Autor:** Ángel Rafael Barrios González  
**Carnet:** 202300733  
**Fecha:** 09 de abril de 2026  

---

## Índice

1. [Introducción](#introducción)
2. [Topología General](#topología-general)
3. [Topología por Área](#topología-por-área)
4. [Tabla de VLANs](#tabla-de-vlans)
5. [Tabla de Dominios de Colisión](#tabla-de-dominios-de-colisión)
6. [Tabla de Dominios de Broadcast](#tabla-de-dominios-de-broadcast)
7. [Jerarquía de Medios e Interfaces](#jerarquía-de-medios-e-interfaces)
8. [Tabla de Subnetting VLSM](#tabla-de-subnetting-vlsm)
9. [Tabla de Direccionamiento IP](#tabla-de-direccionamiento-ip)
10. [Configuración VTP](#configuración-vtp)
11. [Configuración STP](#configuración-stp)
12. [Configuración EtherChannel](#configuración-etherchannel)
13. [Configuración de Trunks](#configuración-de-trunks)
14. [Comandos por Dispositivo](#comandos-por-dispositivo)
15. [Pruebas de Conectividad](#pruebas-de-conectividad)
16. [Presupuesto Estimado](#presupuesto-estimado)
17. [Conclusión](#conclusión)

---

## Introducción

El presente documento corresponde al diseño e implementación de la infraestructura de red del **Aeropuerto Internacional La Aurora**, desarrollado como parte del curso de Redes de Computadoras 1 de la Universidad San Carlos de Guatemala.

El aeropuerto opera con una arquitectura distribuida en **8 áreas operativas reales**, interconectadas mediante switches Cisco 2960-24TT con cableado estructurado de cobre UTP. El objetivo principal es modernizar la infraestructura de red implementando tecnologías de capa 2 del modelo OSI, incluyendo:

- **VLANs** para segmentación lógica por área operativa
- **VTP** para distribución centralizada de VLANs con dominio 202300733
- **STP (Rapid-PVST+)** para redundancia y prevención de bucles
- **EtherChannel (LACP)** para agregación de 3 enlaces entre switches principales en áreas críticas
- **VLSM** para subnetting eficiente ajustado a la cantidad real de hosts por área
- **VLAN Nativa (99)** para tráfico de administración entre switches
- **VLAN Blackhole (999)** para deshabilitar puertos no utilizados

La implementación se realizó en **Cisco Packet Tracer**, utilizando únicamente switches **Cisco 2960-24TT** y dispositivos finales (PCs y Laptops).

---

## Topología General

![Topología Completa](img/topologia_general.png)

La topología conecta las 8 áreas operativas del aeropuerto mediante sus switches principales, formando un backbone de capa 2. Las áreas críticas (Terminal de Pasajeros, Torre de Control y Salas de Abordaje) cuentan con conexiones redundantes mediante EtherChannel LACP de 3 enlaces físicos.

```
SW-TP-P ══[LACP 3 enlaces]══ SW-TC-P
   ║                              ║
[LACP 3 enlaces]          [LACP 3 enlaces]
   ║                              ║
SW-SA-P ────────────────── SW-MG-P ──── SW-SG-P
   │                                        │
SW-CB-P ──── SW-AD-P                    SW-AA-P
```



**Conexiones entre switches principales:**

| Enlace | Tipo | Cables físicos | Puertos origen | Puertos destino |
|--------|------|---------------|----------------|-----------------|
| SW-TP-P ↔ SW-TC-P | EtherChannel LACP | 3 | FA0/5, FA0/6, FA0/7 | FA0/5, FA0/6, FA0/7 |
| SW-TP-P ↔ SW-SA-P | EtherChannel LACP | 3 | FA0/8, FA0/9, FA0/10 | FA0/5, FA0/6, FA0/7 |
| SW-TC-P ↔ SW-MG-P | EtherChannel LACP | 3 | FA0/8, FA0/9, FA0/10 | FA0/5, FA0/6, FA0/7 |
| SW-SA-P ↔ SW-CB-P | Trunk simple | 1 | FA0/8 | FA0/5 |
| SW-CB-P ↔ SW-AD-P | Trunk simple | 1 | FA0/6 | FA0/5 |
| SW-MG-P ↔ SW-SG-P | Trunk simple | 1 | FA0/8 | FA0/5 |
| SW-SG-P ↔ SW-AA-P | Trunk simple | 1 | FA0/6 | FA0/5 |

---

## Topología por Área

### Área 1 — Terminal de Pasajeros (VLAN 13)

![Área 1 - Terminal de Pasajeros](img/area1.png)

**Descripción:** Zona principal del aeropuerto. Incluye mostradores de check-in, atención al viajero y personal de información. Es un área **crítica** por ser el punto de entrada y salida de todos los pasajeros.

**Dispositivos:**
- SW-TP-P (2960-24TT) — Switch principal, VTP Server, Root Bridge VLAN 13
- SW-TP-A1 (2960-24TT) — Switch de acceso, hosts TP1–TP23
- SW-TP-A2 (2960-24TT) — Switch de acceso, hosts TP24–TP46
- SW-TP-A3 (2960-24TT) — Switch de acceso, hosts TP47–TP69
- SW-TP-A4 (2960-24TT) — Switch de acceso, hosts TP70–TP74
- TP1–TP44 (PCs) — Agentes de check-in y mostrador
- TP45–TP74 (Laptops) — Supervisores y personal móvil

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| TP1–TP23 → SW-TP-A1 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| TP24–TP46 → SW-TP-A2 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| TP47–TP69 → SW-TP-A3 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| TP70–TP74 → SW-TP-A4 | UTP Cat5e | NIC | FA0/1–FA0/5 | Straight-Through |
| SW-TP-A1 → SW-TP-P | UTP Cat5e | FA0/24 | FA0/1 | Crossover |
| SW-TP-A2 → SW-TP-P | UTP Cat5e | FA0/24 | FA0/2 | Crossover |
| SW-TP-A3 → SW-TP-P | UTP Cat5e | FA0/24 | FA0/3 | Crossover |
| SW-TP-A4 → SW-TP-P | UTP Cat5e | FA0/24 | FA0/4 | Crossover |

---

### Área 2 — Torre de Control DGAC (VLAN 23)

![Área 2 - Torre de Control](img/area2.png)

**Descripción:** Centro nervioso de la operación aérea. Alberga a los controladores de tráfico aéreo y personal técnico de la DGAC. Es un área **crítica** porque su fallo compromete directamente la seguridad aérea.

**Dispositivos:**
- SW-TC-P (2960-24TT) — Switch principal, VTP Server, Root Bridge VLAN 23
- SW-TC-A1 (2960-24TT) — Switch de acceso, hosts TC1–TC23
- SW-TC-A2 (2960-24TT) — Switch de acceso, hosts TC24–TC25
- TC1–TC20 (PCs) — Controladores de vuelo
- TC21–TC25 (Laptops) — Supervisores y técnicos

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| TC1–TC23 → SW-TC-A1 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| TC24–TC25 → SW-TC-A2 | UTP Cat5e | NIC | FA0/1–FA0/2 | Straight-Through |
| SW-TC-A1 → SW-TC-P | UTP Cat5e | FA0/24 | FA0/1 | Crossover |
| SW-TC-A2 → SW-TC-P | UTP Cat5e | FA0/24 | FA0/2 | Crossover |

---

### Área 3 — Carga y Bodega (VLAN 33)

![Área 3 - Carga y Bodega](img/area3.png)

**Descripción:** Zona de gestión de equipaje y carga aérea. Personal de operaciones con equipos fijos y supervisores de bodega.

**Dispositivos:**
- SW-CB-P (2960-24TT) — Switch principal, VTP Server, Root Bridge VLAN 33
- SW-CB-A1 (2960-24TT) — Switch de acceso, hosts CB1–CB23
- SW-CB-A2 (2960-24TT) — Switch de acceso, hosts CB24–CB32
- CB1–CB22 (PCs) — Operadores de carga
- CB23–CB32 (Laptops) — Supervisores de bodega

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| CB1–CB23 → SW-CB-A1 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| CB24–CB32 → SW-CB-A2 | UTP Cat5e | NIC | FA0/1–FA0/9 | Straight-Through |
| SW-CB-A1 → SW-CB-P | UTP Cat5e | FA0/24 | FA0/1 | Crossover |
| SW-CB-A2 → SW-CB-P | UTP Cat5e | FA0/24 | FA0/2 | Crossover |

---

### Área 4 — Aduana SAT (VLAN 43)

![Área 4 - Aduana SAT](img/area4.png)

**Descripción:** Zona de control aduanero del SAT. Pocos equipos pero de alta sensibilidad operativa.

**Dispositivos:**
- SW-AD-P (2960-24TT) — Switch principal (también hace función de acceso), VTP Server, Root Bridge VLAN 43
- AD1–AD4 (PCs) — Agentes aduaneros
- AD5 (Laptop) — Jefe de aduana

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| AD1–AD5 → SW-AD-P | UTP Cat5e | NIC | FA0/1–FA0/5 | Straight-Through |

---

### Área 5 — Migración (VLAN 53)

![Área 5 - Migración](img/area5.png)

**Descripción:** Control migratorio. Agentes de migración con PCs fijos y supervisores con laptop.

**Dispositivos:**
- SW-MG-P (2960-24TT) — Switch principal, VTP Server, Root Bridge VLAN 53
- SW-MG-A1 (2960-24TT) — Switch de acceso, hosts MG1–MG23
- SW-MG-A2 (2960-24TT) — Switch de acceso, hosts MG24–MG26
- MG1–MG20 (PCs) — Agentes de migración
- MG21–MG26 (Laptops) — Supervisores

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| MG1–MG23 → SW-MG-A1 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| MG24–MG26 → SW-MG-A2 | UTP Cat5e | NIC | FA0/1–FA0/3 | Straight-Through |
| SW-MG-A1 → SW-MG-P | UTP Cat5e | FA0/24 | FA0/1 | Crossover |
| SW-MG-A2 → SW-MG-P | UTP Cat5e | FA0/24 | FA0/2 | Crossover |

---

### Área 6 — Salas de Abordaje / Gates (VLAN 63)

![Área 6 - Salas de Abordaje](img/area6.png)

**Descripción:** Zona de puertas de embarque. Agentes de vuelo y coordinadores de abordaje. Es un área **crítica** porque su fallo impide el embarque de pasajeros.

**Dispositivos:**
- SW-SA-P (2960-24TT) — Switch principal, VTP Server, Root Bridge VLAN 63
- SW-SA-A1 (2960-24TT) — Switch de acceso, hosts SA1–SA23
- SW-SA-A2 (2960-24TT) — Switch de acceso, hosts SA24–SA47
- SA1–SA27 (PCs) — Agentes de gate
- SA28–SA47 (Laptops) — Coordinadores de vuelo

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| SA1–SA23 → SW-SA-A1 | UTP Cat5e | NIC | FA0/1–FA0/23 | Straight-Through |
| SA24–SA47 → SW-SA-A2 | UTP Cat5e | NIC | FA0/1–FA0/24 | Straight-Through |
| SW-SA-A1 → SW-SA-P | UTP Cat5e | FA0/24 | FA0/1 | Crossover |
| SW-SA-A2 → SW-SA-P | UTP Cat5e | FA0/24 | FA0/2 | Crossover |

---

### Área 7 — Seguridad / PNC (VLAN 73)

![Área 7 - Seguridad PNC](img/area7.png)

**Descripción:** Puesto de control de seguridad aeroportuaria. Personal de la Policía Nacional Civil asignado al aeropuerto.

**Dispositivos:**
- SW-SG-P (2960-24TT) — Switch principal (también hace función de acceso), VTP Server, Root Bridge VLAN 73
- SG1–SG6 (PCs) — Agentes de seguridad
- SG7–SG8 (Laptops) — Jefes de turno

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| SG1–SG8 → SW-SG-P | UTP Cat5e | NIC | FA0/1–FA0/8 | Straight-Through |

---

### Área 8 — Área Administrativa (VLAN 83)

![Área 8 - Área Administrativa](img/area8.png)

**Descripción:** Oficinas de administración del aeropuerto. Dirección general, RRHH y finanzas.

**Dispositivos:**
- SW-AA-P (2960-24TT) — Switch principal (también hace función de acceso), VTP Server, Root Bridge VLAN 83
- AA1–AA12 (PCs) — Personal administrativo
- AA13–AA20 (Laptops) — Gerentes y directores

**Conexiones internas:**

| Enlace | Medio | Puerto origen | Puerto destino | Tipo de cable |
|--------|-------|--------------|----------------|---------------|
| AA1–AA20 → SW-AA-P | UTP Cat5e | NIC | FA0/1–FA0/20 | Straight-Through |

---

## Tabla de VLANs

| VLAN ID | Nombre | Área | Red asignada |
|---------|--------|------|--------------|
| 13 | Terminal_Pasajeros | Terminal de Pasajeros | 192.168.10.0/25 |
| 23 | Torre_Control | Torre de Control DGAC | 192.168.11.32/27 |
| 33 | Carga_Bodega | Área de Carga y Bodega | 192.168.10.192/26 |
| 43 | Aduana_SAT | Aduana SAT | 192.168.11.112/29 |
| 53 | Migracion | Migración | 192.168.11.0/27 |
| 63 | Salas_Abordaje | Salas de Abordaje / Gates | 192.168.10.128/26 |
| 73 | Seguridad_PNC | Seguridad / PNC | 192.168.11.96/28 |
| 83 | Area_Administrativa | Área Administrativa | 192.168.11.64/27 |
| 99 | VLAN_Nativa | Tráfico entre switches (trunk) | N/A |
| 999 | Blackhole | Puertos no utilizados | N/A |

![Show VLAN Brief](img/show_vlan_brief.png)

---

## Tabla de Dominios de Colisión

Un dominio de colisión es el segmento de red donde dos dispositivos pueden causar colisión si transmiten simultáneamente. Cada puerto de un switch delimita un dominio de colisión independiente.

| Área | Switch | Dominios de colisión | Hosts que delimitan |
|------|--------|---------------------|-------------------|
| Terminal Pasajeros | SW-TP-A1 | 23 | TP1–TP23 |
| Terminal Pasajeros | SW-TP-A2 | 23 | TP24–TP46 |
| Terminal Pasajeros | SW-TP-A3 | 23 | TP47–TP69 |
| Terminal Pasajeros | SW-TP-A4 | 5 | TP70–TP74 |
| Torre de Control | SW-TC-A1 | 23 | TC1–TC23 |
| Torre de Control | SW-TC-A2 | 2 | TC24–TC25 |
| Carga y Bodega | SW-CB-A1 | 23 | CB1–CB23 |
| Carga y Bodega | SW-CB-A2 | 9 | CB24–CB32 |
| Aduana SAT | SW-AD-P | 5 | AD1–AD5 |
| Migración | SW-MG-A1 | 23 | MG1–MG23 |
| Migración | SW-MG-A2 | 3 | MG24–MG26 |
| Salas de Abordaje | SW-SA-A1 | 23 | SA1–SA23 |
| Salas de Abordaje | SW-SA-A2 | 24 | SA24–SA47 |
| Seguridad PNC | SW-SG-P | 8 | SG1–SG8 |
| Área Administrativa | SW-AA-P | 20 | AA1–AA20 |

**Total de dominios de colisión: 237** (uno por cada host, ya que todos van a switches)

---

## Tabla de Dominios de Broadcast

Un dominio de broadcast abarca todos los dispositivos que reciben un frame de broadcast. Las VLANs delimitan estos dominios, por lo que hay exactamente un dominio de broadcast por VLAN.

| VLAN | Nombre | Dispositivos incluidos | Total hosts |
|------|--------|----------------------|-------------|
| VLAN 13 | Terminal_Pasajeros | TP1–TP74 | 74 |
| VLAN 23 | Torre_Control | TC1–TC25 | 25 |
| VLAN 33 | Carga_Bodega | CB1–CB32 | 32 |
| VLAN 43 | Aduana_SAT | AD1–AD5 | 5 |
| VLAN 53 | Migracion | MG1–MG26 | 26 |
| VLAN 63 | Salas_Abordaje | SA1–SA47 | 47 |
| VLAN 73 | Seguridad_PNC | SG1–SG8 | 8 |
| VLAN 83 | Area_Administrativa | AA1–AA20 | 20 |

**Total de dominios de broadcast: 8** (uno por VLAN)

---

## Jerarquía de Medios e Interfaces

| Nivel | Segmento | Medio | Estándar | Tipo de cable |
|-------|----------|-------|----------|---------------|
| Backbone (entre principales) | Switches principales entre sí | Cobre UTP Cat5e | 100Base-TX | Crossover |
| Distribución (acceso a principal) | Switch acceso → Switch principal | Cobre UTP Cat5e | 100Base-TX | Crossover |
| Acceso (hosts a switches) | PCs / Laptops → Switch de acceso | Cobre UTP Cat5e | 100Base-TX | Straight-Through |

**Nota:** Todo el cableado de la red es cobre UTP Cat5e bajo el estándar TIA/EIA-568B. No se usa fibra óptica dado que todos los switches son Cisco 2960-24TT sin módulos de fibra.

---

## Tabla de Subnetting VLSM

**Red base:** 192.168.10.0  
**Técnica:** VLSM (Variable Length Subnet Masking) — subredes ordenadas de mayor a menor cantidad de hosts.

| Orden | VLAN | Área | Hosts req. | Red | Máscara | Prefijo | Gateway | Rango de hosts | Broadcast | Hosts útiles |
|-------|------|------|-----------|-----|---------|---------|---------|---------------|-----------|-------------|
| 1° | 13 | Terminal Pasajeros | 74 | 192.168.10.0 | 255.255.255.128 | /25 | 192.168.10.1 | .2 – .126 | .127 | 126 |
| 2° | 63 | Salas de Abordaje | 47 | 192.168.10.128 | 255.255.255.192 | /26 | 192.168.10.129 | .130 – .190 | .191 | 62 |
| 3° | 33 | Carga y Bodega | 32 | 192.168.10.192 | 255.255.255.192 | /26 | 192.168.10.193 | .194 – .254 | .255 | 62 |
| 4° | 53 | Migración | 26 | 192.168.11.0 | 255.255.255.224 | /27 | 192.168.11.1 | .2 – .30 | .31 | 30 |
| 5° | 23 | Torre de Control | 25 | 192.168.11.32 | 255.255.255.224 | /27 | 192.168.11.33 | .34 – .62 | .63 | 30 |
| 6° | 83 | Área Administrativa | 20 | 192.168.11.64 | 255.255.255.224 | /27 | 192.168.11.65 | .66 – .94 | .95 | 30 |
| 7° | 73 | Seguridad PNC | 8 | 192.168.11.96 | 255.255.255.240 | /28 | 192.168.11.97 | .98 – .110 | .111 | 14 |
| 8° | 43 | Aduana SAT | 5 | 192.168.11.112 | 255.255.255.248 | /29 | 192.168.11.113 | .114 – .118 | .119 | 6 |

---

## Tabla de Direccionamiento IP

### Área 1 — Terminal de Pasajeros (VLAN 13)
**Red:** 192.168.10.0/25 | **Gateway:** 192.168.10.1 | **Máscara:** 255.255.255.128

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| TP1 | 192.168.10.2 | 255.255.255.128 | 192.168.10.1 |
| TP2 | 192.168.10.3 | 255.255.255.128 | 192.168.10.1 |
| TP3 | 192.168.10.4 | 255.255.255.128 | 192.168.10.1 |
| TP4 | 192.168.10.5 | 255.255.255.128 | 192.168.10.1 |
| TP5 | 192.168.10.6 | 255.255.255.128 | 192.168.10.1 |
| TP6 | 192.168.10.7 | 255.255.255.128 | 192.168.10.1 |
| TP7 | 192.168.10.8 | 255.255.255.128 | 192.168.10.1 |
| TP8 | 192.168.10.9 | 255.255.255.128 | 192.168.10.1 |
| TP9 | 192.168.10.10 | 255.255.255.128 | 192.168.10.1 |
| TP10 | 192.168.10.11 | 255.255.255.128 | 192.168.10.1 |
| TP11 | 192.168.10.12 | 255.255.255.128 | 192.168.10.1 |
| TP12 | 192.168.10.13 | 255.255.255.128 | 192.168.10.1 |
| TP13 | 192.168.10.14 | 255.255.255.128 | 192.168.10.1 |
| TP14 | 192.168.10.15 | 255.255.255.128 | 192.168.10.1 |
| TP15 | 192.168.10.16 | 255.255.255.128 | 192.168.10.1 |
| TP16 | 192.168.10.17 | 255.255.255.128 | 192.168.10.1 |
| TP17 | 192.168.10.18 | 255.255.255.128 | 192.168.10.1 |
| TP18 | 192.168.10.19 | 255.255.255.128 | 192.168.10.1 |
| TP19 | 192.168.10.20 | 255.255.255.128 | 192.168.10.1 |
| TP20 | 192.168.10.21 | 255.255.255.128 | 192.168.10.1 |
| TP21 | 192.168.10.22 | 255.255.255.128 | 192.168.10.1 |
| TP22 | 192.168.10.23 | 255.255.255.128 | 192.168.10.1 |
| TP23 | 192.168.10.24 | 255.255.255.128 | 192.168.10.1 |
| TP24 | 192.168.10.25 | 255.255.255.128 | 192.168.10.1 |
| TP25 | 192.168.10.26 | 255.255.255.128 | 192.168.10.1 |
| TP26 | 192.168.10.27 | 255.255.255.128 | 192.168.10.1 |
| TP27 | 192.168.10.28 | 255.255.255.128 | 192.168.10.1 |
| TP28 | 192.168.10.29 | 255.255.255.128 | 192.168.10.1 |
| TP29 | 192.168.10.30 | 255.255.255.128 | 192.168.10.1 |
| TP30 | 192.168.10.31 | 255.255.255.128 | 192.168.10.1 |
| TP31 | 192.168.10.32 | 255.255.255.128 | 192.168.10.1 |
| TP32 | 192.168.10.33 | 255.255.255.128 | 192.168.10.1 |
| TP33 | 192.168.10.34 | 255.255.255.128 | 192.168.10.1 |
| TP34 | 192.168.10.35 | 255.255.255.128 | 192.168.10.1 |
| TP35 | 192.168.10.36 | 255.255.255.128 | 192.168.10.1 |
| TP36 | 192.168.10.37 | 255.255.255.128 | 192.168.10.1 |
| TP37 | 192.168.10.38 | 255.255.255.128 | 192.168.10.1 |
| TP38 | 192.168.10.39 | 255.255.255.128 | 192.168.10.1 |
| TP39 | 192.168.10.40 | 255.255.255.128 | 192.168.10.1 |
| TP40 | 192.168.10.41 | 255.255.255.128 | 192.168.10.1 |
| TP41 | 192.168.10.42 | 255.255.255.128 | 192.168.10.1 |
| TP42 | 192.168.10.43 | 255.255.255.128 | 192.168.10.1 |
| TP43 | 192.168.10.44 | 255.255.255.128 | 192.168.10.1 |
| TP44 | 192.168.10.45 | 255.255.255.128 | 192.168.10.1 |
| TP45 | 192.168.10.46 | 255.255.255.128 | 192.168.10.1 |
| TP46 | 192.168.10.47 | 255.255.255.128 | 192.168.10.1 |
| TP47 | 192.168.10.48 | 255.255.255.128 | 192.168.10.1 |
| TP48 | 192.168.10.49 | 255.255.255.128 | 192.168.10.1 |
| TP49 | 192.168.10.50 | 255.255.255.128 | 192.168.10.1 |
| TP50 | 192.168.10.51 | 255.255.255.128 | 192.168.10.1 |
| TP51 | 192.168.10.52 | 255.255.255.128 | 192.168.10.1 |
| TP52 | 192.168.10.53 | 255.255.255.128 | 192.168.10.1 |
| TP53 | 192.168.10.54 | 255.255.255.128 | 192.168.10.1 |
| TP54 | 192.168.10.55 | 255.255.255.128 | 192.168.10.1 |
| TP55 | 192.168.10.56 | 255.255.255.128 | 192.168.10.1 |
| TP56 | 192.168.10.57 | 255.255.255.128 | 192.168.10.1 |
| TP57 | 192.168.10.58 | 255.255.255.128 | 192.168.10.1 |
| TP58 | 192.168.10.59 | 255.255.255.128 | 192.168.10.1 |
| TP59 | 192.168.10.60 | 255.255.255.128 | 192.168.10.1 |
| TP60 | 192.168.10.61 | 255.255.255.128 | 192.168.10.1 |
| TP61 | 192.168.10.62 | 255.255.255.128 | 192.168.10.1 |
| TP62 | 192.168.10.63 | 255.255.255.128 | 192.168.10.1 |
| TP63 | 192.168.10.64 | 255.255.255.128 | 192.168.10.1 |
| TP64 | 192.168.10.65 | 255.255.255.128 | 192.168.10.1 |
| TP65 | 192.168.10.66 | 255.255.255.128 | 192.168.10.1 |
| TP66 | 192.168.10.67 | 255.255.255.128 | 192.168.10.1 |
| TP67 | 192.168.10.68 | 255.255.255.128 | 192.168.10.1 |
| TP68 | 192.168.10.69 | 255.255.255.128 | 192.168.10.1 |
| TP69 | 192.168.10.70 | 255.255.255.128 | 192.168.10.1 |
| TP70 | 192.168.10.71 | 255.255.255.128 | 192.168.10.1 |
| TP71 | 192.168.10.72 | 255.255.255.128 | 192.168.10.1 |
| TP72 | 192.168.10.73 | 255.255.255.128 | 192.168.10.1 |
| TP73 | 192.168.10.74 | 255.255.255.128 | 192.168.10.1 |
| TP74 | 192.168.10.75 | 255.255.255.128 | 192.168.10.1 |

### Área 2 — Torre de Control (VLAN 23)
**Red:** 192.168.11.32/27 | **Gateway:** 192.168.11.33 | **Máscara:** 255.255.255.224

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| TC1 | 192.168.11.34 | 255.255.255.224 | 192.168.11.33 |
| TC2 | 192.168.11.35 | 255.255.255.224 | 192.168.11.33 |
| TC3 | 192.168.11.36 | 255.255.255.224 | 192.168.11.33 |
| TC4 | 192.168.11.37 | 255.255.255.224 | 192.168.11.33 |
| TC5 | 192.168.11.38 | 255.255.255.224 | 192.168.11.33 |
| TC6 | 192.168.11.39 | 255.255.255.224 | 192.168.11.33 |
| TC7 | 192.168.11.40 | 255.255.255.224 | 192.168.11.33 |
| TC8 | 192.168.11.41 | 255.255.255.224 | 192.168.11.33 |
| TC9 | 192.168.11.42 | 255.255.255.224 | 192.168.11.33 |
| TC10 | 192.168.11.43 | 255.255.255.224 | 192.168.11.33 |
| TC11 | 192.168.11.44 | 255.255.255.224 | 192.168.11.33 |
| TC12 | 192.168.11.45 | 255.255.255.224 | 192.168.11.33 |
| TC13 | 192.168.11.46 | 255.255.255.224 | 192.168.11.33 |
| TC14 | 192.168.11.47 | 255.255.255.224 | 192.168.11.33 |
| TC15 | 192.168.11.48 | 255.255.255.224 | 192.168.11.33 |
| TC16 | 192.168.11.49 | 255.255.255.224 | 192.168.11.33 |
| TC17 | 192.168.11.50 | 255.255.255.224 | 192.168.11.33 |
| TC18 | 192.168.11.51 | 255.255.255.224 | 192.168.11.33 |
| TC19 | 192.168.11.52 | 255.255.255.224 | 192.168.11.33 |
| TC20 | 192.168.11.53 | 255.255.255.224 | 192.168.11.33 |
| TC21 | 192.168.11.54 | 255.255.255.224 | 192.168.11.33 |
| TC22 | 192.168.11.55 | 255.255.255.224 | 192.168.11.33 |
| TC23 | 192.168.11.56 | 255.255.255.224 | 192.168.11.33 |
| TC24 | 192.168.11.57 | 255.255.255.224 | 192.168.11.33 |
| TC25 | 192.168.11.58 | 255.255.255.224 | 192.168.11.33 |

### Área 3 — Carga y Bodega (VLAN 33)
**Red:** 192.168.10.192/26 | **Gateway:** 192.168.10.193 | **Máscara:** 255.255.255.192

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| CB1 | 192.168.10.194 | 255.255.255.192 | 192.168.10.193 |
| CB2 | 192.168.10.195 | 255.255.255.192 | 192.168.10.193 |
| CB3 | 192.168.10.196 | 255.255.255.192 | 192.168.10.193 |
| CB4 | 192.168.10.197 | 255.255.255.192 | 192.168.10.193 |
| CB5 | 192.168.10.198 | 255.255.255.192 | 192.168.10.193 |
| CB6 | 192.168.10.199 | 255.255.255.192 | 192.168.10.193 |
| CB7 | 192.168.10.200 | 255.255.255.192 | 192.168.10.193 |
| CB8 | 192.168.10.201 | 255.255.255.192 | 192.168.10.193 |
| CB9 | 192.168.10.202 | 255.255.255.192 | 192.168.10.193 |
| CB10 | 192.168.10.203 | 255.255.255.192 | 192.168.10.193 |
| CB11 | 192.168.10.204 | 255.255.255.192 | 192.168.10.193 |
| CB12 | 192.168.10.205 | 255.255.255.192 | 192.168.10.193 |
| CB13 | 192.168.10.206 | 255.255.255.192 | 192.168.10.193 |
| CB14 | 192.168.10.207 | 255.255.255.192 | 192.168.10.193 |
| CB15 | 192.168.10.208 | 255.255.255.192 | 192.168.10.193 |
| CB16 | 192.168.10.209 | 255.255.255.192 | 192.168.10.193 |
| CB17 | 192.168.10.210 | 255.255.255.192 | 192.168.10.193 |
| CB18 | 192.168.10.211 | 255.255.255.192 | 192.168.10.193 |
| CB19 | 192.168.10.212 | 255.255.255.192 | 192.168.10.193 |
| CB20 | 192.168.10.213 | 255.255.255.192 | 192.168.10.193 |
| CB21 | 192.168.10.214 | 255.255.255.192 | 192.168.10.193 |
| CB22 | 192.168.10.215 | 255.255.255.192 | 192.168.10.193 |
| CB23 | 192.168.10.216 | 255.255.255.192 | 192.168.10.193 |
| CB24 | 192.168.10.217 | 255.255.255.192 | 192.168.10.193 |
| CB25 | 192.168.10.218 | 255.255.255.192 | 192.168.10.193 |
| CB26 | 192.168.10.219 | 255.255.255.192 | 192.168.10.193 |
| CB27 | 192.168.10.220 | 255.255.255.192 | 192.168.10.193 |
| CB28 | 192.168.10.221 | 255.255.255.192 | 192.168.10.193 |
| CB29 | 192.168.10.222 | 255.255.255.192 | 192.168.10.193 |
| CB30 | 192.168.10.223 | 255.255.255.192 | 192.168.10.193 |
| CB31 | 192.168.10.224 | 255.255.255.192 | 192.168.10.193 |
| CB32 | 192.168.10.225 | 255.255.255.192 | 192.168.10.193 |

### Área 4 — Aduana SAT (VLAN 43)
**Red:** 192.168.11.112/29 | **Gateway:** 192.168.11.113 | **Máscara:** 255.255.255.248

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| AD1 | 192.168.11.114 | 255.255.255.248 | 192.168.11.113 |
| AD2 | 192.168.11.115 | 255.255.255.248 | 192.168.11.113 |
| AD3 | 192.168.11.116 | 255.255.255.248 | 192.168.11.113 |
| AD4 | 192.168.11.117 | 255.255.255.248 | 192.168.11.113 |
| AD5 | 192.168.11.118 | 255.255.255.248 | 192.168.11.113 |

### Área 5 — Migración (VLAN 53)
**Red:** 192.168.11.0/27 | **Gateway:** 192.168.11.1 | **Máscara:** 255.255.255.224

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| MG1 | 192.168.11.2 | 255.255.255.224 | 192.168.11.1 |
| MG2 | 192.168.11.3 | 255.255.255.224 | 192.168.11.1 |
| MG3 | 192.168.11.4 | 255.255.255.224 | 192.168.11.1 |
| MG4 | 192.168.11.5 | 255.255.255.224 | 192.168.11.1 |
| MG5 | 192.168.11.6 | 255.255.255.224 | 192.168.11.1 |
| MG6 | 192.168.11.7 | 255.255.255.224 | 192.168.11.1 |
| MG7 | 192.168.11.8 | 255.255.255.224 | 192.168.11.1 |
| MG8 | 192.168.11.9 | 255.255.255.224 | 192.168.11.1 |
| MG9 | 192.168.11.10 | 255.255.255.224 | 192.168.11.1 |
| MG10 | 192.168.11.11 | 255.255.255.224 | 192.168.11.1 |
| MG11 | 192.168.11.12 | 255.255.255.224 | 192.168.11.1 |
| MG12 | 192.168.11.13 | 255.255.255.224 | 192.168.11.1 |
| MG13 | 192.168.11.14 | 255.255.255.224 | 192.168.11.1 |
| MG14 | 192.168.11.15 | 255.255.255.224 | 192.168.11.1 |
| MG15 | 192.168.11.16 | 255.255.255.224 | 192.168.11.1 |
| MG16 | 192.168.11.17 | 255.255.255.224 | 192.168.11.1 |
| MG17 | 192.168.11.18 | 255.255.255.224 | 192.168.11.1 |
| MG18 | 192.168.11.19 | 255.255.255.224 | 192.168.11.1 |
| MG19 | 192.168.11.20 | 255.255.255.224 | 192.168.11.1 |
| MG20 | 192.168.11.21 | 255.255.255.224 | 192.168.11.1 |
| MG21 | 192.168.11.22 | 255.255.255.224 | 192.168.11.1 |
| MG22 | 192.168.11.23 | 255.255.255.224 | 192.168.11.1 |
| MG23 | 192.168.11.24 | 255.255.255.224 | 192.168.11.1 |
| MG24 | 192.168.11.25 | 255.255.255.224 | 192.168.11.1 |
| MG25 | 192.168.11.26 | 255.255.255.224 | 192.168.11.1 |
| MG26 | 192.168.11.27 | 255.255.255.224 | 192.168.11.1 |

### Área 6 — Salas de Abordaje (VLAN 63)
**Red:** 192.168.10.128/26 | **Gateway:** 192.168.10.129 | **Máscara:** 255.255.255.192

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| SA1 | 192.168.10.130 | 255.255.255.192 | 192.168.10.129 |
| SA2 | 192.168.10.131 | 255.255.255.192 | 192.168.10.129 |
| SA3 | 192.168.10.132 | 255.255.255.192 | 192.168.10.129 |
| SA4 | 192.168.10.133 | 255.255.255.192 | 192.168.10.129 |
| SA5 | 192.168.10.134 | 255.255.255.192 | 192.168.10.129 |
| SA6 | 192.168.10.135 | 255.255.255.192 | 192.168.10.129 |
| SA7 | 192.168.10.136 | 255.255.255.192 | 192.168.10.129 |
| SA8 | 192.168.10.137 | 255.255.255.192 | 192.168.10.129 |
| SA9 | 192.168.10.138 | 255.255.255.192 | 192.168.10.129 |
| SA10 | 192.168.10.139 | 255.255.255.192 | 192.168.10.129 |
| SA11 | 192.168.10.140 | 255.255.255.192 | 192.168.10.129 |
| SA12 | 192.168.10.141 | 255.255.255.192 | 192.168.10.129 |
| SA13 | 192.168.10.142 | 255.255.255.192 | 192.168.10.129 |
| SA14 | 192.168.10.143 | 255.255.255.192 | 192.168.10.129 |
| SA15 | 192.168.10.144 | 255.255.255.192 | 192.168.10.129 |
| SA16 | 192.168.10.145 | 255.255.255.192 | 192.168.10.129 |
| SA17 | 192.168.10.146 | 255.255.255.192 | 192.168.10.129 |
| SA18 | 192.168.10.147 | 255.255.255.192 | 192.168.10.129 |
| SA19 | 192.168.10.148 | 255.255.255.192 | 192.168.10.129 |
| SA20 | 192.168.10.149 | 255.255.255.192 | 192.168.10.129 |
| SA21 | 192.168.10.150 | 255.255.255.192 | 192.168.10.129 |
| SA22 | 192.168.10.151 | 255.255.255.192 | 192.168.10.129 |
| SA23 | 192.168.10.152 | 255.255.255.192 | 192.168.10.129 |
| SA24 | 192.168.10.153 | 255.255.255.192 | 192.168.10.129 |
| SA25 | 192.168.10.154 | 255.255.255.192 | 192.168.10.129 |
| SA26 | 192.168.10.155 | 255.255.255.192 | 192.168.10.129 |
| SA27 | 192.168.10.156 | 255.255.255.192 | 192.168.10.129 |
| SA28 | 192.168.10.157 | 255.255.255.192 | 192.168.10.129 |
| SA29 | 192.168.10.158 | 255.255.255.192 | 192.168.10.129 |
| SA30 | 192.168.10.159 | 255.255.255.192 | 192.168.10.129 |
| SA31 | 192.168.10.160 | 255.255.255.192 | 192.168.10.129 |
| SA32 | 192.168.10.161 | 255.255.255.192 | 192.168.10.129 |
| SA33 | 192.168.10.162 | 255.255.255.192 | 192.168.10.129 |
| SA34 | 192.168.10.163 | 255.255.255.192 | 192.168.10.129 |
| SA35 | 192.168.10.164 | 255.255.255.192 | 192.168.10.129 |
| SA36 | 192.168.10.165 | 255.255.255.192 | 192.168.10.129 |
| SA37 | 192.168.10.166 | 255.255.255.192 | 192.168.10.129 |
| SA38 | 192.168.10.167 | 255.255.255.192 | 192.168.10.129 |
| SA39 | 192.168.10.168 | 255.255.255.192 | 192.168.10.129 |
| SA40 | 192.168.10.169 | 255.255.255.192 | 192.168.10.129 |
| SA41 | 192.168.10.170 | 255.255.255.192 | 192.168.10.129 |
| SA42 | 192.168.10.171 | 255.255.255.192 | 192.168.10.129 |
| SA43 | 192.168.10.172 | 255.255.255.192 | 192.168.10.129 |
| SA44 | 192.168.10.173 | 255.255.255.192 | 192.168.10.129 |
| SA45 | 192.168.10.174 | 255.255.255.192 | 192.168.10.129 |
| SA46 | 192.168.10.175 | 255.255.255.192 | 192.168.10.129 |
| SA47 | 192.168.10.176 | 255.255.255.192 | 192.168.10.129 |

### Área 7 — Seguridad PNC (VLAN 73)
**Red:** 192.168.11.96/28 | **Gateway:** 192.168.11.97 | **Máscara:** 255.255.255.240

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| SG1 | 192.168.11.98 | 255.255.255.240 | 192.168.11.97 |
| SG2 | 192.168.11.99 | 255.255.255.240 | 192.168.11.97 |
| SG3 | 192.168.11.100 | 255.255.255.240 | 192.168.11.97 |
| SG4 | 192.168.11.101 | 255.255.255.240 | 192.168.11.97 |
| SG5 | 192.168.11.102 | 255.255.255.240 | 192.168.11.97 |
| SG6 | 192.168.11.103 | 255.255.255.240 | 192.168.11.97 |
| SG7 | 192.168.11.104 | 255.255.255.240 | 192.168.11.97 |
| SG8 | 192.168.11.105 | 255.255.255.240 | 192.168.11.97 |

### Área 8 — Área Administrativa (VLAN 83)
**Red:** 192.168.11.64/27 | **Gateway:** 192.168.11.65 | **Máscara:** 255.255.255.224

| Equipo | IP Address | Subnet Mask | Gateway |
|--------|-----------|-------------|---------|
| AA1 | 192.168.11.66 | 255.255.255.224 | 192.168.11.65 |
| AA2 | 192.168.11.67 | 255.255.255.224 | 192.168.11.65 |
| AA3 | 192.168.11.68 | 255.255.255.224 | 192.168.11.65 |
| AA4 | 192.168.11.69 | 255.255.255.224 | 192.168.11.65 |
| AA5 | 192.168.11.70 | 255.255.255.224 | 192.168.11.65 |
| AA6 | 192.168.11.71 | 255.255.255.224 | 192.168.11.65 |
| AA7 | 192.168.11.72 | 255.255.255.224 | 192.168.11.65 |
| AA8 | 192.168.11.73 | 255.255.255.224 | 192.168.11.65 |
| AA9 | 192.168.11.74 | 255.255.255.224 | 192.168.11.65 |
| AA10 | 192.168.11.75 | 255.255.255.224 | 192.168.11.65 |
| AA11 | 192.168.11.76 | 255.255.255.224 | 192.168.11.65 |
| AA12 | 192.168.11.77 | 255.255.255.224 | 192.168.11.65 |
| AA13 | 192.168.11.78 | 255.255.255.224 | 192.168.11.65 |
| AA14 | 192.168.11.79 | 255.255.255.224 | 192.168.11.65 |
| AA15 | 192.168.11.80 | 255.255.255.224 | 192.168.11.65 |
| AA16 | 192.168.11.81 | 255.255.255.224 | 192.168.11.65 |
| AA17 | 192.168.11.82 | 255.255.255.224 | 192.168.11.65 |
| AA18 | 192.168.11.83 | 255.255.255.224 | 192.168.11.65 |
| AA19 | 192.168.11.84 | 255.255.255.224 | 192.168.11.65 |
| AA20 | 192.168.11.85 | 255.255.255.224 | 192.168.11.65 |

---

## Configuración VTP

| Switch | Modo VTP | Dominio | Versión | Contraseña |
|--------|----------|---------|---------|------------|
| SW-TP-P | Server | 202300733 | 2 | area1 |
| SW-TC-P | Server | 202300733 | 2 | area2 |
| SW-CB-P | Server | 202300733 | 2 | area3 |
| SW-AD-P | Server | 202300733 | 2 | area4 |
| SW-MG-P | Server | 202300733 | 2 | area5 |
| SW-SA-P | Server | 202300733 | 2 | area6 |
| SW-SG-P | Server | 202300733 | 2 | area7 |
| SW-AA-P | Server | 202300733 | 2 | area8 |
| SW-TP-A1, A2, A3, A4 | Client | 202300733 | 2 | area1 |
| SW-TC-A1, A2 | Client | 202300733 | 2 | area2 |
| SW-CB-A1, A2 | Client | 202300733 | 2 | area3 |
| SW-MG-A1, A2 | Client | 202300733 | 2 | area5 |
| SW-SA-A1, A2 | Client | 202300733 | 2 | area6 |

![Show VTP Status](img/show_vtp_status.png)

---

## Configuración STP

Se implementó **Rapid PVST+** en todos los switches. Cada switch principal es el Root Bridge para la VLAN de su área, con priority 4096. Los switches de acceso mantienen la priority por defecto (32768).

| Switch | VLAN | Rol STP | Priority |
|--------|------|---------|----------|
| SW-TP-P | 13 | Root Bridge | 4096 |
| SW-TC-P | 23 | Root Bridge | 4096 |
| SW-CB-P | 33 | Root Bridge | 4096 |
| SW-AD-P | 43 | Root Bridge | 4096 |
| SW-MG-P | 53 | Root Bridge | 4096 |
| SW-SA-P | 63 | Root Bridge | 4096 |
| SW-SG-P | 73 | Root Bridge | 4096 |
| SW-AA-P | 83 | Root Bridge | 4096 |
| Todos los SW de acceso | Todas | Designated/Non-designated | 32768 |

![Show Spanning-Tree](img/show_spanning_tree.png)

---

## Configuración EtherChannel

Se utilizó el protocolo **LACP (802.3ad)** con modo **active** en ambos extremos para formar los Port-Channel entre switches principales de áreas críticas. Cada EtherChannel agrupa **3 enlaces FastEthernet** físicos.

| Port-Channel | Switch A | Puertos A | Switch B | Puertos B | Protocolo |
|-------------|---------|-----------|---------|-----------|-----------|
| Po1 | SW-TP-P | FA0/5, FA0/6, FA0/7 | SW-TC-P | FA0/5, FA0/6, FA0/7 | LACP |
| Po2 | SW-TP-P | FA0/8, FA0/9, FA0/10 | SW-SA-P | FA0/5, FA0/6, FA0/7 | LACP |
| Po3 | SW-TC-P | FA0/8, FA0/9, FA0/10 | SW-MG-P | FA0/5, FA0/6, FA0/7 | LACP |

![Show EtherChannel Summary](img/show_etherchannel_summary.png)

---

## Configuración de Trunks

Todos los enlaces entre switches (tanto EtherChannel como simples) se configuraron como puertos trunk 802.1Q con VLAN nativa 99 y permitiendo únicamente las VLANs necesarias.

| Enlace | VLAN nativa | VLANs permitidas |
|--------|------------|-----------------|
| Todos los trunks entre switches | 99 | 13,23,33,43,53,63,73,83,99 |

![Show Interfaces Trunk](img/show_interfaces_trunk.png)

---

## Comandos por Dispositivo

### SW-TP-P — VTP Server / Root Bridge VLAN 13 / EtherChannel LACP

```
enable
configure terminal
hostname SW-TP-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Area Terminal Pasajeros - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area1
spanning-tree mode rapid-pvst
spanning-tree vlan 13 priority 4096
vlan 13
 name Terminal_Pasajeros
vlan 23
 name Torre_Control
vlan 33
 name Carga_Bodega
vlan 43
 name Aduana_SAT
vlan 53
 name Migracion
vlan 63
 name Salas_Abordaje
vlan 73
 name Seguridad_PNC
vlan 83
 name Area_Administrativa
vlan 99
 name VLAN_Nativa
vlan 999
 name Blackhole
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/3
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/4
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/5 - 7
 channel-group 1 mode active
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/8 - 10
 channel-group 2 mode active
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface port-channel 1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
interface port-channel 2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
interface range FastEthernet0/11 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-TP-A1 — VTP Client / Acceso VLAN 13

```
enable
configure terminal
hostname SW-TP-A1
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Area Terminal Pasajeros - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area1
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-TP-A2 — VTP Client / Acceso VLAN 13

```
enable
configure terminal
hostname SW-TP-A2
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Area Terminal Pasajeros - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area1
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-TP-A3 — VTP Client / Acceso VLAN 13

```
enable
configure terminal
hostname SW-TP-A3
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Area Terminal Pasajeros - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area1
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-TP-A4 — VTP Client / Acceso VLAN 13

```
enable
configure terminal
hostname SW-TP-A4
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Area Terminal Pasajeros - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area1
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 5
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast
 no shutdown
interface range FastEthernet0/6 - 23
 switchport mode access
 switchport access vlan 999
 shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-TC-P — VTP Server / Root Bridge VLAN 23 / EtherChannel LACP

```
enable
configure terminal
hostname SW-TC-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Torre de Control - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area2
spanning-tree mode rapid-pvst
spanning-tree vlan 23 priority 4096
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/5 - 7
 channel-group 1 mode active
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/8 - 10
 channel-group 3 mode active
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface port-channel 1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
interface port-channel 3
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
interface range FastEthernet0/11 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-TC-A1 — VTP Client / Acceso VLAN 23

```
enable
configure terminal
hostname SW-TC-A1
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Torre de Control - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area2
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 23
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-TC-A2 — VTP Client / Acceso VLAN 23

```
enable
configure terminal
hostname SW-TC-A2
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Torre de Control - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area2
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 23
 spanning-tree portfast
 no shutdown
interface range FastEthernet0/3 - 23
 switchport mode access
 switchport access vlan 999
 shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-CB-P — VTP Server / Root Bridge VLAN 33

```
enable
configure terminal
hostname SW-CB-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Carga y Bodega - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area3
spanning-tree mode rapid-pvst
spanning-tree vlan 33 priority 4096
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/6
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/7 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-CB-A1 — VTP Client / Acceso VLAN 33

```
enable
configure terminal
hostname SW-CB-A1
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Carga y Bodega - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area3
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 33
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-CB-A2 — VTP Client / Acceso VLAN 33

```
enable
configure terminal
hostname SW-CB-A2
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Carga y Bodega - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area3
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 9
 switchport mode access
 switchport access vlan 33
 spanning-tree portfast
 no shutdown
interface range FastEthernet0/10 - 23
 switchport mode access
 switchport access vlan 999
 shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-AD-P — VTP Server / Root Bridge VLAN 43

```
enable
configure terminal
hostname SW-AD-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Aduana SAT - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area4
spanning-tree mode rapid-pvst
spanning-tree vlan 43 priority 4096
interface range FastEthernet0/1 - 5
 switchport mode access
 switchport access vlan 43
 spanning-tree portfast
 no shutdown
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/6 - 23
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-MG-P — VTP Server / Root Bridge VLAN 53 / EtherChannel LACP

```
enable
configure terminal
hostname SW-MG-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Migracion - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area5
spanning-tree mode rapid-pvst
spanning-tree vlan 53 priority 4096
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/5 - 7
 channel-group 3 mode active
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/8
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface port-channel 3
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
interface range FastEthernet0/9 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-MG-A1 — VTP Client / Acceso VLAN 53

```
enable
configure terminal
hostname SW-MG-A1
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Migracion - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area5
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 53
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-MG-A2 — VTP Client / Acceso VLAN 53

```
enable
configure terminal
hostname SW-MG-A2
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Migracion - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area5
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport access vlan 53
 spanning-tree portfast
 no shutdown
interface range FastEthernet0/4 - 23
 switchport mode access
 switchport access vlan 999
 shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-SA-P — VTP Server / Root Bridge VLAN 63 / EtherChannel LACP

```
enable
configure terminal
hostname SW-SA-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Salas de Abordaje - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area6
spanning-tree mode rapid-pvst
spanning-tree vlan 63 priority 4096
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/5 - 7
 channel-group 2 mode active
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/8
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface port-channel 2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
interface range FastEthernet0/9 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-SA-A1 — VTP Client / Acceso VLAN 63

```
enable
configure terminal
hostname SW-SA-A1
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Salas de Abordaje - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area6
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 23
 switchport mode access
 switchport access vlan 63
 spanning-tree portfast
 no shutdown
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
end
write memory
```

---

### SW-SA-A2 — VTP Client / Acceso VLAN 63

```
enable
configure terminal
hostname SW-SA-A2
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Salas de Abordaje - 202300733 #
vtp mode client
vtp domain 202300733
vtp version 2
vtp password area6
spanning-tree mode rapid-pvst
interface range FastEthernet0/1 - 24
 switchport mode access
 switchport access vlan 63
 spanning-tree portfast
 no shutdown
end
write memory
```

---

### SW-SG-P — VTP Server / Root Bridge VLAN 73

```
enable
configure terminal
hostname SW-SG-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Seguridad PNC - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area7
spanning-tree mode rapid-pvst
spanning-tree vlan 73 priority 4096
interface range FastEthernet0/1 - 8
 switchport mode access
 switchport access vlan 73
 spanning-tree portfast
 no shutdown
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface FastEthernet0/6
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/9 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

### SW-AA-P — VTP Server / Root Bridge VLAN 83

```
enable
configure terminal
hostname SW-AA-P
enable secret 202300733
line console 0
 password 202300733
 login
line vty 0 4
 password 202300733
 login
service password-encryption
no ip domain-lookup
banner motd # Aeropuerto La Aurora - Area Administrativa - 202300733 #
vtp mode server
vtp domain 202300733
vtp version 2
vtp password area8
spanning-tree mode rapid-pvst
spanning-tree vlan 83 priority 4096
interface range FastEthernet0/1 - 20
 switchport mode access
 switchport access vlan 83
 spanning-tree portfast
 no shutdown
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
 no shutdown
interface range FastEthernet0/21 - 24
 switchport mode access
 switchport access vlan 999
 shutdown
end
write memory
```

---

## Pruebas de Conectividad

### Pruebas exitosas (misma VLAN)

![Ping exitoso VLAN 13 - TP1 a TP74](img/ping_ok_vlan13.png)
![Ping exitoso VLAN 23 - TC1 a TC25](img/ping_ok_vlan23.png)
![Ping exitoso VLAN 33 - CB1 a CB32](img/ping_ok_vlan33.png)
![Ping exitoso VLAN 43 - AD1 a AD5](img/ping_ok_vlan43.png)
![Ping exitoso VLAN 53 - MG1 a MG26](img/ping_ok_vlan53.png)
![Ping exitoso VLAN 63 - SA1 a SA47](img/ping_ok_vlan63.png)
![Ping exitoso VLAN 73 - SG1 a SG8](img/ping_ok_vlan73.png)
![Ping exitoso VLAN 83 - AA1 a AA20](img/ping_ok_vlan83.png)

### Pruebas fallidas (entre distintas VLANs — comportamiento esperado)

![Ping fallido VLAN 13 a VLAN 23](img/ping_fail_13_23.png)
![Ping fallido VLAN 33 a VLAN 63](img/ping_fail_33_63.png)
![Ping fallido VLAN 53 a VLAN 83](img/ping_fail_53_83.png)

---

## Presupuesto Estimado

| Equipo | Cantidad | Precio Unit. (USD) | Total (USD) |
|--------|----------|--------------------|-------------|
| Switch Cisco 2960-24TT | 20 | $500.00 | $10,000.00 |
| PC de escritorio | 155 | $400.00 | $62,000.00 |
| Laptop | 82 | $600.00 | $49,200.00 |
| Cable UTP Cat5e Straight-Through (por metro) | 500 | $0.50 | $250.00 |
| Cable UTP Cat5e Crossover (por metro) | 100 | $0.50 | $50.00 |
| Patch Panel 24 puertos | 10 | $80.00 | $800.00 |
| Conectores RJ-45 (caja 100 unidades) | 5 | $15.00 | $75.00 |
| **TOTAL** | | | **$122,375.00** |

---

## Conclusión

La Práctica 2 permitió diseñar e implementar de forma completa la infraestructura de red del Aeropuerto Internacional La Aurora, aplicando tecnologías de capa 2 del modelo OSI en un entorno simulado con Cisco Packet Tracer.

**Subnetting VLSM:** La aplicación de VLSM permitió asignar subredes eficientes y ajustadas a la cantidad real de hosts por área, evitando el desperdicio de direcciones IP que ocurriría con una máscara uniforme /24 para todas las VLANs.

**VLANs y segmentación:** La red quedó dividida en 8 dominios de broadcast independientes, uno por área operativa, garantizando que el tráfico de cada zona quede aislado y no interfiera con las demás.

**VTP:** La distribución centralizada de VLANs mediante VTP simplificó la administración, propagando automáticamente la base de datos de VLANs desde cada switch principal hacia sus switches de acceso.

**STP Rapid PVST+:** La implementación de Rapid PVST+ con cada switch principal como Root Bridge de su VLAN garantizó la prevención de bucles y la convergencia rápida ante fallos de enlace.

**EtherChannel LACP:** La agregación de 3 enlaces físicos entre switches principales de áreas críticas (Terminal de Pasajeros, Torre de Control y Salas de Abordaje) proporcionó redundancia y mayor ancho de banda disponible en los segmentos de mayor demanda.

**Alta disponibilidad:** Las tres áreas críticas cuentan con al menos dos caminos físicos distintos hacia la red backbone, cumpliendo con el requisito de que un fallo en un enlace no interrumpa su conectividad.
