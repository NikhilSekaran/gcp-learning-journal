# Lab 4: Hello Cloud Run

## Lab Info

| Field | Detail |
|---|---|
| **Course** | Google Cloud Fundamentals: Core Infrastructure |
| **Official Title** | Hello Cloud Run |
| **Session** | GET_CERTIFIED-06 · Day 2 (2026.04.09) |
| **Lab Type** | Hands-on (Introductory) |
| **Duration** | 1 hour · 1 Credit |
| **Related Module** | Module 6: Serverless Applications |

---

## 🎯 Objective

Build a simple Node.js Express application, containerize it, push the image to **Artifact Registry**, deploy it to **Cloud Run**, and clean up the resources. This covers the full container-to-serverless deployment lifecycle on GCP.

---

## 🧠 Concepts Covered

- Enabling APIs with `gcloud services enable`
- Writing a containerized Node.js application
- Creating an Artifact Registry Docker repository
- Building and pushing container images with **Cloud Build** (no local Docker daemon needed)
- Locally testing a container from Artifact Registry via Cloud Shell port preview
- Deploying a public Cloud Run service with `gcloud run deploy`
- Cloud Run auto-scaling and pay-per-request billing model
- Cleaning up: deleting container images and Cloud Run services

---

## 🔧 Lab Steps

### Task 1: Enable APIs and Configure the Shell Environment

Open **Cloud Shell** (top-right toolbar → Activate Cloud Shell).

Enable the required APIs:

```bash
gcloud services enable run.googleapis.com artifactregistry.googleapis.com
```

Set the compute region:

```bash
gcloud config set compute/region "REGION"
```

Create a `LOCATION` environment variable (used in later commands):

```bash
LOCATION="REGION"
```

---

### Task 2: Write the Sample Node.js Application

Create and enter a new directory:

```bash
mkdir helloworld && cd helloworld
```

Create `package.json`:

```bash
nano package.json
```

Paste:

```json
{
  "name": "helloworld",
  "description": "Simple hello world sample in Node",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "Google LLC",
  "license": "Apache-2.0",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

Save: `Ctrl+X` → `Y` → Enter

Create `index.js`:

```bash
nano index.js
```

Paste:

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.get('/', (req, res) => {
  const name = process.env.NAME || 'World';
  res.send(`Hello ${name}!`);
});

app.listen(port, () => {
  console.log(`helloworld: listening on port ${port}`);
});
```

Save: `Ctrl+X` → `Y` → Enter

> **How it works:** The app reads the `PORT` environment variable (Cloud Run sets this automatically) and the `NAME` variable (defaults to "World"). This pattern — using environment variables for config — is the Cloud Run best practice.

---

### Task 3: Create an Artifact Registry Docker Repository

```bash
gcloud artifacts repositories create my-repository \
    --repository-format=docker \
    --location=$LOCATION \
    --description="Docker repository"
```

Configure Docker to authenticate with Artifact Registry in your region:

```bash
gcloud auth configure-docker $LOCATION-docker.pkg.dev
```

---

### Task 4: Containerize the App and Upload to Artifact Registry

Create a `Dockerfile` in the `helloworld` directory:

```bash
nano Dockerfile
```

Paste:

```dockerfile
# Use the official lightweight Node.js 20 image.
FROM node:20-slim

# Create and change to the app directory.
WORKDIR /usr/src/app

# Copy package manifests first (layer caching: avoids re-running npm install on every code change)
COPY package*.json ./

# Install production dependencies only.
RUN npm install --only=production

# Copy local code to the container image.
COPY . ./

# Run the web service on container startup.
CMD [ "npm", "start" ]
```

Save: `Ctrl+X` → `Y` → Enter

**Build and push to Artifact Registry using Cloud Build:**

```bash
gcloud builds submit --tag $LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/my-repository/helloworld
```

> **Cloud Build vs. local Docker:** `gcloud builds submit` sends source code to Cloud Build, which builds the image in the cloud and pushes it to Artifact Registry — no local Docker installation needed. On success you see a `SUCCESS` message with the full image path.

**Test the container locally from Cloud Shell:**

```bash
docker run -d -p 8080:8080 $LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/my-repository/helloworld
```

In Cloud Shell, click **Web preview → Preview on port 8080** → browser shows "Hello World!".

Alternatively: `curl localhost:8080`

---

### Task 5: Deploy to Cloud Run

```bash
gcloud run deploy helloworld \
    --image $LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/my-repository/helloworld \
    --allow-unauthenticated \
    --region=$LOCATION
```

**Flag meanings:**

| Flag | Effect |
|---|---|
| `--image` | Container image path in Artifact Registry |
| `--allow-unauthenticated` | Makes the service publicly accessible (no auth token required) |
| `--region` | GCP region where the Cloud Run service runs |

If prompted to enable additional APIs, type `Y`.

On success, the output includes the **Service URL**:

```
Service [helloworld] revision [helloworld-00001-xit] has been deployed
and is serving 100 percent of traffic.

Service URL: https://helloworld-h6cp412q3a-uc.a.run.app
```

Open the URL in a browser → "Hello World!"

> You can also view the service in the Cloud Console: Navigation Menu → **Cloud Run**.

---

### Task 6: Clean Up

Cloud Run doesn't charge when idle, but **Artifact Registry storage** has ongoing costs.

Delete the container image:

```bash
gcloud artifacts docker images delete \
    $LOCATION-docker.pkg.dev/$GOOGLE_CLOUD_PROJECT/my-repository/helloworld
```

Type `Y` to confirm.

Delete the Cloud Run service:

```bash
gcloud run services delete helloworld --region="REGION"
```

Type `Y` to confirm.

---

## 💡 Key Takeaways

| Concept | Insight |
|---|---|
| **Cloud Run = serverless containers** | You provide the container; GCP handles all infrastructure, scaling, and routing |
| **`PORT` environment variable** | Cloud Run injects the `PORT` env var; your app must listen on it (not hardcoded) |
| **Artifact Registry** | GCP's managed container/artifact registry; replaces deprecated Container Registry |
| **Cloud Build** | Builds container images in the cloud without a local Docker daemon (`gcloud builds submit`) |
| **`--allow-unauthenticated`** | Makes the service public; without this flag, requests require a Bearer token |
| **Auto-scaling** | Cloud Run scales to zero when idle, then scales horizontally under load — automatically |
| **Billing** | Pay only for CPU, memory, and networking during request handling (billed per 100ms) |
| **Layer caching** | Copying `package*.json` before `COPY . ./` means Docker reuses the npm install layer unless dependencies change |
| **Clean up** | Deleting the Cloud Run service stops billing for compute; delete Artifact Registry images to stop storage billing |

---

## 🏗️ Architecture of What Was Built

```
Developer (Cloud Shell)
   │
   ├── gcloud builds submit ──► Cloud Build ──► Artifact Registry
   │                                        (my-repository/helloworld image)
   │
   └── gcloud run deploy ──────────────────► Cloud Run Service
                                              URL: https://helloworld-xxxxx-uc.a.run.app
                                              Pulls image from Artifact Registry
                                              Scales to 0 when idle
```

---

## 📌 Certification Relevance

- **ACE Exam:** Know that Cloud Run requires a container image stored in **Artifact Registry** (or Container Registry)
- `--allow-unauthenticated` = public service; without it, requests need Identity-Aware Proxy or audience-bound tokens
- Cloud Run listens on the port specified by the `PORT` environment variable — hardcoding a port is a common mistake
- `gcloud builds submit --tag` = one command to build + push; Cloud Build is the underlying service
- Cloud Run **revision** model: each deploy creates a new revision; traffic can be split between revisions
- Billing stops when no requests are being processed (scales to zero) — key differentiator vs. Compute Engine
- Know when to use Cloud Run vs. Cloud Run Functions: Cloud Run = containerised app; Cloud Run Functions = event-driven single-function code

