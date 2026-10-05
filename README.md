# Clinic Network Lab

I built this network in Cisco Packet Tracer to practice switching, routing, and basic security for a small healthcare clinic.

[Topology](01-topology.png)

Devices: one router (R1), one Layer 3 switch (MLS1), two access switches (SW1 and SW2), one access point, one server, and six PCs.

## IP addressing

I split 192.168.10.0/24 into five subnets with VLSM.

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | Front desk | 192.168.10.0/26 | 192.168.10.1 |
| 20 | Exam rooms | 192.168.10.64/27 | 192.168.10.65 |
| 30 | Guest | 192.168.10.96/27 | 192.168.10.97 |
| 40 | Servers | 192.168.10.128/28 | 192.168.10.129 |
| 99 | Management | 192.168.10.144/29 | 192.168.10.145 |

## What I set up

**VLANs and trunks.** Created the five VLANs on all three switches. Set up trunk links from MLS1 to SW1 and SW2, and put each PC, the server, and the access point in the right VLAN.

**Inter-VLAN routing.** Turned on routing on MLS1 and gave each VLAN a gateway interface. PCs in different VLANs can ping each other and the server.

**DHCP.** MLS1 hands out addresses to each VLAN. I moved the PCs from static addresses to DHCP.

![DHCP bindings](images/02-dhcp-bindings.png)
![PC address from DHCP](images/03-pc-dhcp-ipconfig.png)

**Port security.** On the PC ports I set a maximum of one MAC address, sticky learning, and shutdown on violation. I plugged in a different PC to test it and the port went into err-disabled. I fixed it by reconnecting the original PC and resetting the port.

![Port security on SW2](images/04-port-security-sw2.png)

**Guest Wi-Fi.** The access point has a WPA2 guest network on VLAN 30. A smartphone connected, got an address, and reached the server.

![Smartphone browsing to the server](images/05-guest-wifi-web.png)
![Smartphone ping to the server](images/06-guest-wifi-ping.png)

## Problems I ran into

- The link between R1 and MLS1 stayed red. I had used a cross-over cable instead of straight-through, and the router port was shut down.
- MLS1 would not turn on. It had no power supply module installed.
- SW2 had the wrong hostname.
- One access port was set as a trunk by mistake.
- One PC could not reach its gateway. I checked its IP settings, the firewall, the switch port VLAN, the trunk, and the gateway on MLS1. All were fine, and other PCs in the same VLAN worked. I replaced the PC and it worked. I never found the exact cause.

## Still to do

- Block the guest network from reaching the server with an ACL
- IPv6
- HSRP
- Native VLAN mismatch troubleshooting
- OSPF
