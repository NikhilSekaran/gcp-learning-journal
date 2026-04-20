# Module 6: Serverless Applications - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module 6 — Serverless Applications. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---

## Topic 1: Serverless Concepts

### Q1: Serverless Definition

**Question:**
Which statement best defines "serverless" computing in the context of Google Cloud?

**Options:**
- A) There are literally no servers — computation happens in the user's browser
- B) The cloud provider manages all infrastructure; developers focus on code/containers; billing is based on actual usage
- C) Serverless means zero cost — you only pay for the compute when traffic exceeds 1000 requests/second
- D) Serverless is only applicable to functions; containerized apps always require server management

**Answer:** **B) The cloud provider manages all infrastructure; developers focus on code/containers; billing is based on actual usage**

**Explanation:**
"Serverless" doesn't mean no servers exist — there are definitely servers running your code. It means the **developer doesn't manage, provision, or configure those servers**. The provider handles:
- Server provisioning and teardown
- OS patching
- Scaling up and down (including to zero)
- Load balancing and request routing
- High availability

Billing is per actual execution (requests + duration), not per VM uptime.

**Why not the others:**
- A) Servers absolutely exist in serverless computing — Google runs physical servers that execute your code; "serverless" refers to the developer not managing them, not that they don't exist
- C) Serverless doesn't mean zero cost — billing is per invocation and per compute duration; there is no request-count threshold below which compute is free (beyond the existing free tier)
- D) Serverless extends beyond functions — Cloud Run runs containerized applications in a fully serverless manner; serverless is an operational model, not limited to function-only workloads

---

### Q2: Scale-to-Zero Impact

**Question:**
An internal tool on Cloud Run receives 10 requests per business day (9 AM–5 PM weekdays). What is the approximate monthly compute cost for this service?

**Options:**
- A) Same as a Compute Engine VM running 24/7
- B) Very low — Cloud Run charges only for the milliseconds of compute time during those 10 daily requests
- C) High — Cloud Run charges a flat monthly fee regardless of request count
- D) Zero — services with fewer than 1,000 requests/month are free

**Answer:** **B) Very low — Cloud Run charges only for the milliseconds of compute time during those 10 daily requests**

**Explanation:**
Cloud Run **scales to zero** when there are no incoming requests. For a service receiving only 10 requests/day:
- Outside business hours (16 hours/day, all weekends) → zero cost
- During the 10 daily requests → charged for CPU + memory during those specific ms/seconds of processing

A Cloud Run free tier also covers the first 2 million requests/month and 360,000 GB-seconds of memory/month, so this tool's cost may literally be $0 or a few cents/month.

This contrasts sharply with a Compute Engine VM that runs 24/7 regardless of traffic.

**Why not the others:**
- A) A Compute Engine VM running 24/7 incurs consistent charges regardless of traffic — Cloud Run with scale-to-zero is fundamentally different; it only charges during active request handling
- C) Cloud Run has no flat monthly fee — billing is entirely usage-based (CPU + memory during requests, plus per-million-requests fee); there is no baseline charge
- D) While Cloud Run has a free tier for the first 2 million requests/month, billing beyond the free tier is based on actual usage, not a 1000-request threshold

---

## Topic 2: Cloud Run

### Q3: Cloud Run Stateless Requirement

**Question:**
A Cloud Run service stores user session data in the container's local memory. As traffic increases and multiple container instances spin up, users start experiencing lost sessions. What is the root cause and the correct fix?

**Options:**
- A) Cloud Run instances run out of memory; fix by increasing the memory allocation per instance
- B) Each container instance has its own isolated memory; subsequent requests landing on different instances lose session context — fix by externalizing session state to Memorystore or Firestore
- C) Cloud Run containers restart every 15 minutes by default, clearing in-memory state; fix by increasing the maximum instance lifetime
- D) The session data violates Cloud Run's data residency requirements; fix by storing sessions in a Cloud Storage bucket in the same region

**Answer:** **B) Each container instance has its own isolated memory; subsequent requests landing on different instances lose session context — fix by externalizing session state to Memorystore or Firestore**

**Explanation:**
Cloud Run runs **stateless containers** — it can spin up multiple instances and scale to zero. If session data is stored in the memory of one instance:
- A subsequent request from the same user landing on a **different instance** finds no session → broken user experience
- When an instance **scales to zero**, all in-memory session data is lost

**Fix:** Move session state to an **external, shared storage system**:
- **Memorystore (Redis)** — fast in-memory cache shared across all Cloud Run instances
- **Firestore** — persistent session storage with real-time capabilities
- **Cloud SQL** — relational database for session data

**Why not the others:**
- A) Memory errors would produce OOM crashes, not lost sessions — the symptom is consistent request routing failure, which points to state being instance-local, not a memory size problem
- C) Cloud Run does not have a fixed 15-minute restart cycle — instances are kept alive as long as needed; the issue is multi-instance statefulness, not periodic restarts
- D) In-memory data doesn't violate residency requirements (it stays in the instance) — the problem is architectural statefulness, not data sovereignty

---

### Q4: Deployment Options Comparison

**Question:**
A Python developer wants to deploy a Flask REST API to Cloud Run. They are not familiar with Docker and have never written a Dockerfile. What is the easiest path to deployment?

**Options:**
- A) They must learn Docker first and write a Dockerfile — there's no alternative on Cloud Run
- B) They can use `gcloud run deploy --source .` — Buildpacks will automatically detect Python and build the container
- C) They must use GKE instead, which supports raw Python files
- D) They should deploy to App Engine, which is the only GCP service supporting source-code deployment

**Answer:** **B) They can use `gcloud run deploy --source .` — Buildpacks will automatically detect Python and build the container**

**Explanation:**
Cloud Run supports **source-based deployment** using **Buildpacks**:
1. Developer runs `gcloud run deploy --source .` from their project directory
2. Cloud Build detects `requirements.txt` or `setup.py` → identifies as Python app
3. Buildpacks installs dependencies, sets up the Python runtime, packages the app
4. Container image is pushed to Artifact Registry automatically
5. Cloud Run deploys the image

The developer never touches a Dockerfile. This lowers the barrier to containerized deployment significantly.

**Why not the others:**
- A) A Dockerfile is not required — Cloud Run explicitly supports source-based deployment via Buildpacks; no Docker knowledge needed for common runtimes like Python, Node.js, and Java
- C) GKE also requires containers — it doesn't support raw Python files any more than Cloud Run; you would still need a container image (and thus a Dockerfile) to deploy to GKE
- D) App Engine also supports source-code deployment for Python, but it's not the only GCP service that does — Cloud Run's Buildpacks feature achieves the same result with full Cloud Run flexibility

---

### Q5: Cold Start Mitigation

**Question:**
A critical customer-facing API on Cloud Run must respond in < 100ms for all requests, including the very first request after periods of inactivity. Currently, cold starts take 800ms. What configuration change should the team make?

**Options:**
- A) Switch to GKE — Cloud Run cannot meet low-latency SLAs
- B) Set a minimum number of instances (min-instances = 1 or more) so at least one instance always stays warm
- C) Increase the container memory allocation — more memory reduces cold start time
- D) Use multi-region Cloud Run to distribute cold starts across regions

**Answer:** **B) Set a minimum number of instances (min-instances = 1 or more) so at least one instance always stays warm**

**Explanation:**
Cloud Run's **minimum instances** setting keeps N container instances running at all times, even with zero traffic. The first request always hits a warm instance → no cold start.

**Cost tradeoff:** Minimum instances incur a small charge for idle time (at a reduced idle rate). This is usually much cheaper than running a VM 24/7, while eliminating cold starts.

**Additional cold-start reduction tips:**
- Keep container images small (fewer layers to load)
- Move heavy initialization outside the request handler (top-level code executed once at startup)
- Use the language's built-in connection pooling for databases

**Why not the others:**
- A) Switching to GKE would require managing node pools and does not automatically solve cold start — GKE nodes stay running but idle pods can still have startup delays; the correct solution is Cloud Run configuration
- C) More memory can slightly speed up cold starts but does not eliminate them — the primary solution for eliminating cold starts is minimum instances, not memory allocation
- D) Multi-region deployment reduces latency for geographically distributed users but doesn't prevent cold starts — the first request to an idle instance in any region would still experience a cold start

---

### Q6: Artifact Registry Integration

**Question:**
A Cloud Run service is deployed with a custom service account `run-sa@project.iam.gserviceaccount.com`. The deployment fails because the service account cannot pull the container image from Artifact Registry. Which IAM role is missing?

**Options:**
- A) `roles/artifactregistry.writer` on the repository
- B) `roles/storage.objectViewer` on the project
- C) `roles/artifactregistry.reader` on the Artifact Registry repository
- D) `roles/run.developer` on the Cloud Run service

**Answer:** **C) `roles/artifactregistry.reader` on the Artifact Registry repository**

**Explanation:**
The Cloud Run service uses its **service account** to pull the container image at deployment time. The service account needs:
```
roles/artifactregistry.reader
```
on the specific Artifact Registry repository (or at the project level for convenience).

**Best Practice:** Create a dedicated service account per Cloud Run service with:
- `roles/artifactregistry.reader` — to pull images
- Minimum permissions needed for the application (e.g., `roles/bigquery.dataViewer`)

**Why not the others:**
- A) `roles/artifactregistry.writer` grants the ability to push images — the service account only needs to pull images, not publish them; granting Writer would violate least privilege
- B) `roles/storage.objectViewer` on the project grants access to Cloud Storage buckets — Artifact Registry images are stored separately from Cloud Storage and require Artifact Registry-specific roles
- D) `roles/run.developer` grants the ability to manage Cloud Run services (deploy, update, delete) — it doesn't grant access to Artifact Registry for image pulling

---

## Topic 3: Cloud Functions

### Q7: Event-Driven Architecture

**Question:**
A retailer uploads product images to Cloud Storage. When a new image is uploaded, a function must automatically resize it to thumbnail dimensions and save the thumbnail to another bucket. What is the best implementation using GCP serverless services?

**Options:**
- A) A Cloud Run service that polls Cloud Storage every minute for new images
- B) A Cloud Function (Cloud Run Function) triggered by Cloud Storage object creation events
- C) A Compute Engine VM running a cron job that checks for new files every 5 minutes
- D) A Pub/Sub topic that the retailer must manually publish to after each upload

**Answer:** **B) A Cloud Function (Cloud Run Function) triggered by Cloud Storage object creation events**

**Explanation:**
Cloud Functions natively integrates with Cloud Storage via event triggers:
- When an object is created in the source bucket, Cloud Storage emits an event
- The Cloud Function is triggered automatically, receiving the object's metadata
- The function reads the image, resizes it, and writes the thumbnail to the destination bucket

This is **event-driven architecture** at its simplest:
- No polling required → no wasted compute
- No VM to manage → fully serverless
- Scales automatically to handle thousands of simultaneous uploads
- Pay only for the function invocations that actually occur

**Why not the others:**
- A) Polling Cloud Storage every minute wastes compute (running even when no uploads occur) and has up to 60-second latency before noticing a new upload — event-driven triggers are both cheaper and faster
- C) A cron-based VM solution has the same polling inefficiency, adds server management overhead, doesn't scale automatically for burst uploads, and has up to 5-minute latency
- D) Manually publishing to Pub/Sub after each upload places burden on the retailer to trigger processing themselves — it misses the automatic event detection that Cloud Storage events provide natively

---

### Q8: Cloud Functions vs. Cloud Run

**Question:**
A developer needs to process video files asynchronously where each job takes 10 minutes of CPU time. Which service is the best fit?

**Options:**
- A) Cloud Functions 1st gen — simple event-driven deployment with Pub/Sub trigger
- B) Cloud Functions 2nd gen — longer timeout than 1st gen supports async patterns
- C) Cloud Run — supports long request timeouts and Cloud Run Jobs for batch processing
- D) App Engine Standard — handles background tasks with task queues

**Answer:** **C) Cloud Run — supports long request timeouts and Cloud Run Jobs for batch processing**

**Explanation:**
Cloud Functions has a maximum execution timeout (9 minutes for 1st gen, 60 minutes for 2nd gen via Cloud Run). For reliable 10-minute video processing jobs, Cloud Run is the better architectural fit — especially using **Cloud Run Jobs** which are designed for finite, batch-style workloads without an HTTP trigger. Cloud Run also supports higher CPU/memory configurations required for video processing.

**Why not the others:**
- A) Cloud Functions 1st gen has a 9-minute maximum timeout — a 10-minute job would be killed before completion
- B) Cloud Functions 2nd gen does support up to 60 minutes, but Cloud Run Jobs are the purpose-built primitive for batch/async processing; using Cloud Run directly is cleaner and avoids function-framework overhead
- D) App Engine Standard does not support long-running CPU-intensive background jobs; it is optimized for web request handling, not batch video processing

---

### Q9: Cloud Functions Convergence

**Question:**
Cloud Functions 2nd generation is described as "built on Cloud Run infrastructure." What does this mean for a developer currently using Cloud Functions 1st gen?

**Options:**
- A) Developers must migrate to writing Dockerfiles and deploying services directly via `gcloud run deploy`
- B) The underlying infrastructure is unified with Cloud Run, but the Cloud Functions developer experience (source code deploy + trigger config) remains unchanged
- C) Cloud Functions is deprecated; all new event-driven workloads should use Cloud Run directly
- D) Cloud Functions 2nd gen requires Kubernetes knowledge because it runs on GKE under the hood

**Answer:** **B) The underlying infrastructure is unified with Cloud Run, but the Cloud Functions developer experience (source code deploy + trigger config) remains unchanged**

**Explanation:**
Cloud Functions 2nd generation is **built on top of Cloud Run infrastructure** — when you deploy a function, it's actually running as a Cloud Run service under the hood. The Cloud Functions API is now called "Cloud Run Functions."

**Practically:**
- The developer experience remains the same: deploy source code, configure trigger type, done
- You still write functions using `functions_framework` (Python), `@google-cloud/functions-framework` (Node.js), etc.
- Cloud Run and Cloud Functions share billing model, runtime options, and concurrency controls in 2nd gen
- Cloud Functions remains the right tool for: event-driven handlers, HTTP webhooks, simple integrations

**Why not the others:**
- A) No Dockerfile is required — the source-code deployment model continues to work exactly as before
- C) Cloud Functions is not deprecated — it remains fully supported; the convergence means a unified underlying platform, not removal of the higher-level abstraction
- D) Cloud Functions runs on Cloud Run which runs on GKE internally, but developers never interact with Kubernetes — it's abstracted completely

---

### Q10: Buildpacks Deep Dive

**Question:**
A Python developer runs `gcloud run deploy --source .` from a directory containing `app.py` and `requirements.txt`. No Dockerfile exists. What happens?

**Options:**
- A) The deployment fails because Cloud Run requires a Dockerfile
- B) Cloud Build uses Buildpacks to detect the Python runtime, install dependencies from `requirements.txt`, and produce a container image — no Dockerfile needed
- C) The source files are uploaded directly to Cloud Run without containerization
- D) GCP prompts the developer to choose a base image before proceeding

**Answer:** **B) Cloud Build uses Buildpacks to detect the Python runtime, install dependencies from `requirements.txt`, and produce a container image — no Dockerfile needed**

**Explanation:**
**Buildpacks** are an open-source specification (originally from Heroku, now a CNCF project) for transforming source code into production-ready container images **without a Dockerfile**.

**How `gcloud run deploy --source .` works:**
1. Cloud Build detects `requirements.txt` → identifies as Python app
2. The appropriate builder image is applied — installs runtime, installs dependencies
3. A container image is produced and pushed to Artifact Registry automatically
4. Cloud Run deploys the image

| Without Buildpacks             | With Buildpacks                |
|--------------------------------|--------------------------------|
| Write app code                 | Write app code                 |
| Write `Dockerfile`             | (no Dockerfile needed)         |
| `docker build` + `docker push` | Automatic via Cloud Build      |
| `gcloud run deploy --image`    | `gcloud run deploy --source .` |

**When to still use a Dockerfile:** Multi-stage builds, custom OS packages, complex startup sequences, or full image control.

**Why not the others:**
- A) A Dockerfile is NOT required for common runtimes — this is exactly the problem Buildpacks solve; Cloud Run explicitly supports source-based deployment
- C) Cloud Run only runs containers — source files are always packaged into a container image by Buildpacks; they are not deployed directly as raw files
- D) GCP does not prompt for a base image in source-based deployment — Buildpacks automatically detect and select the correct runtime from your source files

---

### Q11: Knative — What Cloud Run Is Built On

**Question:**
Cloud Run on Google Cloud and Cloud Run for Anthos both use the same developer model (same `gcloud run deploy` commands and YAML). What underlying technology makes this cross-environment portability possible?

**Options:**
- A) Docker — both platforms use the same container image format
- B) Kubernetes — both environments are standard GKE clusters that run the same workloads
- C) Knative — an open-source serverless platform built on Kubernetes that Cloud Run's programming model is based on
- D) Cloud Build — the build pipeline is the same across environments, making deployments portable

**Answer:** **C) Knative — an open-source serverless platform built on Kubernetes that Cloud Run's programming model is based on**

**Explanation:**
**Knative** is an open-source platform built on Kubernetes that provides abstractions for deploying and running serverless workloads. Cloud Run is built on Knative:

| Benefit                   | Detail                                                                             |
|---------------------------|------------------------------------------------------------------------------------|
| **Portability**           | Cloud Run workloads can run anywhere Knative runs — GKE, on-premises, other clouds |
| **Hybrid/multicloud**     | Same programming model across fully managed and self-managed environments          |
| **Open standard**         | Knative spec is public and vendor-neutral; not proprietary lock-in                 |
| **Consistent experience** | Same request-driven, container-based model regardless of deployment target         |

> **Exam Tip:** Cloud Run on Google Cloud = fully managed Knative. Cloud Run for Anthos = Knative on your GKE clusters.

**Why not the others:**
- A) Docker is the container image format used by both, but it's the packaging format — not the runtime/orchestration layer that makes the serverless developer model portable
- B) Cloud Run on Google Cloud is NOT a standard GKE cluster exposed to users — it's a fully managed abstraction; the portability comes from Knative, not raw Kubernetes
- D) Cloud Build is a CI/CD build service — it builds images but is not the runtime layer that enables the same developer model across environments

---

### Q12: Cloud Run Resource Limits

**Question:**
A Cloud Run service needs to load a 6 GB machine learning model into memory for inference. What are the maximum CPU and memory limits for a single Cloud Run instance, and can this workload fit?

**Options:**
- A) Max 2 vCPUs and 4 GB RAM — the 6 GB model cannot fit in a single instance
- B) Max 8 vCPUs and 16 GB RAM — the 6 GB model fits with room to spare
- C) Max 4 vCPUs and 8 GB RAM — the 6 GB model can fit, but leaving only 2 GB for the application itself
- D) Max 1 vCPU and 2 GB RAM — Cloud Run is designed for lightweight workloads only

**Answer:** **C) Max 4 vCPUs and 8 GB RAM — the 6 GB model can fit, but leaving only 2 GB for the application itself**

**Explanation:**
Cloud Run supports up to **4 vCPUs** and **8 GB of memory** per container instance.

- These limits apply to managed Cloud Run on Google Cloud
- CPU allocation can be **request-only** (CPU only during request handling) or **always-on** (CPU persists when idle, useful for background tasks)
- A 6 GB model would fit in 8 GB but leave only 2 GB for the application process overhead — tight but possible

> **Exam Tip:** Know these numbers — 4 vCPUs and 8 GB RAM. If a question describes workloads requiring more resources (large GPU models, high-memory processing), GKE or Compute Engine is the answer.

**Why not the others:**
- A) The 4 vCPU / 8 GB limit is higher than 2 vCPU / 4 GB — a 6 GB model technically fits within 8 GB
- B) The maximum is 4 vCPUs and 8 GB — 8 vCPUs and 16 GB are not available in managed Cloud Run
- D) Cloud Run is not limited to lightweight workloads — the 4 vCPU / 8 GB configuration supports meaningful compute-intensive tasks like ML inference

---

### Q13: Cloud Run Billing Granularity

**Question:**
What is the minimum billing unit for Cloud Run compute time, and what happens to billing when no requests are being processed (scale-to-zero)?

**Options:**
- A) Per second; billing pauses when the instance is idle between requests
- B) Per 100 milliseconds; billing stops completely when all instances scale to zero
- C) Per minute; a minimum of 1 minute is charged per request regardless of duration
- D) Per hour; Cloud Run rounds up to the nearest hour like traditional VMs

**Answer:** **B) Per 100 milliseconds; billing stops completely when all instances scale to zero**

**Explanation:**
Cloud Run billing granularity:

| Phase                             | Billed?     |
|-----------------------------------|-------------|
| Container startup (cold start)    | ✅ Yes       |
| Request handling                  | ✅ Yes       |
| Container shutdown                | ✅ Yes       |
| Idle (no requests, scale-to-zero) | ❌ No charge |

- Minimum unit: **100 milliseconds** (not per second, not per minute)
- A small **per-request fee** also applies per million requests
- **Scale-to-zero** = zero compute cost when there's no traffic; the primary cost advantage over always-on services

> **Exam Tip:** Distractor options will say "per second" or "per minute" — the correct unit is always **100ms**.

**Why not the others:**
- A) Per second is too coarse — Cloud Run's 100ms granularity means short-lived functions that complete in 200ms are only charged for 2 billing units, not a full second
- C) There is no minimum 1-minute charge per request — this resembles Lambda's old billing model; Cloud Run charges only for actual compute time in 100ms increments
- D) Cloud Run has no hourly billing — it is not billed like a VM; per-hour billing would eliminate the cost advantage of scale-to-zero

---

### Q14: Best Scenario for Cloud Run

**Question:**
Which of the following scenarios is the best use case for **Cloud Run** (as opposed to Cloud Run Functions)?

**Options:**
- A) A function that sends a Slack alert when a Pub/Sub message arrives
- B) A dynamic web application where users can upload and share photos
- C) A cron job that runs a database cleanup script once per day
- D) A webhook that receives a GitHub event and writes a log entry

**Answer:** **B) A dynamic web application where users can upload and share photos**

**Official Explanation:**
Correct. Cloud Run is best for **containerized applications** that handle sustained or complex HTTP traffic — like a full web app with multiple endpoints, middleware, authentication, and file handling. It runs any container with any framework, supports long-running requests, and handles concurrent requests within a single instance.

Cloud Run Functions (Cloud Functions) is better for single-purpose, event-driven handlers like the other options.

**Why not the others:**
- A) A function that sends a Slack alert from a Pub/Sub message is a simple event-driven handler — better suited to Cloud Run Functions; no need for full containerized app infrastructure or multi-endpoint routing
- C) A daily database cleanup cron job is a scheduled, short-lived task — better suited to Cloud Scheduler + Cloud Run Functions or Cloud Run Jobs; not a long-running web service
- D) A webhook that receives a GitHub event and logs it is a simple single-purpose HTTP handler — better suited to Cloud Run Functions; no need for multi-endpoint routing or persistent application state

---

### Q15: Three Correct Statements About Cloud Run Functions

**Question:**
Which **three** of the following statements about Cloud Run Functions are correct? (Select 3)

**Options:**
- A) Cloud Run Functions is a scalable, pay-as-you-go Functions-as-a-Service (FaaS) platform
- B) Cloud Run Functions can be used to extend other cloud services with custom logic
- C) Cloud Run Functions requires you to manage your own container runtime and OS
- D) Cloud Run Functions is integrated with Cloud Logging for observability
- E) Cloud Run Functions can only be triggered by HTTP requests, not events

**Answer:** **A, B, D**

**Official Explanation:**
- **A ✅** — Cloud Run Functions is a fully managed, auto-scaling FaaS platform (formerly Cloud Functions)
- **B ✅** — It can extend other GCP services with custom logic (e.g., trigger on Cloud Storage events, Pub/Sub messages, Firestore changes)
- **C ❌** — Wrong; you do NOT manage the container runtime or OS — that's the whole point of serverless
- **D ✅** — Cloud Run Functions is integrated with **Cloud Logging** for built-in observability (logs, metrics, traces)
- **E ❌** — Wrong; Cloud Run Functions supports both HTTP triggers AND event-based triggers (Pub/Sub, Cloud Storage, Firestore, etc.)

**Why not the others:**
- C) Cloud Run Functions is fully managed — Google handles the container runtime, OS patching, and infrastructure; developers only write and deploy function code
- E) Cloud Run Functions supports multiple trigger types — HTTP triggers, Pub/Sub push subscriptions, Cloud Storage object events, Firestore document changes, and more; HTTP is not the only supported trigger

---

### Q16: Why Use Cloud Run Functions?

**Question:**
What is the primary reason a developer would choose Cloud Run Functions for their workload?

**Options:**
- A) They have event-driven code and do not want to provision or manage compute infrastructure
- B) They need to run containers with custom OS configurations
- C) They want to run stateful applications that require persistent local storage
- D) They need fine-grained control over the number of running instances at all times

**Answer:** **A) They have event-driven code and do not want to provision or manage compute infrastructure**

**Official Explanation:**
Correct. Cloud Run Functions is designed for **event-driven code** — logic that should execute in response to an event (a file upload, a message in a queue, an HTTP request, a database change). The key benefit is that developers write the handler function and let Google manage everything else: provisioning, scaling, runtime, availability.

**Exam Key Point:** If a scenario describes code that responds to events and the team doesn't want to manage servers, the answer is Cloud Run Functions.

**Why not the others:**
- B) Custom OS configurations in containers describes Cloud Run, not Cloud Functions — Cloud Functions manages the runtime environment; you cannot customize the underlying OS or container
- C) Cloud Functions is stateless — local filesystem storage is ephemeral and is lost between invocations; persistent storage requires external services like Cloud Storage or Firestore
- D) Cloud Functions scales automatically and you cannot control the exact number of running instances — fine-grained instance control is available in GKE or Cloud Run (via min/max instances), not Cloud Functions
