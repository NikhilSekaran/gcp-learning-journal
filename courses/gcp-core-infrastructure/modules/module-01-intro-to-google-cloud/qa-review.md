# Module 1: Introduction to Google Cloud - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module 1 — Introduction to Google Cloud. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---

## Topic 1: NIST Cloud Computing Definition

### Q1: The Five Characteristics

**Question:**
Which of the following is NOT one of the NIST five essential characteristics of cloud computing?

**Options:**
- A) On-demand self-service
- B) Dedicated hardware per customer
- C) Rapid elasticity
- D) Measured service

**Answer:** **B) Dedicated hardware per customer**

**Explanation:**
The NIST definition describes cloud computing as using **pooled resources** shared across multiple customers (multi-tenancy), not dedicated hardware. The five characteristics are: on-demand self-service, broad network access, **resource pooling**, rapid elasticity, and measured service.

**Why not the others:**
- A) On-demand self-service ✓ — NIST characteristic #1
- C) Rapid elasticity ✓ — NIST characteristic #4
- D) Measured service ✓ — NIST characteristic #5

---

### Q2: Rapid Elasticity in Practice

**Question:**
A video streaming platform experiences 10× the normal traffic during a major live event. The platform is hosted on Google Cloud and its infrastructure scales automatically within minutes to handle the load, then scales back down after the event ends. Which NIST cloud characteristic does this best demonstrate?

**Options:**
- A) Broad network access
- B) Resource pooling
- C) Rapid elasticity
- D) On-demand self-service

**Answer:** **C) Rapid elasticity**

**Explanation:**
Rapid elasticity is the characteristic that allows resources to scale up or down quickly and automatically to match demand. The ability to expand capacity 10× during a live event and then release those resources afterward is a textbook example of rapid elasticity.

**Why not the others:**
- A) Broad network access — refers to resources being accessible from any device/location over the network; it describes *how* resources are reached, not the ability to scale automatically
- B) Resource pooling — describes multiple customers sharing physical resources (multi-tenancy); it's about shared infrastructure, not dynamic scaling in response to load
- D) On-demand self-service — means users can provision resources without human help from the provider; it's about *self-provisioning*, not automatic scaling in response to traffic

---

## Topic 2: Service Models (IaaS / PaaS / SaaS)

### Q3: Choosing the Right Service Model

**Question:**
A startup wants to deploy a web application quickly. They want to focus entirely on writing application code and not worry about patching operating systems, managing servers, or configuring load balancers. Which GCP service model best fits their needs?

**Options:**
- A) Infrastructure as a Service (IaaS)
- B) Platform as a Service (PaaS)
- C) Software as a Service (SaaS)
- D) On-premises colocation

**Answer:** **B) Platform as a Service (PaaS)**

**Explanation:**
PaaS abstracts all infrastructure management (servers, OS, middleware, load balancers) so developers focus only on writing application code. The provider manages everything below the application layer.

**Why not the others:**
- A) IaaS — the team would still manage VMs, OS patches, and application runtime
- C) SaaS — SaaS provides finished applications (like Gmail); you cannot deploy your own code
- D) On-premises — maximum control but maximum management burden

---

### Q4: IaaS Billing Concern

**Question:**
A company migrated its batch processing application to Compute Engine VMs. The application runs processing jobs for only 2 hours a day. An engineer notices the bill seems higher than expected. What is the most likely cause?

**Options:**
- A) Compute Engine charges extra for batch workloads
- B) The company is paying for VM uptime even when the application is idle
- C) Sustained use discounts do not apply to batch jobs
- D) The company is being double-charged for network egress

**Answer:** **B) The company is paying for VM uptime even when the application is idle**

**Explanation:**
IaaS model (Compute Engine) charges for VM uptime regardless of whether the application is processing work. If the VM runs 24/7 but the app only works 2 hours/day, the company still pays for all 24 hours. The fix: stop VMs when not needed, or use a PaaS/serverless alternative.

**Why not the others:**
- A) Compute Engine does not charge extra for batch workloads — all VMs are billed the same way regardless of workload type
- C) Sustained use discounts apply to all eligible VMs including batch workloads — they kick in automatically based on runtime, not workload type
- D) Network egress is charged per-GB transferred, not as a fixed "double charge" — and wouldn't produce the pattern of a higher-than-expected bill for an idle VM

---

## Topic 3: Google Cloud Global Infrastructure

### Q5: Region vs. Zone

**Question:**
A compliance requirement states that all customer data must remain within Germany. You also need to ensure that your service remains available even if one data center fails. What is the correct deployment approach?

**Options:**
- A) Deploy in a single zone in `europe-west3` (Frankfurt)
- B) Deploy across multiple zones in `europe-west3` (Frankfurt)
- C) Deploy across the `europe-west3` and `us-central1` regions
- D) Deploy in a multi-regional Cloud Storage bucket

**Answer:** **B) Deploy across multiple zones in `europe-west3` (Frankfurt)**

**Explanation:**
`europe-west3` is Frankfurt, Germany. Deploying across multiple zones within this region keeps data in Germany (compliance satisfied) while providing fault tolerance against a single data center failure. Deploying to `us-central1` would violate the German data residency requirement.

**Why not the others:**
- A) Single zone in `europe-west3` — data stays in Germany (compliant), but provides no fault tolerance against a data center failure; one zone outage takes down the service
- C) `europe-west3` and `us-central1` — adding `us-central1` violates the German data residency requirement; customer data would leave Germany
- D) Multi-regional Cloud Storage bucket — Cloud Storage stores objects, not compute workloads; a bucket doesn't serve an application or provide HA for application logic

---

### Q6: Edge PoPs and CDN

**Question:**
A user in Tokyo requests a static image from a web app hosted on a Cloud Run service in `us-central1`. Cloud CDN is enabled and the image is cached. What happens?

**Options:**
- A) The request travels to the `us-central1` origin for every request regardless of CDN
- B) Google's network routes the request to the nearest edge Point of Presence (PoP), where the cached image is served without reaching the origin region
- C) Cloud CDN only caches content for users in North America and Europe
- D) The request is served from a Compute Engine VM in the nearest GCP region

**Answer:** **B) Google's network routes the request to the nearest edge Point of Presence (PoP), where the cached image is served without reaching the origin region**

**Explanation:**
Edge Points of Presence (PoPs) are locations at the outer edge of Google's global network where it interconnects with the public internet. They are distinct from regions and zones — they are networking entry/exit points, not compute data centers.

Cloud CDN uses edge PoPs to cache static content (images, CSS, JS, video) close to end users. When a user requests cached content, Google serves it from the nearest PoP rather than routing the request all the way back to the origin server in a GCP region. This reduces latency significantly and offloads traffic from origin servers.

**Why not the others:**
- A) Without CDN, every request would travel to the origin — but CDN caching at the PoP intercepts the request before it reaches `us-central1`, which is the entire point of CDN
- C) Google's CDN operates from PoPs worldwide including Asia-Pacific; Tokyo would be served from a nearby Japanese or Asian PoP
- D) Cloud CDN serves from PoP cache nodes at Google's network edge, not from Compute Engine VMs; PoPs are separate from the regional compute infrastructure

---

## Topic 4: Security & Shared Responsibility

### Q7: Titan Chip Purpose

**Question:**
What is the purpose of Google's Titan security chip in its data center servers?

**Options:**
- A) Encrypts all customer data at rest inside Cloud Storage
- B) Verifies the integrity of firmware and bootloader at system startup
- C) Manages multi-factor authentication for Cloud Console users
- D) Provides hardware-accelerated TLS termination for HTTPS traffic

**Answer:** **B) Verifies the integrity of firmware and bootloader at system startup**

**Explanation:**
The Titan chip is a custom-designed security microcontroller that verifies firmware integrity at boot. It ensures that only authorized, trusted firmware runs on Google's servers — protecting against attacks that attempt to compromise the hardware layer below the OS.

**Why not the others:**
- A) Cloud Storage encryption is handled by Google's key management system at the storage service layer — the Titan chip operates at the hardware firmware level, not the storage service layer
- C) Cloud Console MFA uses Google's Identity infrastructure (login challenges, U2F security keys) — not the Titan chip, which is a server hardware boot integrity mechanism
- D) TLS termination is handled by the Google Front End (GFE) proxy — the Titan chip is a server-level security chip, not a network-level TLS accelerator

---

### Q8: Shared Responsibility Model

**Question:**
Under GCP's shared responsibility model, which of the following is the customer's responsibility?

**Options:**
- A) Physical security of Google's data center buildings and hardware
- B) Patching hypervisor vulnerabilities on Google's servers
- C) Configuring IAM roles, firewall rules, and securing application code deployed on VMs
- D) DDoS mitigation at the network layer

**Answer:** **C) Configuring IAM roles, firewall rules, and securing application code deployed on VMs**

**Explanation:**
Google is responsible for the security **of** the cloud — physical infrastructure, hardware, the hypervisor, encryption in transit, and DDoS mitigation.

The customer is responsible for security **in** the cloud:
- **IAM configuration:** Who has access to what resources
- **Firewall rules:** Controlling which network traffic reaches VMs
- **Application security:** Securing the code they deploy
- **Operating system patches:** On IaaS (Compute Engine) VMs
- **Data classification:** Deciding what data goes to the cloud
- **Encryption key management:** Using customer-managed keys (CMEK) in Cloud KMS if needed

**Why not the others:**
- A) Physical data center security is entirely Google's responsibility — customers never have physical access to Google's infrastructure
- B) Hypervisor patching is Google's responsibility as part of securing the underlying infrastructure layer
- D) Network-level DDoS mitigation is built into Google's infrastructure and is Google's responsibility; customers benefit from it automatically

---

## Topic 5: Pricing

### Q9: Sustained Use Discounts

**Question:**
A company runs Compute Engine VMs 24/7 throughout the month. Which statement about pricing is correct?

**Options:**
- A) The company must sign a 1-year contract to receive a discount
- B) The company automatically receives a sustained use discount of up to 30%
- C) The company receives a discount only if they use Preemptible VMs
- D) No discounts apply to 24/7 workloads; only burst workloads are discounted

**Answer:** **B) The company automatically receives a sustained use discount of up to 30%**

**Explanation:**
Sustained use discounts start automatically when a VM runs for more than 25% of the billing month, and increase as the VM runs more. Running 100% of the month earns the maximum discount of up to 30% — no contract or action required.

**Why not the others:**
- A) Signing a 1-year contract describes **Committed Use Discounts** — a separate program with deeper savings (up to 57%) but requiring an upfront commitment; sustained use discounts require no contract
- C) Preemptible/Spot VMs have their own pricing model (up to 91% off) — they are not a prerequisite for sustained use discounts, which apply to regular on-demand VMs
- D) Sustained use discounts are specifically designed for VMs running continuously — a 24/7 workload is the ideal candidate, not an exception

---

### Q10: Cost Optimization — Custom Machine Types

**Question:**
Your application requires 6 vCPUs and 20 GB of memory. The closest predefined machine type offers 8 vCPUs and 32 GB. What GCP feature lets you avoid paying for the extra resources?

**Options:**
- A) Sustained use discounts
- B) Committed use discounts
- C) Custom machine types
- D) Spot VMs

**Answer:** **C) Custom machine types**

**Explanation:**
Custom machine types let you specify the exact number of vCPUs and amount of memory your workload needs. Instead of paying for an 8 vCPU / 32 GB predefined VM when you only need 6 vCPU / 20 GB, you create a custom machine with exactly those specs and pay only for what you configure.

**Why not the others:**
- A) Sustained use discounts reduce the per-hour rate based on runtime — they don't change the fact that you're paying for unused vCPUs and RAM
- B) Committed use discounts offer savings in exchange for a 1 or 3-year commitment — you'd still pay for the overprovisioned 8 vCPU / 32 GB machine, just at a discounted rate
- D) Spot VMs offer deep discounts but can be preempted (reclaimed) by Google at any time — not suitable if the service requires continuous availability

---

## Topic 6: Open Source & Vendor Lock-in

### Q11: Kubernetes and Portability

**Question:**
A company deploys its microservices on GKE using standard Kubernetes YAML manifests. Later, they want to move some workloads to Amazon EKS. What is true about this migration?

**Options:**
- A) The workloads must be rewritten because GKE uses a proprietary Kubernetes fork incompatible with EKS
- B) The Kubernetes YAML manifests are portable and will work on EKS with minimal or no changes, because GKE uses upstream open-source Kubernetes
- C) GKE will automatically replicate the workloads to AWS EKS without any migration effort
- D) Portability is only possible if the company purchased the GKE Enterprise tier

**Answer:** **B) The Kubernetes YAML manifests are portable and will work on EKS with minimal or no changes, because GKE uses upstream open-source Kubernetes**

**Explanation:**
Kubernetes is an open-source container orchestration platform that runs on any cloud or on-premises environment. Kubernetes workloads are defined using standard YAML manifests that work across GKE, Amazon EKS, Azure AKS, and even on-premises clusters.

Because GKE is based on upstream Kubernetes (not a proprietary fork), the same application manifests, Helm charts, and kubectl commands that work on GKE also work on other Kubernetes distributions. If you need to move workloads away from GCP, you can lift-and-shift Kubernetes applications without rewriting code.

Google's philosophy is to use and contribute to open standards, so customers are never forced to stay on GCP due to proprietary API dependencies.

**Why not the others:**
- A) GKE runs standard upstream Kubernetes — not a proprietary fork; it passes the CNCF Kubernetes conformance tests, which guarantees API compatibility with other certified distributions
- C) There is no automatic cross-cloud workload replication; migration still requires deploying to the target cluster, but the manifests themselves are reusable
- D) Kubernetes portability is a feature of the open standard itself and applies to all GKE tiers; it doesn't require Enterprise licensing

---

### Q12: Identifying the Service Model

**Question:**
Which of the following correctly identifies the service model of the GCP service listed?

**Options:**
- A) Compute Engine — PaaS; Google manages the OS and runtime
- B) Google Workspace (Gmail) — SaaS; a fully managed application users interact with directly
- C) Cloud SQL — IaaS; you must install and patch the database engine yourself
- D) Cloud Run — IaaS; you provide the server configuration and manage the runtime

**Answer:** **B) Google Workspace (Gmail) — SaaS; a fully managed application users interact with directly**

**Explanation:**
The full mapping across common GCP services:

| Offering                 | Service Model | Reason                                                                            |
|--------------------------|---------------|-----------------------------------------------------------------------------------|
| Compute Engine           | **IaaS**      | You manage the OS, patches, and runtime on a raw VM                               |
| Cloud Run                | **PaaS**      | You provide a container; Google manages infrastructure, scaling, runtime          |
| Google Workspace (Gmail) | **SaaS**      | Fully managed application; you just use it                                        |
| Cloud SQL                | **PaaS**      | Google manages the database engine, patching, backups; you manage schema and data |

**Why not the others:**
- A) Compute Engine is **IaaS** — you receive a raw VM and must manage the OS, patches, runtimes, and everything above the hypervisor
- C) Cloud SQL is **PaaS** — Google manages the database engine (MySQL/PostgreSQL), backups, and patching; you only manage schema, users, and data
- D) Cloud Run is **PaaS** (some classify it as serverless/CaaS) — you provide a container image and Google handles provisioning, scaling, load balancing, and runtime

---

### Q13: Google's Sustainability Commitments

**Question:**
Which of the following statements about Google Cloud's sustainability commitments is INCORRECT?

**Options:**
- A) Google has been carbon neutral since 2007
- B) Google matches 100% of its global electricity consumption with renewable energy purchases
- C) Google aims to operate on 24/7 carbon-free energy by 2030
- D) Google's data centers are ISO 9001 certified for environmental management

**Answer:** **D) Google's data centers are ISO 9001 certified for environmental management**

**Explanation:**
ISO 9001 is a **quality management** standard — not environmental. The correct certification is **ISO 14001**, which is the international standard for environmental management systems. Google's data centers hold ISO 14001 certification.

Google Cloud's actual sustainability commitments:
- **ISO 14001 certified** — internationally recognized environmental management framework
- **Carbon neutral since 2007** — Google has offset all carbon emissions since 2007
- **100% renewable energy matched** — Google matches 100% of its global electricity consumption with renewable energy purchases
- **Hamina, Finland data center** — uses recycled seawater from the Gulf of Finland for cooling
- Target: **24/7 carbon-free energy by 2030**

**Why not the others:**
- A) Carbon neutral since 2007 is correct — Google has offset 100% of its operational carbon since 2007
- B) 100% renewable energy matching is correct — Google has matched its consumption with renewable energy purchases globally
- C) The 2030 target for 24/7 carbon-free energy is correct — this goes beyond matching to actually running on clean energy at all times

---

### Q14: Rate Quota vs. Allocation Quota

**Question:**
A CI/CD pipeline fails with a quota error after making hundreds of API calls per minute. The error resolves automatically after a few minutes without any changes. Which quota type was hit, and what is the correct response?

**Options:**
- A) An Allocation quota — delete unused resources or request a quota increase
- B) A Rate quota — wait for the time window to reset, then retry; optionally add exponential backoff
- C) An Allocation quota — wait for the time window to reset, then retry
- D) A Rate quota — delete unused API keys to free up quota

**Answer:** **B) A Rate quota — wait for the time window to reset, then retry; optionally add exponential backoff**

**Explanation:**

| Quota Type           | Definition                                                                   | Resets?                                  | Example                                                |
|----------------------|------------------------------------------------------------------------------|------------------------------------------|--------------------------------------------------------|
| **Rate quota**       | Limits how many API requests or operations you can make **per unit of time** | Yes — resets after the time window       | 1,000 API requests per 100 seconds per project         |
| **Allocation quota** | Limits the **total number of resources** you can hold at any point in time   | No — must be deleted/released to free up | Max 15 VPC networks per project; max 8 CPUs per region |

- Rate quotas protect against **API abuse** and **runaway automation** — they reset automatically after the time window
- Allocation quotas protect against **resource sprawl** — they cap how many resources can exist simultaneously; they don't reset automatically

The key tell in the scenario: **resolves automatically after a few minutes** = Rate quota (time-window based).

**Why not the others:**
- A) Allocation quotas don't resolve automatically — they require deleting resources or requesting an increase; the self-resolving behavior rules out Allocation
- C) Allocation quotas don't reset on a timer — they are a hard cap on concurrent resources, not on request frequency
- D) Deleting API keys does not free Rate quota — Rate quota is tracked per project/user against API calls per time window, not against the number of keys

---

### Q15: Fault Tolerance with Zones

**Question:**
What is the primary benefit to a Google Cloud customer of using resources in several zones within a region?

**Options:**
- A) For improved fault tolerance
- B) For better performance
- C) For expanding services to customers in new areas
- D) For getting discounts on other zones

**Answer:** **A) For improved fault tolerance**

**Official Explanation:**
Correct. As part of building a fault-tolerant application, you can spread your resources across multiple zones in a region. If one zone fails (data center outage), your other zones continue serving traffic.

**Why not the others:**
- B) Better performance — using multiple zones doesn't inherently improve performance; that's achieved through CDN, load balancers, and choosing regions close to users
- C) New areas — expanding to new geographic areas requires additional **regions**, not zones
- D) Discounts on zones — zone usage doesn't affect pricing for other zones
