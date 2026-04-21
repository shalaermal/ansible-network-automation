# Ansible Network Automation Lab — SP-Grade Infrastructure

> End-to-end Service Provider network automation using Ansible, GitHub Actions CI/CD, and NetBox — simulating real-world carrier infrastructure with Cisco IOS devices in EVE-NG.

---

## Network Topology

```
                    ┌─────────────────────────────────────────┐
                    │           PROVIDER NETWORK (AS 65000)   │
                    │                                         │
  CE1 (AS65001)     │    ┌────────┐      ┌────────┐           │     CE2 (AS65002)
  10.10.20.0/24 ────┼────┤  PE1   ├──────┤  PE2   ├───────────┼──── 10.20.20.0/24
  10.10.30.0/24     │    │10.255.0.3    │10.255.0.4│          │     10.20.30.0/24
       │            │    └────┬───┘      └────┬───┘           │          │
      SW1           │         │    iBGP RR    │               │         SW2
  (VLAN 20,30)      │    ┌────┴───┐      ┌────┴───┐           │    (VLAN 20,30)
                    │    │   P1   ├──────┤   P2   │           │
                    │    │10.255.0.1    │10.255.0.2│          │
                    │    │  IS-IS │      │  IS-IS │           │
                    │    │  RR    │      │  RR    │           │
                    │    └────────┘      └────────┘           │
                    └─────────────────────────────────────────┘

  Management Network: 192.168.0.x/24
  Transit Links:      10.0.x.x/30
  Loopbacks:          10.255.0.x/32
  CE1 LAN:            10.10.20.0/24 (VLAN 20), 10.10.30.0/24 (VLAN 30)
  CE2 LAN:            10.20.20.0/24 (VLAN 20), 10.20.30.0/24 (VLAN 30)
```

---

## IP Addressing

| Device    | Interface    | IP Address      | Role                |
|-----------|-------------|-----------------|---------------------|
| Router_P1 | Gi0/0       | 192.168.0.11/24 | Management          |
| Router_P1 | Gi0/1       | 10.0.12.1/30    | Transit → P2        |
| Router_P1 | Gi0/2       | 10.0.13.1/30    | Transit → PE1       |
| Router_P1 | Gi0/3       | 10.0.14.1/30    | Transit → PE2       |
| Router_P1 | Lo0         | 10.255.0.1/32   | Router-ID / LDP     |
| Router_P2 | Gi0/1       | 10.0.12.2/30    | Transit → P1        |
| Router_P2 | Gi0/2       | 10.0.24.1/30    | Transit → PE2       |
| Router_P2 | Gi0/3       | 10.0.21.1/30    | Transit → PE1       |
| Router_P2 | Lo0         | 10.255.0.2/32   | Router-ID / LDP     |
| Router_PE1| Gi0/1       | 10.0.13.2/30    | Transit → P1        |
| Router_PE1| Gi0/2       | 10.2.1.1/30     | CE1 uplink (VRF)    |
| Router_PE1| Gi0/3       | 10.0.21.2/30    | Transit → P2        |
| Router_PE1| Lo0         | 10.255.0.3/32   | Router-ID / LDP     |
| Router_PE2| Gi0/1       | 10.0.14.2/30    | Transit → P1        |
| Router_PE2| Gi0/2       | 10.2.2.1/30     | CE2 uplink (VRF)    |
| Router_PE2| Gi0/3       | 10.0.24.2/30    | Transit → P2        |
| Router_PE2| Lo0         | 10.255.0.4/32   | Router-ID / LDP     |
| Router_CE1| Gi0/1       | 10.2.1.2/30     | PE1 uplink          |
| Router_CE1| Gi0/2.20    | 10.10.20.1/24   | VLAN 20 (Users)     |
| Router_CE1| Gi0/2.30    | 10.10.30.1/24   | VLAN 30 (Voice)     |
| Router_CE2| Gi0/1       | 10.2.2.2/30     | PE2 uplink          |
| Router_CE2| Gi0/2.20    | 10.20.20.1/24   | VLAN 20 (Users)     |
| Router_CE2| Gi0/2.30    | 10.20.30.1/24   | VLAN 30 (Voice)     |

---

## Stack Overview

```
┌─────────────────────────────────────────────────────────┐
│  Layer 3  │  BGP (iBGP RR + eBGP PE-CE)                 │
├───────────┼─────────────────────────────────────────────┤
│  Layer 2.5│  MPLS L3VPN (LDP + VRF + MP-BGP)            │
├───────────┼─────────────────────────────────────────────┤
│  Layer 2  │  IS-IS Level-2 (Core IGP)                   │
├───────────┼─────────────────────────────────────────────┤
│  Layer 1  │  Physical Links (EVE-NG)                    │
└─────────────────────────────────────────────────────────┘
```

---

## Technologies Used

| Category         | Technology                                      |
|------------------|-------------------------------------------------|
| Automation       | Ansible, GitHub Actions CI/CD                   |
| Inventory        | NetBox (Source of Truth), Static YAML           |
| Protocols        | IS-IS, BGP (iBGP RR + eBGP), MPLS LDP, OSPF    |
| Services         | MPLS L3VPN, VRF, DHCP, 802.1Q Trunk            |
| Platforms        | Cisco IOS (`cisco.ios` collection)              |
| Simulation       | EVE-NG Community                                |
| Connectivity     | Tailscale (secure tunnel to lab)                |
| Monitoring       | Grafana, Nagios, Prometheus                     |

---

## Project Structure

```
ansible-network-automation/
├── .github/
│   └── workflows/
│       ├── network-ci.yml       # Main CI/CD pipeline
│       └── rollback.yml         # Manual rollback workflow
├── inventories/
│   └── lab/
│       ├── hosts.yml            # Device inventory
│       ├── group_vars/
│       │   ├── all.yml          # Global vars (credentials, AS numbers, VRF)
│       │   ├── cisco_ios.yml    # IOS connection settings
│       │   └── access_switches.yml
│       └── host_vars/
│           ├── router_P1.yml    # P1 — IS-IS NET, BGP RR, MPLS interfaces
│           ├── router_P2.yml    # P2 — IS-IS NET, BGP RR, MPLS interfaces
│           ├── router_PE1.yml   # PE1 — IS-IS, MPLS, VRF, CE neighbor
│           ├── router_PE2.yml   # PE2 — IS-IS, MPLS, VRF, CE neighbor
│           ├── router_CE1.yml   # CE1 — BGP, customer networks, DHCP
│           ├── router_CE2.yml   # CE2 — BGP, customer networks, DHCP
│           ├── sw01.yml
│           └── sw02.yml
├── playbooks/
│   ├── site.yml                 # Main playbook — full stack deployment
│   ├── isis.yml                 # IS-IS standalone playbook
│   ├── ping.yml                 # Connectivity check
│   ├── precheck.yml             # Pre-change validation + backup
│   ├── postcheck.yml            # Post-change validation
│   ├── rollback.yml             # Configuration rollback
│   └── mop/
│       ├── vlan_mop.yml         # VLAN change MOP
│       ├── precheck.yml
│       ├── deploy.yml
│       ├── postcheck.yml
│       └── rollback.yml
├── roles/
│   ├── cisco_base/              # Base config (SSH, users, NTP, logging)
│   ├── router_interfaces/       # Interface IP configuration
│   ├── subinterfaces/           # 802.1Q sub-interfaces
│   ├── dhcp/                    # DHCP pools
│   ├── ospf/                    # OSPF (legacy, CE areas)
│   ├── access_switch/           # Switch VLANs, trunk, SVI
│   ├── vlan_lifecycle/          # VLAN deploy/validate/rollback
│   ├── isis/                    # IS-IS Level-2 core
│   ├── bgp/                     # BGP RR + PE + CE
│   └── mpls_l3vpn/              # MPLS LDP + VRF + L3VPN
├── backups/                     # Device configuration backups
├── logs/                        # Verification outputs per device
├── requirements.yml
└── ansible.cfg
```

---

## Roles Overview

| Role              | Hosts              | Purpose                                              |
|-------------------|--------------------|------------------------------------------------------|
| `cisco_base`      | All                | SSH, users, NTP, syslog, logging level               |
| `router_interfaces` | All routers      | Physical interface IP addressing                     |
| `subinterfaces`   | CE routers         | 802.1Q sub-interfaces for VLANs                     |
| `dhcp`            | CE routers         | DHCP pools per VLAN                                  |
| `ospf`            | Selected           | OSPF (legacy, available but not active on P routers) |
| `access_switch`   | Switches           | VLANs, trunk ports, SVI, default gateway             |
| `vlan_lifecycle`  | Switches           | VLAN lifecycle: deploy → verify → rollback           |
| `isis`            | P + PE routers     | IS-IS Level-2, point-to-point, metric-style wide     |
| `bgp`             | P + PE + CE        | iBGP Route Reflector, VPNv4, eBGP PE-CE             |
| `mpls_l3vpn`      | PE routers         | MPLS LDP, VRF CUSTOMER_A, BGP VRF                   |

---

## NetBox Integration

NetBox is used as **Source of Truth** for device inventory and IP addressing.

```
NetBox
  └── Devices → feeds Ansible dynamic inventory
  └── IP Addresses → validates against host_vars
  └── Interfaces → used for IS-IS/MPLS interface lists
  └── VRFs → VRF definitions per customer

Dynamic inventory:
  ansible-playbook -i inventories/netbox.yml playbooks/site.yml
```

NetBox inventory file: `inventories/netbox.yml`

When a new device is added to NetBox → Ansible automatically picks it up via the NetBox plugin without modifying `hosts.yml`.

---

## CI/CD Pipeline

Located in `.github/workflows/network-ci.yml`

```
Git Push → GitHub Actions
              │
              ├── 1. git pull on EVE-NG server
              ├── 2. Run site.yml
              │       ├── Configure switches
              │       ├── Configure routers (interfaces, DHCP, subinterfaces)
              │       ├── Configure IS-IS (P + PE routers)
              │       ├── [BGP — tag: never, manual only]
              │       └── [MPLS — tag: never, manual only]
              └── 3. Save logs to ./logs/
```

### Manual Execution — BGP & MPLS

BGP and MPLS are tagged `never` in `site.yml` to prevent accidental deployment via CI/CD. Execute manually:

```bash
# Deploy BGP
ansible-playbook playbooks/site.yml \
  -i inventories/lab/hosts.yml \
  --tags bgp

# Deploy MPLS L3VPN
ansible-playbook playbooks/site.yml \
  -i inventories/lab/hosts.yml \
  --tags mpls
```

### Rollback

```bash
# Via CLI
ansible-playbook playbooks/rollback.yml \
  -i inventories/lab/hosts.yml

# Via GitHub Actions
Actions → Manual Rollback → Run workflow
```

---

## IS-IS Configuration

- **Type:** Level-2 only (SP standard)
- **Metric:** Wide (required for MPLS TE)
- **Interfaces:** `point-to-point` on all physical links (no DIS election)
- **Loopback:** Registered in IS-IS for LDP reachability

```
NET addressing format:
49.0001.0102.5500.000X.00
│    │    └──────────┘  └── NSEL (always 00)
│    │    System ID (from Loopback IP)
│    └── Area ID
└── AFI (49 = private)
```

---

## BGP Design

```
Route Reflectors: P1 (10.255.0.1), P2 (10.255.0.2)
RR Clients:       PE1, PE2
AS Number:        65000 (provider)

iBGP sessions use Loopback addresses (redundant paths via IS-IS)
VPNv4 address-family with extended communities (Route Target)

eBGP:
  PE1 ↔ CE1 (AS 65001)
  PE2 ↔ CE2 (AS 65002)
```

---

## MPLS L3VPN Design

```
VRF:            CUSTOMER_A
RD:             65000:1
Route-Target:   65000:100 (import + export)

Label stack:
  [Outer Label] = LSP label (transport, swapped by P routers)
  [Inner Label] = VPN label (VRF identification at PE egress)

PHP (Penultimate Hop Popping) enabled by default via LDP Implicit Null
```

---

## MOP — Method of Procedure

All changes follow a structured MOP:

```
1. Precheck  → validate current state, backup configs
2. Deploy    → apply changes via Ansible
3. Postcheck → verify expected state
4. Rollback  → restore from backup if needed (manual trigger)
```

MOP playbooks located in `playbooks/mop/`.

---

## Verification Commands

```bash
# IS-IS
show isis neighbors
show ip route isis
show isis database

# BGP
show bgp vpnv4 unicast all summary
show bgp ipv4 unicast summary   # on CE

# MPLS
show mpls ldp neighbor
show mpls forwarding-table
show mpls interfaces

# VPN end-to-end
show ip route vrf CUSTOMER_A
ping vrf CUSTOMER_A 10.20.20.1 source 10.10.20.1
```

---

## Lab Tasks Completed

| Task | Description                        | Status |
|------|------------------------------------|-------|
| 1    | IP Addressing                      |  Done |
| 2    | Loopbacks                          |  Done |
| 3    | Static Routes                      |  Done |
| 4    | OSPF Single-Area                   |  Done |
| 5    | OSPF Advanced (multi-area, stub)   |  Done |
| 6    | Hosts / LANs / Switches            |  Done |
| 7    | ACL / Prefix-list / Route-map      |  Done |
| 8    | IS-IS                              |  Done |
| 9    | BGP + Route Reflector              |  Done |
| 10   | MPLS L3VPN                         |  Done |

---

## Best Practices Implemented

-  Infrastructure as Code (IaC)
-  Idempotent configurations
-  NetBox as Source of Truth
-  Separation of concerns (roles)
-  CI/CD with controlled execution
-  Automated precheck / postcheck
-  Safe manual rollback strategy
-  Per-device verification logs
-  MOP-based change management

---

## Author

Built as a hands-on SP-grade network automation lab simulating real carrier infrastructure and production workflows.

> Cisco IOS · Ansible · GitHub Actions · NetBox · EVE-NG · IS-IS · BGP · MPLS L3VPN
