# LAB 1 – OPNsense and Ubuntu Network Connectivity

## 1. Introduction

This lab was carried out to set up and test a simple network using **OPNsense as a firewall/router** and **Ubuntu as the client machine**.

The purpose of the lab was to understand how the firewall connects the internal network to the internet and how the Ubuntu client communicates through the firewall.

The laboratory used two virtual machines:

* **`icdfa-nslab-firewall-v1`** – OPNsense firewall
* **`icdfa-nslab-client-v1`** – Ubuntu client

The internal network was named **ICDFA-LAN** and used the **10.10.10.0/24** network.

---

## 2. Lab Objectives

The main objectives of this lab were to:

* Configure the VirtualBox network adapters.
* Configure and verify the OPNsense WAN and LAN interfaces.
* Connect the Ubuntu client to the OPNsense LAN.
* Confirm that Ubuntu receives a correct IP address.
* Confirm that Ubuntu uses OPNsense as its default gateway.
* Test communication between Ubuntu and the firewall.
* Test internet connectivity.
* Test DNS name resolution.
* Use Wireshark to observe ARP, ICMP and DNS traffic.

---

## 3. Network Topology

The lab used the following network arrangement:

```text
                         Internet
                            |
                            |
                       VirtualBox
                           NAT
                            |
                            |
                    OPNsense Firewall
                       /         \
                    WAN           LAN
                                  |
                            10.10.10.1
                                  |
                             ICDFA-LAN
                                  |
                           Ubuntu Client
                         10.10.10.x/24
```

The **WAN interface** was connected to VirtualBox NAT so that OPNsense could access the outside network.

The **LAN interface** was connected to the isolated **ICDFA-LAN** network.

The Ubuntu client was also connected to ICDFA-LAN, allowing it to communicate with the OPNsense firewall.

---

## 4. Part A – VirtualBox Network Configuration

I first shut down both virtual machines completely before making any network changes.

### OPNsense Firewall

For **`icdfa-nslab-firewall-v1`**, I configured:

| Adapter   | Setting                        |
| --------- | ------------------------------ |
| Adapter 1 | Internal Network – `ICDFA-LAN`                       |
| Adapter 2 | NAT |
| Adapter 3 | Disabled                       |
| Adapter 4 | Disabled                       |

### Ubuntu Client

For **`icdfa-nslab-client-v1`**, I configured:

| Adapter        | Setting                        |
| -------------- | ------------------------------ |
| Adapter 1      | Internal Network – `ICDFA-LAN` |
| Other adapters | Disabled                       |

Both the OPNsense LAN adapter and Ubuntu adapter were connected to the same **ICDFA-LAN** network.

This allowed the two virtual machines to communicate with each other.

### Evidence

**E1 – OPNsense VirtualBox network settings**

Screenshot showing Adapter 1 using NAT and Adapter 2 using ICDFA-LAN.

**E2 – Ubuntu VirtualBox network settings**

Screenshot showing the Ubuntu adapter connected to ICDFA-LAN.

---

## 5. Part B – OPNsense Interface Configuration

After starting **`icdfa-nslab-firewall-v1`**, I checked the OPNsense console to confirm the network interfaces.

The OPNsense LAN interface was configured with:

* **LAN IP:** `10.10.10.1`
* **Subnet:** `/24`
* **Network:** `10.10.10.0/24`
* **DHCP range:** `10.10.10.100 – 10.10.10.200`

The LAN interface did not have an upstream gateway because it is the internal side of the firewall.

The WAN interface was configured to obtain its IP address automatically through DHCP.

### Evidence

**E3 – OPNsense WAN and LAN status**

Screenshot showing the OPNsense WAN and LAN interfaces and their addresses.

---

## 6. Part C – Ubuntu IP Address and Routing

After the OPNsense firewall was ready, I started **`icdfa-nslab-client-v1`** and checked the Ubuntu network configuration.

I used:

```bash
ip -4 -br address
```

This command displayed the active network interface and its IPv4 address.

The Ubuntu address was within the **10.10.10.0/24** network and was different from the firewall address of **10.10.10.1**.

I then checked the routing table using:

```bash
ip route
```

The default route pointed to:

```text
10.10.10.1
```

This showed that Ubuntu was using the OPNsense LAN interface as its default gateway.

I also checked the DNS configuration using:

```bash
resolvectl status
```

### Evidence

**E4 – Ubuntu IP address and routing**

Screenshot showing the output of `ip -4 -br address` and `ip route`.

---

## 7. Why Ubuntu Uses 10.10.10.1 as Its Default Gateway

The Ubuntu client uses **10.10.10.1 as its default gateway because 10.10.10.1 is the LAN address of the OPNsense firewall**.

When Ubuntu needs to communicate with a network outside its local `10.10.10.0/24` network, it sends the traffic to OPNsense.

The firewall then handles the routing of the traffic to the outside network.

---

## 8. Part D – Connectivity Tests

I performed connectivity tests to confirm that each part of the network was working correctly.

### Test 1 – Local Gateway

Command:

```bash
ping -c 4 10.10.10.1
```

**Result:** Replies were received from `10.10.10.1`.

This confirmed that the Ubuntu client could communicate with the OPNsense LAN interface.

---

### Test 2 – Internet Connectivity

Command:

```bash
ping -c 4 1.1.1.1
```

**Result:** Replies were received from `1.1.1.1`.

This showed that traffic could pass through the OPNsense firewall and reach an external IP address.

---

### Test 3 – DNS Resolution

Command:

```bash
getent hosts opnsense.org
```

**Result:** An IP address was returned.

This confirmed that DNS name resolution was working.

---

### Test 4 – Web Connectivity

Command:

```bash
curl -I https://opnsense.org
```

**Result:** HTTP response headers were received.

This confirmed that Ubuntu could establish web connectivity using TCP and HTTPS.

### Evidence

**E5 – Connectivity tests**

Screenshot showing the successful gateway ping, internet ping and DNS test.

---

## 9. Accessing the OPNsense Web Interface

I opened a browser from the Ubuntu client and accessed:

```text
https://10.10.10.1
```

The OPNsense login page was displayed.

Since this was the authorised laboratory firewall, I accepted the self-signed certificate warning and logged in using the supplied laboratory credentials.

The dashboard showed the status of the WAN and LAN interfaces.

This confirmed that the Ubuntu client could access the OPNsense web interface through the internal network.

---

## 10. Part E – Wireshark Packet Capture

I used Wireshark on the Ubuntu client to observe the network traffic generated during the tests.

I selected the active Ethernet interface carrying the **10.10.10.x** address and started a packet capture.

I then ran:

```bash
ping -c 4 10.10.10.1
```

and:

```bash
getent hosts opnsense.org
```

After the commands completed, I stopped the capture.

---

### ARP

I used the filter:

```text
arp
```

The capture showed the ARP request asking for the MAC address associated with **10.10.10.1** and the ARP reply from the firewall.

This showed how Ubuntu learned the firewall's MAC address before sending traffic to it.

---

### ICMP

I used:

```text
icmp
```

The capture showed ICMP Echo Request packets sent from Ubuntu and Echo Reply packets received from OPNsense.

This matched the successful ping test.

---

### DNS

I used:

```text
dns
```

The capture showed that the DNS query for **opnsense.org** and the response containing the resolved address.

This confirmed that DNS traffic was being exchanged successfully.

### Evidence

**E6 – Wireshark packet capture**

Screenshots showing the ARP, ICMP and DNS packets with the relevant filters visible.

---

# 11. Problem Encountered During Setup

During the setup, internet connectivity did not work immediately.

The client initially experienced a routing/connectivity problem when trying to reach:

```text
1.1.1.1
```

The ping showed that the destination could not initially be reached.

The network configuration was checked, including:

* OPNsense WAN status
* OPNsense LAN configuration
* Ubuntu IP address
* Default gateway
* Network connection between the virtual machines

After the configuration was corrected and the network connection was established, the connectivity test worked.

This was useful because it showed me that having an IP address on the client is not enough. The gateway and firewall WAN connection must also be working.

---

# 12. Completion Questions

### 1. What is the difference between the OPNsense WAN and LAN interfaces?

The **WAN** interface connects OPNsense to the outside network or internet, while the **LAN** interface connects OPNsense to the Ubuntu client inside the network.

---

### 2. Why must both internal adapters use the same VirtualBox network name?

They must use the same network name so that the **Ubuntu client and OPNsense LAN can communicate with each other** on the same virtual network.

---

### 3. What information does the default route provide to Ubuntu?

The default route tells Ubuntu **where to send traffic that is going outside its own network**.

In this lab, it uses:

```text
10.10.10.1
```

which is the OPNsense LAN address.

---

### 4. Which packet exchange allows Ubuntu to learn the firewall MAC address?

**ARP** allows Ubuntu to find the MAC address of the OPNsense firewall.

Ubuntu asks for the MAC address of:

```text
10.10.10.1
```

and OPNsense responds.

---

### 5. Why does a successful ping to 1.1.1.1 not automatically prove that DNS is working?

Because **1.1.1.1 is already an IP address**, so the ping does not need DNS.

DNS needs to be tested separately by using a domain name such as:

```text
opnsense.org
```

---

# 13. Results Summary

The laboratory setup was successfully configured and tested.

| Test              | Command                        | Result                         |
| ----------------- | ------------------------------ | ------------------------------ |
| Check client IP   | `ip -4 -br address`            | Successful                     |
| Check route       | `ip route`                     | Successful                     |
| Ping gateway      | `ping -c 4 10.10.10.1`         | Successful                     |
| Ping internet     | `ping -c 4 1.1.1.1`            | Successful after configuration |
| DNS resolution    | `getent hosts opnsense.org`    | Successful                     |
| HTTPS test        | `curl -I https://opnsense.org` | Successful                     |
| OPNsense GUI      | `https://10.10.10.1`           | Accessible                     |
| Wireshark capture | `enp0s3`                       | ARP/ICMP/DNS observed          |

---

# 14. What I Learned

This lab helped me understand the basic role of a firewall in a network.

I learned that:

* A firewall can sit between a private network and the internet.
* The LAN and WAN interfaces have different roles.
* The LAN interface connects the internal devices to the firewall.
* The WAN interface connects the firewall to the external network.
* DHCP can automatically assign IP addresses to clients.
* The default gateway tells the client where to send traffic outside its local network.
* DNS changes a domain name into an IP address.
* ICMP is used by tools such as `ping`.
* Wireshark can be used to observe network traffic.
* ARP helps devices find each other on a local network.
* OPNsense provides a web interface for managing the firewall.
* A working client connection requires more than just an IP address; the gateway, routing, and external connection must also work.

---

# 15. Conclusion

Lab 1 gave me practical experience setting up an OPNsense firewall and connecting an Ubuntu client to it.

I configured the OPNsense firewall with a WAN interface connected through VirtualBox NAT and a LAN interface connected to the **ICDFA-LAN** network.

The LAN was configured with the `10.10.10.0/24` network and the OPNsense LAN address was `10.10.10.1`.

The Ubuntu client successfully received an IP address through DHCP and used OPNsense as its default gateway.

I tested communication with the firewall, tested internet connectivity, checked DNS resolution, and tested HTTPS access. I also used Wireshark to observe ARP, ICMP, and DNS traffic.

The lab helped me understand how a firewall connects an internal network to the internet and how different network components work together to provide communication.

Overall, Lab 1 gave me a practical foundation for the firewall rule configuration and traffic-control activities completed in **Lab 2**.

---

# 16. Evidence List

| Evidence | Description                                              |
| -------- | -------------------------------------------------------- |
| **E1**   | OPNsense VirtualBox network settings – NAT and ICDFA-LAN |
| **E2**   | Ubuntu VirtualBox network settings – ICDFA-LAN           |
| **E3**   | OPNsense WAN and LAN status                              |
| **E4**   | Ubuntu IP address and routing table                      |
| **E5**   | Successful gateway, internet and DNS tests               |
| **E6**   | Wireshark ARP, ICMP and DNS captures                     |
| **E7**   | Explanation of the Ubuntu default gateway                |

---

# 17. Evidence Files

The screenshots for this lab are stored in the `SCREENSHOTS` folder.

Recommended file names:

```text
SCREENSHOTS/
├── E1-opnsense-virtualbox-network.png
├── E2-ubuntu-virtualbox-network.png
├── E3-opnsense-wan-lan-status.png
├── E4-ubuntu-ip-and-routing.png
├── E5-connectivity-tests.png
├── E6-wireshark-arp-icmp-dns.png
└── E7-default-gateway.png
```

---

## Lab Status

**Status:** Completed

**Firewall:** OPNsense

**Client:** Ubuntu

**LAN:** `10.10.10.0/24`

**OPNsense LAN:** `10.10.10.1`

**Internal Network:** `ICDFA-LAN`

**Main Tests:** Gateway, Internet, DNS, HTTPS and Wireshark

**Result:** Successful

