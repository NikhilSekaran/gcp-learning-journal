# Lab 3: Getting Started with Cloud Storage and Cloud SQL

## Lab Info

| Field | Detail |
|---|---|
| **Course** | Google Cloud Fundamentals: Core Infrastructure |
| **Official Title** | Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL |
| **Session** | GET_CERTIFIED-06 · Day 2 (2026.04.09) |
| **Lab Type** | Hands-on (Introductory) |
| **Duration** | 50 minutes · 1 Credit |
| **Related Module** | Module 4: Cloud Storage & Databases |

---

## 🎯 Objective

Build a three-tier web application on GCP by:
1. Provisioning an **Apache + PHP** Compute Engine VM using a startup script
2. Creating a **Cloud Storage bucket** (multi-region), uploading a public image with `gcloud storage`
3. Creating a **Cloud SQL (MySQL)** instance, adding a user account, and authorizing the VM's external IP
4. Connecting the PHP app on the VM to the Cloud SQL database and embedding the Cloud Storage image

---

## 🧠 Concepts Covered

- Compute Engine VM provisioning with a startup script
- Cloud Storage bucket creation via `gcloud storage` CLI
- Making a Cloud Storage object publicly readable with `gsutil acl`
- Cloud SQL (MySQL) instance creation, user management, authorized networks
- Connecting Compute Engine to Cloud SQL via public IP + authorized network
- Editing a PHP app on a VM via nano/SSH to wire up database and image URLs
- Progressive debugging: testing a broken DB connection first, then fixing it

---

## 🔧 Lab Steps

### Task 1: Sign In to the Google Cloud Console

Use the **lab-provided credentials** from the Lab Details panel — do NOT use your personal account.

---

### Task 2: Deploy the Web Server VM (`bloghost`)

1. Navigation Menu → **Compute Engine → VM instances**
2. Click **Create instance**

| Setting | Value |
|---|---|
| **Name** | `bloghost` |
| **Region** | Lab-assigned Region |
| **Zone** | Lab-assigned Zone |
| **Machine type** | `e2-standard-2` |
| **Boot disk image** | Debian GNU/Linux 12 (bookworm) |

3. Click **Networking** → In **Firewall**, check **Allow HTTP traffic**
4. Click **Advanced** → In **Automation → Startup script**, paste:

```bash
#!/bin/bash
apt-get install apache2 php php-mysql -y
service apache2 restart
```

5. Click **Create**

> Note: The VM takes ~2 minutes to launch. Note the **internal** and **external IP** addresses — you'll need both later.

---

### Task 3: Create a Cloud Storage Bucket with `gcloud storage`

Open **Cloud Shell** (top-right toolbar → Activate Cloud Shell).

**Set your location variable** (choose the multi-region closest to your lab region):

```bash
export LOCATION=US
# or: export LOCATION=EU
# or: export LOCATION=ASIA
```

**Create the bucket** (named after your Project ID, which is globally unique):

```bash
gcloud storage buckets create -l $LOCATION gs://$DEVSHELL_PROJECT_ID
```

**Download the banner image** from Google's public training bucket:

```bash
gcloud storage cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
```

**Upload the image to your bucket:**

```bash
gcloud storage cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

**Make the object publicly readable:**

```bash
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```

> **Why `gsutil acl` not `gcloud storage`?** The `gsutil acl ch` command sets fine-grained ACL (Access Control List) on individual objects — this is the way to make a single object publicly readable when the bucket uses fine-grained access control.

---

### Task 4: Create the Cloud SQL Instance (`blog-db`)

1. Navigation Menu → **Cloud SQL**
2. Scroll to bottom → click **Create instance**
3. For database engine, select **MySQL**
4. For edition, click **Enterprise** → preset: **Sandbox**

| Setting | Value |
|---|---|
| **Instance ID** | `blog-db` |
| **Password** | `Passw0rd1!` |
| **Region** | Lab-assigned Region (same as bloghost VM) |
| **Zonal availability** | Single zone |
| **Primary zone** | Lab-assigned Zone (same as bloghost VM) |

5. Expand **Show configuration options → Security** → enable **Allow unencrypted network traffic**

> Note: This lab doesn't use SSL. In production, always use SSL/TLS for database connections.

6. Click **Create instance** (takes several minutes)

#### Configure User Account

1. From the instance details page, copy the **Public IP address** to a text editor
2. Left pane → **Users** → **Add user account**

| Field | Value |
|---|---|
| **User name** | `blogdbuser` |
| **Password** | `Passw0rd1!` |

3. Click **Add**

#### Authorize the VM's External IP

1. Left pane → **Connections** → **Networking tab**
2. Click **Add a network**

| Field | Value |
|---|---|
| **Name** | `web front end` |
| **Network** | `<bloghost external IP>/32` (e.g., `35.192.208.2/32`) |

> Use the **external IP** of the `bloghost` VM + `/32` (single host CIDR). Do NOT use the internal IP.

3. Click **Done** → **Save**

---

### Task 5: Connect the Web Server to Cloud SQL

SSH into `bloghost`:

1. Navigation Menu → **Compute Engine → VM instances**
2. Click **SSH** next to `bloghost` → click **Authorize** if prompted

Navigate to the web server document root:

```bash
cd /var/www/html
```

Create `index.php`:

```bash
sudo nano index.php
```

Paste this content (leave `CLOUDSQLIP` and `DBPASSWORD` as placeholders for now):

```php
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<?php
 $dbserver = "CLOUDSQLIP";
$dbuser = "blogdbuser";
$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store
//  it in a Secret Manager.

 try {
  $conn = new PDO("mysql:host=$dbserver;dbname=mysql", $dbuser, $dbpassword);
  $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
  echo "Connected successfully";
} catch(PDOException $e) {
  echo "Database connection failed:: " . $e->getMessage();
}
?>
</body></html>
```

Save: `Ctrl+O` → Enter → `Ctrl+X`

Restart Apache:

```bash
sudo service apache2 restart
```

**Test (expect failure):** Open `http://<bloghost external IP>/index.php` in a browser.  
You should see: _"Database connection failed: ..."_ — expected, because placeholders are still in place.

**Fix the placeholders:**

```bash
sudo nano index.php
```

- Replace `CLOUDSQLIP` with the **Cloud SQL Public IP** you copied earlier
- Replace `DBPASSWORD` with `Passw0rd1!`

Save and exit, then restart Apache:

```bash
sudo service apache2 restart
```

Reload the browser tab → you should now see: **"Connected successfully"**

---

### Task 6: Embed the Cloud Storage Image in the Web App

1. Navigation Menu → **Cloud Storage → Buckets**
2. Click the bucket named after your Project ID
3. Find `my-excellent-blog.png` → copy its **Public link URL** (looks like `https://storage.googleapis.com/PROJECT_ID/my-excellent-blog.png`)

Back in the SSH session on `bloghost`:

```bash
cd /var/www/html
sudo nano index.php
```

Locate the `<h1>` line and insert **above it** (on a new line):

```html
<img src='https://storage.googleapis.com/YOUR_PROJECT_ID/my-excellent-blog.png'>
```

The file should now have this order:
```html
<img src='https://storage.googleapis.com/...'>
<h1>Welcome to my excellent blog</h1>
```

Save, exit, restart Apache:

```bash
sudo service apache2 restart
```

Reload the browser → the page now shows the **banner image** above the heading.

---

## 💡 Key Takeaways

| Concept | Insight |
|---|---|
| **Startup scripts** | Automate software install at VM boot; eliminates manual setup post-launch |
| **Bucket name = Project ID** | Project IDs are globally unique → use them for guaranteed unique bucket names |
| **`gsutil acl ch -u allUsers:R`** | Makes a single object publicly readable (fine-grained ACL on the object) |
| **Cloud SQL authorized networks** | External IP + `/32` CIDR allows only that one VM to connect; security boundary |
| **Same region/zone for VM + DB** | Co-locating VM and Cloud SQL in the same zone minimises latency |
| **Public IP connection** | Cloud SQL exposed via public IP to the authorized VM external IP; production should use Cloud SQL Auth Proxy or Private IP |
| **Placeholder workflow** | Lab deliberately shows a broken state first, then fixing it — reinforces understanding of what each config change does |
| **Cloud Storage as static host** | Images served directly from Cloud Storage via HTTPS — no web server needed for static files |

---

## 🔬 Architecture Diagram

```
Internet
   │
   ▼
[bloghost VM]  ←── startup script: apache2 + php + php-mysql
   │
   ├── PHP connects to ──► [Cloud SQL: blog-db (MySQL)]
   │                          authorized network: bloghost external IP/32
   │
   └── HTML img src ──────► [Cloud Storage bucket]
                               public object: my-excellent-blog.png
```

---

## 📌 Certification Relevance

- **ACE Exam:** Cloud SQL authorized networks (external IP + CIDR) is a well-tested configuration detail
- Know the difference between Cloud SQL Auth Proxy (recommended, production) vs. authorized networks (simple, lab/dev)
- `gsutil acl ch -u allUsers:R` = fine-grained public access on a single object (vs. IAM-based uniform bucket access)
- Startup scripts on Compute Engine = key automation pattern; stored in instance metadata
- Co-locating database and VM in the same zone is a performance best practice
- Storing credentials in code (`index.php`) is flagged as insecure; production uses **Secret Manager** — this is called out directly in the lab code comments

