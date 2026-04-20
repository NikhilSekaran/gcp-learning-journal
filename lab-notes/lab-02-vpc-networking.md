# Lab 2: Get Started with Virtual Private Cloud Networking and Compute Engine

## Lab Info

| Field | Detail |
|---|---|
| **Course** | Google Cloud Fundamentals: Core Infrastructure |
| **Official Title** | Get Started with Virtual Private Cloud Networking and Compute Engine |
| **Session** | GET_CERTIFIED-06 · Day 1 (2026.04.08) |
| **Lab Type** | Hands-on (Introductory) |
| **Duration** | 40 minutes · 1 Credit |
| **Related Module** | Module 3: VPC & Compute Engine |

---

## 🎯 Objective

Explore the default VPC network, delete it, prove VMs cannot exist without a VPC, then recreate it as an auto mode VPC named `mynetwork`. Deploy two VMs in different regions and test connectivity while systematically removing firewall rules to observe the effect on each traffic type.

---

## 🧠 Concepts Covered

- Default VPC network structure: subnets, routes, firewall rules
- Auto mode vs. custom mode VPC networks
- How deleting a VPC removes all routes and firewall rules with it
- VM instance creation requires a VPC network
- Firewall rule types: allow-icmp, allow-ssh, allow-rdp, allow-custom
- Testing connectivity: SSH (tcp:22), internal IP ping, external IP ping
- Effect of firewall rule removal on each traffic type

---

## 🔧 Lab Steps

### Task 1: Explore the Default Network

#### View Subnets
1. Navigation Menu → **VPC network → VPC networks**
2. Click **default** → click **Subnets**
3. Observe: one subnet per Google Cloud region, each with an RFC 1918 private IP range and a gateway

#### View Routes
1. In the left pane, click **Routes**
2. In **Effective Routes**, set Network = `default`, select your Region, click **View**
3. Observe: one route per subnet (managed automatically; custom static routes can be added)

#### View Firewall Rules
1. In the left pane, click **Firewall**
2. The default network has **4 ingress firewall rules**:

| Rule | Traffic Allowed | Source |
|---|---|---|
| `default-allow-icmp` | ICMP (ping) | `0.0.0.0/0` (anywhere) |
| `default-allow-rdp` | TCP:3389 (RDP) | `0.0.0.0/0` |
| `default-allow-ssh` | TCP:22 (SSH) | `0.0.0.0/0` |
| `default-allow-internal` | All TCP, UDP, ICMP within the network | `10.128.0.0/9` |

> There are also 2 **implied rules** that cannot be deleted: block all ingress, allow all egress.

#### Delete Firewall Rules and the Default Network

1. Select **all** firewall rules → click **Delete** → confirm
2. Navigation Menu → **VPC network → VPC networks**
3. Select **default** → click **Delete VPC network** → confirm
4. Wait for deletion to complete
5. Verify: **Routes** → no routes; **Firewall** → no rules

> **Key insight:** Deleting a VPC removes all its routes and firewall rules simultaneously.

#### Try to Create a VM Without a VPC (Expected Failure)

1. Navigation Menu → **Compute Engine → VM instances**
2. Click **Create instance** → accept defaults → click **Create**
3. An error appears on the **Networking tab**: _"no more networks"_ / _"no network available"_
4. Click **Cancel**

> **Confirmed:** You cannot create a VM instance without a VPC network.

---

### Task 2: Create a VPC Network and VM Instances

#### Create an Auto Mode VPC: `mynetwork`

1. Navigation Menu → **VPC network → VPC networks**
2. Click **Create VPC network**

| Setting | Value |
|---|---|
| **Name** | `mynetwork` |
| **Subnet creation mode** | **Automatic** (auto mode — creates a subnet in every region) |
| **Firewall rules** | Select **all available rules** (allow-icmp, allow-custom, allow-rdp, allow-ssh) |

3. Click **Create**
4. Observe: a subnet was created automatically for each region

> Note: The `deny-all-ingress` and `allow-all-egress` implied rules are shown but cannot be checked or unchecked — they are always present with lower priority (higher integer = lower priority).

#### Create VM 1: `mynet-us-vm`

1. Navigation Menu → **Compute Engine → VM instances**
2. Click **Create Instance**

| Setting | Value |
|---|---|
| **Name** | `mynet-us-vm` |
| **Region** | Region 1 (lab-assigned) |
| **Zone** | Zone 1 (lab-assigned) |
| **Series** | E2 |
| **Machine type** | `e2-micro` (2 vCPU, 1 GB memory) |

3. Click **Create**

#### Create VM 2: `mynet-r2-vm`

1. Click **Create Instance**

| Setting | Value |
|---|---|
| **Name** | `mynet-r2-vm` |
| **Region** | Region 2 (lab-assigned, different from Region 1) |
| **Zone** | Zone 2 |
| **Series** | E2 |
| **Machine type** | `e2-micro` |

2. Click **Create**

> **Note:** Both VMs receive ephemeral external IPs. If a VM is stopped and restarted, it gets a new external IP. Use reserved static IPs when a permanent external IP is required.

---

### Task 3: Explore Connectivity and Firewall Rule Effects

#### Baseline Connectivity Test (All Rules Present)

1. Navigation Menu → **Compute Engine → VM instances**
2. Note the **internal** and **external IP** of `mynet-r2-vm`
3. Click **SSH** for `mynet-us-vm`

> SSH works because of `mynetwork-allow-ssh`, which allows tcp:22 from `0.0.0.0/0`.

**Ping by internal IP:**
```bash
ping -c 3 <mynet-r2-vm internal IP>
```
✅ Succeeds — `mynetwork-allow-custom` allows all traffic within the VPC

**Ping by external IP:**
```bash
ping -c 3 <mynet-r2-vm external IP>
```
✅ Succeeds — `mynetwork-allow-icmp` allows ICMP from `0.0.0.0/0`

> **Lab quiz answer:** The rule that allows pinging the external IP is **`mynetwork-allow-icmp`**.

---

#### Remove `allow-icmp` — Effect on External IP Ping

1. Navigation Menu → **VPC network → Firewall**
2. Select `mynetwork-allow-icmp` → **Delete** → confirm; wait for deletion

**Ping by internal IP:**
```bash
ping -c 3 <mynet-r2-vm internal IP>
```
✅ Still succeeds — `allow-custom` still permits intra-VPC traffic

**Ping by external IP:**
```bash
ping -c 3 <mynet-r2-vm external IP>
```
❌ 100% packet loss — `allow-icmp` deleted; ICMP from outside the VPC is now blocked

---

#### Remove `allow-custom` — Effect on Internal IP Ping

1. Select `mynetwork-allow-custom` → **Delete** → confirm; wait for deletion

**Ping by internal IP:**
```bash
ping -c 3 <mynet-r2-vm internal IP>
```
❌ 100% packet loss — `allow-custom` deleted; intra-VPC traffic now blocked

```bash
exit
```

---

#### Remove `allow-ssh` — Effect on SSH Access

1. Select `mynetwork-allow-ssh` → **Delete** → confirm; wait for deletion
2. Navigation Menu → **Compute Engine → VM instances**
3. Click **SSH** for `mynet-us-vm`

❌ **Connection failed** — `allow-ssh` deleted; tcp:22 ingress is now blocked

---

## 💡 Key Takeaways

| Observation | Insight |
|---|---|
| **Deletion cascades** | Deleting a VPC also deletes all its routes and firewall rules |
| **No VPC = No VMs** | A Compute Engine VM cannot be created without a VPC network |
| **Auto mode VPC** | Creates one subnet per region automatically; easiest way to recreate the default network |
| **allow-icmp** | Controls ICMP ping from the internet (to external IP) |
| **allow-custom** | Controls internal traffic between VMs on the same VPC (internal IP) |
| **allow-ssh** | Controls SSH access (tcp:22 from the internet) |
| **Implied rules** | `deny-all-ingress` and `allow-all-egress` always exist; cannot be deleted; lowest priority |
| **Ephemeral external IPs** | VMs get a new external IP each restart; use static IPs for permanence |

---

## 🔬 Firewall Rule Removal — Effect Summary

| Firewall State | SSH | Internal Ping | External Ping |
|---|---|---|---|
| All rules present (baseline) | ✅ | ✅ | ✅ |
| `allow-icmp` removed | ✅ | ✅ | ❌ |
| `allow-custom` also removed | ✅ | ❌ | ❌ |
| `allow-ssh` also removed | ❌ | ❌ | ❌ |

---

## 📌 Certification Relevance

- **ACE Exam:** Every GCP project has a **default VPC with 4 default ingress firewall rules**
- Auto mode VPC creates subnets in every region; custom mode gives full control over IP ranges
- Firewall rules have **priority** — lower number = higher priority; implied rules are always lowest (65534/65535)
- VMs can only be created when a VPC with available subnets in that region exists
- Ephemeral vs. static external IP behavior is a common scenario question on ACE
- Know which rule controls which traffic: `allow-icmp` (external ping), `allow-custom` (internal traffic), `allow-ssh` (SSH), `allow-rdp` (Windows RDP)

