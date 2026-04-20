# GCP Key Terms & Glossary

A reference glossary of Google Cloud Platform terminology. Organized alphabetically within topic groups.

---

## Cloud Fundamentals

| Term | Definition |
|---|---|
| **Availability Zone / Zone** | A single isolated data center within a GCP region (e.g., `us-central1-a`). Zones are independent — a failure in one does not affect others. |
| **Broad Network Access** | NIST cloud characteristic: capabilities available over the network and accessible via standard mechanisms. |
| **CapEx** | Capital Expenditure — upfront investment in physical hardware. Traditional on-premises model. |
| **Cloud Computing** | NIST definition: on-demand self-service, broad network access, resource pooling, rapid elasticity, and measured service. |
| **Colocation (Colo)** | Renting physical data center space to install and manage your own hardware. |
| **IaaS** | Infrastructure as a Service — provider manages physical hardware and hypervisor; customer manages VMs, OS, and everything above. |
| **Measured Service** | NIST characteristic: resource usage monitored and billed per consumption. |
| **Multi-Tenancy** | Multiple customers sharing the same physical infrastructure with logical isolation. |
| **NIST** | National Institute of Standards and Technology — defines the five essential characteristics of cloud computing. |
| **On-Demand Self-Service** | NIST characteristic: provision resources without requiring human interaction with the provider. |
| **OpEx** | Operational Expenditure — pay-as-you-go model. Cloud computing converts CapEx to OpEx. |
| **PaaS** | Platform as a Service — provider manages infrastructure + runtime; customer manages application code and data. |
| **Point of Presence (PoP)** | A Google network edge location where Google's backbone connects to the public internet. Used by Cloud CDN and Interconnect. |
| **Rapid Elasticity** | NIST characteristic: resources can scale up or down quickly on demand. |
| **Region** | A geographic area containing 3+ zones (e.g., `us-central1` = Iowa). Resources deployed to a region for data residency and latency. |
| **Resource Pooling** | NIST characteristic: provider pools resources for multiple consumers using a multi-tenant model. |
| **SaaS** | Software as a Service — provider manages everything including the application. Customer just uses it (e.g., Gmail, Workspace). |
| **Sustained Use Discount** | Automatic GCP discount of up to 30% for VMs running > 25% of a billing month. No contract required. |

---

## Resource Hierarchy & IAM

| Term | Definition |
|---|---|
| **Basic Role** | Broad IAM roles: Viewer, Editor, Owner. Apply to all resources in a project. Generally discouraged for production. |
| **Cloud Identity** | Google's identity platform for managing corporate user accounts, groups, SSO, and MFA. Free tier available. |
| **Custom Role** | User-defined IAM role with a specific set of permissions. Use when predefined roles are too permissive. |
| **etcd** | Distributed key-value store used by Kubernetes as the source of truth for all cluster state. |
| **Folder** | Optional organizational layer in the GCP resource hierarchy. Groups projects under a department or team. |
| **Google Group** | A collection of Google/Cloud Identity accounts. Best practice: grant IAM roles to groups, not individuals. |
| **IAM** | Identity and Access Management — controls who can do what on which GCP resource. |
| **IAM Binding** | Associates a principal (who) with a role (what) on a resource (which). |
| **IAM Policy** | Document that contains one or more IAM bindings; attached to a GCP resource. |
| **Least Privilege** | Security principle: grant only the minimum permissions required to perform a task. |
| **Organization Node** | Root of the GCP resource hierarchy. Represents a company domain. Requires Cloud Identity or Google Workspace. |
| **Organization Policy** | Controls what resources and configurations can be created (e.g., restrict VM deployment to specific regions). |
| **Permission** | A granular action on a resource. Format: `service.resource.verb` (e.g., `compute.instances.create`). |
| **Predefined Role** | Curated IAM role scoped to a specific service (e.g., `roles/bigquery.dataViewer`). Recommended for most use cases. |
| **Principal** | The identity being granted access in an IAM binding. Can be a user, group, or service account. |
| **Project** | Primary unit of resource management in GCP. Resources belong to exactly one project. Has a unique Project ID, Number, and Name. |
| **Project ID** | Globally unique, user-chosen string identifier for a GCP project. Immutable after creation. |
| **Project Number** | Numerically assigned immutable identifier for a GCP project. Used internally by Google. |
| **Service Account** | Special identity representing an application or VM (not a human). Used for machine-to-machine API authentication. |

---

## Compute & Networking

| Term | Definition |
|---|---|
| **Auto Mode VPC** | VPC that automatically creates one subnet per region with predefined IP ranges. Convenient but inflexible. |
| **Autoscaler** | Kubernetes or Compute Engine feature that automatically adjusts instance count based on load. |
| **Cloud CDN** | Google's content delivery network. Caches static content at edge PoPs to reduce latency and origin load. |
| **Cloud DNS** | Managed authoritative DNS service with 100% uptime SLA. Supports public and private zones. |
| **Compute Engine** | GCP's IaaS virtual machine service. Supports custom machine types, GPUs, and per-second billing. |
| **Custom Machine Type** | Compute Engine VM with user-specified vCPU count and RAM (instead of predefined types). |
| **Dedicated Interconnect** | Direct physical fiber connection between on-premises network and Google. High bandwidth, no internet, SLA. |
| **Default VPC** | Auto-mode VPC created automatically in each new GCP project. Includes pre-configured firewall rules. |
| **Deployment (K8s)** | Kubernetes controller that manages a desired count of Pod replicas and handles rolling updates. |
| **Firewall Rule** | GCP rule that allows or denies traffic based on protocol, port, source, and target (VM tags or service accounts). |
| **Horizontal Scaling** | Adding more VM or container instances to distribute load. Requires load balancing; no downtime. |
| **Instance Template** | Blueprint for creating identical Compute Engine VM instances in a Managed Instance Group. |
| **Kubernetes** | Open-source container orchestration platform. Manages container lifecycle, scaling, networking, and rollout. |
| **Managed Instance Group (MIG)** | Group of identical Compute Engine VMs created from an instance template. Supports autoscaling and health checking. |
| **Network Tag** | Label applied to Compute Engine VMs to associate them with specific firewall rules. Enables micro-segmentation. |
| **Partner Interconnect** | Connectivity to Google via a third-party network provider. For sites without direct Dedicated Interconnect access. |
| **Persistent Disk** | Block storage for Compute Engine VMs. Zonal or regional. Decoupled from VM lifecycle. |
| **Pod** | Smallest deployable unit in Kubernetes. Contains one or more containers sharing a network namespace and storage. |
| **Service (K8s)** | Kubernetes object that provides a stable virtual IP and load-balances traffic across a set of pods. |
| **Shared VPC** | Network architecture where a host project shares its VPC subnets with service projects for centralized management. |
| **Spot VM** | Preemptible VM that Google may reclaim anytime. Up to 91% discount. Not for critical workloads. |
| **Vertical Scaling** | Resizing a VM to add more CPU/RAM. Requires stopping the VM; causes downtime. |
| **VPC** | Virtual Private Cloud — global, isolated private network in GCP. |
| **VPC Peering** | Private connectivity between two VPC networks (no internet). Non-transitive. IP ranges must not overlap. |
| **Cloud VPN** | Encrypted IPsec tunnel over the public internet connecting on-premises to GCP. |

---

## Storage & Databases

| Term | Definition |
|---|---|
| **Archive (storage class)** | Lowest-cost Cloud Storage class. For data accessed < once per year. Minimum 365-day storage duration. |
| **Artifact Registry** | GCP service for storing container images, Maven, npm, Python packages. Replaced Container Registry. |
| **Autoclass** | Cloud Storage feature that auto-transitions objects between storage classes based on access patterns. |
| **BigQuery** | Serverless, columnar data warehouse for analytical SQL queries. Not for transactional workloads. |
| **Bigtable** | High-throughput NoSQL wide-column database. For IoT, time-series, and analytics data at petabyte scale. |
| **Bucket** | Container for objects in Cloud Storage. Requires globally unique name. Location set at creation and immutable. |
| **CMEK** | Customer-Managed Encryption Key — customer controls encryption keys via Cloud KMS. |
| **Cloud SQL** | Managed relational database supporting MySQL, PostgreSQL, and SQL Server. Regional; max 64 TB/instance. |
| **Cloud Spanner** | Globally distributed relational database with strong consistency. Petabyte-scale; 99.999% SLA. |
| **Coldline (storage class)** | Cloud Storage class for data accessed < once per quarter. Minimum 90-day storage. Retrieval fees apply. |
| **Filestore** | Managed NFS (Network File System) service for shared file access across multiple VMs. |
| **Firestore** | Serverless document NoSQL database. Supports real-time sync and offline access for mobile/web apps. |
| **Lifecycle Management** | Automated policy to delete or transition objects in Cloud Storage based on age, access, or version count. |
| **Multi-Regional** | Cloud Storage location type replicating data across 3+ regions on a continent (US, EU, Asia). |
| **Nearline (storage class)** | Cloud Storage class for data accessed < once per month. 30-day minimum. Retrieval fees apply. |
| **Object** | Immutable binary blob stored in a Cloud Storage bucket. Maximum size: 5 TB. |
| **Object Versioning** | Cloud Storage feature that retains prior versions of objects when they are replaced or deleted. |
| **Regional (storage location)** | Cloud Storage data replicated across multiple zones within one region. |
| **Retention Policy** | Cloud Storage setting enforcing minimum object retention duration. Used for compliance. Can be locked. |
| **Standard (storage class)** | Highest-cost Cloud Storage class. No retrieval fees. For hot, frequently accessed data. |
| **Storage Transfer Service** | Managed service for transferring data to Cloud Storage from other clouds (S3, Azure Blob) or HTTP sources. |
| **Transfer Appliance** | Physical storage device shipped by Google for offline large-scale data migration to Cloud Storage. |

---

## Serverless & Containers

| Term | Definition |
|---|---|
| **Buildpacks** | Open-source toolset that builds production container images from source code without a Dockerfile. |
| **Cloud Build** | Fully managed CI/CD build service. Builds container images, runs tests, triggers deployments. |
| **Cloud Functions** | Event-driven serverless compute. Now called Cloud Run Functions. Triggered by 100+ event types. |
| **Cloud Run** | Fully managed serverless container platform. Stateless; autoscales 0→∞; pay per request. |
| **Cold Start** | Latency from spinning up a new Cloud Run or Cloud Functions instance after being idle. |
| **Container** | Lightweight, portable software package including the application and all its dependencies. Shares host OS kernel. |
| **Control Plane (K8s)** | Kubernetes master components: API Server, Scheduler, Controller Manager, etcd. Managed by Google in GKE. |
| **GKE Autopilot** | GKE mode where Google manages nodes. Billed per Pod resource request. Recommended default. |
| **GKE Standard** | GKE mode where you manage and configure node pools. Billed per node VM uptime. |
| **kubelet** | Agent running on each Kubernetes node. Receives instructions from control plane; manages containers. |
| **kubectl** | CLI tool for managing Kubernetes resources. Communicates with the API Server. |
| **Managed Instance Group** | Group of identically configured VMs managed as a single entity with autoscaling and healing. |
| **Minimum Instances** | Cloud Run setting to keep N instances warm, eliminating cold starts (at idle cost). |
| **ReplicaSet** | Kubernetes controller ensuring a specified number of Pod replicas are running at all times. |
| **Rolling Update** | Kubernetes strategy: incrementally replace old Pods with new ones; no downtime. |
| **Scale-to-Zero** | Cloud Run/Functions feature: instances drop to 0 when no traffic; cost drops to $0 at idle. |
| **Stateless** | Application that stores no session/state locally; all state is externalized. Required for Cloud Run. |
