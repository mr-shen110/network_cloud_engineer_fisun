DRAFT

Cloud-net-project_evpn-vxlan






# EVPN/VXLAN Фабрика

> Архитектура: 2 POD (POD-A, POD-B), Super-Spine, IS-IS underlay, BGP EVPN overlay, MLAG, Anycast Gateway
> Статус: Рабочая конфигурация

---

## 1. Таблица адресного пространства

### Подсети

| Префикс | Назначение | Используется? |
|--------|-----------|---------------|
| 10.0.0.0/24 | Loopback'и, MLAG Peer-Link | ✅ Да |
| 172.16.0.0/16 | POD-A Underlay (Leaf → Spine) | ✅ Частично (`172.16.1.0/31`, `172.16.2.0/31`) |
| 172.18.0.0/16 | POD-B Underlay (Leaf → Spine) | ✅ Частично (`172.18.1.0/31`, `172.18.2.0/31`) |
| 172.17.0.0/24 | Super-Spine линки (Spine → Super-Spine) | ✅ Да |
| 192.168.10.0/24 | Клиентская сеть (VLAN 10) | ✅ Да |
| 172.16.100.0/24 | Management (резерв) | ⚠️ Не используется (можно выделить) |

---

### Loopback'и

| Устройство | Loopback0 | Назначение |
|-----------|-----------|-----------|
| Leaf-A1 | 10.0.0.11/32 | BGP RID, VTEP |
| Leaf-A2 | 10.0.0.12/32 | BGP RID, VTEP |
| Leaf-B1 | 10.0.0.51/32 | BGP RID, VTEP |
| Leaf-B2 | 10.0.0.52/32 | BGP RID, VTEP |
| Spine-A1 | 10.0.0.21/32 | BGP RID, IS-IS |
| Spine-A2 | 10.0.0.22/32 | BGP RID, IS-IS |
| Spine-B1 | 10.0.0.41/32 | BGP RID, IS-IS |
| Spine-B2 | 10.0.0.42/32 | BGP RID, IS-IS |
| Super-Spine-1 | 10.0.0.31/32 | BGP RID, IS-IS |
| Super-Spine-2 | 10.0.0.32/32 | BGP RID, IS-IS |

---

### P2P Underlay (Leaf → Spine)

| Устройство | Интерфейс | IP | Назначение |
|-----------|----------|-----|-----------|
| Leaf-A1 → Spine-A1 | Et2 ↔️ Et4 | 172.16.1.1/31 ↔️ 172.16.1.0/31 | IS-IS L2 |
| Leaf-A2 → Spine-A1 | Et2 ↔️ Et5 | 172.16.1.3/31 ↔️ 172.16.1.2/31 | IS-IS L2 |
| Leaf-A1 → Spine-A2 | Et3 ↔️ Et4 | 172.16.2.1/31 ↔️ 172.16.2.0/31 | IS-IS L2 |
| Leaf-A2 → Spine-A2 | Et3 ↔️ Et5 | 172.16.2.3/31 ↔️ 172.16.2.2/31 | IS-IS L2 |
| Leaf-B1 → Spine-B1 | Et2 ↔️ Et4 | 172.18.1.1/31 ↔️ 172.18.1.0/31 | IS-IS L2 |
| Leaf-B2 → Spine-B1 | Et2 ↔️ Et5 | 172.18.2.1/31 ↔️ 172.18.2.0/31 | IS-IS L2 |
| Leaf-B1 → Spine-B2 | Et3 ↔️ Et4 | 172.18.3.1/31 ↔️ 172.18.3.0/31 | IS-IS L2 |
| Leaf-B2 → Spine-B2 | Et3 ↔️ Et5 | 172.18.4.1/31 ↔️ 172.18.4.0/31 | IS-IS L2 |

---

### P2P Underlay (Spine → Super-Spine)

| Устройство | Интерфейс | IP | Назначение |
|-----------|----------|-----|-----------|
| Spine-A1 → Super-Spine-1 | Et6 ↔️ Et1 | 172.17.1.1/31 ↔️ 172.17.1.0/31 | IS-IS L2 |
| Spine-A2 → Super-Spine-1 | Et6 ↔️ Et2 | 172.17.2.1/31 ↔️ 172.17.2.0/31 | IS-IS L2 |
| Spine-B1 → Super-Spine-1 | Et6 ↔️ Et3 | 172.17.3.1/31 ↔️ 172.17.3.0/31 | IS-IS L2 |
| Spine-B2 → Super-Spine-1 | Et6 ↔️ Et4 | 172.17.4.1/31 ↔️ 172.17.4.0/31 | IS-IS L2 |
| Spine-A1 → Super-Spine-2 | Et7 ↔️ Et1 | 172.17.5.1/31 ↔️ 172.17.5.0/31 | IS-IS L2 |
| Spine-A2 → Super-Spine-2 | Et7 ↔️ Et2 | 172.17.6.1/31 ↔️ 172.17.6.0/31 | IS-IS L2 |
| Spine-B1 → Super-Spine-2 | Et7 ↔️ Et3 | 172.17.7.1/31 ↔️ 172.17.7.0/31 | IS-IS L2 |
| Spine-B2 → Super-Spine-2 | Et7 ↔️ Et4 | 172.17.8.1/31 ↔️ 172.17.8.0/31 | IS-IS L2 |

---

### MLAG Peer-Link (L3)

| Устройство | Интерфейс | IP | Назначение |
|-----------|----------|-----|-----------|
| Leaf-A1 ↔️ Leaf-A2 | Vlan4094 | 10.0.0.1/30 ↔️ 10.0.0.2/30 | MLAG Peer-Link |
| Leaf-B1 ↔️ Leaf-B2 | Vlan4094 | 10.0.0.5/30 ↔️ 10.0.0.6/30 | MLAG Peer-Link |

---

### Overlay (L2 VNI)

| VLAN | VNI | Назначение |
|------|-----|-----------|
| 10 | 10010 | Клиентская сеть (растянута между POD'ами) |

---

### Клиентская сеть (k8s)

| Назначение | IP | Устройство |
|-----------|-----|-----------|
| Anycast Gateway | 192.168.10.254/24 | На всех Leaf в Vlan10 |
| Host-A | 192.168.10.10/24 | Host-A |
| Host-B | 192.168.10.20/24 | Host-B |

---

## 2. Список узлов (конфигурации)

> Кликни по названию узла для ознакомления с конфигом

<details>
<summary>📍 Host-A</summary>

Host-A#sh run
! Command: show running-config
! device: Host-A (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname Host-A
!
spanning-tree mode mstp
!
vlan 10
!
interface Port-Channel100
   description Uplink to MLAG
   switchport trunk allowed vlan 10
   switchport mode trunk
!
interface Ethernet1
   description To-Leaf-A1
   channel-group 100 mode active
!
interface Ethernet2
   description To-Leaf-A2
   channel-group 100 mode active
!
interface Ethernet3
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Management1
!
interface Vlan10
   description Client Interface
   ip address 192.168.10.10/24
!
ip routing
!
ip route 0.0.0.0/0 192.168.10.254
!
end
Host-A#

</details>




<details>
<summary>📍 Host-B</summary>

</details>





<details>
<summary>📍 Leaf-A1</summary>

</details>





<details>
<summary>📍 Leaf-A1</summary>

</details>




<details>
<summary>📍 Leaf-B1</summary>

</details>





<details>
<summary>📍 Leaf-B2</summary>

</details>




<details>
<summary>📍 Spine-A1</summary>

</details>



<details>
<summary>📍 Spine-A2</summary>

</details>





<details>
<summary>📍 Spine-B1</summary>

</details>




<details>
<summary>📍 Spine-B2</summary>

</details>





<details>
<summary>📍 Super-Spine-1</summary>

</details>





<details>
<summary>📍 Super-Spine-2</summary>

</details>



---

## Примечания

- ✅ IS-IS — основа underlay, уровень L2
- ✅ BGP EVPN — только на Leaf (AS 65001), Spine не участвуют в BGP
- ✅ MLAG — включён, домены: POD-A, POD-B
- ✅ Anycast Gateway: 192.168.10.254, MAC: 00:00:11:11:22:22
- ✅ VXLAN: VNI 10010, flood list: все VTEP
- ✅ Route Reflector: Leaf-A1 (`10.0.0.11`)

---
