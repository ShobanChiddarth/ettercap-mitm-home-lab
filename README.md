# Ettercap MITM Home Lab

This is an Man in the Middle (MITM) attack home lab using `ettercap` for ARP poisoning and Wireshark for credential sniffing in insecure HTTP connections.

## Setup

### Network Architecture

![network-architecture](./assets/network-architecture.png)

#### VM Details

- 1 pfSense as router
- 1 Kali VM as attacker
- 1 Debian Desktop VM that runs 10 docker containers using MACVLAN adapter to stimulate victims

#### Networks

| Network Name | Purpose | IP Addressing |
| --- | --- | --- |
| VirtualBox Host Only Ethernet Adapter 4 (`vboxnet4`) | LAN for the Attacker and Victims | Network: `192.168.60.0/24`<br>Gateway: `192.168.60.2`<br>DNS:`192.168.60.2` |
| NAT | Public Internet connectivity | Fully managed by VirtualBox |

#### Adapter details

##### pfSense VM

| Adapter Number | Adapter Name | Network it is attached to |
| --- | --- | --- |
| 1 | `em0` (WAN Adapter) | `NAT` |
| 2 | `em1` (LAN Adapter) | `vboxnet4` |

##### Kali VM

| Adapter Number | Adapter Name | Network it is attached to |
| --- | --- | --- |
| 1 | `eth0` | `vboxnet4` |

##### Debia Victims VM

| Adapter Number | Adapter Name | Network it is attached to |
| --- | --- | --- |
| 1 | `enp0s3` | `vboxnet4` |

