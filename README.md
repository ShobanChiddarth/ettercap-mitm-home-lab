# Ettercap MITM Home Lab

This is an Man in the Middle (MITM) attack home lab using `ettercap` for ARP poisoning and Wireshark for credential sniffing over insecure HTTP connections.

## Network Architecture Setup

![network-architecture](./assets/network-architecture.png)

### VM Details

- 1 pfSense as router
- 1 Kali VM as attacker
- 2 Debian VMs as victims

### Networks

| Network Name | Purpose | IP Addressing |
| --- | --- | --- |
| VirtualBox Host Only Ethernet Adapter 4 (`vboxnet4`) | LAN for the Attacker and Victims | Network: `192.168.60.0/24`<br>Gateway: `192.168.60.2`<br>DNS:`192.168.60.2` |
| NAT | Public Internet connectivity | Fully managed by VirtualBox |

### Adapter details

#### 1. pfSense VM

| Adapter Number | Adapter Name | Network it is attached to |
| --- | --- | --- |
| 1 | `em0` (WAN Adapter) | `NAT` |
| 2 | `em1` (LAN Adapter) | `vboxnet4` |

#### 2. Kali VM

| Adapter Number | Adapter Name | Network it is attached to |
| --- | --- | --- |
| 1 | `eth0` | `vboxnet4` |

#### 3. Both Debia Victims VM

| Adapter Number | Adapter Name | Network it is attached to |
| --- | --- | --- |
| 1 | `enp0s3` | `vboxnet4` |


## Kali attacker setup

### Attack Scope

What we are doing is,
1. Make the router add Kali's MAC address to its ARP table for every host IP
2. Make every victim host add Kali's MAC address to its ARP table for gateway IP

### IP Forwarding

So all traffic will be routed through Kali. In order to do that, we need to enable IP forwarding (temporarily) in Kali VM using the command

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Verify it with

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Screenshot:

![ip-forwarding-screenshot](./assets/ip-forwarding-screenshot.png)


## Recon

Objectives:

We need to know
1. Private IP of Attacker
2. Subnet
3. Gateway IP
4. Victim IPs (Other hosts in the same network)

We already know this as we designed the network while setting up the lab. But here is how an attacked would do reconnaisance in a real attack scenario.

### 1. Private IP and 2. Subnet

```bash
ifconfig eth0 <or any other interface name>
```

Will print the private IP of the attacker's machine, along with the subnet mask.

![ifconfig](./assets/ifconfig-screenshot.png)

From that, we can use it to reconstruct the network address. In our case, it is `192.168.60.0/24`


### 3. Gateway IP

We can issue this command to show the route table to find out the default gateway's address

```bash
ip route
```

![ip-route-screenshot](./assets/ip-route-screenshot.png)

Now we know that the router's IP address is `192.168.60.2`

### 4. List of other hosts IP addresses

We can use the tool `netdiscover` to identify other hosts in the network connected to the network the attacker's machine is connected to.

```bash
sudo netdiscover -i eth0
```

![netdiscover-screenshot](./assets/netdiscover-screenshot.png)

Apart from Kali, the list of hosts shown through netdiscover are

| IP | Details |
| --- | --- |
| `192.168.60.1` | This is the bare metal host on which VirtualBox is running. Since we are using VirtualBox Host Only Ethernet adapter the host needs an IP in the subnet to access the VMs in this Host Only Network. This will not exist in real world, and also we don't have to consider this for the lab. |
| `192.168.60.2` | This is the gateway as confirmed by the `ip route` command |
| `192.168.60.102` | Unknown - Victim |
| `192.168.60.103` | Unknown - Victim |


## Recon Summary

Our reconnaissance is over. Here is the information we have gathered

| Key | Value |
| --- | --- |
| Private IP of Attacker | `192.168.60.101` |
| Subnet | `192.168.60.0/24` |
| Gateway IP | `192.168.60.2` |
| Victim IPs | `192.168.60.102`, `192.168.60.103` |

