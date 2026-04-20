# Module 2: Resource Hierarchy and IAM - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module 2 — Resource Hierarchy & IAM. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---

## Topic 1: Resource Hierarchy

### Q1: Hierarchy Level Identification

**Question:**
A company wants to apply a security policy that restricts all GCP workloads to deploy VMs only in US regions. At which level of the resource hierarchy should they apply this policy for maximum effect?

**Options:**
- A) At the resource level on each VM
- B) At each project individually
- C) At the Organization level
- D) At the folder level under "Production"

**Answer:** **C) At the Organization level**

**Explanation:**
Applying the restriction at the Organization level means it automatically inherits to all folders, projects, and resources under the organization. Applying at a lower level (project or resource) would require managing the policy on each item individually — which is inefficient and error-prone. Organization Policies at the top ensure consistent enforcement across the entire company.

**Why not the others:**
- A) Resource level on each VM — requires creating and managing individual policies on potentially thousands of VMs; any new VM would need the policy applied manually
- B) Each project individually — more manageable than per-VM, but still requires updating every project; new projects would also need the policy applied each time they are created
- D) Folder level under "Production" — would only apply to resources in the Production folder; development, staging, and other folders would still allow non-US deployments

---

### Q2: Project Identifier Mutability

**Question:**
After creating a project in Google Cloud, which of the following can you change?

**Options:**
- A) Project Number
- B) Project ID
- C) Project Name
- D) Both Project Number and Project ID

**Answer:** **C) Project Name**

**Explanation:**
- **Project Number** — Assigned by Google at creation; immutable
- **Project ID** — Chosen at creation; globally unique; immutable after creation
- **Project Name** — Human-readable display name; can be changed at any time in the Console

**Why not the others:**
- A) Project Number — assigned by Google automatically at creation; neither you nor Google can change it after the project is created
- B) Project ID — globally unique and chosen during creation (or auto-generated); immutable after the project is created; used in API calls and resource URLs
- D) Both Project Number and Project ID — both are immutable; only the Project Name can be changed post-creation

---

### Q3: Policy Inheritance Scenario

**Question:**
Alice has `roles/editor` granted at the Organization level. Bob has `roles/viewer` granted at the project level for `project-alpha`. Eve has no IAM bindings. Which statement is correct?

**Options:**
- A) Alice can only edit resources in projects she's explicitly added to
- B) Alice can edit resources across all projects in the organization
- C) Bob has Editor access to `project-alpha` due to the org-level policy
- D) Eve can view all resources since Organization policies are public

**Answer:** **B) Alice can edit resources across all projects in the organization**

**Explanation:**
IAM policies at a higher level in the hierarchy apply to all resources below. Alice's Editor role at the Organization level means she inherits Editor permissions on all folders, projects, and resources under that organization. Bob's Viewer role applies only to `project-alpha`. Eve has no access anywhere.

**Why not the others:**
- A) Alice does NOT need explicit project-level bindings — org-level IAM grants automatically cascade down to all folders, projects, and resources beneath
- C) Bob has Viewer, not Editor — IAM policies are additive; Alice's org-level Editor role doesn't change Bob's project-level Viewer role; each binding is independent
- D) Eve has no IAM bindings anywhere — GCP IAM is deny-by-default; without any bindings, Eve has zero access to any resource

---

## Topic 2: IAM Roles

### Q4: Choosing the Right Role Type

**Question:**
A data engineer needs to run queries on BigQuery but should not be able to read raw data in Cloud Storage, modify VM instances, or change IAM policies. Which role type and approach best follows least privilege?

**Options:**
- A) Grant `roles/owner` at the project level
- B) Grant `roles/editor` at the project level
- C) Grant the predefined role `roles/bigquery.dataViewer` and `roles/bigquery.jobUser`
- D) Grant a custom role with `bigquery.*` permissions

**Answer:** **C) Grant the predefined role `roles/bigquery.dataViewer` and `roles/bigquery.jobUser`**

**Explanation:**
Predefined roles follow least privilege by being scoped to a specific service. `roles/bigquery.dataViewer` allows reading BigQuery data, and `roles/bigquery.jobUser` allows running query jobs. Together they provide exactly what the engineer needs — nothing more. Owner and Editor are too broad.

**Why not the others:**
- A) `roles/owner` — grants full control of all project resources including the ability to delete the project, modify billing, and change IAM policies; massively violates least privilege
- B) `roles/editor` — grants write access to most resources across all services in the project; the engineer could modify VMs, delete storage buckets, and change non-IAM configurations
- D) Custom role with `bigquery.*` — a wildcard is overly broad and includes permissions like deleting datasets and tables; custom roles should be used when predefined roles are still too permissive, not to bundle all permissions for a service

---

### Q5: Basic Roles Risk

**Question:**
Why are Basic Roles (Owner, Editor, Viewer) generally discouraged for production GCP environments?

**Options:**
- A) Basic Roles expire automatically after 90 days and must be renewed, creating an operational risk
- B) Basic Roles are project-wide and grant access across ALL GCP services, violating the principle of least privilege
- C) Basic Roles cannot be assigned to service accounts, only to human users
- D) Basic Roles require Multi-Factor Authentication which slows down CI/CD pipelines

**Answer:** **B) Basic Roles are project-wide and grant access across ALL GCP services, violating the principle of least privilege**

**Explanation:**
Basic Roles grant permissions across **all services and resources within an entire project**. This violates the principle of least privilege:

1. **Blast radius:** If an account with Editor role is compromised, an attacker can modify or access any resource in the project
2. **No service scope:** There's no way to grant "only BigQuery access" using Basic Roles — Editor gives access to Compute, Storage, Databases, etc.
3. **Poor auditability:** Broad roles make it harder to understand what a user actually needs vs. has access to

**Recommended approach:** Use **Predefined Roles** scoped to specific services, or **Custom Roles** when predefined roles are still too broad.

**Why not the others:**
- A) Basic Roles have no expiry — IAM role bindings are permanent until explicitly removed; expiry requires Conditions
- C) Basic Roles can be assigned to service accounts — assigning `roles/editor` to a service account is possible (and a common mistake that creates over-privileged services)
- D) Basic Roles have no special MFA requirement — MFA is a policy applied via Identity Platform/BeyondCorp, not tied to specific role types

---

### Q6: Role Permissions Naming

**Question:**
Which of the following represents a valid GCP IAM permission?

**Options:**
- A) `createVM`
- B) `compute:instances:create`
- C) `compute.instances.create`
- D) `gcp/compute/instances/create`

**Answer:** **C) `compute.instances.create`**

**Explanation:**
GCP IAM permissions follow the format `service.resource.verb`:
- `service` = GCP service (e.g., `compute`, `storage`, `bigquery`)
- `resource` = resource type (e.g., `instances`, `buckets`, `datasets`)
- `verb` = action (e.g., `create`, `list`, `get`, `delete`)

**Why not the others:**
- A) `createVM` — not a valid format; GCP permissions use dot notation (`service.resource.verb`), not camelCase verbs without service context
- B) `compute:instances:create` — uses colons, not dots; this resembles AWS IAM notation, not GCP
- D) `gcp/compute/instances/create` — uses slashes; not a valid GCP permission format; GCP permissions never start with `gcp/`

---

## Topic 3: Service Accounts

### Q7: Service Account Best Practice

**Question:**
A developer is building a Cloud Run service that needs to read from BigQuery. Which approach is the most secure?

**Options:**
- A) Hardcode a service account key file in the container image
- B) Store a service account key in a Cloud Storage bucket the container reads at startup
- C) Create a service account with `roles/bigquery.dataViewer`, attach it to the Cloud Run service
- D) Grant the Cloud Run service `roles/owner` for simplicity

**Answer:** **C) Create a service account with `roles/bigquery.dataViewer`, attach it to the Cloud Run service**

**Explanation:**
The correct approach is to:
1. Create a dedicated service account with only the permissions needed (least privilege)
2. Attach the service account directly to the Cloud Run service
3. Cloud Run automatically uses this identity for API calls — no key files needed

Google manages the credential rotation automatically. Storing key files in images or buckets creates security risks (key exposure, no automatic rotation). Granting `owner` violates least privilege.

**Why not the others:**
- A) Hardcoding a key file in the container image — the key is embedded in the image and accessible to anyone who can pull the image; no automatic rotation; credentials could be leaked via image registry access
- B) Storing key in Cloud Storage — the container still needs credentials to access Cloud Storage (circular dependency); the key is still not automatically rotated and adds an extra failure point at startup
- D) `roles/owner` — violates least privilege; the service only needs to read BigQuery data, not have full ownership over all project resources including deletion and billing access

---

### Q8: Service Account as Resource

**Question:**
A junior engineer has `roles/editor` on a GCP project. There is a service account `deploy-sa@project.iam.gserviceaccount.com` with `roles/owner`. The engineer tries to attach this service account to a new Compute Engine VM. What additional IAM permission is required?

**Options:**
- A) No additional permission is needed; `roles/editor` includes the ability to use any service account in the project
- B) The engineer needs `roles/iam.serviceAccountUser` on the specific service account
- C) The engineer needs `roles/owner` on the project to assign service accounts to VMs
- D) The engineer must first create a new service account key before attaching it to a VM

**Answer:** **B) The engineer needs `roles/iam.serviceAccountUser` on the specific service account**

**Explanation:**
A service account is a **dual-role entity** in GCP IAM:

**As a Principal (identity):**
- Google Cloud grants IAM roles **to** the service account (e.g., `roles/storage.objectViewer`)
- When a VM or Cloud Run service uses this service account, it inherits those permissions

**As a Resource:**
- You can control **who can use or impersonate** the service account
- Granting `roles/iam.serviceAccountUser` on a service account allows a user to attach it to compute resources
- This prevents privilege escalation: without `serviceAccountUser` on a specific SA, a user cannot deploy a VM that runs as that service account (and inherits its potentially elevated permissions)

**Why not the others:**
- A) `roles/editor` does NOT include the right to attach arbitrary service accounts to VMs — this is intentional to prevent privilege escalation via high-privilege service accounts
- C) `roles/owner` would work but is far too broad — granting Owner just to attach a service account to a VM is a massive violation of least privilege
- D) A service account key is not required to attach a SA to a VM — the VM uses the SA's identity automatically via the instance metadata server; no key file is needed

---

## Topic 4: Cloud Identity & Interaction Methods

### Q9: Groups vs. Individual Users

**Question:**
A team of 15 developers all need the same set of permissions to a GCP project. What is the recommended IAM approach?

**Options:**
- A) Grant the permissions to each developer's email address individually
- B) Create a Google Group, add developers to the group, grant the group the required IAM roles
- C) Create a custom role for each developer with their specific permissions
- D) Grant `roles/owner` to the entire team for simplicity

**Answer:** **B) Create a Google Group, add developers to the group, grant the group the required IAM roles**

**Explanation:**
Granting IAM to a **group** is best practice because:
- Adding a new developer = add them to the group (no IAM changes)
- Removing a developer = remove from group (no IAM changes)
- Permission changes = update the group binding once (not 15 individual bindings)
- Easier to audit who has access by reviewing group membership

**Why not the others:**
- A) Granting to each individual email — requires 15 separate IAM bindings; adding or removing anyone requires updating IAM directly; onboarding and offboarding is error-prone at scale
- C) Custom role per developer — overcomplicated; all developers need the same permissions, so per-person custom roles add no value and are much harder to audit and maintain
- D) `roles/owner` — violates least privilege; Owner allows deleting projects, modifying billing, and accessing all resources — far more than developers need for their work

---

### Q10: Cloud Identity vs. Google Workspace

**Question:**
A company already uses Microsoft 365 for email and collaboration but wants to manage GCP access with corporate identities (`user@company.com`). Which Google identity platform should they use?

**Options:**
- A) Google Workspace — required for all corporate GCP users because it provides the most complete identity management
- B) Cloud Identity — provides managed corporate accounts and SSO without requiring the Google collaboration apps they don't need
- C) Firebase Authentication — the standard GCP identity service for enterprise corporate accounts
- D) Google Workspace is mandatory; GCP cannot integrate with Microsoft 365 user accounts

**Answer:** **B) Cloud Identity — provides managed corporate accounts and SSO without requiring the Google collaboration apps they don't need**

**Explanation:**
Both **Cloud Identity** and **Google Workspace** are Google's identity management platforms that let organizations manage corporate accounts. The key differences:

| Feature                      | Cloud Identity      | Google Workspace      |
|------------------------------|---------------------|-----------------------|
| User account management      | ✅ Yes               | ✅ Yes                 |
| SSO and MFA                  | ✅ Yes               | ✅ Yes                 |
| Active Directory / LDAP sync | ✅ Yes (via GCDS)    | ✅ Yes                 |
| Gmail, Drive, Meet, etc.     | ❌ No                | ✅ Yes                 |
| Cost                         | Free tier available | Per-user subscription |

Since the company already has Microsoft 365, they don't need Google's collaboration apps. Cloud Identity gives them GCP identity management without paying for productivity apps they won't use.

**Why not the others:**
- A) Google Workspace is not required for GCP — Cloud Identity provides all IAM-related identity features; Google Workspace adds the productivity apps
- C) Firebase Authentication is for authenticating end users of mobile/web apps, not for managing corporate employee identities accessing GCP
- D) GCP fully supports federated identity with Microsoft Azure AD via SAML/OIDC; the company can sync Microsoft 365 identities through Cloud Identity's GCDS (Google Cloud Directory Sync)

---

### Q11: Choosing an Interaction Method

**Question:**
A DevOps engineer needs to automate the nightly provisioning of 50 Compute Engine VMs with identical configurations as part of a CI/CD pipeline. Which interaction method is most appropriate?

**Options:**
- A) Cloud Console — manually click through the UI for each VM
- B) Cloud Mobile App — send provisioning commands from a phone
- C) gcloud CLI scripts or Terraform — automate with code in the pipeline
- D) Direct REST API calls written from scratch in Python

**Answer:** **C) gcloud CLI scripts or Terraform — automate with code in the pipeline**

**Explanation:**
Automation tasks in CI/CD pipelines are best handled with scripting tools (`gcloud` CLI) or Infrastructure as Code (Terraform). These can be version-controlled, repeatable, and executed without human interaction. The Cloud Console requires manual clicking (not automatable), and writing raw REST API calls is unnecessarily complex when `gcloud` and client libraries exist.

**Why not the others:**
- A) Cloud Console — cannot be automated; manually clicking through the UI 50 times each night is not a CI/CD pipeline; human-error-prone and does not scale
- B) Cloud Mobile App — designed for monitoring and quick administrative tasks from a phone, not for scripted automation or integration into CI/CD pipelines
- D) Raw REST API calls from scratch — technically works but reinvents the wheel; `gcloud` CLI and client libraries provide tested abstractions, built-in authentication handling, and error handling

---

## Topic 5: IAM Policy Scenarios

### Q12: Least Privilege Scenario

**Question:**
A compliance auditor needs to review all IAM policies across a GCP organization but must not be able to modify any resources or change any policies. Which predefined role should be assigned at the Organization level?

**Options:**
- A) `roles/owner`
- B) `roles/editor`
- C) `roles/viewer`
- D) `roles/iam.securityReviewer`

**Answer:** **D) `roles/iam.securityReviewer`**

**Explanation:**
`roles/iam.securityReviewer` is specifically designed for auditing — it provides read access to IAM policies and security configurations across the resource hierarchy without granting access to modify data or configurations. While `roles/viewer` grants read access to most resources, `securityReviewer` is more targeted for IAM/security auditing and better aligns with least privilege for this specific use case.

> **Note:** On the ACE exam, when a scenario specifies a security audit function, look for audit or reviewer-specific predefined roles before defaulting to `viewer`.

**Why not the others:**
- A) `roles/owner` — grants full control of all project resources including deletion and billing; completely violates least privilege for a read-only auditor
- B) `roles/editor` — grants write access to most resources across all services; allows modifying data and configurations, which is inappropriate for a read-only auditor
- C) `roles/viewer` — grants broad read access to most resources but is not specifically scoped to IAM/security policy review; `securityReviewer` is more targeted and follows least privilege more closely

---

### Q13: Services and APIs Are Enabled Per...

**Question:**
Consider a single hierarchy that has an organization node, folders, and projects. If a policy applied at the project level contradicts a policy applied at the organization level, which policy takes precedence?

Choose the BEST completion of this sentence: Services and APIs are enabled on a per-_____ basis.

**Options:**
- A) Organization
- B) Project
- C) Folder
- D) Resource

**Answer:** **B) Project**

**Official Explanation:**
Correct. In Google Cloud, services and APIs are enabled or disabled at the **project** level. Each project has its own set of enabled APIs. This allows different projects to use different subsets of GCP services even within the same organization.

**Why not the others:**
- A) Organization — organization-level settings control policies and structure, but API enablement is not managed at the organization level; APIs are enabled per project
- C) Folder — folders group projects for policy and billing organization, but API/service enablement is at the project level within each folder
- D) Resource — individual resources within a project cannot independently enable APIs; the API must be enabled at the project level first

---

### Q14: IAM Role Types — Broadest to Finest-Grained

**Question:**
Which of the following correctly orders the types of IAM roles from the broadest to the finest-grained level of access?

**Options:**
- A) Custom roles, Predefined roles, Basic roles
- B) Predefined roles, Custom roles, Basic roles
- C) Basic roles, Predefined roles, Custom roles
- D) Custom roles, Basic roles, Predefined roles

**Answer:** **C) Basic roles, Predefined roles, Custom roles**

**Official Explanation:**
Correct. Basic roles (Owner, Editor, Viewer) are the broadest — they affect all resources in a project. Predefined roles are curated by Google for specific services with a limited set of permissions. Custom roles are the finest-grained — they let you define the exact set of permissions you need, down to individual API methods.

**Exam Tip:** Order for exam = **Basic → Predefined → Custom** (broadest to finest).

**Why not the others:**
- A) Custom, Predefined, Basic — this is the reverse order (finest to broadest); Custom roles are the most specific, not the broadest
- B) Predefined, Custom, Basic — incorrect; Basic roles are the broadest (project-wide across all services), not the finest
- D) Custom, Basic, Predefined — also incorrect ordering; Basic roles are always the broadest tier in the IAM role hierarchy

---

### Q15: Project ID Properties

**Question:**
Which value associated with a Google Cloud Project is:
- Globally unique
- Permanent (cannot be changed after creation)
- Modifiable **during** creation

**Options:**
- A) Project Name
- B) Project ID
- C) Project Number
- D) Project Label

**Answer:** **B) Project ID**

**Official Explanation:**
Correct. The **Project ID** is globally unique and is assigned by you (or auto-generated by GCP). You can change it during the creation wizard but once the project is created, it is **permanent and immutable**. It is used in API calls, `gcloud` commands, and resource URLs.

**Compare:**
- **Project Name**: human-readable, not globally unique, can be changed after creation
- **Project Number**: assigned by Google, globally unique and permanent, but you don't choose it
- **Project ID**: you choose it (or GCP generates one), globally unique, cannot change after creation

**Why not the others:**
- A) Project Name — human-readable display name; not globally unique (two projects can have the same name); can be changed after creation
- C) Project Number — assigned automatically by Google; globally unique and permanent, but you cannot choose or modify it at any point
- D) Project Label — labels are key-value pairs attached to resources for organization and billing tracking; not a project identifier used in APIs or commands
