# Red Universitaria con Alta Disponibilidad

## 📖 Descripción del Proyecto
Este proyecto presenta la guía, topología y configuración paso a paso de una red de campus universitario diseñada para analizar la tolerancia a fallos. Se comparan dos infraestructuras paralelas: una red básica sin redundancia (con un único punto de fallo) y una red con alta disponibilidad que duplica equipos críticos en Capa 2 y Capa 3.

El objetivo principal es implementar y evaluar el comportamiento de los mecanismos de redundancia ante caídas de enlace o fallas en los equipos de enrutamiento y conmutación.

## 🗺️ Topología de la Red
![Topología de la red con y sin redundancia](topologia.png)

* **Red sin redundancia (Izquierda):** Un único router, un switch de acceso, tres VLANs y el servidor web. Si el router o switch falla, la red se cae por completo.
* **Red con redundancia (Derecha):** Dos switches de acceso, EtherChannel entre ellos, dos routers operando con redundancia de gateway (FHRP/HSRP) y un router de salida hacia el servidor.

---

## 🏗️ Estructura y Configuración de la Red

### 1. Segmentación de Red (VLANs)
| VLAN | Grupo | Red | PCs (Hosts) | Gateway Virtual (HSRP) |
|---|---|---|---|---|
| 10 | Estudiantes | 192.168.10.0/24 | .4 y .5 | 192.168.10.1 |
| 20 | Cámaras | 192.168.20.0/24 | .4 y .5 | 192.168.20.1 |
| 30 | Laboratorio | 192.168.30.0/24 | .4 y .5 | 192.168.30.1 |

### 2. Tecnologías e Implementación Técnica
* **EtherChannel (LACP):** Agrupación de enlaces entre SW-A y SW-B (puertos Fa0/10-11) para evitar bucles y garantizar ancho de banda y tolerancia a fallos en Capa 2.
* **FHRP / HSRP:** Balanceo y redundancia de primer salto entre Router3 y Router2:
  * **Router3:** Activo para VLAN 10 (Estudiantes) y VLAN 20 (Cámaras) [Prioridad 110].
  * **Router2:** Activo para VLAN 30 (Laboratorio) [Prioridad 110].
* **Enrutamiento OSPF:** Conectividad dinámica entre los routers de la red interna (Router2 y Router3) y el router de salida (Router4) hacia la red `200.0.0.0/24` del servidor (`cct.com`).

---

## 🧪 Resumen de Configuración por Pasos

### En Switches (SW-A y SW-B)
```bash
! Crear VLANs
vlan 10
 name ESTUDIANTES
vlan 20
 name CAMARAS
vlan 30
 name LABORATORIO

! Configurar EtherChannel entre switches
interface range FastEthernet0/10-11
 channel-group 1 mode active
interface port-channel 1
 switchport mode trunk
```

### En Routers (Ejemplo HSRP en Router3)
```bash
! Subinterfaz VLAN 10 (Estudiantes - Activo)
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt

! Subinterfaz VLAN 30 (Laboratorio - Standby)
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.2 255.255.255.0
 standby 30 ip 192.168.30.1
 standby 30 priority 90
```

---

## 📊 Pruebas de Falla y Medición de Convergencia

> *Nota: Esta sección se actualizará próximamente con las mediciones de tiempo de recuperación (failover) obtenidas en la simulación.*

### Metodología de Prueba
1. Dejar un `ping -t` continuo desde una PC hacia la dirección IP del servidor (`200.0.0.2`).
2. Provocar la falla de forma intencional (desconectar un enlace de EtherChannel o apagar el router activo).
3. Contar la cantidad de paquetes perdidos y calcular el tiempo en segundos/milisegundos que le toma a la red volver a estabilizarse.

| Escenario de Falla | Comportamiento Red SIN Redundancia | Tiempo de Failover Red CON Redundancia |
|---|---|---|
| Apagado de Router Activo (HSRP) | Caída permanente | *[Pendiente]* |
| Desconexión de un enlace EtherChannel | Caída permanente | *[Pendiente]* |
| Falla de enlace hacia Router4 (WAN) | Caída permanente | *[Pendiente]* |

---

## 📁 Archivos del Repositorio
* `Red Universitaria con Alta Disponibilidad.docx`: Guía detallada y marco teórico del proyecto.
* `red redundancia cct.pkt`: Archivo de simulación ejecutable en Cisco Packet Tracer.
* `topologia.png`: Imagen del diagrama de red.
