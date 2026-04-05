# Networking-Projects_
Networking labs that showcase skills in the field 
# 🌐 OSPF Routing — Cisco Packet Tracer Lab



---

## 📋 Table of Contents

- [Overview](#overview)
- [Network Topology](#network-topology)
- [Device Inventory](#device-inventory)
- [IP Addressing Scheme](#ip-addressing-scheme)
- [Cabling](#cabling)
- [Router Configuration](#router-configuration)
- [Verification & Testing](#verification--testing)


---

## Overview

This lab configures OSPF routing across three interconnected routers. Unlike RIP, OSPF is a **link-state** protocol that supports larger networks, faster convergence, and more efficient routing decisions using **cost metrics based on bandwidth**.

**Goal:** Enable full connectivity between all six PCs across three separate subnets using OSPF dynamic routing.

---

## Network Topology

```
        192.168.1.10 / .11                     192.168.3.14 / .15
              |                                       |
           [Sw0]                                   [Sw2]
              |                                       |
            [R3] ————————— (Serial) ————————————— [R1]
               \                                   /
                \————————— [R2] ——————————————————/
                             |
                           [Sw1]
                             |
               192.168.2.12 / .13
```

**Subnets:**
- `192.168.1.0/24` → Left LAN (Sw0, R3)
- `192.168.2.0/24` → Bottom LAN (Sw1, R2)
- `192.168.3.0/24` → Right LAN (Sw2, R1)
- `10.0.0.0/30`, `11.0.0.0/30`, `12.0.0.0/30` → WAN links between routers

---

## Device Inventory

| Device | Type | Label |
|---|---|---|
| Router x3 | Router-PT-Empty | R1, R2, R3 |
| Switch x3 | 2950T-24 | Sw0, Sw1, Sw2 |
| PC x6 | PC-PT | 192.168.1.10, .11, .12, .13, .14, .15 |

> **Note:** Use `Router-PT-Empty` devices. Each router requires **two Serial (PT-ROUTER-NM-1S)** and **two FastEthernet (PT-ROUTER-NM-1CFE)** modules. Power off the router before adding modules, then power back on.

---

## IP Addressing Scheme

### PC Configuration

| PC | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC0 (192.168.1.10) | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 (192.168.1.11) | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| PC2 (192.168.2.12) | 192.168.2.12 | 255.255.255.0 | 192.168.2.1 |
| PC3 (192.168.2.13) | 192.168.2.13 | 255.255.255.0 | 192.168.2.1 |
| PC4 (192.168.3.14) | 192.168.3.14 | 255.255.255.0 | 192.168.3.1 |
| PC5 (192.168.3.15) | 192.168.3.15 | 255.255.255.0 | 192.168.3.1 |

### Subnet Allocation

| Subnet | Devices | Subnet Mask |
|---|---|---|
| 192.168.1.0/24 | PC0, PC1, R0 | 255.255.255.0 |
| 192.168.2.0/24 | PC2, PC3, R1 | 255.255.255.0 |
| 192.168.3.0/24 | PC4, PC5, R2 | 255.255.255.0 |
| 10.0.0.0/30 | R0 ↔ R1 | 255.0.0.0 |
| 11.0.0.0/30 | R1 ↔ R2 | 255.0.0.0 |
| 12.0.0.0/30 | R0 ↔ R2 | 255.0.0.0 |

---

## Cabling

### Copper Straight-Through

| From | To | Ports |
|---|---|---|
| PC0 | Sw0 | fa0/1 |
| PC1 | Sw0 | fa0/2 |
| Sw0 | R0 | fa0/24 → fa2/0 |
| PC2 | Sw1 | fa0/1 |
| PC3 | Sw1 | fa0/2 |
| Sw1 | R1 | fa0/24 → fa2/0 |
| PC4 | Sw2 | fa0/1 |
| PC5 | Sw2 | fa0/2 |
| Sw2 | R2 | fa0/24 → fa2/0 |

### Serial DTE

| From | To | Ports |
|---|---|---|
| R0 | R1 | se0/0 ↔ se1/0 |
| R1 | R2 | se0/0 ↔ se1/0 |
| R0 | R2 | se1/0 ↔ se0/0 |

---

## Router Configuration

### R0

```cisco
enable
configure terminal
hostname R0

interface fa2/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

interface se0/0
 ip address 10.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
exit

interface se1/0
 ip address 12.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
exit

router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.255.255.255 area 0
 network 12.0.0.0 0.255.255.255 area 0
exit

write memory
```

### R1

```cisco
enable
configure terminal
hostname R1

interface fa2/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

interface se1/0
 ip address 10.0.0.2 255.0.0.0
 no shutdown
exit

interface se0/0
 ip address 11.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
exit

router ospf 1
 network 192.168.2.0 0.0.0.255 area 0
 network 10.0.0.0 0.255.255.255 area 0
 network 11.0.0.0 0.255.255.255 area 0
exit

write memory
```

### R2

```cisco
enable
configure terminal
hostname R2

interface fa2/0
 ip address 192.168.3.1 255.255.255.0
 no shutdown
exit

interface se1/0
 ip address 11.0.0.2 255.0.0.0
 no shutdown
exit

interface se0/0
 ip address 12.0.0.2 255.0.0.0
 no shutdown
exit

router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 11.0.0.0 0.255.255.255 area 0
 network 12.0.0.0 0.255.255.255 area 0
exit

write memory
```

---

## Verification & Testing

### Check OSPF Routing Tables

Run on each router to confirm OSPF routes (`O`) are present:

```cisco
show ip route
```

To show only OSPF routes:

```cisco
show ip route O
```

You should see entries like:
```
O   192.168.2.0/24 [110/65] via 10.0.0.2, Serial0/0
O   192.168.3.0/24 [110/65] via 12.0.0.2, Serial1/0
```

### Test Connectivity (from PC0)

```bash
ping 192.168.1.11   # same subnet
ping 192.168.2.12   # across R1
ping 192.168.3.14   # across R2
```

### Cross-Network Ping (from PC3)

```bash
ping 192.168.3.15
```

> ⚠️ The first ping packet may time out — this is normal as OSPF finishes converging. Subsequent packets should succeed.

---

## Related Tutorials

| # | Topic |
|---|---|
| **11** | **OSPF Routing in Packet Tracer** ← you are here |

---

## 📁 Files

| File | Description |
|---|---|
| `*.pkt` | Cisco Packet Tracer project file |
| `README.md` | This file |
---
