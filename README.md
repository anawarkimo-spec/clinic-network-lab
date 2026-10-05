# Healthcare Clinic Network Lab (Cisco Packet Tracer)

A small clinic network built and tested in Cisco Packet Tracer 9.0. It covers VLANs, trunking, Layer 3 switching, DHCP, port security, and a guest Wi-Fi network, with a troubleshooting log of the problems I ran into and how I fixed them.

## Topology

![Clinic network topology](images/01-topology.png)

| Device | Model | Role |
|---|---|---|
| R1 | Cisco 2911 | Router, connected to MLS1 |
| MLS1 | Cisco 3650-24PS | Layer 3 switch: inter-VLAN routing and DHCP |
| SW1 | Cisco 2950-24 | Front desk access switch |
| SW2 | Cisco 2950-24 | Exam rooms and server access switch |
| AP1 | Access Point-PT | Guest Wi-Fi |
| SRV1 | Server-PT | Internal server |
| PCs | PC-PT | 3 front desk PCs, 3 exam room PCs |

## Addressing plan (VLSM)

Starting block: `192.168.10.0/24`. Host counts are planned sizes for this lab. Each subnet is the smallest block that fits, placed largest first.

| VLAN | Name | Planned hosts | Network | Mask | Gateway (MLS1) |
|---|---|---|---|---|---|
| 10 | FRONT-DESK | 50 | 192.168.10.0/26 | 255.255.255.192 | 192.168.10.1 |
| 20 | EXAM-ROOMS | 25 | 192.168.10.64/27 | 255.255.255.224 | 192.168.10.65 |
| 30 | GUEST | 25 | 192.168.10.96/27 | 255.255.255.224 | 192.168.10.97 |
| 40 | SERVERS | 10 | 192.168.10.128/28 | 255.255.255.240 | 192.168.10.129 |
| 99 | MGMT | 5 | 192.168.10.144/29 | 255.255.255.248 | 192.168.10.145 |

## What I built

### 1. VLANs and trunks
- Created VLANs 10, 20, 30, 40, and 99 on MLS1, SW1, and SW2.
- Configured 802.1Q trunks: MLS1 Gi1/0/2 to SW1 Fa0/3, and MLS1 Gi1/0/3 to SW2 Fa0/1. Allowed VLANs 10, 20, 30, 40, 99.
- Assigned access ports: front desk PCs to VLAN 10, exam room PCs to VLAN 20, guest AP to VLAN 30, server to VLAN 40.

<!-- Add screenshots: `show vlan brief` (SW1, SW2) and `show interfaces trunk` (MLS1) -->

### 2. Inter-VLAN routing (Layer 3 switching)
- Enabled `ip routing` on MLS1 and created a gateway interface (SVI) for each VLAN.
- Verified with pings between PCs in different VLANs and to the server. TTL 127 on replies shows the traffic was routed.

<!-- Add screenshots: `show ip interface brief | include Vlan` on MLS1, and a cross-VLAN ping from a PC -->

### 3. DHCP
- Configured DHCP pools on MLS1 for the front desk, exam rooms, and guest VLANs, with excluded addresses for gateways and spares.
- Switched all PCs from static to DHCP. Each received an address, mask, and gateway automatically.

![DHCP bindings on MLS1](images/02-dhcp-bindings.png)

![PC receiving its address by DHCP](images/03-pc-dhcp-ipconfig.png)

### 4. Port security
- Enabled port security on all PC ports on SW1 and SW2: maximum 1 MAC address, sticky learning, shutdown on violation.
- Tested by connecting an unauthorized PC to a protected port. The port went to `err-disabled`. I recovered it by reconnecting the authorized PC and resetting the port with `shutdown` / `no shutdown`.

![Port security on SW2: Fa0/2 err-disabled after the unauthorized PC connected](images/04-port-security-sw2.png)

### 5. Guest Wi-Fi
- Configured AP1 with a WPA2-secured guest SSID on VLAN 30, with its own DHCP pool on MLS1.
- Connected a smartphone. It received a guest address and reached the server across VLANs.

![Smartphone browsing to the server](images/05-guest-wifi-web.png)

![Smartphone ping to the server](images/06-guest-wifi-ping.png)

## Troubleshooting log

| Problem | How I found it | Fix |
|---|---|---|
| Link between R1 and MLS1 stayed red | Checked the cable type and port state | Used a straight-through cable instead of cross-over, and ran `no shutdown` on the router port |
| MLS1 end of the link stayed red | Opened the CLI and got a "device must be powered on" error | Installed an AC power supply module on the Layer 3 switch |
| SW2 had the wrong hostname | Prompt said `SW1` on the SW2 window | Set `hostname SW2` |
| An access port was accidentally set as a trunk | Reviewed `show interfaces trunk` | Set the port back to access mode |
| One PC could not reach its gateway | Checked its IP settings, firewall, switch port VLAN, trunk, and the VLAN gateway on MLS1. All were correct, and other devices in the same VLAN worked | Replaced the PC, reconnected it to the same port, and re-entered its settings. I did not find the exact cause on the PC itself |
| Pings to a PC timed out | Realized the address belonged to the PC I had deleted | Checked the other PCs' addresses and tested using the addresses that actually existed |

## Skills practiced
VLANs, 802.1Q trunking, VLSM subnetting, Layer 3 switching (SVIs), DHCP, port security, WPA2 wireless, Cisco IOS CLI, and systematic troubleshooting (IP settings, VLAN assignment, trunks, gateways, port state).

## Next steps
- Restrict the guest VLAN from reaching internal servers with an ACL
- IPv6 addressing and testing
- HSRP for gateway redundancy
- Troubleshoot a native VLAN mismatch
- Diagnose and repair an OSPF adjacency failure

## Files
- `clinic-network.pkt`: the Packet Tracer file (add your latest save here)
- `images/`: screenshots used in this README
