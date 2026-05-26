# 🌐 Phase 3 FortiGate Hybrid WAN Architecture

## 📋 Project Overview
This project designs and simulates a highly redundant, secure enterprise Hybrid WAN environment linking two customer branches. The architecture leverages a private MPLS network as the primary transit pathway for high-performance data delivery, with dual-ISP paths providing an alternative encrypted Site-to-Site IPsec VPN tunnel during primary link failure events. 🛡️

The goal of this project was to validate automated failover behavior and analyze the granular stateful routing mechanics of FortiGate Firewalls.

---

## 🏗️ Network Architecture & Topology

### 🛠️ Design Components
* **🏢 Customer Edge (CE) Routers:** Interface customer branch sites directly with the service provider network.
* **🛰️ Provider Edge (PE) Routers:** Distribute customer routes securely across the core using provider protocols.
* **🎛️ Provider (P) Core Router:** Core MPLS switching fabric.
* **☁️ ISP Routers:** Infrastructure paths simulating the public internet cloud.
* **🔥 FortiGate Next-Generation Firewalls (NGFW):** Deployed at the edge of both Branch/HQ zones to govern routing policies, traffic classification, and security profiles.

### 🛣️ Routing Implementation
1. **🔄 Internal Routing:** Multi-Area OSPF configures LAN routing and routes advertisements into customer edge segments.
2. **🤝 Provider Transit Routing:** Multiprotocol BGP (MP-BGP) running inside the service provider core exchanges routes seamlessly between PE nodes.
3. **⚠️ Backup Routing Logic:** Floating static paths pointing toward the ISP-facing interfaces trigger automatic VPN failover upon primary route withdrawal.
4. **🔒 Encryption Layer:** Strong cryptographic Site-to-Site IPsec VPN tunnels secure multi-branch traffic across public infrastructure.

---

## 🚦 Traffic Path Orchestration

### 🟢 1. Primary Path (Normal Operation)
* **Path Flow:** `FortiGate (Branch) ➔ CE Router ➔ PE Router ➔ P Core Router ➔ PE Router ➔ CE Router ➔ FortiGate (HQ)`
* ⭐ Premium MPLS transport is favored due to better routing preferences, satisfying zero packet loss, and stable line performance.

### 🟡 2. Secondary Path (Failover Operation)
* **Path Flow:** `FortiGate (Branch) ➔ ISP-1 ➔ Public Internet Cloud ➔ ISP-2 ➔ FortiGate (HQ)`
* ⚡ Triggered natively when the primary MPLS circuit drops. Traffic dynamically transfers to the secondary ISP loop where an IPsec VPN tunnel is immediately established to encapsulate corporate communications.

---

## 🔍 Detailed Technical Observation: The Stateful Failback Dilemma

A key finding during physical link reconstruction and failback testing:
* **❌ The Problem:** When the primary MPLS line was restored and OSPF/MP-BGP converged perfectly back into the routing table, existing traffic streams remained trapped over the lower-priority backup IPsec VPN path.
* **🧠 The Root Cause:** FortiGate operates as a **stateful firewall**, putting precedence on its **internal session tracking table** over real-time routing table lookups. If a continuous packet stream (e.g., active ICMP, file transfer, socket connection) maintains an open session in the state table, the firewall continuously points egress traffic out the backup path to prevent temporary socket termination. 
* **💡 The Resolution Architecture:** To mitigate this without relying on rigid static policy rules, an **SD-WAN framework** should be layered to actively monitor link health SLA parameters and explicitly manage automated, bidirectional session preservation and clean failback.

---

## ⚙️ Configuration Summary Reference

* **📊 Subnets Sample Configured:**
  * 🏢 HQ Internal Network Area: `172.16.1.0/24`
  * 🏬 Branch Internal Network Area: `172.16.3.0/24`
  * 🌐 Service Provider AS System: `65400`

* **✅ Verification Methods:**
  * Consistent validation achieved via End-to-End ICMP Ping tests and detailed trace paths showing exact label configurations inside the MPLS cloud and encapsulation transitions through the IPsec virtual interface tunnel.

---
*👨‍💻 Developed by Engr. Sam Laurence Manapsal ECT, CCNA*
