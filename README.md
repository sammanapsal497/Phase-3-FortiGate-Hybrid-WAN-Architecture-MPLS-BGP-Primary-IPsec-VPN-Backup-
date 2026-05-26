# Phase 3 FortiGate Hybrid WAN Architecture

## Project Overview
[cite_start]This project designs and simulates a highly redundant, secure enterprise Hybrid WAN environment linking two customer branches[cite: 758, 764]. [cite_start]The architecture leverages a private MPLS network as the primary transit pathway for high-performance data delivery, with dual-ISP paths providing an alternative encrypted Site-to-Site IPsec VPN tunnel during primary link failure events[cite: 759, 760, 761].

[cite_start]The goal of this project was to validate automated failover behavior and analyze the granular stateful routing mechanics of FortiGate Firewalls[cite: 780, 781].

---

## Network Architecture & Topology

### Design Components
* [cite_start]**Customer Edge (CE) Routers:** Interface customer branch sites directly with the service provider network[cite: 768, 769].
* [cite_start]**Provider Edge (PE) Routers:** Distribute customer routes securely across the core using provider protocols[cite: 768, 770].
* [cite_start]**Provider (P) Core Router:** Core MPLS switching fabric[cite: 768, 771].
* [cite_start]**ISP Routers:** Infrastructure paths simulating the public internet cloud[cite: 768, 772].
* [cite_start]**FortiGate Next-Generation Firewalls (NGFW):** Deployed at the edge of both Branch/HQ zones to govern routing policies, traffic classification, and security profiles[cite: 768, 773, 783].

### Routing Implementation
1. [cite_start]**Internal Routing:** Multi-Area OSPF configures LAN routing and routes advertisements into customer edge segments[cite: 794, 1190].
2. [cite_start]**Provider Transit Routing:** Multiprotocol BGP (MP-BGP) running inside the service provider core exchanges routes seamlessly between PE nodes[cite: 795, 1190].
3. [cite_start]**Backup Routing Logic:** Floating static paths pointing toward the ISP-facing interfaces trigger automatic VPN failover upon primary route withdrawal[cite: 796].
4. [cite_start]**Encryption Layer:** Strong cryptographic Site-to-Site IPsec VPN tunnels secure multi-branch traffic across public infrastructure[cite: 761, 797].

---

## Traffic Path Orchestration

### 1. Primary Path (Normal Operation)
* [cite_start]**Path Flow:** `FortiGate (Branch) ➔ CE Router ➔ PE Router ➔ P Core Router ➔ PE Router ➔ CE Router ➔ FortiGate (HQ)`[cite: 789].
* [cite_start]Premium MPLS transport is favored due to better routing preferences, satisfying zero packet loss, and stable line performance[cite: 783, 788].

### 2. Secondary Path (Failover Operation)
* [cite_start]**Path Flow:** `FortiGate (Branch) ➔ ISP-1 ➔ Public Internet Cloud ➔ ISP-2 ➔ FortiGate (HQ)`[cite: 791, 792].
* [cite_start]Triggered natively when the primary MPLS circuit drops[cite: 791]. [cite_start]Traffic dynamically transfers to the secondary ISP loop where an IPsec VPN tunnel is immediately established to encapsulate corporate communications[cite: 784].

---

## Detailed Technical Observation: The Stateful Failback Dilemma

A key finding during physical link reconstruction and failback testing:
* [cite_start]**The Problem:** When the primary MPLS line was restored and OSPF/MP-BGP converged perfectly back into the routing table, existing traffic streams remained trapped over the lower-priority backup IPsec VPN path.
* [cite_start]**The Root Cause:** FortiGate operates as a **stateful firewall**, putting precedence on its **internal session tracking table** over real-time routing table lookups. [cite_start]If a continuous packet stream (e.g., active ICMP, file transfer, socket connection) maintains an open session in the state table, the firewall continuously points egress traffic out the backup path to prevent temporary socket termination. 
* [cite_start]**The Resolution Architecture:** To mitigate this without relying on rigid static policy rules, an **SD-WAN framework** should be layered to actively monitor link health SLA parameters and explicitly manage automated, bidirectional session preservation and clean failback[cite: 746, 747].

---

## Configuration Summary Reference

* **Subnets Sample Configured:**
  * [cite_start]HQ Internal Network Area: `172.16.1.0/24` [cite: 1196]
  * [cite_start]Branch Internal Network Area: `172.16.3.0/24` [cite: 1206]
  * [cite_start]Service Provider AS System: `65400` [cite: 1225]

* **Verification Methods:**
  * [cite_start]Consistent validation achieved via End-to-End ICMP Ping tests and detailed trace paths showing exact label configurations inside the MPLS cloud and encapsulation transitions through the IPsec virtual interface tunnel[cite: 741, 1194].

---
*Developed by Engr. [cite_start]Sam Laurence Manapsal ECT, CCNA* [cite: 756]
