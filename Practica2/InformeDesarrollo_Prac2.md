# INFORME DE DESARROLLO — Práctica 2: Red del Aeropuerto Internacional La Aurora

---

**Institución:** Universidad San Carlos de Guatemala  
**Facultad:** Ingeniería en Ciencias y Sistemas  
**Curso:** Redes de Computadoras 1  
**Autor:** Ángel Rafael Barrios González  
**Carnet:** 202300733  
**Fecha:** 09 de abril de 2026  

---

## Índice

1. [Descripción General del Proyecto](#descripción-general-del-proyecto)
2. [Proceso de Implementación](#proceso-de-implementación)
3. [Problemas Encontrados y Soluciones](#problemas-encontrados-y-soluciones)
4. [Decisiones de Diseño Justificadas](#decisiones-de-diseño-justificadas)
5. [Capturas de Funcionamiento](#capturas-de-funcionamiento)
6. [Conclusiones del Desarrollo](#conclusiones-del-desarrollo)

---

## Descripción General del Proyecto

Esta práctica consistió en diseñar e implementar desde cero la infraestructura de red del **Aeropuerto Internacional La Aurora** de Guatemala, utilizando Cisco Packet Tracer como herramienta de simulación.

El proyecto se estructuró en las siguientes fases de trabajo:

| Fase | Descripción | Estado |
|------|-------------|--------|
| 0 | Diseño de arquitectura física (áreas, hosts, switches, cables) | ✅ Completada |
| 1 | Asignación de IPs a todos los hosts mediante VLSM | ✅ Completada |
| 2 | Configuración básica de los 20 switches | ✅ Completada |
| 3 | Creación de VLANs en switches principales | ✅ Completada |
| 4 | Configuración de VTP (servidor y clientes) | ✅ Completada |
| 5 | Configuración de puertos de acceso, trunks, VLAN nativa y Blackhole | ✅ Completada |
| 6 | Implementación de STP Rapid PVST+ | ✅ Completada |
| 7 | Implementación de EtherChannel LACP | ✅ Completada |
| 8 | Verificación y pruebas de conectividad | ✅ Completada |

---

## Proceso de Implementación

### Fase 0 — Arquitectura Física

La primera decisión fue identificar las **8 áreas operativas reales** del Aeropuerto La Aurora. Se investigaron las instalaciones reales del aeropuerto para seleccionar áreas que tuvieran sentido operativo:

1. Terminal de Pasajeros (la más grande, 74 hosts)
2. Torre de Control DGAC (crítica para la seguridad aérea, 25 hosts)
3. Área de Carga y Bodega (32 hosts)
4. Aduana SAT (5 hosts, la más pequeña)
5. Migración (26 hosts)
6. Salas de Abordaje / Gates (47 hosts, la segunda más grande)
7. Seguridad / PNC (8 hosts)
8. Área Administrativa (20 hosts)

Se determinó que las áreas **críticas** para la operación aeroportuaria son la Terminal de Pasajeros, la Torre de Control y las Salas de Abordaje, ya que su fallo tiene impacto directo en la continuidad operativa del aeropuerto. Estas tres áreas recibieron conexiones EtherChannel redundantes entre sus switches principales.

Para los switches, se eligió exclusivamente el modelo **Cisco 2960-24TT** por ser el estándar de la práctica. Al tener solo 24 puertos FastEthernet por switch y considerar que el puerto FA0/24 se reserva para la conexión hacia el switch principal, cada switch de acceso puede conectar máximo **23 hosts**. Esto determinó la cantidad de switches necesarios por área.

**Total de switches desplegados: 20** (8 principales + 12 de acceso)  
**Total de hosts configurados: 237** (155 PCs + 82 Laptops)

### Fase 1 — Subnetting VLSM

Se aplicó VLSM partiendo de la red base `192.168.10.0`. Las áreas se ordenaron de mayor a menor cantidad de hosts y se asignaron subredes del tamaño justo para cada una. Esto permitió aprovechar eficientemente el espacio de direcciones usando únicamente dos bloques /24: `192.168.10.0/24` y parte de `192.168.11.0/24`.

A cada host se le configuró manualmente su IP, máscara y gateway desde la opción Desktop > IP Configuration > Static en Packet Tracer.

### Fases 2 a 7 — Configuración CLI de Switches

Toda la configuración de los switches se realizó mediante **CLI (Command Line Interface)** en modo configuración global. El orden seguido fue siempre el mismo para mantener consistencia:

1. Hostname y contraseñas
2. Banner motd
3. VTP (modo y credenciales)
4. STP (modo Rapid PVST+ y priority del root bridge)
5. Creación de VLANs (solo en servers)
6. Configuración de interfaces (acceso, trunk, EtherChannel, Blackhole)
7. `write memory` para guardar

---

## Problemas Encontrados y Soluciones

### Problema 1 — Límite de 24 puertos por switch subestimado

**Descripción:** En el diseño inicial se calculó que con un solo switch de acceso por área era suficiente para conectar todos los hosts. Sin embargo, al considerar que el puerto FA0/24 debe reservarse para la conexión hacia el switch principal, solo quedan **23 puertos** disponibles para hosts. Áreas con más de 23 hosts necesitaron switches adicionales.

**Áreas afectadas y solución:**

| Área | Hosts | Switches de acceso iniciales | Switches adicionales requeridos |
|------|-------|-----------------------------|---------------------------------|
| Terminal Pasajeros | 74 | 3 (solo 69 hosts) | +1 SW-TP-A4 para TP70–TP74 |
| Torre de Control | 25 | 1 (solo 23 hosts) | +1 SW-TC-A2 para TC24–TC25 |
| Carga y Bodega | 32 | 1 (solo 23 hosts) | +1 SW-CB-A2 para CB24–CB32 |
| Migración | 26 | 1 (solo 23 hosts) | +1 SW-MG-A2 para MG24–MG26 |

**Solución:** Se agregaron los switches faltantes y se redistribuyeron los hosts. El total final fue de **20 switches** en lugar de los 16 originalmente planificados.

---


### Problema 2 — Planificación de puertos en switches principales con EtherChannel

**Descripción:** Los switches principales de áreas críticas necesitan conectar tanto a sus switches de acceso (trunks simples) como a otros switches principales (EtherChannel de 3 cables). Al sumar los puertos necesarios había riesgo de quedarse sin puertos en el switch principal.

**Ejemplo con SW-TP-P:**
- 4 puertos para switches de acceso (A1, A2, A3, A4)
- 3 puertos para EtherChannel hacia SW-TC-P
- 3 puertos para EtherChannel hacia SW-SA-P
- Total: **10 puertos utilizados** de 24 disponibles ✅

**Solución:** Se planificó cuidadosamente la asignación de puertos antes de configurar, asignando rangos específicos: FA0/1–FA0/4 para accesos, FA0/5–FA0/7 para EtherChannel 1, FA0/8–FA0/10 para EtherChannel 2, y FA0/11–FA0/24 a VLAN 999 (Blackhole).

---

### Problema 3 — VLAN nativa y puertos trunk

**Descripción:** Al configurar los puertos trunk entre switches, si no se especifica explícitamente la VLAN nativa, Cisco usa por defecto la VLAN 1, lo que genera advertencias de seguridad en Packet Tracer y puede causar inconsistencias si un switch tiene configurada VLAN 1 como nativa y otro tiene VLAN 99.

**Solución:** En todos los puertos trunk se especificó explícitamente:
```
switchport trunk native vlan 99
switchport trunk allowed vlan 13,23,33,43,53,63,73,83,99
```
Esto garantiza que ambos extremos del trunk coincidan en la VLAN nativa y que solo viajen las VLANs necesarias, siguiendo el principio de mínimo privilegio.

---

### Problema 4 — EtherChannel requiere configuración idéntica en ambos extremos

**Descripción:** Al configurar EtherChannel LACP, si los parámetros (modo, VLANs permitidas, VLAN nativa, tipo de puerto) no son exactamente iguales en ambos switches, el Port-Channel no se levanta y los puertos quedan en estado `err-disabled` o simplemente no forman el canal.

**Solución:** Se configuraron siempre los dos switches del EtherChannel en la misma sesión, verificando que los comandos fueran idénticos en ambos extremos. Se usó el modo `active` en ambos lados (LACP activo-activo), que es la configuración más robusta.

**Verificación utilizada:**
```
show etherchannel summary
```
El estado esperado es `SU` (Switch, en uso) para el Port-Channel y `P` (en el Port-Channel) para las interfaces individuales.

---

### Problema 5 — Switches de áreas pequeñas funcionan como principal y acceso simultáneamente

**Descripción:** Las áreas con pocos hosts (Aduana SAT con 5, Seguridad PNC con 8 y Área Administrativa con 20) no requirieron switches de acceso separados porque todos sus hosts caben en el switch principal. Sin embargo, esto genera una configuración mixta donde el mismo switch tiene puertos de acceso (para hosts) y puertos trunk (hacia la red backbone).

**Solución:** Se configuraron ambos tipos de interfaces en el mismo switch, respetando el orden: primero los puertos de acceso hacia los hosts con su VLAN correspondiente, luego los puertos trunk hacia los switches principales vecinos, y finalmente los puertos no utilizados asignados a VLAN 999 y apagados con `shutdown`.

---

## Decisiones de Diseño Justificadas

### ¿Por qué topología en estrella extendida con redundancia selectiva?

Se eligió una topología de estrella extendida entre los switches principales porque ofrece el mejor balance entre simplicidad de administración y rendimiento. Cada área tiene un switch principal que centraliza el tráfico de sus switches de acceso, facilitando el diagnóstico de problemas y la gestión del STP.

La redundancia (EtherChannel de 3 enlaces) se aplicó **solo** a las tres áreas críticas para no desperdiciar puertos en áreas donde una interrupción temporal no compromete la seguridad operativa del aeropuerto.

### ¿Por qué VLSM y no subredes fijas /24?

Usar /24 para todas las VLANs hubiera asignado 254 hosts disponibles a cada área, incluso al Área de Aduana SAT que solo necesita 5. Esto habría desperdiciado más de 1,600 direcciones IP. Con VLSM cada área recibe exactamente el bloque que necesita, dejando el espacio restante disponible para futuras expansiones.

### ¿Por qué Rapid PVST+ y no PVST clásico?

Rapid PVST+ converge en menos de 1 segundo ante fallos de enlace, en comparación con los 30–50 segundos del PVST clásico (basado en 802.1D). En un entorno crítico como un aeropuerto, una interrupción de 50 segundos en la red podría afectar operaciones en tiempo real. Rapid PVST+ garantiza continuidad operativa ante fallos.

### ¿Por qué LACP y no PAgP para EtherChannel?

LACP (802.3ad) es un estándar abierto IEEE compatible con equipos de cualquier fabricante, mientras que PAgP es propietario de Cisco. Aunque en esta práctica todos los switches son Cisco, se eligió LACP por ser la opción más apropiada en entornos profesionales reales donde puede haber equipos de diferentes marcas.

### ¿Por qué el switch de acceso SA-A2 no tiene puertos en Blackhole?

El switch SW-SA-A2 conecta a SA24–SA47, que son exactamente **24 hosts**, ocupando todos sus puertos FA0/1–FA0/24. En este caso no hay puertos sin usar, por lo que no se aplica la VLAN Blackhole. El puerto FA0/24 del switch de acceso normalmente va hacia el principal, pero al estar todos los puertos ocupados por hosts, SA-A2 usa sus 24 puertos de host y el uplink hacia SW-SA-P se hace por GE0/1 o se reorganizan los puertos según disponibilidad.

---

## Capturas de Funcionamiento

### Topología implementada

![Topología general completa](img/topologia_general.png)

### Configuración de VLANs

![Show VLAN Brief en SW-TP-P](img/show_vlan_brief.png)

### Configuración VTP

![Show VTP Status](img/show_vtp_status.png)

### Configuración STP

![Show Spanning-Tree VLAN 13](img/show_spanning_tree_vlan13.png)

### EtherChannel activo

![Show EtherChannel Summary](img/show_etherchannel_summary.png)

### Puertos trunk activos

![Show Interfaces Trunk](img/show_interfaces_trunk.png)

### Pruebas de conectividad exitosas

![Ping TP1 a TP74 - misma VLAN 13](img/ping_ok_vlan13.png)
![Ping TC1 a TC25 - misma VLAN 23](img/ping_ok_vlan23.png)
![Ping SA1 a SA47 - misma VLAN 63](img/ping_ok_vlan63.png)

### Pruebas de aislamiento entre VLANs (comportamiento esperado)

![Ping fallido VLAN 13 hacia VLAN 63](img/ping_fail_13_63.png)
![Ping fallido VLAN 23 hacia VLAN 83](img/ping_fail_23_83.png)

---

## Conclusiones del Desarrollo

La implementación de la red del Aeropuerto Internacional La Aurora permitió aplicar de manera integrada los conceptos de capa 2 del modelo OSI aprendidos durante el curso.

El principal aprendizaje práctico fue la **planificación previa**. Antes de tocar un solo comando en Packet Tracer fue necesario tener completamente definida la arquitectura: cuántos switches por área, qué puerto va a qué lugar, cuáles VLANs existen, y qué IPs corresponden a cada host. Sin esa planificación, los errores de configuración en CLI son difíciles de rastrear y corregir.

El uso de **VLSM** demostró ser fundamental para una gestión eficiente del espacio de direcciones, especialmente en redes con áreas de tamaños muy distintos como es el caso del aeropuerto, donde el área más grande necesita 74 hosts y la más pequeña solo 5.

La implementación de **EtherChannel LACP** en áreas críticas mostró cómo la redundancia de enlace protege la continuidad operativa sin requerir hardware adicional, simplemente aprovechando mejor los cables físicos existentes entre switches.

Finalmente, la práctica evidenció la importancia de las **VLANs y el aislamiento de broadcast** en entornos con múltiples departamentos o áreas operativas, garantizando que el tráfico de cada zona no interfiera con las demás y que la red sea segura, ordenada y escalable.
