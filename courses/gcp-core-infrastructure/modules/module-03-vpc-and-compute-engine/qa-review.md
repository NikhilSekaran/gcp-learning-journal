# Module 3: VPC and Compute Engine - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module 3 — VPC & Compute Engine. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---

## Topic 1: VPC Fundamentals

### Q1: GCP VPC Scope

**Question:**
What makes Google Cloud VPCs fundamentally different from VPCs in other major cloud providers?

**Options:**
- A) GCP VPCs are limited to a single availability zone
- B) GCP VPCs are global resources that span all regions
- C) GCP VPCs do not support custom IP address ranges
- D) GCP VPCs require a dedicated physical network switch

**Answer:** **B) GCP VPCs are global resources that span all regions**

**Explanation:**
GCP VPCs are global — a single VPC can contain subnets in `us-central1`, `europe-west1`, and `asia-east1`. VMs in different regions within the same VPC can communicate via internal IPs using Google's global private network. This eliminates the need to create separate VPCs per region and configure inter-region peering, which is required in cloud providers where VPCs are regional.

**Why not the others:**
- A) GCP VPCs are not limited to a single zone — zones are the smallest compute unit; VPCs are software-defined networks that span globally
- C) GCP Custom Mode VPCs specifically exist to let you define your own CIDR ranges; Auto Mode assigns pre-defined ranges per region, but custom ranges are fully supported
- D) GCP VPCs are entirely software-defined; no physical hardware is dedicated per customer — isolation is achieved through software-defined networking

---

### Q2: Auto Mode vs. Custom Mode

**Question:**
A company is deploying a production workload that includes a data processing pipeline with multi-region VMs. They need full control over IP address ranges to avoid conflicts with their on-premises network (192.168.0.0/16). Which VPC mode should they choose?

**Options:**
- A) Auto Mode — it automatically assigns IP ranges that don't conflict
- B) Custom Mode — they can specify their own non-overlapping CIDR ranges
- C) Default Mode — the default VPC already avoids conflicts with on-premises
- D) Either mode — VPC mode has no effect on IP range assignment

**Answer:** **B) Custom Mode — they can specify their own non-overlapping CIDR ranges**

**Explanation:**
Auto Mode assigns predefined CIDR blocks per region (e.g., `10.128.0.0/20` for `us-central1`). If the company's on-premises network uses the same ranges, connectivity via VPN or Interconnect will break. Custom Mode lets them choose IP ranges that don't overlap with existing networks — essential for hybrid connectivity.

**Why not the others:**
- A) Auto Mode assigns fixed pre-defined CIDRs like `10.128.0.0/20` for `us-central1` — if the on-premises network uses overlapping ranges, VPN or Interconnect routing will fail silently
- C) The "default" VPC is Auto Mode with the same predefined IP ranges — it has no mechanism to detect or avoid conflicts with on-premises networks
- D) VPC mode directly controls whether IP ranges are pre-assigned (Auto) or user-defined (Custom) — it critically determines whether range conflicts can be avoided

---

## Topic 2: Firewall Rules

### Q3: Default Firewall Behavior

**Question:**
A new Compute Engine VM is deployed in a custom VPC with no firewall rules configured. A developer tries to SSH into the VM from the internet. What happens?

**Options:**
- A) SSH succeeds because all traffic is allowed by default
- B) SSH fails because all inbound traffic is denied by default in a custom VPC
- C) SSH succeeds only if the VM has a public IP address
- D) SSH results in a timeout because custom VPCs don't support SSH

**Answer:** **B) SSH fails because all inbound traffic is denied by default in a custom VPC**

**Explanation:**
In GCP, **all inbound (ingress) traffic is denied by default** in a custom VPC. The developer must explicitly create a firewall rule that allows TCP port 22 from the appropriate source IP. The default VPC comes with pre-configured SSH rules, but a custom VPC does not.

**Why not the others:**
- A) All inbound traffic is NOT allowed by default in a custom VPC — only egress is allowed by default; all ingress is denied
- C) A public IP is required to reach the VM from the internet, but it alone doesn't override firewall rules — traffic would still be blocked by the default deny-all ingress policy even with a public IP
- D) SSH (port 22) is not blocked by VPC type — it's blocked by the absence of a firewall rule allowing it; custom VPCs fully support SSH once an allow rule is added

---

### Q4: Firewall Rule Targeting

**Question:**
A Compute Engine VM is tagged `db-server`. A firewall rule allows TCP:3306 to VMs with target tag `db-server` from source tag `app-server`. A new `app-server` VM is created with a different IP address. Does it automatically gain access to the database?

**Options:**
- A) No — firewall rules based on tags only apply when explicitly reapplied after a VM is replaced
- B) Yes — the firewall rule targets tags, so any VM tagged `app-server` automatically satisfies the source condition regardless of IP
- C) No — tag-based rules only work within the same VPC subnet
- D) Yes — but only if the new VM has the same internal IP as the original

**Answer:** **B) Yes — the firewall rule targets tags, so any VM tagged `app-server` automatically satisfies the source condition regardless of IP**

**Explanation:**
Network tags are arbitrary strings attached to Compute Engine VM instances. Firewall rules can target VMs based on these tags rather than IP addresses, enabling micro-segmentation:

**Example Architecture:**
- `web-server` tag: Allow TCP:80,443 inbound from internet (`0.0.0.0/0`)
- `app-server` tag: Allow TCP:8080 inbound only from VMs with tag `web-server`
- `db-server` tag: Allow TCP:3306 inbound only from VMs with tag `app-server`

Each tier only receives traffic it needs from the tier that should be calling it. As VMs are added, replaced, or scaled out, simply apply the appropriate tag — firewall rules follow automatically without any IP changes needed.

**Why not the others:**
- A) Tag-based firewall rules apply to any VM with the tag at the time of evaluation; no manual reapplication is needed when VMs change
- C) Network tags work across subnets within the same VPC — they are not subnet-scoped
- D) Tags are independent of IP addresses entirely — the IP of the source VM does not need to match anything; only the tag matters

---

## Topic 3: Compute Engine

### Q5: Custom Machine Types Use Case

**Question:**
A machine learning inference service requires exactly 12 vCPUs and 45 GB of RAM. The closest predefined machine type is `n2-standard-16` with 16 vCPUs and 64 GB. What should you do to minimize costs?

**Options:**
- A) Use `n2-standard-16` — predefined types are always more cost-effective
- B) Create a custom machine type with 12 vCPUs and 45 GB of RAM
- C) Use 4 smaller VMs and distribute the workload
- D) Use Preemptible VMs to offset the cost of overprovisioning

**Answer:** **B) Create a custom machine type with 12 vCPUs and 45 GB of RAM**

**Explanation:**
Custom machine types let you specify exact CPU and memory, so you only pay for `12 vCPU + 45 GB RAM` instead of `16 vCPU + 64 GB RAM`. This avoids paying for 4 unused vCPUs and 19 GB of unused RAM. Custom types are billed per vCPU and per GB of memory.

**Why not the others:**
- A) `n2-standard-16` costs ~33% more CPU and ~42% more memory than needed — predefined types are not more cost-effective when they significantly overprovision
- C) Distributing across 4 smaller VMs introduces inter-VM networking overhead, increases management complexity, and may not match the exact 12 vCPU / 45 GB single-machine requirement
- D) Spot VMs offer deep discounts but can be reclaimed by Google at any time — if the ML inference service must be continuously available, Spot VMs are not appropriate

---

### Q6: Vertical vs. Horizontal Scaling

**Question:**
A web application is experiencing high traffic. The team needs to scale the infrastructure. The application is stateless (sessions stored in Memorystore). What is the recommended approach?

**Options:**
- A) Vertical scaling — resize the single VM to add more CPU and RAM
- B) Horizontal scaling — add more VM instances behind a load balancer with autoscaling
- C) Vertical scaling — it is always more cost-effective than horizontal
- D) Add more persistent disks to the existing VM

**Answer:** **B) Horizontal scaling — add more VM instances behind a load balancer with autoscaling**

**Explanation:**
Since the application is **stateless** (sessions are externalized to Memorystore), it's safe to run multiple identical instances. Horizontal scaling provides:
- **No downtime** — new instances join the load balancer while existing ones keep serving
- **Elasticity** — autoscaler adds/removes instances based on real demand
- **Reliability** — if one VM fails, others continue serving

Vertical scaling requires the VM to stop for resizing, causing downtime. A single vertical VM is also a single point of failure.

**Why not the others:**
- A) Vertical scaling requires stopping the VM for resizing — causing downtime; the resized VM also remains a single point of failure with no redundancy
- C) Vertical scaling is not always more cost-effective — paying for a large VM 24/7 when traffic is variable wastes money; horizontal autoscaling adjusts to actual demand
- D) Adding persistent disks increases storage capacity — it doesn't address CPU or memory bottlenecks that cause high-traffic performance issues

---

## Topic 4: VPC Connectivity

### Q7: VPC Peering Limitation

**Question:**
Company A has three VPCs: VPC-1 (project-alpha), VPC-2 (project-beta), and VPC-3 (project-gamma). VPC-1 is peered with VPC-2, and VPC-2 is peered with VPC-3. Can VMs in VPC-1 communicate with VMs in VPC-3?

**Options:**
- A) Yes — peering is transitive, so VPC-1→2→3 communication is automatic
- B) No — VPC Peering is non-transitive; VPC-1 needs a direct peering with VPC-3
- C) Yes — but only if all VPCs are in the same project
- D) No — VPC Peering only works within the same organization

**Answer:** **B) No — VPC Peering is non-transitive; VPC-1 needs a direct peering with VPC-3**

**Explanation:**
GCP VPC Peering is **non-transitive**. A peering between A and B, and B and C, does NOT create a path from A to C. Each pair of VPCs that needs to communicate must have a **direct peering** established. For large-scale multi-VPC connectivity, consider Shared VPC or Cloud VPN.

**Why not the others:**
- A) VPC Peering is explicitly non-transitive by design — peerings between A↔B and B↔C do not create an implicit A↔C path; this is a key exam fact
- C) VPC Peering works cross-project and even cross-organization — the project boundary is not the constraint; the issue is the non-transitive topology
- D) VPC Peering supports cross-organization peering — being in the same organization is not a requirement

---

### Q8: Shared VPC Use Case

**Question:**
A large organization has 15 GCP projects managed by different teams. A central network security team must enforce consistent firewall rules and control all subnet definitions across all projects. Which GCP networking approach achieves this?

**Options:**
- A) VPC Peering — connect all 15 VPCs to each other in a full-mesh topology
- B) Cloud VPN — create encrypted tunnels between all project VPCs
- C) Shared VPC — a host project owns the network; service projects deploy workloads into shared subnets controlled by the central team
- D) Separate independent VPCs with project-level firewall rules reviewed in a quarterly audit

**Answer:** **C) Shared VPC — a host project owns the network; service projects deploy workloads into shared subnets controlled by the central team**

**Explanation:**
**Shared VPC** is preferred when you want **centralized network management** across multiple projects:

- A **host project** owns and controls the VPC (subnets, firewall rules, routes)
- **Service projects** consume resources from the host VPC — their VMs, GKE clusters, and other resources get IPs from the shared subnets
- A central networking team manages security policy for all projects in one place
- Individual project teams cannot modify network configuration — they just deploy workloads

**Choose VPC Peering when:** Two independently managed networks (possibly different orgs) need to communicate while each retaining control of their own VPC.

**Why not the others:**
- A) VPC Peering is non-transitive and requires each pair to establish their own peering — 15 projects would require up to 105 peering connections, and each team still controls their own firewall rules
- B) Cloud VPN is for connecting on-premises networks or networks across organizations over encrypted tunnels — not designed for centralized network management within a single org's projects
- D) Separate VPCs with audits don't enforce rules in real time — teams can still misconfigure their networks between audits; Shared VPC enforces policy continuously

---

## Topic 5: Load Balancing

### Q9: Choosing a Load Balancer

**Question:**
A global e-commerce site serves users across the US, Europe, and Asia. They need:
- HTTPS with SSL termination
- URL-based routing (`/api/*` to backend services, `/static/*` to CDN buckets)
- Automatic failover if a region is down

Which load balancer type should they use?

**Options:**
- A) Regional External Network TCP/UDP Load Balancer
- B) Internal TCP/UDP Load Balancer
- C) Global External HTTP(S) Load Balancer
- D) Regional External HTTP(S) Load Balancer

**Answer:** **C) Global External HTTP(S) Load Balancer**

**Explanation:**
The **Global External HTTP(S) Load Balancer** is the right choice because:
- **Global scope**: runs on Google's edge PoPs; users hit the nearest PoP globally
- **HTTPS and SSL termination**: handles TLS certificates and offloads encryption
- **URL-based routing**: route based on URL path, hostname, or headers to different backends
- **Cross-region failover**: automatically routes to healthy backends in other regions if one fails

Regional LBs cannot do cross-region failover. Internal LBs are for private VPC traffic, not internet-facing applications.

**Why not the others:**
- A) Regional External Network TCP/UDP Load Balancer is L4 (TCP/UDP) and regional only — cannot do URL-based routing and has no cross-region failover
- B) Internal TCP/UDP Load Balancer is for private VPC traffic only — it doesn't serve internet traffic and cannot be used for a public-facing application
- D) Regional External HTTP(S) Load Balancer supports L7 features (URL routing, SSL) but is scoped to one region — no cross-region failover for a globally distributed site

---

### Q10: Layer 4 vs. Layer 7 Load Balancing

**Question:**
A team needs to load balance a multiplayer game server that uses TCP on custom ports 7000–7100. The backend protocol is not HTTP. Which GCP load balancer type is appropriate?

**Options:**
- A) Global External HTTP(S) Load Balancer — handles all TCP traffic globally
- B) Regional External Network TCP/UDP Load Balancer — operates at Layer 4, routes TCP/UDP without requiring HTTP
- C) Internal HTTP(S) Load Balancer — L7 load balancer for private VPC traffic
- D) Cloud CDN with origin failover — handles high-throughput TCP connections

**Answer:** **B) Regional External Network TCP/UDP Load Balancer — operates at Layer 4, routes TCP/UDP without requiring HTTP**

**Explanation:**

| Feature               | Layer 4 (TCP/UDP)              | Layer 7 (HTTP/HTTPS)                  |
|-----------------------|--------------------------------|---------------------------------------|
| **Traffic visibility** | IP addresses, ports, TCP state | Full HTTP headers, URL paths, cookies |
| **Routing decisions** | Based on IP/port | Based on URL, hostname, HTTP headers |
| **Protocol support** | Any TCP/UDP protocol | HTTP and HTTPS only |
| **Session persistence** | IP-based (5-tuple) | Cookie-based or header-based |
| **Use cases** | Non-HTTP apps, gaming, databases, raw TCP | Web apps, REST APIs, microservices |

**Choose Layer 4 when:** Load balancing non-HTTP traffic (game servers, TCP messaging, database proxies).
**Choose Layer 7 when:** URL-based routing, SSL termination, WAF (Cloud Armor), or request-level observability.

**Why not the others:**
- A) The Global External HTTP(S) LB is L7 and only processes HTTP/HTTPS traffic — it cannot route arbitrary TCP protocols on custom ports
- C) The Internal HTTP(S) LB is L7 and only handles HTTP/HTTPS traffic — also internal-only, not suitable for internet-facing game servers
- D) Cloud CDN is a content delivery service for caching static assets — not a load balancer for stateful TCP game connections

---

## Topic 6: Hybrid Connectivity

### Q11: Choosing Connectivity Option

**Question:**
A financial services company needs to connect their on-premises data center to GCP with guaranteed bandwidth of 10 Gbps, a latency SLA, and traffic that must never traverse the public internet. What is the most appropriate connectivity option?

**Options:**
- A) Cloud VPN with high-availability tunnels
- B) Dedicated Interconnect
- C) Partner Interconnect
- D) Cloud CDN with origin shielding

**Answer:** **B) Dedicated Interconnect**

**Explanation:**
**Dedicated Interconnect** provides a direct physical fiber connection between the on-premises network and Google's network — traffic never traverses the public internet. It offers:
- 10 Gbps or 100 Gbps dedicated circuits
- 99.9% to 99.99% SLA
- Consistent, low-latency performance

Cloud VPN encrypts traffic but it still travels over the **public internet**, which doesn't meet the requirement. Partner Interconnect is suitable when you can't get Dedicated Interconnect directly (e.g., the data center isn't near a Google colocation facility), but Dedicated Interconnect is preferred for enterprise-grade requirements.

**Why not the others:**
- A) Cloud VPN encrypts traffic with IPsec but routes it over the public internet — doesn't satisfy the "never traverse the public internet" requirement
- C) Partner Interconnect avoids the public internet via a third-party provider but relies on the partner's network; Dedicated Interconnect is preferred when enterprise-grade 10 Gbps with a direct Google SLA is required
- D) Cloud CDN with origin shielding is a content delivery optimization tool — not a network connectivity option between on-premises and GCP

---

### Q12: Cloud VPN vs. Interconnect

**Question:**
A startup has 50 remote employees who need private access to GCP-hosted internal tools. Their budget is limited. They currently have a 500 Mbps internet connection at the office. Which hybrid connectivity option is most appropriate?

**Options:**
- A) Dedicated Interconnect — provides the most reliable connection with a 10 Gbps circuit and Google SLA
- B) Partner Interconnect — avoids the public internet and costs less than Dedicated
- C) Cloud VPN — cost-effective, fast to deploy, encrypted over IPsec, sufficient bandwidth for 50 users
- D) Direct Peering — free BGP peering with Google's network, most cost-effective option

**Answer:** **C) Cloud VPN — cost-effective, fast to deploy, encrypted over IPsec, sufficient bandwidth for 50 users**

**Explanation:**
For this scenario:
- **Cost:** Cloud VPN is significantly cheaper than Dedicated or Partner Interconnect (which involve leased line costs)
- **Bandwidth:** 50 employees accessing internal apps don't need 10 Gbps — Cloud VPN supports up to ~3 Gbps per tunnel (can add parallel tunnels)
- **Setup time:** VPN is quick to configure; Interconnect requires physical provisioning that takes weeks
- **Traffic:** IPsec encryption is sufficient; these aren't high-frequency database replications

**Tradeoff:** VPN traffic traverses the public internet (encrypted via IPsec) — less predictable latency than Interconnect. If the company later grows significantly or adds high-bandwidth workloads (e.g., DB replication), Partner Interconnect would be a natural upgrade.

**Why not the others:**
- A) Dedicated Interconnect requires a minimum 10 Gbps physical circuit and colocation facility access — massive overkill and cost for 50 users; setup takes weeks to months
- B) Partner Interconnect avoids the public internet but involves service provider fees and provisioning delays — the latency improvement over VPN doesn't justify the cost for 50 users on an internal tool
- D) Direct Peering provides BGP peering to reach Google services (like Google APIs) but is not designed for private enterprise connectivity to GCP VPC resources; it has no Google-backed SLA

---

### Q13: VPC Subnet Scope

**Question:**
What is the scope (geographic boundary) of a VPC subnet in Google Cloud?

**Options:**
- A) Global (spans all regions)
- B) Regional (spans zones within a single region)
- C) Zonal (limited to a single zone)
- D) It depends on the VPC type (Auto or Custom)

**Answer:** **B) Regional (spans zones within a single region)**

**Official Explanation:**
Correct. VPC subnets in Google Cloud are **regional** resources. A subnet's IP range spans all zones within that region. This means a single subnet can have instances in different zones simultaneously, which simplifies fault-tolerant designs without needing separate subnets per zone.

**Exam Key Point:** "Subnets span the zones that make up a region" — this is why they're regional, not zonal.

**Why not the others:**
- A) Global — the VPC itself is global, but subnets are regional; confusing VPC scope with subnet scope is a common exam mistake
- C) Zonal — GCP subnets span all zones within a region; a subnet in `us-east1` covers `us-east1-b`, `us-east1-c`, and `us-east1-d` simultaneously
- D) It depends on VPC type — subnet scope is always regional regardless of whether the VPC is Auto Mode or Custom Mode

---

### Q14: Main Reason for Preemptible / Spot VMs

**Question:**
What is the main reason customers choose Preemptible or Spot VMs in Google Cloud?

**Options:**
- A) They have dedicated hardware with no interference from other tenants
- B) To reduce cost significantly
- C) Because they have higher CPU and memory performance
- D) They offer committed use contract pricing

**Answer:** **B) To reduce cost significantly**

**Official Explanation:**
Correct. Preemptible (now called **Spot VMs**) offer per-hour prices that incorporate a **substantial discount** compared to regular VMs, because Google can reclaim these VMs when it needs the capacity for higher-priority workloads. The performance is the same as a regular VM — the difference is reliability and availability.

**When to use:** Batch jobs, fault-tolerant workloads, data processing pipelines that can handle interruption.

**Why not the others:**
- A) Spot VMs do not have dedicated hardware — they run on the same shared infrastructure as regular VMs; Google can reclaim the capacity at any time
- C) Performance is identical to regular VMs — the discount reflects the risk of interruption, not any hardware advantage or disadvantage
- D) Committed use contracts are a separate pricing program for regular VMs — not related to Spot/Preemptible pricing

---

### Q15: Which Interconnect Option Has an SLA?

**Question:**
Which of the following GCP connectivity options comes with a Service Level Agreement (SLA) for uptime?

**Options:**
- A) Dedicated Interconnect
- B) Direct Peering
- C) Carrier Peering
- D) Standard Internet

**Answer:** **A) Dedicated Interconnect**

**Official Explanation:**
Correct. **Dedicated Interconnect** and **Partner Interconnect** both come with SLAs (up to 99.99% with redundant connections). **Direct Peering** and **Carrier Peering** do **not** come with Google-backed SLAs — they rely on bilateral peering agreements, making them less suitable for enterprise workloads with strict uptime requirements.

**Why not the others:**
- B) Direct Peering is BGP-based peering at internet exchange points — no Google-backed SLA; primarily for accessing Google services at reduced latency, not for enterprise workloads
- C) Carrier Peering allows access to Google via a service provider's network — also no Google-backed SLA; reliability depends on the carrier
- D) Standard internet has no SLA at all — reliability depends entirely on the routing path and ISPs involved

---

### Q16: Global Cloud Load Balancing

**Question:**
How does Google Cloud's global Cloud Load Balancing distribute incoming HTTP traffic?

**Options:**
- A) Using round-robin DNS across multiple IP addresses
- B) Based on the proximity of each backend to one specific region
- C) Only across instances within a single Compute Engine zone
- D) Across multiple Compute Engine regions to the closest healthy backend

**Answer:** **D) Across multiple Compute Engine regions to the closest healthy backend**

**Official Explanation:**
Correct. Google Cloud's global external Application Load Balancer operates at layer 7 and can route traffic to backends in **multiple regions worldwide**. It uses a single anycast IP address and routes requests to the closest healthy instance based on load, health, and proximity — enabling low-latency responses globally from a single VIP.

**Why not the others:**
- A) Round-robin DNS is a simple technique that rotates between multiple IP addresses — GCP's global LB uses a single anycast IP and routes based on proximity and health, not DNS rotation
- B) GCP's global LB considers multiple factors including proximity, load, and health across all configured regions — not just one specific region
- C) A single-zone load balancer would be a regional resource, not a global one — the question specifically refers to Google Cloud's global LB

---

### Q17: Describing a VPC

**Question:**
Which of the following best describes a Virtual Private Cloud (VPC) in Google Cloud?

**Options:**
- A) A set of virtual machines that share a single IP address block
- B) A managed private data center physically allocated to a single customer
- C) A secure, individual, private cloud-computing model hosted within a public cloud
- D) A VLAN segment that spans a single data center

**Answer:** **C) A secure, individual, private cloud-computing model hosted within a public cloud**

**Official Explanation:**
Correct. A VPC is a **logically isolated** virtual network within Google's public cloud infrastructure. It provides the same benefits as a private network (isolation, internal routing, private IP space) while leveraging the scale and global reach of Google's public cloud. VPCs are software-defined, not tied to physical network segments.

**Why not the others:**
- A) VMs in a VPC each have unique IP addresses — they don't share a single IP block; IP ranges are assigned per subnet, and individual VMs receive unique IPs from those subnets
- B) GCP VPCs are software-defined and multi-tenant — multiple customers share the same physical hardware with logical isolation; no dedicated hardware is physically allocated per customer
- D) A VLAN is a layer 2 network segment in traditional physical networking — GCP VPCs are layer 3 software-defined networks that span globally across regions, not a VLAN in a single data center
