# Module 5: Containers and Kubernetes - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module 5 — Containers & Kubernetes. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---

## Topic 1: Containers vs. VMs

### Q1: Container Benefits

**Question:**
A team is deploying a microservices-based application. A developer says "it works on my laptop but breaks in staging." What container benefit most directly solves this problem?

**Options:**
- A) Containers share the host OS kernel, reducing memory usage
- B) Containers package the application together with its dependencies, ensuring consistent runtime environments
- C) Containers start faster than VMs
- D) Containers provide better network isolation than VMs

**Answer:** **B) Containers package the application together with its dependencies, ensuring consistent runtime environments**

**Explanation:**
The "works on my machine" problem arises because the laptop has different library versions, runtime versions, or OS packages than staging. A container image bundles the application code **together with all its dependencies** (exact library versions, runtime, config) into a single portable artifact. Running the same container image on the laptop and in staging produces identical behavior — the environment is consistent everywhere the image runs.

**Why not the others:**
- A) Sharing the host OS kernel reduces memory usage — true, but that's a resource efficiency benefit; it doesn't address inconsistent dependency versions between environments
- C) Faster startup than VMs — true, but irrelevant to environment consistency; a container that starts quickly but has wrong dependencies still fails in staging
- D) Better network isolation than VMs — containers actually provide weaker security isolation than VMs (shared kernel); and network isolation doesn't address the dependency consistency problem

---

### Q2: Containers vs. VMs — Resource Overhead

**Question:**
What is the fundamental architectural difference that makes containers more lightweight than virtual machines?

**Options:**
- A) Containers run on dedicated physical hardware; VMs share hardware among multiple tenants
- B) Containers share the host OS kernel using Linux namespaces and cgroups, eliminating the separate OS per instance that VMs require
- C) VMs use interpreted runtimes (Python, Node.js) which consume more memory; containers use only compiled binaries
- D) Containers use Type 1 hypervisors that operate closer to hardware; VMs use slower Type 2 hypervisors

**Answer:** **B) Containers share the host OS kernel using Linux namespaces and cgroups, eliminating the separate OS per instance that VMs require**

**Explanation:**
The key difference is at the **virtualization layer**:

**Virtual Machine:**
- Uses a **hypervisor** to virtualize hardware
- Each VM runs a **complete, independent OS** — its own kernel, system libraries, and OS processes
- VM image: typically several GBs; Boot time: 1–5 minutes

**Container:**
- Uses **OS-level virtualization** — Linux namespaces isolate processes; cgroups limit resource usage
- Containers share the **host OS kernel** — no separate kernel per container
- Container image: typically tens to hundreds of MBs; Startup: seconds or milliseconds

**Implication:** On the same hardware, you can run hundreds of containers vs. tens of VMs.

**Why not the others:**
- A) Both VMs and containers run on shared physical hardware in cloud environments — neither gets dedicated physical hardware
- C) VMs fully support all language runtimes including Python and Node.js; memory overhead comes from the OS copy, not the runtime
- D) Containers don't use hypervisors at all — they use OS-level kernel features (namespaces/cgroups); Type 1 vs. Type 2 is a VM hypervisor distinction, not relevant to containers

---

## Topic 2: Kubernetes Architecture

### Q3: Control Plane Role

**Question:**
What is the role of the Kubernetes control plane?

**Options:**
- A) It runs the application containers and handles incoming user requests
- B) It maintains the desired cluster state, schedules pods to nodes, and reconciles actual state with declared state
- C) It provides the physical network infrastructure connecting worker nodes
- D) It stores container images for deployment to worker nodes

**Answer:** **B) It maintains the desired cluster state, schedules pods to nodes, and reconciles actual state with declared state**

**Explanation:**
The control plane is the **brain** of the Kubernetes cluster. Its components:
- **API Server:** Receives all management commands (from kubectl, CI/CD, Autopilot)
- **Scheduler:** Decides which node a new Pod should run on
- **Controller Manager:** Continuously reconciles actual state → desired state (e.g., if a Pod crashes, starts a replacement)
- **etcd:** Stores all cluster state (desired + actual)

The control plane does NOT run application containers — that's the job of the worker nodes.

**Why not the others:**
- A) Running application containers is the job of **worker nodes**, not the control plane; the control plane only manages cluster state and scheduling decisions
- C) Physical networking between nodes is handled by the underlying infrastructure (Compute Engine VMs and VPC networking); the control plane is a software orchestration layer
- D) Container images are stored in a registry (Artifact Registry, Docker Hub); the control plane references images but doesn't store them; etcd stores cluster state, not images

---

### Q4: etcd Importance

**Question:**
A GKE cluster's etcd becomes permanently unavailable and cannot be restored from backup. What is the impact on the cluster?

**Options:**
- A) All running Pods are immediately terminated because etcd stores the container runtime
- B) Running Pods continue serving traffic but the cluster cannot be managed; new deployments, scaling, and configuration changes all fail
- C) GKE automatically creates a new etcd cluster and restores state from the node kubelet caches
- D) Only the scheduler is affected; existing deployments continue reconciling normally

**Answer:** **B) Running Pods continue serving traffic but the cluster cannot be managed; new deployments, scaling, and configuration changes all fail**

**Explanation:**
**etcd** is the distributed key-value store used by Kubernetes as its **source of truth** for all cluster state:
- All declared desired state (Deployments, Services, ConfigMaps)
- All actual observed state (which nodes are ready, which Pods are running)
- All configuration and secrets

**Why it's critical:**
- Without etcd, the control plane cannot function — the API server has no state to read or write
- Existing Pods running on worker nodes continue to run (kubelet keeps them alive)
- But: no new pods scheduled, no changes accepted, cluster is effectively read-only at the runtime level

In GKE, Google manages etcd replication and backups automatically.

**Why not the others:**
- A) Running Pods are NOT immediately terminated — they continue running because the kubelet on each node manages them independently of the control plane
- C) GKE cannot restore etcd from node kubelet caches — node kubelets don't hold a full copy of cluster state; etcd is the only authoritative store
- D) All control plane components depend on etcd — the scheduler, controller manager, and API server all lose function; reconciliation requires reading/writing etcd

---

## Topic 3: Core Kubernetes Objects

### Q5: Pod vs. Deployment

**Question:**
Why should you almost never create a standalone Pod directly for a production workload? What should you use instead?

**Options:**
- A) Pods are deprecated; use Services instead
- B) Pods are ephemeral and not self-healing; use a Deployment to manage desired replica count and enable restarts
- C) Pods cannot be exposed to network traffic without a Deployment
- D) Pods do not support environment variables or volume mounts

**Answer:** **B) Pods are ephemeral and not self-healing; use a Deployment to manage desired replica count and enable restarts**

**Explanation:**
A standalone Pod is **not self-healing**:
- If it crashes → it stays dead (nothing restarts it)
- If the node it runs on fails → the Pod is gone
- You cannot easily scale it

A **Deployment** wraps a Pod template with a desired replica count. The Deployment Controller continuously reconciles:
- If a Pod crashes → a new Pod is created automatically
- If a node fails → the Pod is rescheduled on a healthy node
- You can scale by changing the replica count
- Rolling updates and rollbacks are built-in

> **Rule:** In production, always use a Deployment (or StatefulSet for stateful workloads) rather than creating raw Pods.

**Why not the others:**
- A) Pods are not deprecated — they are the fundamental building block of Kubernetes; Deployments manage Pods, not replace them; Services expose network endpoints to Pods, not the other way around
- C) Pods can be exposed to network traffic directly — a Service selects Pods by label and doesn't require a Deployment; the issue is self-healing, not network exposure
- D) Standalone Pods fully support environment variables, ConfigMaps, Secrets, and volume mounts — this is unrelated to why you shouldn't use them for production workloads

---

### Q6: Kubernetes Service Types

**Question:**
You have a Kubernetes Deployment running a web application on GKE. You want the application to be accessible from the internet via a public IP address with automatic load balancing across all pod replicas. Which Service type should you create?

**Options:**
- A) `ClusterIP` — internal service visible only within the cluster
- B) `NodePort` — exposes on a static port on each node's IP
- C) `LoadBalancer` — creates a GCP External Load Balancer with a public IP
- D) `ExternalName` — routes to an external DNS name

**Answer:** **C) `LoadBalancer` — creates a GCP External Load Balancer with a public IP**

**Explanation:**
On GKE, a Service of type `LoadBalancer` automatically provisions:
1. A GCP External (regional) Load Balancer
2. A dedicated public IP address
3. Load balancing across all healthy Pods matching the selector

Traffic flow: `Internet → External LB (public IP) → NodePort → Pods`

`ClusterIP` is internal-only. `NodePort` exposes on each node's IP (requires knowing node IPs; no managed LB). `LoadBalancer` is the production choice for internet-facing apps on GKE.

**Why not the others:**
- A) `ClusterIP` creates a virtual IP accessible only from within the cluster — no external traffic can reach it; used for internal service-to-service communication
- B) `NodePort` exposes on a high port (30000–32767) on each node's external IP — requires knowing individual node IPs, no managed load balancer, non-standard ports; not production-ready for internet-facing apps
- D) `ExternalName` maps the Service to an external DNS name (CNAME) — used to proxy to external services outside the cluster; it doesn't expose internal Pods to external traffic

---

### Q7: Declarative vs. Imperative

**Question:**
A DevOps engineer runs `kubectl run web --image=nginx:1.25`. What is the primary risk of this approach compared to `kubectl apply -f web-deployment.yaml`?

**Options:**
- A) `kubectl run` creates a standalone Pod without self-healing; it also cannot be version-controlled or reproduced consistently
- B) `kubectl run` requires cluster-admin permissions; `kubectl apply` works with lower RBAC permissions
- C) Imperative commands execute slower than declarative YAML because they require more API round trips
- D) `kubectl run` is deprecated and will be removed in a future Kubernetes version

**Answer:** **A) `kubectl run` creates a standalone Pod without self-healing; it also cannot be version-controlled or reproduced consistently**

**Explanation:**
`kubectl run` is **imperative** — it directly commands Kubernetes to create a resource.

**Risks in production:**
1. **No version control:** The command is not stored anywhere; changes can't be reviewed or audited
2. **Not reproducible:** Recreating after a disaster requires remembering exact flags and options
3. **No GitOps:** Cannot integrate with PR-based workflows
4. **Drift:** Cluster state diverges from what's documented

**Correct approach:**
```yaml
# web-deployment.yaml stored in Git
apiVersion: apps/v1
kind: Deployment
...
```
```bash
kubectl apply -f web-deployment.yaml
git commit -m "Deploy web app with nginx 1.25, 3 replicas"
```

**Why not the others:**
- B) `kubectl run` uses the same RBAC permissions as `kubectl apply` for creating workloads — the issue is not permissions but reproducibility and auditability
- C) The number of API calls is similar — the problem with imperative commands is operational, not performance
- D) `kubectl run` is not deprecated — it's still supported but discouraged in production in favor of declarative manifests

---

## Topic 4: GKE — Standard vs. Autopilot

### Q8: GKE Mode Selection

**Question:**
A startup team of 3 engineers is building a new product on GKE. They have limited Kubernetes expertise and want to focus on deploying their application without managing node pools, node sizes, or OS patching. Which GKE mode should they choose?

**Options:**
- A) GKE Standard — gives them the most control over their cluster
- B) GKE Autopilot — Google manages nodes; team focuses on workloads
- C) A self-managed Kubernetes cluster on Compute Engine VMs
- D) GKE Standard with node auto-provisioning enabled

**Answer:** **B) GKE Autopilot — Google manages nodes; team focuses on workloads**

**Explanation:**
GKE Autopilot removes all node management responsibilities:
- No need to choose node machine types or sizes
- No node OS patching or upgrades to manage
- Automatic node provisioning based on Pod resource requests
- Google applies security best practices to nodes by default
- Team writes Deployments and Services; Autopilot handles the rest

This is ideal for small teams and teams new to Kubernetes. The tradeoff is less control over node configuration.

**Why not the others:**
- A) GKE Standard gives more control but requires the team to manage node pools, choose machine types, patch node OSes, and handle capacity planning — too much overhead for a 3-person team wanting to focus on their application
- C) Self-managed Kubernetes on Compute Engine VMs means the team must also manage the control plane (etcd, API server, scheduler) in addition to nodes — the most complex option; not recommended for teams without deep Kubernetes expertise
- D) GKE Standard with node auto-provisioning (NAP) helps with node sizing but the team still manages node pools and underlying node infrastructure — Autopilot removes this entirely

---

### Q9: GKE Standard vs. Autopilot Billing

**Question:**
A company runs a GKE Standard cluster with 10 nodes. On weekends only 5% of pod capacity is used, but all 10 nodes keep running. How would GKE Autopilot change the weekend billing?

**Options:**
- A) No difference — both GKE Standard and Autopilot bill per node VM regardless of pod utilization
- B) Autopilot would bill for cluster management fees only; nodes are free in Autopilot
- C) Autopilot would scale the cluster down to zero and charge nothing on weekends with no pods scheduled
- D) Autopilot bills per Pod resource request; with only 5% of workloads active, the weekend bill drops by approximately 95%

**Answer:** **D) Autopilot bills per Pod resource request; with only 5% of workloads active, the weekend bill drops by approximately 95%**

**Explanation:**
**GKE Standard billing:**
- All 10 nodes run 24/7 regardless of pod utilization
- You pay for **all 10 VMs** even on weekends with 5% utilization — 95% wasted cost

**GKE Autopilot billing:**
- Bills per **Pod resource request** (CPU + memory declared in the Pod spec)
- If weekend workloads only use 5% of weekday resources, the bill drops by ~95% automatically
- Nodes scale to zero when no Pods need scheduling — zero compute cost during idle periods

**Why not the others:**
- A) GKE Standard does bill per node VM regardless of utilization — but Autopilot is fundamentally different; it provisions and bills only for what Pods actually request
- B) Autopilot does not have a separate "management fee" structure that makes nodes free — the billing model is per Pod resource request, not per node VM
- C) If any Pods are scheduled on weekends (5% utilization), some nodes exist and are billed; only a zero-pod state would be completely free — but the key point is proportional billing, not zero billing

---

### Q10: GKE Managed Control Plane

**Question:**
A team is choosing between self-managed Kubernetes on Compute Engine and GKE. Which statement about GKE's managed control plane is correct?

**Options:**
- A) With GKE, you must still provision control plane VMs but Google assists with patching
- B) GKE control plane VMs appear in your project's Compute Engine console and are billed like regular VMs
- C) GKE fully manages the control plane — provisioning, HA, upgrades, and backups are Google's responsibility at no charge to the customer
- D) Only GKE Autopilot has a managed control plane; GKE Standard requires you to manage the control plane yourself

**Answer:** **C) GKE fully manages the control plane — provisioning, HA, upgrades, and backups are Google's responsibility at no charge to the customer**

**Explanation:**
In self-managed Kubernetes, the control plane (API server, etcd, scheduler, controller manager) runs on machines that you must: provision, patch, upgrade, back up, monitor, and scale.

In **GKE**, Google manages all of this:
- Control plane VMs are invisible to the customer — they don't appear in your Compute Engine console
- Google handles HA (multi-zone control plane), version upgrades, and etcd backups
- **You are not billed for control plane VMs** — they are included in the GKE service at no charge
- GKE automatically upgrades the control plane during maintenance windows

**Why not the others:**
- A) You have no access to or responsibility for control plane VMs in GKE — Google handles everything; you don't need to provision or patch them
- B) Control plane VMs do NOT appear in your Compute Engine console — they run in Google's managed project, completely transparent to the customer
- D) Both GKE Standard and GKE Autopilot have a managed control plane — neither requires the customer to manage control plane infrastructure

---

### Q11: Rolling Update Strategy

**Question:**
You have a Deployment with 5 Pod replicas. You update the container image. By default, how does a Kubernetes rolling update proceed?

**Options:**
- A) All 5 Pods are deleted simultaneously, then 5 new Pods are started
- B) New Pods are started one at a time; old Pods are removed only as new ones become ready
- C) All 5 Pods continue running; new Pods are added until 10 exist, then old ones are deleted
- D) The rollout pauses and awaits manual approval before updating each Pod

**Answer:** **B) New Pods are started one at a time; old Pods are removed only as new ones become ready**

**Explanation:**
Kubernetes rolling updates use two parameters:
- `maxSurge` (default: 25%) — max extra Pods beyond desired count during update
- `maxUnavailable` (default: 25%) — max Pods that can be unavailable during update

With 5 replicas (defaults rounded): 
1. Start 1–2 new Pods with the new image
2. Wait for them to pass readiness checks
3. Terminate 1–2 old Pods
4. Repeat until all Pods are updated

At no point does the service go down — there are always healthy Pods serving traffic. If the new Pods fail readiness checks, the rollout stops automatically, protecting the service.

**Why not the others:**
- A) Deleting all 5 Pods simultaneously then starting new ones describes a **Recreate** strategy — this causes full service downtime and is only used when backward compatibility between versions is impossible
- C) Running both old and new sets simultaneously until all new Pods are ready describes a **Blue/Green** approach — Kubernetes rolling updates don't wait for all new Pods before removing any old ones
- D) Kubernetes rolling updates proceed automatically without requiring manual approval between each Pod — manual pausing requires an explicit `kubectl rollout pause` command

---

### Q12: GKE Node Pools

**Question:**
A GKE cluster runs both GPU-intensive ML training jobs and standard web API services. What GKE feature lets both workload types share the same cluster while running on appropriately sized hardware?

**Options:**
- A) Kubernetes Namespaces — each namespace automatically gets assigned different hardware based on resource requests
- B) Node pools — create separate pools (one with GPU/high-CPU machines for ML, one standard for APIs) and assign pods via node selectors or taints
- C) Pod Disruption Budgets — limit how many pods of each type run simultaneously to fit on available hardware
- D) GKE Autopilot — automatically selects GPU machines when a pod requests GPU resources without any configuration needed

**Answer:** **B) Node pools — create separate pools (one with GPU/high-CPU machines for ML, one standard for APIs) and assign pods via node selectors or taints**

**Explanation:**
A **node pool** is a group of nodes within a GKE cluster that all share the same configuration (machine type, OS image, disk size, labels, taints). You add additional node pools when different workloads have different infrastructure needs:

| Scenario                                      | Solution                                                    |
|-----------------------------------------------|-------------------------------------------------------------|
| ML training needs GPUs, web pods don't        | GPU node pool for ML; standard pool for web                 |
| Cost-sensitive batch workloads                | Spot VM node pool for batch; standard for critical services |
| Compliance requires specific machine families | Separate pools with different machine types                 |
| Windows containers needed                     | Separate Windows OS node pool                               |

**Pod assignment:** Use **node selectors** or **node affinity** to target specific pools; use **taints and tolerations** to repel/allow pods on certain node pools.

**Why not the others:**
- A) Kubernetes Namespaces provide logical isolation for resource management and RBAC — they do not control which physical hardware or machine type pods run on
- C) Pod Disruption Budgets control how many pods can be simultaneously unavailable during voluntary disruptions — unrelated to hardware selection
- D) GKE Autopilot does handle GPU workloads automatically when a pod specifies GPU resource requests, but the question context (Standard cluster with both workload types) implies GKE Standard node pool management

---

### Q13: kubectl Command Selection

**Question:**
A Deployment's pods are all crashing after a bad image update. Which `kubectl` command immediately rolls back to the previous working version?

**Options:**
- A) `kubectl delete deployment web-app && kubectl apply -f web-deployment.yaml`
- B) `kubectl scale deployment web-app --replicas=0`
- C) `kubectl rollout undo deployment/web-app`
- D) `kubectl set image deployment/web-app web=nginx:previous`

**Answer:** **C) `kubectl rollout undo deployment/web-app`**

**Explanation:**
`kubectl rollout undo` reverts a Deployment to its previous revision, applying a reverse rolling update to replace failing pods with pods from the last good image.

Full `kubectl` command reference for common scenarios:

| Scenario                 | Command                                                           |
|--------------------------|-------------------------------------------------------------------|
| Deploy from YAML         | `kubectl apply -f deployment.yaml`                                |
| Scale replicas           | `kubectl scale deployment web-app --replicas=8`                   |
| Update image version     | `kubectl set image deployment/web-app web=nginx:1.26`             |
| **Roll back**            | **`kubectl rollout undo deployment/web-app`**                     |
| Expose via load balancer | `kubectl expose deployment web-app --type=LoadBalancer --port=80` |

> **Key principle:** `kubectl apply` is declarative (desired state from file). In production, store YAML in Git and use `kubectl apply` for all changes — making rollbacks a YAML edit + apply rather than an imperative command.

**Why not the others:**
- A) Deleting and recreating the deployment causes downtime and loses rollout history — this is not a rollback, it's a recreation
- B) Scaling to 0 replicas stops all traffic to the service (downtime) without restoring the previous version — it would then need a separate step to update the image
- D) `kubectl set image` requires knowing the exact previous image tag — `rollout undo` automatically uses the previously recorded revision without needing to look up the tag

---

### Q14: What Is a Kubernetes Pod?

**Question:**
In Kubernetes, what is a **Pod**?

**Options:**
- A) A physical server node in the cluster
- B) A running container image stored in Artifact Registry
- C) A namespace used to isolate workloads within a cluster
- D) A group of containers that share storage and networking

**Answer:** **D) A group of containers that share storage and networking**

**Official Explanation:**
Correct. A **Pod** is the smallest deployable unit in Kubernetes. It represents one or more containers that are always scheduled together on the same node, share the same network namespace (same IP address and port space), and can share mounted storage volumes.

**Common pattern:** A Pod often contains one primary application container plus lightweight **sidecar containers** (e.g., a logging agent, a proxy like Envoy) that support the main app.

**Exam Key Point:** A Pod ≠ a container. A Pod *groups* one or more containers. In practice, most Pods have exactly one container.

**Why not the others:**
- A) A physical server node in the cluster is called a **Node** in Kubernetes — a Node is a VM or physical machine that runs the container runtime and kubelet
- B) Container images stored in a registry (Artifact Registry, Docker Hub) are not Pods — Pods are running instances, not stored images
- C) A Kubernetes **Namespace** provides logical isolation for resources within a cluster (like virtual sub-clusters) — not a Pod

---

### Q15: Where Do GKE Cluster Resources Come From?

**Question:**
The resources used to build a Google Kubernetes Engine (GKE) cluster come from which GCP service?

**Options:**
- A) Compute Engine
- B) App Engine
- C) Cloud Functions
- D) Bare Metal Solution

**Answer:** **A) Compute Engine**

**Official Explanation:**
Correct. GKE cluster **nodes** are **Compute Engine VM instances**. When you create a GKE cluster, GKE provisions Compute Engine VMs as worker nodes. This means GKE clusters benefit from Compute Engine capabilities — including custom machine types, GPUs, persistent disks, and Google VPC networking.

**Exam Key Point:** GKE clusters = Compute Engine VMs under the hood. You can see node VMs in the Compute Engine console. In GKE Standard, you pay for those VMs. In GKE Autopilot, node provisioning is abstracted.

**Why not the others:**
- B) App Engine is a separate PaaS platform for deploying web applications — GKE node infrastructure does not come from App Engine
- C) Cloud Functions (Cloud Run Functions) is a serverless FaaS platform — its execution environment is completely separate from GKE cluster node infrastructure
- D) Bare Metal Solution provides dedicated physical servers for specialized workloads requiring direct hardware access — GKE uses Compute Engine VMs by default, not Bare Metal
