What is MAC Address
---

MAC Address is 12 character haxadecimal address and has attached to NIC.

```bash
ifconfig -a

# Look for en01 is your local.
# Look for ether - will have MAC Address like F0:24:1F:99:8J:48

ip link

# Look for link/ether
```

wireshark
---

- Wireshark is a free and open source `network protocol analyzer` or `sniffer`.
- Used to analyze the traffic passing through a computer network.
- It may be wired or wireless.

- wireshark allow you to **inspect individual packets** to see detailed informations including `source & dest`, `pkg size`, `type of data transmitted`.

- This info is useful for Troubleshooting network issues.

- Wiresharks supports protocols like **TCP/IP**, **DNS**, **HTTP** and many more.

DHCP
---

- DHCP (dynamic host configuration protocol) is nothing but a dynamically way to assign IP Address to Switch, Router on same network or diff network, as we did earlier using `ip addr add` and `route add`.

- It eliminates the need for manual configuration and ensuring devices can connect and communicate seamlessly.

- **DHCP Server** - A dedicated device or server that manages a pool of IP addresses and Assigns them to clients.

- **DHCP Client** - A device where we want to assign IP Address like SmartPhone, Laptop etc.

1. DHCP Client - Broadcast a DHCP DISCOVER pkg - Ask for Ip address to DHCP Server
2. DHCP Server - Server will give offer to client, "Hey!, I have this much of IP Addresses!. Do you wanna use it ?"

3. DHCP client - Will Request to DHCP Server, "Yes, Please! , Give me IP Address!"

4. DHCP Server - Will assign IP Address to client and Acknowledge it.

How DHCP Lease renewal works
---

what is `lease` ?
- Lease means, once ip has assgined to your device by DHCP Server, That IP will usable by your device till next 8 days only by default.

- Normally after 8 days it will expired, and your device will get a diff new ip address.

- However you want a static Ip address for long time, like ElasticIP.

- Here, DHCP lease renewal comes in picture.

## Renewal Process Timeline Std way

### 1. 50% - Day4 

  - While your lease duration used 50% time like 4 days.

  - Client will send a request to DHCP Server to renew my IP address. And Ask to renew same ip.

  - If `DHCP Server` is online - Lease will reset to next 8 days from 4 days.

  - Your IP Address keeps extending begore it expires untill your DHCP Server is online.

## 2. 87.5% - 6 Days

  - Somehow, your DHCP is not responding and offline, your DHCP Lease Renewal Request will failed.

  - Client will retries at 87.5% of lease, like on 6th days, it will retry to renew lease and ask to DHCP Server to renew the same ip.

  - If it succeeds - Lease renewed.

## 3. 100% - Expiratoin Day

  - If both attepmts was failed (On 50% and 87.5%), your lease will have been expired.

  - your device will get a **New different IP Address**.

Automatic Private IP Addressing (APIPA)
---

- This will automatically assign an IP Address to the devices while **DHCP is UnAvailable**.

- This is a settings in Mobile, Laptops.

- While DHCP is UnAvailable, Client (Devices) will automatically creates an IP Address from `169.254.X.X`

What is DNS Zones and DNS Records ?
---

- A `DNS zone` is a specific portion of DNS namespace that contains DNS Records.

- **DNS Zones Types**
  1. `Forward lookup zone` - Resolves Domain to IP -- google.com to 12.46.78.24

  2. `Reverse lookup zone` - Resolves IP to Domain -- 12.46.78.24 to google.com

**DNS Records**

  1. **A** - Stores Domain to IPv4
  `google.com → 142.250.183.14`

  2. **AAAA** - Stores Domains to IPv6
  `google.com → 2404:6800:4007:80f::200e`

  3. **MX** - Which mail server handles emails for that domains

```bash
dig google.com MX

# OUTPUT
google.com mail is handled by 10 smtp.google.com
```

  4. **CNAME** - Stores Alias records.

```bash
www.mycompany.com → mycompany.com
```
 
  5. **TXT** - Stores Text informations.

    - Domain Verifications
    - Google Verifications
    - AWS SES Verifications
    - SPF (email security)

  6. **NS** - Stores Name Server Records

  7. **SRV** - SRV Records used for:
    - VoIP Systems
    - Microsoft Active Directory
    - XMPP (Chat Servers)
    - Kubernetes Internal Services
    - LDAP

    Example:
    - Microsoft Active Directory heavily uses SRV records.

    - When a Windows machine joins a domain:

    `_ldap._tcp.dc._msdcs.domain.com`


    - It queries SRV record to find:

    - 👉 Which Domain Controller
    - 👉 On which port


How DNS Works ?
---

  1. While you ping or send request to any of domain, It will first look into `/etc/host` file.

```bash
192.168.1.115 test
```

  2. If it is found locally, it will never varify your local dns resolver.

  - Local dns resolver is located at `/etc/resolve.conf`

![alt text](dns2.png)

  - This order may change depend on how you configured `/etc/nsswitch.conf`

```bash

hosts:          files dns
networks:       files
```

  - hosts is configured with 1. files and 2. dns
  - So, Always it will varify in `/etc/host` (1. files) and then in the `/etc/resolve.conf` (2. dns).

  3. This is not your real dns server, this is just local dns resolver.

![alt text](dns2.png)

  - To check which is your dns server and assigning ips to your pc's network interfaces

```bash
resolvectl status
```

![alt text](dnsr.png)

  - `Current DNS Server` is **10.100.49.55** is your Primary DNS Server.
  - Your Secondary DNS Server is **10.126.52.5**.

- So, Your dns flow will like this

```bash
Your App or Ping Query
   ↓
10.126.30.1 Default Gateway or Router
   ↓
127.0.0.53 (local DNS stub)
   ↓
10.100.49.55 (real corporate DNS server)
   ↓
Internet & Domain NS
   ↓
Youtube Server
```

**Step 1** - Know your PC IP and MAC

- Your PC has 2 diff IP & MAC, Wifi NIC and Ethernet NIC

```bash
# To know MAC and Private IP of your PC's NIC
ifconfig 

# To know only MAC Address of your PC's NIC
ip link
```

![alt text](ifcfg.png)

**Step 2** - Know your default gateway or router's MAC and IP.

```bash
ip route
# default via 10.126.30.1 dev eno1 proto dhcp metric 100 

ip neigh
# 10.126.30.1 dev eno1 lladdr 90:77:ee:a4:4a:89 REACHABLE
```

**Step 3** - Know which DHCP Server used by your PC

```bash
resolvectl
```

![alt text](dnsr.png)

- While you send req to youtube.com it will first goes to default gateway `10.126.30.1` and look for youtube.com is available in local by `/etc/host`.

- If it is not available in `/etc/host` it will goes to `/etc/resolve.conf` -- loal dns resovler `127.0.0.53`based on how route defined in this `/etc/nsswitch.conf`.

- If it is not found in local dns resolver, it will goes to internet trhough router.

- Goes to Multiple Hops to Original DNS NS.

**Step 4** - Confirm to reach to DNS

```bash
traceroute youtube.com
```

![alt text](tr.png)

```bash
9  192.178.110.109 (192.178.110.109)  11.326 ms lcboma-as-in-f14.1e100.net (142.250.194.238)  10.337 ms  10.286 ms
```

**lcboma-as-in-f14.1e100.net** is a Name Server Address and  **(142.250.194.238)** is a IP of youtube.com



Networking
---

Each devices has assigned network interface itslef by physical or virtual.
This is called MAC Address

To see this ip , use

```bash
ip link
```

**OUTPUT**

```bash
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether c9:05:b5:42:j7:12 brd ff:ff:ff:ff:ff:ff
    altname enp1s0
```
- Here, `c9:05:b5:42:j7:12` is a MAC Address.

- We then assign the systems with IP Addresses on the same network. 
- It means, We will assign IP Address to this network `eno1`.
- for this we use,

```bash
ip addr add 10.126.31.232/23 dev eno1
```

- Varify this,

```bash
ip addr show eno1
```

**OUTPUT**

```bash
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether c9:05:b5:42:j7:12 brd ff:ff:ff:ff:ff:ff
    altname enp1s0
    inet 10.126.31.232/23 brd 10.126.31.255 scope global dynamic noprefixroute eno1
       valid_lft 168710sec preferred_lft 168710sec
    inet6 fe80::8d6:777c:a4d0:3585/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

- Here, `inet` **`10.126.31.232/23`** is assigned IP Address.
- Also, `brd 10.126.31.255` is brodcast ip address to use for communicate from this network `eno1` to `other network`.

**Router** - helps to get connect 2 diff networks

- **Router** has connected to 2 diff networks. So, we would have to assign 2 diff ip address from 2 diff networks to this router for communications.

# DNS Zone Types

## Purpose

Understand the four Windows DNS zone types and when to use them.

## DNS Zone

A DNS zone stores DNS records for a domain such as A, CNAME, MX, etc.

## 1. Primary Zone

Primary zone is the main read-write copy of DNS records.

Characteristics:

* Stores the master DNS records
* Administrators can add, edit, or delete records
* Other DNS servers copy from this zone

Use Case:
Used as the main DNS server where records are managed.

## 2. Secondary Zone

Secondary zone is a read-only copy of a primary zone.

Characteristics:

* Records cannot be modified
* Receives updates from the primary server
* Used to reduce load on the primary server

Use Case:
Useful for branch offices to reduce WAN traffic.

## 3. Active Directory Integrated Zone

DNS data is stored inside Active Directory.

Characteristics:

* Replicates automatically using AD replication
* Multiple DNS servers can update records
* Requires DNS server to be a domain controller

Use Case:
Recommended option when Active Directory is available.

## 4. Stub Zone

Stub zone stores only limited DNS information.

Characteristics:

* Stores NS records
* Helps locate authoritative DNS servers
* Does not store full DNS records

Use Case:
Used to speed up DNS lookups and maintain delegation.

## Quick Comparison

| Zone Type     | Read/Write | Stores Full Records | Replication Method           |
| ------------- | ---------- | ------------------- | ---------------------------- |
| Primary       | Yes        | Yes                 | Manual / Zone Transfer       |
| Secondary     | No         | Yes                 | Zone Transfer                |
| AD Integrated | Yes        | Yes                 | Active Directory Replication |
| Stub          | No         | No                  | Maintains NS Records         |

## Key Takeaway

* Primary = Main DNS database
* Secondary = Read-only copy for redundancy and performance
* AD Integrated = Best option in Windows domain environments
* Stub = Helps locate authoritative DNS servers

Understanding these helps design reliable DNS infrastructure in enterprise environments.


# Active Directory–Integrated DNS Zones

## Overview

Active Directory–integrated zones simplify DNS management by combining DNS data storage with **Active Directory replication and security features**.

## 1. What Is an Active Directory–Integrated DNS Zone?

An **Active Directory–integrated DNS zone** stores DNS records directly inside **Active Directory (AD)** instead of using a traditional DNS zone file.

Because DNS data is stored in Active Directory, it automatically replicates between **domain controllers** that run the DNS service.

This architecture removes the dependency on traditional **primary and secondary DNS server roles**.

Key idea:

* DNS data becomes part of the **Active Directory database**
* Replication is handled by **Active Directory replication mechanisms**
* Multiple DNS servers can host and manage the same zone

## 2. Multi-Master DNS Updates

In traditional DNS configurations:

* **Primary DNS server** → the only server allowed to make changes
* **Secondary DNS server** → read-only copy of the primary

With **Active Directory–integrated DNS**:

* Any DNS server hosting the zone can **create, modify, or delete DNS records**
* There is **no single authoritative write server**

This model is known as **multi-master replication**.

### Benefits

* Eliminates a single point of failure
* Any domain controller can process DNS updates
* Improves DNS availability and resilience

## 3. Replication Through Active Directory

DNS records stored in Active Directory replicate using the **existing Active Directory replication topology**.

### Characteristics

* Replication follows **Active Directory site and replication rules**
* Updates are **incremental**
* Only **changed records** are replicated between domain controllers

### Benefits

* Efficient network utilization
* Faster propagation of DNS changes
* No manual **zone transfers** required

## 4. Secure Dynamic DNS Updates

Active Directory–integrated zones support **secure dynamic DNS updates**.

This means:

* Only **domain-joined computers** are allowed to automatically register DNS records
* Unauthorized or unmanaged devices cannot create DNS entries

### Example

When a computer joins the domain, it can automatically register its DNS record:

```
computer01.company.local → 10.10.1.25
```

### Security Advantage

Secure updates prevent devices such as:

* Personal laptops
* Mobile phones
* Unmanaged systems

from automatically creating DNS records that could pollute the DNS database.

Administrators can disable secure updates if they want **all devices** to be allowed to register DNS records.

## 5. Why Active Directory–Integrated DNS Zones Are Preferred

In most enterprise Windows environments, **Active Directory–integrated DNS zones are the recommended configuration** because they provide:

* Automatic replication
* Multi-master DNS updates
* Improved security
* Reduced administrative overhead

### Comparison With Traditional DNS Zones

| Feature               | Traditional DNS (Primary/Secondary) | AD–Integrated DNS            |
| --------------------- | ----------------------------------- | ---------------------------- |
| Write Capability      | Only Primary Server                 | Any DNS Server               |
| Replication Method    | Zone Transfer                       | Active Directory Replication |
| Security              | Basic                               | Secure Dynamic Updates       |
| Administrative Effort | Higher                              | Lower                        |

## 6. Typical Enterprise Architecture

Example Windows domain architecture using AD-integrated DNS:

```
        Domain Controller 1
        DNS Server
             │
             │  Active Directory Replication
             │
        Domain Controller 2
        DNS Server
             │
             │  Active Directory Replication
             │
        Domain Controller 3
        DNS Server
```

All DNS servers in this environment:

* Host the same DNS zone
* Allow record updates
* Replicate DNS changes automatically through Active Directory

## 7. Key Takeaways

* Active Directory–integrated DNS stores DNS data **inside Active Directory**
* Supports **multi-master DNS updates**
* Uses **Active Directory replication** instead of zone transfers
* Provides **secure dynamic DNS updates**
* Recommended DNS configuration for **Windows domain environments**

# DNS Stub Zone

## Overview

A **Stub Zone** is a lightweight DNS zone that contains only the minimum information required to locate the **authoritative DNS servers** for another zone. Instead of storing every DNS record, it stores a small subset of records that allow your DNS server to determine where to send queries.

Stub zones are commonly used in environments with **multiple domains or forests** where DNS servers need to resolve names across domains without replicating and storing all records locally.

## One‑Line Definition

A **Stub Zone** is a DNS zone that stores only the **NS (Name Server) records and related authority information** for another DNS zone so that your DNS server knows which authoritative server to query.

## Simple Analogy

Think of a full DNS zone as a **complete phonebook** containing every person's phone number in a city.

A **stub zone** is like a **directory assistance page** that only lists the phone numbers of the offices that manage the phonebook. When you need someone's number, you call the directory office instead of carrying the entire phonebook.

This keeps your local "phonebook" very small while still allowing you to find any number when needed.


![alt text](stubzone.png)


## Records Contained in a Stub Zone

A stub zone typically contains only a small number of DNS record types:

* **SOA (Start of Authority)** — Identifies the primary DNS server responsible for the zone
* **NS (Name Server) Records** — Lists the authoritative DNS servers for that zone
* **Glue A Records** — Provides the IP addresses of those authoritative name servers

A stub zone **does NOT contain standard host records**, such as:

* A records
* AAAA records
* MX records
* TXT records
* CNAME records

Because of this, stub zones remain **very small and lightweight**.

## How Stub Zones Work (Step‑by‑Step)

Example scenario with two domains:

* **contoso.com**
* **fabrikam.com**

### Name Resolution Flow

1. A DNS server in **contoso.com** receives a query for a hostname in **fabrikam.com**.
2. Instead of storing every host record for fabrikam.com, the DNS server maintains a **stub zone** for that domain.
3. The stub zone contains the **NS records** that identify fabrikam.com's authoritative DNS servers.
4. The contoso DNS server checks the stub zone to locate those authoritative servers.
5. It sends the DNS query directly to one of the fabrikam DNS servers.
6. The authoritative server returns the correct DNS record.
7. The contoso DNS server forwards the response back to the client.

This process enables **cross-domain resolution without replicating the entire zone**.

## DNS Resolution Diagram (Stub Zone Example)

```
Client
   |
   | DNS Query: app.fabrikam.com
   v
+----------------------+
| Contoso DNS Server   |
| (Contains Stub Zone) |
+----------------------+
           |
           | Checks Stub Zone
           | Finds NS records
           v
   +----------------------+
   | Fabrikam DNS Server  |
   | (Authoritative)      |
   +----------------------+
           |
           | Returns A Record
           v
Contoso DNS Server -> Client
```

The stub zone acts as a **pointer that tells the DNS server where to ask the question**.

## Why Use Stub Zones

### 1. Reduced Replication Traffic

Stub zones prevent thousands of DNS records from being replicated across WAN links.

### 2. Minimal Storage Requirements

Only a few authority records are stored locally, keeping the DNS database small.

### 3. Automatic Discovery of DNS Servers

If authoritative name servers change, the stub zone can update the **NS record list automatically**.

### 4. Efficient Cross‑Domain Name Resolution

DNS servers can locate the correct authoritative DNS server without storing the entire zone.

## When Not to Use Stub Zones

Stub zones may not be the best option when:

* A **complete local copy** of all DNS records is required
* DNS resolution must continue even if **authoritative servers are temporarily unreachable**
* Very **high‑performance local DNS resolution** is required for all records

In these cases, a **Secondary Zone** may be more appropriate.


## Stub Zones in Active Directory Environments

In **Active Directory environments**, stub zones are frequently used to support **cross-domain DNS resolution** without replicating all DNS records between domains.

Advantages in AD environments include:

* Reduced Active Directory replication traffic
* Lightweight DNS infrastructure
* Accurate awareness of authoritative DNS servers

This makes stub zones useful in **multi-domain forests**.

## Key Takeaways

* Stub zones store **only minimal DNS authority information**.
* They allow DNS servers to **locate authoritative name servers** for another zone.
* They reduce **replication overhead, storage requirements, and WAN traffic**.
* They are commonly used in **multi-domain or Active Directory environments**.

## Quick Summary

| Feature                          | Stub Zone  |
| -------------------------------- | ---------- |
| Stores all DNS records           | No         |
| Stores NS records                | Yes        |
| Stores SOA record                | Yes        |
| Used for cross-domain resolution | Yes        |
| Replication size                 | Very small |

# DNS Forwarding

## One-Line Summary

DNS **forwarding** means a local DNS server sends queries it cannot resolve to another DNS server (called a **forwarder**) instead of resolving them itself using root servers.

---

## How DNS Forwarding Works

1. A client device sends a DNS query (for example `microsoft.com`) to the **local DNS server**.
2. The local DNS server checks whether it can resolve the query locally (for example internal domain records).
3. If the record is not found, the DNS server **forwards the query** to a configured external DNS resolver.
4. The external resolver performs the DNS lookup (or returns a cached result).
5. The result is sent back to the local DNS server and then returned to the client.

---

## Example Forwarders

Common public DNS forwarders include:

* **8.8.8.8** – Google Public DNS
* **8.8.4.4** – Google Public DNS

---

## DNS Forwarding Flow

```
Client → Local DNS Server → Forwarder (External DNS) → Internet DNS Servers
          (forwards query)      (performs lookup / returns cached result)
```

![alt text](dnsfwd.png)

## Why DNS Forwarding Is Faster

* Public DNS resolvers maintain **large caches** for frequently requested domains.
* Queries for common domains can be answered immediately from cache.
* The local DNS server avoids performing full recursive lookups.

## Advantages

* Faster resolution for common internet domains
* Reduced workload on the local DNS server
* Simple configuration

## Considerations

* DNS queries are sent to external servers (privacy considerations).
* Reliance on external DNS availability.
* Internal DNS zones should not be forwarded.

# Networking Commands (Equivalent to Windows ipconfig)

## Overview

In Ubuntu (Linux), the `ipconfig` command used in Windows does not exist. Linux uses different networking tools that provide the same functionality and often more detailed information.

This document maps common Windows `ipconfig` commands to their Ubuntu equivalents and explains how to use them for basic network troubleshooting.

---

## Windows vs Ubuntu Networking Commands

| Windows Command      | Ubuntu Equivalent                      |
| -------------------- | -------------------------------------- |
| ipconfig             | ip addr or ip a                        |
| ipconfig /all        | ip addr + ip route + resolvectl status |
| ipconfig /release    | dhclient -r                            |
| ipconfig /renew      | dhclient                               |
| ipconfig /displaydns | resolvectl statistics                  |
| ipconfig /flushdns   | sudo resolvectl flush-caches           |

---

## 1. Check IP Address

### Command

```
ip a
```

or

```
ip addr
```

### Example Output

```
2: wlp2s0: <BROADCAST,MULTICAST,UP>
    inet 192.168.1.105/24
    inet6 fe80::9f3:21ff:fe4a:1b2
```

### Explanation

| Field  | Meaning           |
| ------ | ----------------- |
| wlp2s0 | Network interface |
| inet   | IPv4 address      |
| inet6  | IPv6 address      |
| /24    | Subnet mask       |

Example:

```
192.168.1.105/24
```

Means:

```
IP Address: 192.168.1.105
Subnet Mask: 255.255.255.0
```

---

## 2. Check Default Gateway

### Command

```
ip route
```

### Example Output

```
default via 192.168.1.1 dev wlp2s0
```

| Field       | Meaning          |
| ----------- | ---------------- |
| default     | Internet route   |
| 192.168.1.1 | Router / Gateway |
| wlp2s0      | Interface        |

---

## 3. Check DNS Server

### Commands

```
resolvectl status
```

or

```
cat /etc/resolv.conf
```

### Example

```
nameserver 192.168.1.1
nameserver 8.8.8.8
```

| Value       | Meaning    |
| ----------- | ---------- |
| 192.168.1.1 | Router DNS |
| 8.8.8.8     | Google DNS |

---

## 4. DHCP Concept

Ubuntu usually receives network configuration automatically using DHCP.

DHCP provides:

* IP Address
* Subnet Mask
* Default Gateway
* DNS Server

These settings are typically assigned by a router, ISP modem, or enterprise DHCP server.

---

## 5. Release IP Address

### Command

```
sudo dhclient -r
```

This releases the currently assigned IP address from the DHCP server.

Verify:

```
ip a
```

---

## 6. Renew IP Address

### Command

```
sudo dhclient
```

The system requests a new IP address from the DHCP server.

---

## 7. View DNS Cache Statistics

### Command

```
resolvectl statistics
```

Shows DNS cache metrics such as cache hits, misses, and queries.

---

## 8. Flush DNS Cache

### Command

```
sudo resolvectl flush-caches
```

Clears stored DNS records from the local cache.

This is useful if DNS records have changed but the system still resolves to an old IP address.

---

## 9. Basic Network Troubleshooting Steps

### Check IP

```
ip a
```

### Check Gateway

```
ip route
```

### Check DNS

```
cat /etc/resolv.conf
```

### Test Gateway

```
ping 192.168.1.1
```

### Test Internet

```
ping 8.8.8.8
```

### Test DNS Resolution

```
ping google.com
```

---

## 10. Quick Command Cheat Sheet

| Purpose          | Command                      |
| ---------------- | ---------------------------- |
| Check IP address | ip a                         |
| Check routes     | ip route                     |
| Check DNS        | cat /etc/resolv.conf         |
| Release IP       | sudo dhclient -r             |
| Renew IP         | sudo dhclient                |
| Flush DNS cache  | sudo resolvectl flush-caches |
| Test gateway     | ping 192.168.1.1             |
| Test internet    | ping 8.8.8.8                 |


# Linux Ping Command (Ubuntu) – Complete Guide

## Overview

`ping` is a network troubleshooting utility used to verify connectivity between two systems. It sends an **ICMP Echo Request** to a target host and waits for an **ICMP Echo Reply**. If replies are received, the network path between the systems is working.

---

## How Ping Works

1. Your system sends an **ICMP Echo Request** packet to the destination.
2. The destination device responds with an **ICMP Echo Reply**.
3. Successful replies indicate working network connectivity.
4. No reply may indicate a network, routing, firewall, or DNS issue.

Example:

```bash
ping google.com
```

Example Output:

```
PING google.com (142.250.183.14) 56(84) bytes of data.
64 bytes from 142.250.183.14: icmp_seq=1 ttl=117 time=22 ms
64 bytes from 142.250.183.14: icmp_seq=2 ttl=117 time=20 ms
```

### Output Fields

| Field    | Description                     |
| -------- | ------------------------------- |
| icmp_seq | Packet sequence number          |
| ttl      | Time To Live (remaining hops)   |
| time     | Round‑trip time between systems |

---

## Default Behavior: Linux vs Windows

| System  | Default Behavior                |
| ------- | ------------------------------- |
| Windows | Sends 4 packets then stops      |
| Linux   | Runs continuously until stopped |

Stop continuous ping in Linux using:

```
Ctrl + C
```

Example summary:

```
--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss
```

---

## Send a Specific Number of Packets

Linux uses the `-c` (count) option to control the number of packets.

```bash
ping -c 4 google.com
```

Result:

```
4 packets transmitted, 4 received
```

---

## Ping an IP Address

```bash
ping 8.8.8.8
```

If replies are received, internet connectivity is working.

---

## Hostname Resolution

Linux automatically attempts to resolve hostnames when pinging an IP address.

Example:

```bash
ping 192.168.1.1
```

Possible output:

```
PING router.local (192.168.1.1)
```

---

## Force IPv4

```bash
ping -4 google.com
```

Forces the command to use **IPv4 only**.

---

## Force IPv6

```bash
ping -6 google.com
```

Forces the command to use **IPv6 only**.

---

## TTL (Time To Live)

TTL represents the maximum number of network hops a packet can travel before being discarded.

Each router decreases the TTL value by **1**.

Example:

```
ttl=117
```

This indicates the packet passed through multiple routers before reaching the destination.

---

## Basic Network Troubleshooting Workflow

### 1. Test Local Network (Gateway)

```bash
ping 192.168.1.1
```

If successful, your system can reach the router.

### 2. Test Internet Connectivity

```bash
ping 8.8.8.8
```

If successful, internet connectivity is available.

### 3. Test DNS Resolution

```bash
ping google.com
```

If this fails but the IP ping works, the issue is likely related to DNS.

---

## DNS Troubleshooting Example

Problem: Internet websites are not opening.

Test connectivity:

```bash
ping 8.8.8.8
```

If successful, internet connectivity exists.

Test DNS:

```bash
ping google.com
```

If you see:

```
ping: google.com: Temporary failure in name resolution
```

Conclusion: DNS configuration or DNS server issue.

## DNS Load Balancing (Round Robin)

Running:

```bash
ping google.com
```

May return different IP addresses such as:

```
142.250.183.14
142.250.183.46
142.250.183.78
```

This technique is called **DNS Round Robin**, which distributes traffic across multiple servers for:

* Load balancing
* High availability
* Fault tolerance

## Useful Ping Commands

| Command              | Purpose                      |
| -------------------- | ---------------------------- |
| ping google.com      | Continuous connectivity test |
| ping -c 4 google.com | Send 4 packets               |
| ping -4 google.com   | Force IPv4                   |
| ping -6 google.com   | Force IPv6                   |
| ping 8.8.8.8         | Test internet connectivity   |
| ping 192.168.1.1     | Test router / gateway        |

TraceRoute & MTR
---

**TraceRoute** - To show you that from how many hops your request goes and finally reach to DNS server to resolve name to ip.

```bash
traceroute google.com
```

![alt text](tr.png)

**mtr** - Is shows pkg loss and send / received by all hops from where your requests was goes.

```bash
mtr google.com
```

![alt text](mtr.png)

