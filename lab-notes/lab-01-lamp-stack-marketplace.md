# Lab 1: Deploy a LAMP Stack from the Google Cloud Marketplace

## Lab Info

| Field | Detail |
|---|---|
| **Course** | Google Cloud Fundamentals: Core Infrastructure |
| **Official Title** | Google Cloud Fundamentals: Getting Started with Cloud Marketplace |
| **Session** | GET_CERTIFIED-06 · Day 1 (2026.04.08) |
| **Lab Type** | Hands-on (Introductory) |
| **Duration** | 25 minutes · 1 Credit |
| **Related Module** | Module 1: Intro to Google Cloud |

---

## 🎯 Objective

Deploy a fully configured LAMP stack on a Compute Engine instance using **Google Cloud Marketplace**. This lab demonstrates how Marketplace solutions (specifically Google Click to Deploy) handle all infrastructure and software configuration automatically via a single workflow.

---

## 🧠 Concepts Covered

- Google Cloud Marketplace navigation
- Google Click to Deploy solutions
- Infrastructure Manager API and Compute Engine API enablement
- Compute Engine VM creation via Marketplace
- LAMP stack architecture: Linux + Apache HTTP Server + MySQL + PHP + phpMyAdmin
- Verifying a running web server via the deployed site URL

---

## 🧱 LAMP Stack Components

| Component | Role |
|---|---|
| **Linux** | Operating system |
| **Apache HTTP Server** | Web server |
| **MySQL** | Relational database |
| **PHP** | Web application framework |
| **phpMyAdmin** | PHP administration tool |

---

## 🔧 Lab Steps

### Task 1: Sign In to the Google Cloud Console

1. Click **Start Lab** to provision a temporary project with credentials
2. Use the **Lab Details panel** credentials — do NOT use your personal Google account
3. Accept terms; skip recovery options and free trial prompts

> Note: Using your own account may incur charges.

### Task 2: Deploy LAMP Stack via Cloud Marketplace

1. In the Cloud Console, click the **Navigation Menu (☰)**
2. Click **Marketplace**
3. In the search bar, type **LAMP** and press Enter
4. Select **LAMP Stack, by Google Click to Deploy**
   > ⚠️ Must be the Google Click to Deploy version — other LAMP stacks will break the lab steps
5. Click **GET STARTED**
6. On the Agreements page, check **Terms and agreements** → click **AGREE**
7. On the confirmation pop-up, click **DEPLOY**
8. If prompted, click **Enable** for the **Compute Engine API** and the **Infrastructure Manager API**

**Configure the deployment:**

| Setting | Value |
|---|---|
| **Zone** | Lab-assigned zone (shown in Lab Details) |
| **Machine Type (Series)** | E2 |
| **Machine Type** | `e2-medium` |
| All other settings | Leave as defaults |

9. Click **Deploy**

> Note: Warnings may appear during deployment — these can be disregarded for this lab.

**Status progression:**  
`lamp-1 is being deployed` → `lamp-1 has been deployed`

### Task 3: Verify the Deployment

1. When deployment completes, click **lamp-1-vm → Details**
2. Click the **Site URL** link
   - If the site doesn't respond, wait 30 seconds and retry
   - If redirected, follow the redirect link
3. A new tab opens showing a **congratulations page** — this confirms Apache HTTP Server is running

> Note: If the page doesn't load on a corporate laptop, try disabling VPN or testing on another device.

---

## 💡 Key Takeaways

| Concept | Insight |
|---|---|
| **Marketplace** | Pre-configured solutions deploy in minutes with no manual installation or configuration |
| **Google Click to Deploy** | Google-maintained Marketplace template category; uses Deployment Manager under the hood |
| **Infrastructure Manager API** | Required API for Marketplace deployments — must be enabled before deploying |
| **Compute Engine API** | Required for any VM-based Marketplace solution |
| **Deployment name** | Becomes prefix for all created resources (VM named `lamp-1-vm`, firewall rules, etc.) |
| **phpMyAdmin** | Web-based MySQL admin tool included in this LAMP stack — accessible via the admin URL |
| **External IP** | VMs deployed via Marketplace get an ephemeral external IP by default |

---

## 📌 Certification Relevance

- **ACE Exam:** Know that Cloud Marketplace uses **Deployment Manager** / **Infrastructure Manager** templates to provision resources
- Marketplace solutions require specific APIs to be **enabled** before deployment (Compute Engine API, Infrastructure Manager API)
- Understanding that a Marketplace deployment creates real Compute Engine VMs with real firewall rules ties directly to Module 3 networking
- The lab reinforces that **Compute Engine is IaaS** — the VM underlying the LAMP stack is a standard Compute Engine instance you can SSH into
- `e2-medium` = 2 vCPU, 4 GB RAM — knowing machine type families is an ACE exam objective
