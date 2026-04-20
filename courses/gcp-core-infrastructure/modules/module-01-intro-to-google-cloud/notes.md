# Module 1: Introduction to Google Cloud

## Status: ✅ Completed (Day 1 · 2026.04.08)

## 🔗 Quick Navigation

- Q&A Review: [qa-review.md](qa-review.md)

---

## 📝 Learning Objectives

By the end of this module, you will understand:

- [x] The NIST definition of cloud computing and its five essential characteristics
- [x] The historical evolution from on-premises infrastructure to cloud computing
- [x] The three cloud service models (IaaS, PaaS, SaaS) and their billing differences
- [x] Google Cloud's global infrastructure: regions, zones, and edge points of presence
- [x] Google's shared responsibility model for security
- [x] GCP's pricing model — per-second billing, sustained use discounts, custom machine types
- [x] How open source technologies (Kubernetes, TensorFlow) help avoid vendor lock-in

---

## 📚 Key Concepts

### 1. Cloud Computing — NIST Definition

The **National Institute of Standards and Technology (NIST)** defines cloud computing with five essential characteristics:

| # | Characteristic             | Description                                                          | GCP Implementation                           |
|---|----------------------------|----------------------------------------------------------------------|----------------------------------------------|
| 1 | **On-demand self-service** | Provision resources without human interaction with the provider      | Cloud Console, gcloud CLI, REST APIs         |
| 2 | **Broad network access**   | Capabilities available over the network; accessible from any device  | Global HTTPS endpoints, Cloud Console        |
| 3 | **Resource pooling**       | Multi-tenant model; provider resources pooled for multiple customers | Shared physical infra with logical isolation |
| 4 | **Rapid elasticity**       | Resources scale up/down quickly, appearing unlimited to the user     | VM autoscaling, Cloud Run scale-to-zero      |
| 5 | **Measured service**       | Usage is monitored, controlled, reported; pay for what you use       | Per-second billing, Cloud Monitoring         |

> **Exam Tip:** The NIST five characteristics are a foundational exam topic. Memorize the acronym **O-B-R-R-M**: On-demand, Broad network, Resource pooling, Rapid elasticity, Measured service.

---

### 2. Historical Evolution of Infrastructure

```
On-Premises → Colocation → Virtualization → Containerization (Cloud)
```

| Era                          | Description                                                             | Customer Controls            | Cost Model            |
|------------------------------|-------------------------------------------------------------------------|------------------------------|-----------------------|
| **On-Premises**              | All hardware owned and operated in your own data center                 | Everything (hardware → apps) | High CapEx            |
| **Colocation**               | Rent rack space and power in a shared data center; you own the hardware | Hardware + software          | High CapEx (hardware) |
| **Virtualization**           | Run multiple VMs on shared physical hardware managed by hypervisors     | VMs, OS, apps                | Lower CapEx           |
| **Containerization / Cloud** | Lightweight app packaging sharing OS kernel; deploy to managed cloud    | App code and config          | OpEx / pay-per-use    |

**Key Drivers of Cloud Adoption:**
- **Agility:** Provision infrastructure in minutes vs. weeks for hardware procurement
- **Cost:** Eliminate idle resource costs; pay only for what you consume
- **Scale:** Access to global infrastructure on demand without pre-planning capacity
- **Focus:** Let Google manage hardware, hypervisors, and physical security

---

### 3. Cloud Service Models

| Model    | Full Name                   | Provider Manages                       | Customer Manages                  | GCP Examples                          |
|----------|-----------------------------|----------------------------------------|-----------------------------------|---------------------------------------|
| **IaaS** | Infrastructure as a Service | Physical hardware, network, hypervisor | VMs, OS, patches, apps, data, IAM | Compute Engine, Cloud Storage         |
| **PaaS** | Platform as a Service       | Infrastructure + runtime + middleware  | Application code and data         | App Engine, Cloud Run, Cloud SQL      |
| **SaaS** | Software as a Service       | Everything, including the application  | User config and data input        | Google Workspace (Gmail, Meet, Drive) |

**Billing Model Comparison:**

| Model | You Pay For                                                  | Idle Cost                                     |
|-------|--------------------------------------------------------------|-----------------------------------------------|
| IaaS  | Resources **allocated** (VM uptime regardless of traffic)    | Yes — running VM incurs charges               |
| PaaS  | Resources **actually used** (app runtime / request duration) | Minimal — scales to zero with serverless PaaS |
| SaaS  | Per-user or subscription                                     | Yes — flat fee regardless of actual usage     |

> **Official slide:** "In the IaaS model, customers pay for the resources they allocate ahead of time; in the PaaS model, customers pay for the resources they actually use."

**Control vs. Convenience Trade-off:**

```
MORE CONTROL ◄─────────────────────────────────► MORE MANAGED
On-Premises        IaaS        PaaS        SaaS
```

|                    | On-Premises                               | IaaS                                   | PaaS                                 | SaaS                              |
|--------------------|-------------------------------------------|----------------------------------------|--------------------------------------|-----------------------------------|
| **GCP Example**    | Your own data center                      | Compute Engine                         | App Engine / Cloud Run               | Google Workspace                  |
| **You manage**     | Hardware, network, OS, runtime, app, data | OS, patches, runtime, app, data        | App code and data only               | Nothing                           |
| **Google manages** | Nothing                                   | Physical hardware, network, hypervisor | Hardware + OS + runtime + middleware | Everything                        |
| **Billing**        | CapEx (upfront hardware cost)             | Per VM uptime (allocated resources)    | Per request / active use             | Per user / subscription           |
| **Best for**       | Full control, strict compliance           | Custom OS/software, lift-and-shift     | Modern apps, fast deployment         | Productivity tools, no-code users |

> **Q: What is the billing difference between IaaS and PaaS on Google Cloud?**
>
> **A:** IaaS (Compute Engine) charges for VM uptime — you pay even when your app serves zero requests. PaaS services like Cloud Run scale to zero and charge only for active request duration.

---

### 4. Google Cloud Global Infrastructure

**Scale (as of 2026 training):**
- **43 Regions** across all major continents
- **130+ Zones** (each zone is an independent data center)
- **Edge Points of Presence (PoPs)** globally for CDN and low-latency routing

**Region:**
- A geographic area (e.g., `us-central1` = Iowa, USA; `europe-west1` = Belgium)
- Contains **3 or more** zones
- Resources deployed per-region for compliance and latency optimization

**Zone:**
- A single isolated data center within a region (e.g., `us-central1-a`, `us-central1-b`)
- A failure in one zone does **not** affect other zones
- Zonal resources (like Compute Engine VMs) live in a specific zone

**Edge Points of Presence (PoPs):**
- Locations where Google's network connects to the public internet
- Used by Cloud CDN, Cloud Interconnect, and Google's backbone routing
- Provide low latency and high throughput globally

**High Availability Design Patterns:**

| Goal                          | Deployment Strategy                          |
|-------------------------------|----------------------------------------------|
| Fault tolerance (same region) | Deploy across 2–3 zones in one region        |
| Disaster recovery             | Deploy across 2+ regions                     |
| Global low-latency access     | Global load balancers + multi-region storage |

**Region Selection Criteria:**

1. **Compliance / Data Residency** — Laws may require data to stay in a specific country or region
2. **Latency** — Deploy closer to end users for lower response times
3. **Cost** — Pricing varies slightly by region
4. **Feature Availability** — Not all GCP services are available in every region

**Environmental Commitment:**

| Milestone           | Achievement                                           |
|---------------------|-------------------------------------------------------|
| **Founding decade** | First major company to be **carbon neutral**          |
| **Second decade**   | First company to achieve **100% renewable energy**    |
| **2030 goal**       | First major company to operate **carbon free** (24/7) |

- Data centers use roughly 2% of world's electricity globally
- Google data centers were the **first to achieve ISO 14001 certification** — a framework for improving resource efficiency and reducing waste
- Notable: Hamina, Finland data center uses **sea water** from the Bay of Finland for cooling — first of its kind in the world
- Data centers optimized for Power Usage Effectiveness (PUE)

---

### 5. Security & Shared Responsibility (Overview)

> Full detail in [Module 2 — IAM & Identity](../module-02-resource-hierarchy-and-iam/notes.md)

**Shared Responsibility Model:**

| Security Layer                                    | Responsibility                                  |
|---------------------------------------------------|-------------------------------------------------|
| Physical data center (building, power, cooling)   | **Google**                                      |
| Custom hardware design                            | **Google** (Titan chips for firmware integrity) |
| 6-layer physical data center security             | **Google**                                      |
| Communication encryption within GCP network       | **Google**                                      |
| DDoS mitigation at network edge                   | **Google**                                      |
| Operating system (for IaaS VMs)                   | **Customer**                                    |
| Application code and configuration                | **Customer**                                    |
| IAM roles, permissions, and policies              | **Customer**                                    |
| Firewall rules for VMs                            | **Customer**                                    |
| Data classification and encryption key management | **Customer**                                    |

**Default Protections (Zero-Config):**
- All customer data encrypted **at rest** by default
- All traffic encrypted **in transit** inside Google's network
- Bug bounty programs reward independent security researchers

**Security Layers (Official ILT Detail):**

| Layer                      | Google's Protection                                                                                                                                    |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Hardware**               | Custom server boards + networking chips; Titan chip for secure boot (verifies BIOS, bootloader, kernel); multi-layer data center premises security     |
| **Service Deployment**     | Infrastructure auto-encrypts all **inter-service RPC traffic** between data centers; hardware crypto accelerators                                      |
| **User Identity**          | Login challenges based on risk factors (device, location history); U2F (Universal 2nd Factor) open standard for MFA                                    |
| **Storage Services**       | Apps access storage via centrally managed encryption keys; hardware encryption on HDDs and SSDs                                                        |
| **Internet Communication** | **Google Front End (GFE)**: terminates TLS, enforces perfect forward secrecy, applies multi-tier/multi-layer **DoS protection**                        |
| **Operational Security**   | Rules + machine intelligence intrusion detection; Red Team exercises; employee U2F security keys; two-party code review; Vulnerability Rewards Program |

> **Google Front End (GFE):** The perimeter proxy that all internet-facing Google services register with. TLS is terminated here, and Google's DoS mitigation scale is so large it can absorb many attacks outright.

---

### 6. Pricing Model

**Core Principle:** Pay-per-use with no upfront commitments required.

| Pricing Feature             | Description                                        | Max Saving                   |
|-----------------------------|----------------------------------------------------|------------------------------|
| **Per-second billing**      | VMs billed per second (1-minute minimum)           | Eliminates idle-second waste |
| **Sustained use discounts** | Auto-applied for VMs running > 25% of the month    | Up to **30% off**            |
| **Committed use discounts** | 1 or 3-year commitment for VM resources            | Up to **57% off**            |
| **Spot (Preemptible) VMs**  | Short-lived VMs Google can reclaim anytime         | Up to **91% off**            |
| **Custom machine types**    | Specify exact vCPU + RAM to avoid overprovisioning | Right-sized savings          |

**Sustained Use Discount Detail:**
- Kicks in automatically once a VM runs > 25% of the billing month
- The longer the VM runs, the greater the discount (up to 30%)
- **No contract, no action required** — applied automatically

**Cost Management Tools:**

| Tool                   | Purpose                                                                                     |
|------------------------|---------------------------------------------------------------------------------------------|
| **Pricing Calculator** | Estimate monthly costs before deploying (cloud.google.com/products/calculator)              |
| **Budgets**            | Set spending limit per billing account or project; fixed amount or % of prior month's spend |
| **Alerts**             | Email notifications at configurable % thresholds (e.g., 50%, 90%, 100%); can use Pub/Sub    |
| **Reports**            | Visual dashboard in Cloud Console to monitor spend by project or service                    |
| **Quotas**             | Hard limits on resource consumption; protects against errors and malicious attacks          |

**Quota Types:**

| Type                 | Description                         | Example                                          |
|----------------------|-------------------------------------|--------------------------------------------------|
| **Rate Quota**       | Resets after a specific time window | GKE API: 3,000 calls per 100 seconds per project |
| **Allocation Quota** | Governs total resource count        | Default: max 15 VPC networks per project         |

> **Exam Tip:** Sustained use discounts are **automatic** — no contract needed. Committed use discounts require an upfront commitment but offer deeper savings.

---

### 7. Open Source & Avoiding Vendor Lock-in

**Google's Philosophy:** Build on and contribute to open standards; customers should be able to move workloads anywhere.

| Open Source Technology | Purpose                                    | Google's Role                                    |
|------------------------|--------------------------------------------|--------------------------------------------------|
| **Kubernetes**         | Container orchestration                    | Created internally; open-sourced to CNCF in 2014 |
| **TensorFlow**         | Machine learning / deep learning framework | Developed by Google Brain; open-sourced 2015     |
| **Apache Beam**        | Unified stream + batch data processing     | Donated by Google; powers Cloud Dataflow         |
| **Istio**              | Service mesh for microservices             | Co-created by Google                             |
| **gRPC**               | High-performance RPC framework             | Open-sourced by Google                           |

**Portability Benefits:**
- Applications built on open standards run on **any cloud or on-premises**
- GCP exposes **REST and gRPC APIs** for all services (not proprietary protocols)
- Client libraries available in Python, Java, Go, Node.js, Ruby, .NET, PHP
- Kubernetes workloads are portable — same manifests work on GKE, AKS, EKS, or on-prem

---

## 🔗 References & Links

| **Resource**                                                                                     | **Description**                                                   |
|--------------------------------------------------------------------------------------------------|-------------------------------------------------------------------|
| [Google Cloud Overview](https://cloud.google.com/docs/overview)                                  | Official introduction to Google Cloud services and concepts       |
| [Google Cloud Regions & Zones](https://cloud.google.com/compute/docs/regions-zones)              | Complete list of GCP regions, zones, and availability             |
| [Google Cloud Pricing](https://cloud.google.com/pricing)                                         | Overview of GCP pricing model and cost management tools           |
| [GCP Pricing Calculator](https://cloud.google.com/products/calculator)                           | Estimate monthly costs before deploying resources                 |
| [Google Sustainability](https://sustainability.google/reports/google-2024-environmental-report/) | Google's environmental commitments and data center efficiency     |
| [Open Source at Google](https://opensource.google/)                                              | Google's open source projects including Kubernetes and TensorFlow |

---

## ❓ Key Questions to Review

- What are the five NIST characteristics of cloud computing? Can you name them without looking?
- What is the difference between IaaS, PaaS, and SaaS billing models?
- How many regions and zones does GCP have (as of 2026 training)?
- What is the difference between a region and a zone?
- What is the purpose of Edge Points of Presence (PoPs)?
- What are the four region selection criteria?
- What does Google manage vs. what does the customer manage in the shared responsibility model?
- What is the Titan chip and what does it protect against?
- What is the Google Front End (GFE) and what does it do?
- What is per-second billing and what is the minimum charge?
- What triggers a sustained use discount and what is the maximum saving?
- What is the difference between sustained use discounts and committed use discounts?
- What are the two types of quotas in GCP? How do they differ?
- Name three open source technologies Google created or contributed to.
- Why does Google's open source contribution help avoid vendor lock-in?

---

## 📌 Summary

| Concept                  | Key Point                                                                                             |
|--------------------------|-------------------------------------------------------------------------------------------------------|
| NIST Cloud Definition    | 5 characteristics: On-demand · Broad network · Resource pooling · Rapid elasticity · Measured service |
| Infrastructure Evolution | On-premises → Colocation → Virtualization → Containerization                                          |
| Service Models           | IaaS (manage VMs) · PaaS (manage code) · SaaS (just use it)                                           |
| GCP Scale                | 43 regions · 130+ zones · zones = independent data centers                                            |
| HA Design                | Multiple zones = fault tolerance; multiple regions = disaster recovery                                |
| Security                 | Google secures hardware/infra; customer secures IAM, app config, firewall rules                       |
| Pricing                  | Per-second billing · sustained use discounts (automatic, up to 30%) · custom machine types            |
| Open Source              | Kubernetes · TensorFlow · Apache Beam → avoids vendor lock-in                                         |
