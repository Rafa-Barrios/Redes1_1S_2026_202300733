<div align="center">

# MANUAL TECNICO

## Practica 1

---

**Institución:** Universidad San Carlos de Guatemala  

**Autor:** Angel Rafael Barrios González  
**Carnet:** 202300733  
**Curso:** Redes de Computadoras 1  
**Fecha:** 19 de Febrero de 2026  

---

</div>

# 1. Introducción

La presente práctica tuvo como objetivo el diseño e implementación de una red empresarial de tres niveles utilizando el simulador Packet Tracer. El proyecto consistió en estructurar la conexión de una empresa distribuida en tres pisos, donde cada nivel cuenta con distintas áreas de trabajo.

Cada piso está compuesto por entre tres y cuatro áreas, y cada una de ellas dispone de un switch propio que conecta diferentes dispositivos finales, como computadoras y laptops. Estos switches de área se enlazan a un switch central por cada piso, y a su vez, los switches centrales de cada nivel se interconectan entre sí, permitiendo la comunicación y el intercambio de archivos entre todos los equipos de la empresa.

Durante el desarrollo de la práctica se aplicaron conocimientos fundamentales de redes, tales como la conexión de dispositivos mediante cable directo y cable cruzado, la asignación manual de direcciones IP a computadoras, laptops y servidores, y la configuración básica de switches, incluyendo la asignación de hostname y el establecimiento de contraseñas para mayor seguridad.

Esta práctica permitió comprender la estructura jerárquica de una red empresarial, la importancia de una correcta segmentación por áreas y la configuración adecuada de los dispositivos para garantizar una comunicación eficiente entre todos los equipos.

# 2. Tabla de Direcciones IP

| Nº | Equipo    | Nombre     | IP              |
|----|-----------|------------|-----------------|
| 1  | PC        | PC1        | 192.178.33.1    |
| 2  | PC        | PC2        | 192.178.33.2    |
| 3  | PC        | PC3        | 192.178.33.3    |
| 4  | PC        | PC4        | 192.178.33.4    |
| 5  | PC        | PC5        | 192.178.33.5    |
| 6  | PC        | PC6        | 192.178.33.6    |
| 7  | PC        | PC7        | 192.178.33.7    |
| 8  | PC        | PC8        | 192.178.33.8    |
| 9  | PC        | PC9        | 192.178.33.9    |
| 10 | PC        | PC10       | 192.178.33.10   |
| 11 | PC        | PC11       | 192.178.33.11   |
| 12 | PC        | PC12       | 192.178.33.12   |
| 13 | PC        | PC13       | 192.178.33.13   |
| 14 | PC        | PC14       | 192.178.33.14   |
| 15 | PC        | PC15       | 192.178.33.15   |
| 16 | PC        | PC16       | 192.178.33.16   |
| 17 | PC        | PC17       | 192.178.33.17   |
| 18 | PC        | PC18       | 192.178.33.18   |
| 19 | PC        | PC19       | 192.178.33.19   |
| 20 | PC        | PC20       | 192.178.33.20   |
| 21 | PC        | PC21       | 192.178.33.23   |
| 22 | PC        | PC22       | 192.178.33.24   |
| 23 | PC        | PC23       | 192.178.33.25   |
| 24 | PC        | PC24       | 192.178.33.28   |
| 25 | PC        | PC25       | 192.178.33.29   |
| 26 | PC        | PC26       | 192.178.33.35   |
| 27 | PC        | PC27       | 192.178.33.36   |
| 28 | PC        | PC28       | 192.178.33.37   |
| 29 | PC        | PC29       | 192.178.33.38   |
| 30 | PC        | PC30       | 192.178.33.39   |
| 31 | PC        | PC31       | 192.178.33.40   |
| 32 | PC        | PC32       | 192.178.33.41   |
| 33 | PC        | PC33       | 192.178.33.42   |
| 34 | PC        | PC34       | 192.178.33.43   |
| 35 | PC        | PC35       | 192.178.33.44   |
| 36 | PC        | PC36       | 192.178.33.45   |
| 37 | PC        | PC37       | 192.178.33.54   |
| 38 | Laptop    | Laptop1    | 192.178.33.21   |
| 39 | Laptop    | Laptop2    | 192.178.33.22   |
| 40 | Laptop    | Laptop3    | 192.178.33.26   |
| 41 | Laptop    | Laptop4    | 192.178.33.27   |
| 42 | Laptop    | Laptop5    | 192.178.33.30   |
| 43 | Laptop    | Laptop6    | 192.178.33.31   |
| 44 | Laptop    | Laptop7    | 192.178.33.32   |
| 45 | Laptop    | Laptop8    | 192.178.33.33   |
| 46 | Laptop    | Laptop9    | 192.178.33.34   |
| 47 | Laptop    | Laptop10   | 192.178.33.46   |
| 48 | Laptop    | Laptop11   | 192.178.33.47   |
| 49 | Laptop    | Laptop12   | 192.178.33.48   |
| 50 | Laptop    | Laptop13   | 192.178.33.49   |
| 51 | Laptop    | Laptop14   | 192.178.33.50   |
| 52 | Laptop    | Laptop15   | 192.178.33.51   |
| 53 | Laptop    | Laptop16   | 192.178.33.52   |
| 54 | Laptop    | Laptop17   | 192.178.33.53   |
| 55 | Servidor  | Server1    | 192.178.33.55   |
| 56 | Servidor  | Server2    | 192.178.33.56   |
| 57 | Servidor  | Server3    | 192.178.33.57   |
| 58 | Servidor  | Server4    | 192.178.33.58   |
| 59 | Servidor  | Server5    | 192.178.33.59   |
| 60 | Servidor  | Server6    | 192.178.33.60   |

# 3. Configuración de Switches

Este apartado muestra los comandos necesarios para la configuración básica de cada switch, permitiendo asignar un hostname y establecer una contraseña de acceso para mayor seguridad.

```bash
Switch> enable
Switch# configure terminal
Switch(config)# hostname "Nombre a Colocar"
DATACENTER(config)# line console 0
DATACENTER(config-line)# password "Contraseña a Colocar"
DATACENTER(config-line)# login
```

# 4. Departamentos Creados

## 1. Departamento de Recepción

![Departamento de Recepción](re.png)

## 2. Departamento de Contabilidad

![epartamento de Contabilidad](c.png)

## 3. Departamento Legal

![Departamento Legal](l.png)

## 4. Sala de Reuniones

![Sala de Reuniones](r.png)

## 5. Departamento de Arquitectura

![Departamento de Arquitectura](a.png)

## 6. Departamento de Urbanismo

![Departamento de Urbanismo](u.png)

## 7. Sala de revisión de planos

![Sala de revisión de planos](p.png)

## 8. Departamento de Dirección General

![Departamento de Dirección General](d.png)

## 9. Departamento de Ingeniería Civil

![Departamento de Ingeniería Civil](i.png)

## 10. Departamento de Servidores Principales

![Departamento de Servidores Principales](s.png)

# 5. Evidencia de Pings

![Ping 13](ping13.png)

![Ping 46](ping46.png)

![Ping 79](ping79.png)

![Ping 10](ping10.png)

# 6. Paquetes en Modo Simulación

![Simulación 1](s1.png)

![Simulación 2](s2.png)

![Simulación 3](s3.png)

![Simulación 4](s4.png)

![Simulación 5](s5.png)

![Simulación 6](s6.png)

![Simulación 7](s7.png)

![Simulación 8](s8.png)

![Simulación 9](s9.png)

![Simulación 10](s10.png)

# 7. Conclusión

La realización de la Práctica 1 permitió aplicar de manera práctica los conceptos fundamentales de redes de computadoras mediante el uso de Packet Tracer. A través del diseño de una red empresarial de tres niveles, se logró comprender la estructura jerárquica de conexión entre áreas de trabajo y la importancia de los switches centrales para la comunicación entre distintos pisos.

Durante el desarrollo del proyecto se configuraron dispositivos finales como PCs, laptops y servidores, asignándoles direcciones IP de forma manual para garantizar la conectividad dentro de la red. Asimismo, se realizó la configuración básica de los switches, incluyendo la asignación de hostname y contraseñas, fortaleciendo los conocimientos sobre administración y seguridad básica de dispositivos de red.

Las pruebas de conectividad mediante comandos ping y la observación de paquetes en modo simulación permitieron verificar el correcto funcionamiento de la topología implementada. En conclusión, la práctica contribuyó al fortalecimiento de las habilidades técnicas necesarias para el diseño, configuración y verificación de redes locales empresariales.

