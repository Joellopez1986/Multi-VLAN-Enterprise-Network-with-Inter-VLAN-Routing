# Multi-VLAN Network with Inter-VLAN Routing — Cisco Packet Tracer Lab

A small enterprise network built in Cisco Packet Tracer, demonstrating VLAN segmentation, inter-VLAN routing on a Layer 3 switch, and router connectivity.

## Topology

```
                         [ ROUTER0 ]  (Cisco ISR)
                             |
                     Gi0/0/0 | Gi0/1
                             |
                     [ SW-CORE ]  (Cisco 3560-24PS, Multilayer Switch)
                      /   |    |    \
                     /    |    |     \
                  ADMIN FINANCE IT  MANAGEMENT   + SERVERS + Laptop
                (PC1-4) (PC5-7)(PC8-10) (PC11-12)  (Server0)  (Laptop1)
```

- **Router0** — Layer 3 router, provides the WAN-facing / upstream link
- **SW-CORE** — Cisco 3560 multilayer switch, handles VLAN segmentation and inter-VLAN routing
- **12 PCs** — end devices split across four departments
- **1 Server** — shared resource, isolated on its own VLAN
- **1 Laptop** — additional device, joined to the ADMIN VLAN

## VLAN Design

| VLAN | Name | Devices | Network | Usable Range | Gateway (SVI) |
|---|---|---|---|---|---|
| 10 | ADMIN | PC1–PC4, Laptop1 | 192.168.10.0/27 | .2 – .30 | 192.168.10.1 |
| 20 | FINANCE | PC5–PC7 | 192.168.10.32/27 | .34 – .62 | 192.168.10.33 |
| 30 | IT | PC8–PC10 | 192.168.10.64/27 | .66 – .94 | 192.168.10.65 |
| 40 | MANAGEMENT | PC11–PC12 | 192.168.10.96/27 | .98 – .126 | 192.168.10.97 |
| 50 | SERVERS | Server0 | 192.168.10.128/27 | .130 – .158 | 192.168.10.129 |

Subnet mask for all VLANs: `255.255.255.224` (/27)

## Port Assignments (SW-CORE)

| Port(s) | Device | VLAN |
|---|---|---|
| Fa0/1 – Fa0/4 | PC1 – PC4 | 10 (ADMIN) |
| Fa0/5 – Fa0/7 | PC5 – PC7 | 20 (FINANCE) |
| Fa0/8 – Fa0/10 | PC8 – PC10 | 30 (IT) |
| Fa0/11 – Fa0/12 | PC11 – PC12 | 40 (MANAGEMENT) |
| Fa0/13 | Server0 | 50 (SERVERS) |
| Fa0/14 | Laptop1 | 10 (ADMIN) |
| Gi0/1 | Router0 (uplink) | Routed port (no switchport) |

## Router ↔ Switch Link

| Device | Interface | IP Address |
|---|---|---|
| Router0 | GigabitEthernet0/0/0 | 10.0.0.1/30 |
| SW-CORE | GigabitEthernet0/1 | 10.0.0.2/30 |

## Key Configuration Steps

1. **Enable IP routing on the switch** (required for inter-VLAN routing on a 3560):
   ```
   ip routing
   ```

2. **Create VLANs and assign access ports**, e.g. for ADMIN:
   ```
   vlan 10
   name ADMIN
   interface range FastEthernet0/1-4
   switchport mode access
   switchport access vlan 10
   ```

3. **Configure VLAN interfaces (SVIs)** as default gateways, e.g.:
   ```
   interface vlan 10
   ip address 192.168.10.1 255.255.255.224
   no shutdown
   ```

4. **Configure the router-to-switch link as a routed port**:
   ```
   interface GigabitEthernet0/1
   no switchport
   ip address 10.0.0.2 255.255.255.252
   no shutdown
   ```

5. **Configure each end device** with a static IP, subnet mask `255.255.255.224`, and the appropriate VLAN gateway.

## Testing / Verification

- ✅ Same-VLAN connectivity (e.g. PC1 → PC2)
- ✅ Gateway reachability from each VLAN
- ✅ Inter-VLAN routing (e.g. PC1 in ADMIN → PC8 in IT)
- ✅ Server reachability from all departments
- ✅ Router ↔ Switch link (`ping 10.0.0.2` from Router0)

## Notes / Lessons Learned

- The ISR4331 uses **three-part interface naming** (`GigabitEthernet0/0/0`), unlike ISR routers such as the 2911/1841 which use two-part naming (`GigabitEthernet0/0`).
- Switch ports default to **Layer 2 switchport mode** — use `no switchport` before assigning an IP address to turn a port into a routed Layer 3 interface.
- Always confirm which physical port a cable actually lands on using `show ip interface brief` (Status/Protocol columns) rather than assuming based on labels — cables can end up on unexpected ports.
- Static IPs on end devices must be set via the **Desktop → IP Configuration** GUI in Packet Tracer; `ipconfig` in the CLI is read-only.

## Tools

- Cisco Packet Tracer
