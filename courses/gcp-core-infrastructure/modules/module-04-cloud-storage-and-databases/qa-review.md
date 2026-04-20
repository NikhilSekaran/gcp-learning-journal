# Module 4: Cloud Storage and Databases - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module 4 — Cloud Storage & Databases. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---

## Topic 1: Cloud Storage Fundamentals

### Q1: Object Storage vs. File Storage

**Question:**
An application running on Compute Engine needs a shared file system where multiple VMs can mount the same directory using NFS and read/write files simultaneously. Should they use Cloud Storage?

**Options:**
- A) Yes — Cloud Storage supports NFS and POSIX file system access
- B) No — Cloud Storage is object storage; they should use Filestore for NFS access
- C) Yes — Cloud Storage buckets can be mounted as disks on any Compute Engine VM
- D) No — they should use Persistent Disk shared between VMs

**Answer:** **B) No — Cloud Storage is object storage; they should use Filestore for NFS access**

**Explanation:**
Cloud Storage does NOT expose a file system. It stores immutable objects accessed via HTTP/API — there's no concept of in-place editing, POSIX permissions, or NFS mounting.

For shared file system access:
- **Filestore** = managed NFS service; multiple VMs can mount and use simultaneously
- **Persistent Disk** = block storage; can be mounted read-write on one VM at a time (or read-only to multiple)
- **Cloud Storage** = object storage; ideal for static files, backups, data lakes — not for shared POSIX filesystems

**Why not the others:**
- A) Cloud Storage does NOT support NFS or POSIX file system access — it is accessed only via HTTP/REST API or gRPC; no kernel-level mounting is possible
- C) Cloud Storage buckets cannot be mounted as disks on Compute Engine VMs — for disk-like block storage access, use Persistent Disk
- D) Persistent Disk supports block storage for VMs but is limited for sharing — a single Persistent Disk can only be mounted read-write on one VM at a time; Filestore is better for true simultaneous multi-VM NFS access

---

### Q2: Bucket Naming

**Question:**
A developer creates a bucket named `my-company-data` in their GCP project. Another developer at a different company tries to create a bucket with the same name. What happens?

**Options:**
- A) The second bucket is created in the second company's project with no conflict
- B) The bucket creation fails because bucket names are globally unique across all of GCP
- C) The second bucket is created but with a numeric suffix auto-appended
- D) Both companies can have the same bucket name but in different regions

**Answer:** **B) The bucket creation fails because bucket names are globally unique across all of GCP**

**Explanation:**
Cloud Storage bucket names are part of the object's public URL: `https://storage.googleapis.com/BUCKET_NAME/object`. Because this URL must be globally unique, bucket names must be globally unique across the entire GCP ecosystem — not just within your project or organization.

**Why not the others:**
- A) Unlike many GCP resources scoped to a project, Cloud Storage bucket names are globally accessible in the URL — uniqueness must hold across all GCP projects worldwide
- C) GCP does not auto-append numeric suffixes to bucket names — the creation request simply fails with a "bucket already exists" error
- D) Bucket names are globally unique regardless of region or location — location only affects where data is stored, not the naming namespace

---

### Q3: Object Versioning

**Question:**
A developer accidentally overwrites a critical object `config.json` in Cloud Storage. Object versioning is enabled on the bucket. What is the state of the bucket after the overwrite, and how can the previous version be restored?

**Options:**
- A) The previous version is permanently deleted; there is no way to recover it
- B) The previous version becomes a non-current version with its original generation number; it can be restored by copying it back to the current version
- C) The previous version is automatically moved to a separate backup bucket configured by GCP
- D) Object versioning must be manually triggered before each write to preserve a version

**Answer:** **B) The previous version becomes a non-current version with its original generation number; it can be restored by copying it back to the current version**

**Explanation:**
Object versioning preserves every previous version of an object each time it is replaced or deleted. When enabled:
- Overwriting `config.json` creates a **new current version** while the old one becomes a **non-current version** with its original generation number
- Deleting an object creates a **delete marker** but doesn't permanently remove non-current versions
- You can list all versions with their **generation numbers** and restore any prior version

**When to enable:** Accidental deletion recovery, audit trails, compliance requirements.

**Cost consideration:** Non-current versions continue to incur storage charges. Pair versioning with a **lifecycle policy** to auto-delete non-current versions after a set number of days.

**Why not the others:**
- A) Object versioning specifically prevents permanent deletion on overwrite — the previous version is retained as a non-current version
- C) GCP does not create automatic backup buckets — versioning stores all versions in the same bucket under different generation numbers
- D) Versioning is a bucket-level configuration that applies automatically to every write once enabled — no manual triggering is needed per operation

---

## Topic 2: Storage Classes & Location Types

### Q4: Choosing Storage Class

**Question:**
A company stores regulatory audit logs that must be kept for 7 years to satisfy compliance requirements. The logs are never accessed during the retention period but must be retrievable within 24 hours if a regulator requests them. Which storage class is most cost-effective?

**Options:**
- A) Standard — fastest retrieval
- B) Nearline — accessed less than once per month
- C) Coldline — accessed less than once per quarter
- D) Archive — accessed less than once per year

**Answer:** **D) Archive**

**Explanation:**
Archive class is designed exactly for this use case:
- Logs that are never accessed during normal operations → accessed < once per year
- Archive has the **lowest storage cost** per GB
- Retrieval time "within 12 hours" is typical for Archive (milliseconds to minutes for data retrieval from Archive, though it has the highest retrieval fee)
- 7-year retention with a minimum 365-day storage requirement perfectly matches Archive class

> **Note:** Archive retrieval is fast (milliseconds to minutes) — the "archive" name refers to infrequent access, not slow retrieval. This distinguishes it from AWS Glacier's hours-long retrieval.

**Why not the others:**
- A) Standard has the fastest retrieval but the highest per-GB storage cost — for data never accessed during 7 years, Standard would be expensive overkill
- B) Nearline is for data accessed less than once per month — higher storage cost per GB than Coldline/Archive for long-term retention with no expected access
- C) Coldline is for data accessed less than once per quarter — still more expensive per GB than Archive for data stored for years without access

---

### Q5: Storage Location and Compliance

**Question:**
A European company must ensure that all customer PII data stays within the European Union due to GDPR requirements. They need high availability within the EU. Which Cloud Storage location type should they choose?

**Options:**
- A) Multi-Regional in the `us` (United States) location
- B) Multi-Regional in the `eu` (Europe) location
- C) Single-Zone in `europe-west3` (Frankfurt)
- D) Standard class storage in any region

**Answer:** **B) Multi-Regional in the `eu` (Europe) location**

**Explanation:**
- **Multi-Regional `eu`** replicates data across 3+ locations within the European continent, satisfying GDPR data residency
- Provides the highest availability within EU
- Data never leaves the European Union

**Why not the others:**
- A) `us` multi-region — data stored in the United States; GDPR violation for EU customer PII
- C) Single-zone in `europe-west3` (Frankfurt) — data stays in Germany but provides no fault tolerance; a single zone failure takes the data unavailable
- D) Standard class in any region — storage class controls cost and access frequency, not data location; this doesn't specify an EU region and doesn't satisfy the GDPR residency requirement

---

### Q6: Autoclass Feature

**Question:**
A company has a Cloud Storage bucket with hundreds of thousands of objects. The engineering team doesn't know which objects will be accessed in the future — some are hot, some accumulate "dust" over time. What feature should they enable to optimize storage costs automatically?

**Options:**
- A) Object versioning
- B) Autoclass
- C) Bucket-level default storage class
- D) Lifecycle management with a fixed 30-day transition

**Answer:** **B) Autoclass**

**Explanation:**
Autoclass monitors each object's access patterns and automatically moves it through storage classes:
- Frequently accessed objects stay in Standard
- Objects not accessed for 30 days move to Nearline
- Not accessed for 90 days → Coldline
- Not accessed for 365 days → Archive

This is ideal when access patterns are unpredictable or vary by object. A fixed lifecycle rule would incorrectly demote objects that are actually accessed regularly.

**Why not the others:**
- A) Object versioning preserves previous versions of objects when overwritten or deleted — it has nothing to do with storage cost optimization based on access patterns
- C) A bucket-level default storage class sets the initial class for new objects — it doesn't dynamically adjust based on per-object access frequency over time
- D) A fixed 30-day lifecycle transition moves ALL objects to Nearline after 30 days regardless of whether they're still being accessed — this would increase retrieval costs for hot objects that happen to be older than 30 days

---

## Topic 3: Database Selection

### Q7: Cloud SQL vs. Cloud Spanner

**Question:**
A global e-commerce company runs a product inventory system. The inventory must be strongly consistent — when a product sells out in New York, warehouse managers in Tokyo must immediately see the updated stock count. The system handles 50,000 transactions per second globally. Which database should they choose?

**Options:**
- A) Cloud SQL with read replicas in each region
- B) Cloud Spanner
- C) Firestore
- D) Bigtable

**Answer:** **B) Cloud Spanner**

**Explanation:**
Cloud Spanner is designed for exactly this scenario:
- **Global, multi-region replication** with **strong consistency** — writes in New York are immediately visible in Tokyo (not eventually consistent)
- **Petabyte-scale** horizontal scaling — 50,000 TPS is within Cloud Spanner's range
- **Relational SQL** — inventory is a structured, relational workload with complex queries and transactions

**Why not the others:**
- A) Cloud SQL with read replicas — Cloud SQL is regional; strong global consistency requires a complex multi-master setup that Cloud SQL doesn't natively support; also limited to 64 TB per instance
- C) Firestore — a document database designed for mobile/web apps with real-time sync; not suited for high-throughput relational inventory management or the 50,000 TPS requirement
- D) Bigtable — a high-throughput NoSQL wide-column database; doesn't support SQL queries, relational transactions, or the inventory management use case

---

### Q8: BigQuery Misconception

**Question:**
A developer wants to use BigQuery as the primary database for a banking application that processes thousands of individual account debit/credit transactions per second with strict ACID guarantees. Is this appropriate?

**Options:**
- A) Yes — BigQuery is a fully managed serverless database suitable for high-frequency transactions
- B) Yes — BigQuery supports INSERT/UPDATE/DELETE at high throughput with row locking
- C) No — BigQuery is an analytical data warehouse, not a transactional database
- D) No — BigQuery is not ACID-compliant at all

**Answer:** **C) No — BigQuery is an analytical data warehouse, not a transactional database**

**Explanation:**
BigQuery excels at **analytical queries** — scanning billions of rows for aggregations (SUM, COUNT, AVG) across large datasets. It is NOT designed for:
- High-frequency individual row INSERT/UPDATE/DELETE
- Low-latency single-record lookups (e.g., "get account balance for account #12345")
- Row-level locking and high-throughput OLTP transactions

For this banking scenario, use **Cloud Spanner** (global, strongly consistent, relational) or **Cloud SQL** (if regional scope is sufficient).

**Why not the others:**
- A) BigQuery is not suitable for high-frequency transactions — it's an analytical data warehouse designed for batch analytics, not OLTP workloads
- B) BigQuery supports DML (INSERT/UPDATE/DELETE) but is optimized for batch operations, not thousands of individual row mutations per second — the columnar storage model makes row-level updates expensive
- D) BigQuery does support ACID guarantees for individual DML statements — the reason it's unsuitable is performance, not ACID compliance; ACID compliance doesn't equal OLTP performance

---

### Q9: Choosing the Right Database

**Question:**
A mobile app needs to work offline and sync data back to the server with real-time push updates to other users when connectivity is restored. Which Google Cloud database is purpose-built for this use case?

**Options:**
- A) Bigtable — designed for high-throughput data ingestion and time-series data
- B) Cloud Spanner — globally consistent relational database with high availability
- C) Firestore — serverless document database with native offline SDKs and real-time listeners
- D) BigQuery — serverless analytical warehouse with streaming ingestion support

**Answer:** **C) Firestore — serverless document database with native offline SDKs and real-time listeners**

**Explanation:**
Firestore is specifically designed for mobile and web applications. The full database-to-scenario mapping:

| Scenario                                 | Best Database | Reason                                                           |
|------------------------------------------|---------------|------------------------------------------------------------------|
| Mobile app offline sync + real-time push | **Firestore** | Native offline SDKs; real-time listeners push changes to clients |
| IoT time-series at 1M events/second      | **Bigtable**  | High-throughput NoSQL for wide-column time-series patterns       |
| SQL analytics on 50 TB of event data     | **BigQuery**  | Serverless analytical warehouse for petabyte-scale queries       |
| Managed MySQL hosting                    | **Cloud SQL** | Managed MySQL with automated backups, patching, and HA replicas  |

**Why not the others:**
- A) Bigtable is for high-throughput IoT/time-series ingestion accessed by row key and range scans — it has no mobile SDK, no offline support, and no real-time push to clients
- B) Cloud Spanner is a globally consistent relational database for high-TPS transactional workloads — not designed for mobile offline sync or real-time document listeners
- D) BigQuery is an analytical warehouse — not suitable for real-time mobile app data or offline sync; it doesn't have a mobile SDK

---

### Q10: Firestore vs. Bigtable

**Question:**
An IoT platform ingests 2 million sensor readings per second. Queries are always by device ID + time range. There are no ad-hoc joins or complex WHERE clauses. Which NoSQL database is most appropriate?

**Options:**
- A) Firestore — serverless and automatically scales to handle high write throughput
- B) Cloud SQL — relational databases support indexed range queries on timestamp columns
- C) Bigtable — designed for high-throughput write ingestion and row key/range scan access patterns
- D) BigQuery — serverless and handles streaming ingestion at any scale

**Answer:** **C) Bigtable — designed for high-throughput write ingestion and row key/range scan access patterns**

**Explanation:**

| Factor                | Firestore                                               | Bigtable                                      |
|-----------------------|---------------------------------------------------------|-----------------------------------------------|
| **Data model**        | Document → Collection hierarchy                         | Wide-column (row key + column families)       |
| **Access pattern**    | Flexible queries; multiple indexes; real-time listeners | Key-based lookups; range scans; no ad-hoc SQL |
| **Client SDK**        | Rich mobile/web SDKs with offline sync                  | No mobile SDK; accessed via API/HBase client  |
| **Operational model** | Fully serverless; scales to zero                        | Node-based; minimum 1 node; not serverless    |
| **Real-time sync**    | Yes — push updates to clients                           | No                                            |
| **Throughput**        | Moderate; for app-scale workloads                       | Very high; millions of rows/second            |

**Choose Firestore when:** Building mobile/web apps, need real-time data sync, offline capability, or document-oriented data model.
**Choose Bigtable when:** Massive write throughput for IoT, time-series, or analytics ingestion; data is accessed by row key and range scans.

**Why not the others:**
- A) Firestore is serverless but designed for app-scale workloads, not millions of writes per second from IoT devices; also no HBase-style range scan API
- B) Cloud SQL would require sharding and careful indexing to approach this scale — relational databases are not designed for millions of writes per second from high-cardinality time-series
- D) BigQuery's streaming inserts support high throughput, but its columnar format is optimized for analytics queries, not real-time row-key lookups by device ID

---

## Topic 4: Data Ingestion

### Q11: Moving Large Data to GCP

**Question:**
A company needs to migrate 2 PB of on-premises research data to Cloud Storage. Their internet connection is 1 Gbps. Approximately how long would an online transfer take at full speed, and what should they use instead?

**Options:**
- A) Approximately 2 weeks; Storage Transfer Service over the internet would be faster
- B) Approximately 185 days; they should use Transfer Appliance to physically ship the data
- C) Approximately 18 days; the company should upgrade to a 10 Gbps dedicated line first
- D) Online transfer is not possible for datasets over 1 TB; a VPN connection is required

**Answer:** **B) Approximately 185 days; they should use Transfer Appliance to physically ship the data**

**Explanation:**
**Time calculation:**
- 2 PB = 2,000 TB = 2,000,000 GB
- 1 Gbps = 125 MB/s = 0.125 GB/s
- Time = 2,000,000 GB ÷ 0.125 GB/s ≈ 16,000,000 seconds ≈ **185 days**

Even at full 1 Gbps with no other network traffic, an online transfer would take over 6 months.

**Transfer Appliance:**
- Google ships a physical high-capacity storage appliance (up to 1 PB per appliance)
- Customer loads data offline, ships it back to Google
- Google ingests the data into Cloud Storage — total time: days not months

> **Rule of thumb:** If transferring more than 100 TB over a limited internet connection, consider Transfer Appliance.

**Why not the others:**
- A) Storage Transfer Service still uses the internet connection — same 185-day constraint; it's designed for cloud-to-cloud transfers or HTTP-accessible data, not large on-premises migrations
- C) Upgrading to 10 Gbps would reduce the time to ~18 days, but leasing a dedicated 10 Gbps line for a one-time migration is expensive and complex — Transfer Appliance is much more practical
- D) Online transfer IS possible for datasets over 1 TB (no hard limit exists) — the problem is time and bandwidth, not a protocol restriction

---

### Q12: Uniform vs. Fine-Grained Access Control

**Question:**
A developer needs to make a few specific objects in a Cloud Storage bucket publicly readable while all other objects remain private. What is Google's recommended approach?

**Options:**
- A) Enable fine-grained ACLs on the bucket and set `allUsers: READER` on the individual public objects
- B) Enable uniform bucket-level access and grant `allUsers` the `roles/storage.objectViewer` role on the whole bucket
- C) Keep uniform bucket-level access; separate public objects into a dedicated bucket configured for public access
- D) Change the bucket's default storage class to Standard; this makes all new objects publicly readable

**Answer:** **C) Keep uniform bucket-level access; separate public objects into a dedicated bucket configured for public access**

**Explanation:**
While fine-grained ACLs technically enable per-object permissions, Google recommends against mixing public and private objects in the same bucket:

| Method                  | Granularity  | Recommended?                    |
|-------------------------|--------------|---------------------------------|
| **Uniform (IAM)**       | Bucket-level | ✅ Yes — for all new buckets     |
| **Fine-Grained (ACLs)** | Per-object   | ⚠️ Legacy only; not recommended |

**Best practice:** Create two buckets — one for public objects (with `allUsers` at bucket level), one for private objects. Uniform access is simpler, auditable, and avoids accidental data exposure from misconfigured ACLs.

**Why not the others:**
- A) Fine-grained ACLs work technically, but per-object permissions are error-prone, hard to audit, and Google explicitly recommends uniform bucket-level access for new buckets
- B) Granting `allUsers` at the bucket level makes ALL objects in the bucket publicly readable — violates the requirement that other objects remain private
- D) Storage class (Standard, Nearline, Coldline, Archive) controls cost and access frequency, not access permissions — changing the storage class has no effect on public vs. private visibility

---

### Q13: Lifecycle Management Policy

**Question:**
Cloud Storage lifecycle rules that transition objects between storage classes can move objects in which direction?

**Options:**
- A) Both directions — warmer or colder — based on a time threshold you configure
- B) Only to colder classes (Standard → Nearline → Coldline → Archive); transitions to warmer classes require Autoclass
- C) Only to warmer classes; you must manually transition to colder classes
- D) Both directions when Autoclass is enabled; fixed lifecycle rules are unidirectional in either direction

**Answer:** **B) Only to colder classes (Standard → Nearline → Coldline → Archive); transitions to warmer classes require Autoclass**

**Explanation:**
Fixed lifecycle rules can only move objects to **colder** (less expensive) storage classes. You cannot write a lifecycle rule that moves an Archive object back to Standard.

A typical log retention policy:
```
Age 0–30 days   → Standard Storage     (active access)
Age 31–90 days  → Nearline Storage     (transition at day 30)
Age 91–365 days → Coldline Storage     (transition at day 90)
Age 365+ days   → Archive Storage      (transition at day 365)
Age 7 years     → Delete               (transition at day 2555)
```

> **Exam Tips:**
> - Lifecycle rules are evaluated **once per day**
> - Transitions move objects to **colder** classes only; Autoclass can move objects both ways based on actual access
> - Combine with a **Retention Policy lock** for legal hold requirements

**Why not the others:**
- A) Fixed lifecycle rules are unidirectional (colder only) — you cannot configure a time-threshold rule to move objects to warmer classes
- C) Fixed lifecycle rules only move to **colder** classes, not warmer; you cannot write a rule to promote Archive → Standard
- D) Autoclass moves objects both ways based on access patterns — but fixed lifecycle rules can only move objects to colder classes regardless of Autoclass being enabled

---

### Q14: Cloud Storage Use Case

**Question:** What is the correct use case for Cloud Storage?

**Options:**
- A) Cloud Storage is well suited to providing the root file system of a Linux virtual machine
- B) Cloud Storage is well suited to providing durable and highly available object storage
- C) Cloud Storage is well suited to providing data warehousing services
- D) Cloud Storage is well suited to providing RDBMS services

**Answer:** **B) Cloud Storage is well suited to providing durable and highly available object storage**

**Explanation:**
Cloud Storage is object storage, not file storage, not data warehousing, not RDBMS. Compute Engine VMs use Persistent Disk for their file systems. BigQuery is for data warehousing. Cloud SQL/Spanner for relational databases.

**Why not the others:**
- A) Cloud Storage cannot be used as the root file system for a VM — VMs need block storage (Persistent Disk) for their OS; Cloud Storage is object storage, not block storage
- C) Data warehousing is BigQuery's role — serverless SQL analytics over large datasets; Cloud Storage can store data lakes that feed into BigQuery, but it is not itself a warehouse
- D) Cloud SQL provides relational database management (MySQL, PostgreSQL, SQL Server) — Cloud Storage stores unstructured objects, not structured relational data

---

### Q15: Relational Database Scale

**Question:** Which relational database service can scale to higher database sizes?

**Options:**
- A) Cloud Spanner
- B) Cloud SQL
- C) Bigtable
- D) Firestore

**Answer:** **A) Cloud Spanner**

**Explanation:**
Spanner scales to **petabyte** database sizes. Cloud SQL is limited to **64 TB** max. Bigtable is NoSQL (not relational). Firestore is NoSQL document database.

**Why not the others:**
- B) Cloud SQL supports up to 64 TB per instance — adequate for most applications, but not for the very largest relational workloads requiring petabyte-scale
- C) Bigtable is a NoSQL wide-column database — not a relational database; doesn't support SQL joins or traditional relational transactions
- D) Firestore is a NoSQL document database — not relational; doesn't support SQL or the relational data model
---

### Q16: Coldline Storage Class

**Question:** Why would a customer consider the Coldline storage class?

**Options:**
- A) To save money on storing infrequently accessed data
- B) To save money on storing frequently accessed data
- C) To improve security
- D) To use the Coldline Storage API

**Answer:** **A) To save money on storing infrequently accessed data**

**Explanation:**
Coldline has a low monthly storage rate but charges a retrieval fee. Best for data accessed **at most once every 90 days**. Storing frequently accessed data in Coldline would actually cost more due to retrieval fees. Security and API access are identical across all storage classes.

**Why not the others:**
- B) Storing frequently accessed data in Coldline would be more expensive overall — the low storage cost is offset by high per-GB retrieval fees; Coldline is only cost-effective when access is infrequent
- C) All storage classes have the same security — IAM, encryption at rest, and encryption in transit apply equally; Coldline provides no additional security features
- D) There is no "Coldline Storage API" — all storage classes use the same Cloud Storage JSON/XML API; Coldline is a pricing/availability tier, not a separate API
