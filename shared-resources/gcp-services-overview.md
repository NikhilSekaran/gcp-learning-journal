# GCP Services Overview

Quick reference of Google Cloud Platform services covered in courses. Updated as new topics are studied.

---

## Compute

| Service | Type | Description | When to Use |
|---|---|---|---|
| **Compute Engine** | IaaS | Virtual machines (fully configurable) | Full OS control; custom workloads; lift-and-shift migrations |
| **App Engine** | PaaS | Managed app hosting (Standard and Flexible) | Web apps without managing servers |
| **Cloud Run** | Serverless | Managed containerized apps; scale to zero | Stateless containers; APIs; event-driven microservices |
| **Cloud Functions** | Serverless | Event-driven single-purpose functions | Webhooks; background processing; event triggers |
| **GKE (Standard)** | CaaS | Managed Kubernetes; you manage nodes | Containerized apps needing Kubernetes with node control |
| **GKE Autopilot** | Serverless K8s | Managed Kubernetes; Google manages nodes | Kubernetes workloads without node management overhead |
| **Cloud Batch** | Managed | Scheduled batch computing jobs | HPC and data processing batch jobs |

---

## Storage

| Service | Type | Description | When to Use |
|---|---|---|---|
| **Cloud Storage** | Object | Immutable blob storage; globally addressed URLs | Backups; static website content; data lakes; serving images/media |
| **Filestore** | File | Managed NFS file server | Multi-VM shared filesystems; legacy NFS workloads |
| **Persistent Disk** | Block | Zonal or regional VM block storage | VM boot disks; database storage; high-IOPS workloads |
| **Hyperdisk** | Block | Next-gen block storage for VMs | Ultra-high-performance storage needs |

---

## Databases

| Service | Type | Scale | Serverless | When to Use |
|---|---|---|---|---|
| **Cloud SQL** | Relational (MySQL, PG, SQL Server) | Up to 64 TB | No | Traditional relational apps; managed MySQL/PostgreSQL |
| **Cloud Spanner** | Relational (global) | Petabyte | No | Global OLTP with strong consistency; financial systems |
| **Firestore** | NoSQL Document | Automatic | Yes | Mobile/web apps; real-time sync; offline-capable apps |
| **Bigtable** | NoSQL Wide-column | Petabyte | No | IoT; time-series; high-throughput analytics ingestion |
| **BigQuery** | Data Warehouse | Petabyte | Yes | Analytical SQL; BI dashboards; ML on large datasets |
| **Memorystore** | In-memory cache | Regional | No | Redis/Memcached caching; session storage; leaderboards |
| **AlloyDB** | PostgreSQL-compatible | Regional | No | High-performance PostgreSQL; OLTP + HTAP workloads |

---

## Networking

| Service | Description | When to Use |
|---|---|---|
| **VPC** | Global Virtual Private Cloud | Foundation of all GCP networking |
| **Cloud Load Balancing** | Global and regional load balancers (L4 and L7) | Distribute traffic; multi-region failover; SSL termination |
| **Cloud DNS** | Managed authoritative DNS (100% SLA) | Host domain DNS records |
| **Cloud CDN** | Content delivery network at edge PoPs | Cache static content globally; reduce latency |
| **Cloud NAT** | Network Address Translation | Allow VMs without public IPs to reach the internet |
| **Cloud VPN** | Encrypted IPsec tunnel over internet | Site-to-site hybrid connectivity; low-cost |
| **Dedicated Interconnect** | Direct physical fiber to Google | High-bandwidth, low-latency, no-internet enterprise connectivity |
| **Partner Interconnect** | Via third-party provider to Google | Interconnect without direct colocation at Google PoP |
| **Cloud Armor** | WAF and DDoS protection | Protect HTTP(S) load balancers from attacks |
| **VPC Service Controls** | API access perimeter | Prevent data exfiltration from GCP APIs |

---

## Identity & Security

| Service | Description | When to Use |
|---|---|---|
| **Cloud IAM** | Identity and Access Management for GCP | All access control; roles and permissions |
| **Cloud Identity** | Employee identity management (free tier) | Manage corporate user accounts; AD sync; SSO |
| **Google Workspace** | Cloud Identity + productivity apps (Gmail, Drive, Meet) | When Google apps are also needed |
| **Secret Manager** | Managed secrets storage (API keys, passwords, certs) | Store and retrieve secrets securely in apps |
| **Cloud KMS** | Key Management Service | Customer-managed encryption keys (CMEK) |
| **Cloud Certificate Manager** | Managed TLS certificates | Auto-renewing SSL certs for Load Balancers and Cloud Run |
| **Binary Authorization** | Container image signing policy | Enforce that only signed, trusted images deploy to GKE |

---

## DevOps & CI/CD

| Service | Description | When to Use |
|---|---|---|
| **Cloud Build** | Fully managed CI/CD build service | Build container images; run tests; trigger deployments |
| **Artifact Registry** | Container image and artifact storage | Store Docker images, Maven, npm, Python packages |
| **Cloud Deploy** | Managed continuous delivery pipelines | Deploy to GKE, Cloud Run, Compute Engine |
| **Cloud Source Repositories** | Managed Git hosting | Host private Git repos within GCP |

---

## Data & Analytics

| Service | Description | When to Use |
|---|---|---|
| **BigQuery** | Serverless analytical data warehouse | SQL analytics on petabyte-scale datasets |
| **Dataflow** | Managed Apache Beam (stream + batch pipelines) | ETL; real-time and batch data processing |
| **Pub/Sub** | Managed message queue / event streaming | Decouple producers and consumers; event-driven systems |
| **Dataproc** | Managed Hadoop and Spark | Lift-and-shift Spark/Hadoop workloads |
| **Looker** | Business intelligence and data exploration | BI dashboards and data exploration |

---

## AI & Machine Learning

| Service | Description | When to Use |
|---|---|---|
| **Vertex AI** | Unified ML platform (train, deploy, manage models) | End-to-end ML workflows; AutoML; custom model training |
| **Cloud Vision API** | Pre-trained image analysis | Object detection, OCR, face detection without ML expertise |
| **Cloud Natural Language API** | Pre-trained NLP | Entity extraction, sentiment analysis |
| **Cloud Translation API** | Text translation | Translate content between languages |
| **Dialogflow** | Conversational AI / chatbots | Build virtual agents and chatbots |

---

## Management & Operations

| Service | Description | When to Use |
|---|---|---|
| **Cloud Monitoring** | Metrics, dashboards, alerting | Monitor GCP resource health and performance |
| **Cloud Logging** | Log aggregation and analysis | Centralized logs for all GCP services |
| **Cloud Trace** | Distributed tracing | Debug latency in microservices |
| **Error Reporting** | Auto-detect and alert on app errors | Surface application errors quickly |
| **Cloud Deployment Manager** | Infrastructure as Code (GCP-native) | Template-based GCP resource provisioning |
| **Terraform on GCP** | HashiCorp IaC tool | Multi-cloud infrastructure as code |
