# GCP IAM

## Table of Contents
1. [Prerequisites & Setup](#prerequisites--setup)
2. [IAM Mental Model & Resource Hierarchy](#iam-mental-model--resource-hierarchy)
3. [Principals (Members) & Identities](#principals-members--identities)
4. [Roles: Basic, Predefined, Custom](#roles-basic-predefined-custom)
5. [IAM Policies (Bindings) & Inheritance](#iam-policies-bindings--inheritance)
6. [IAM Conditions (ABAC)](#iam-conditions-abac)
7. [Service Accounts](#service-accounts)
8. [Impersonation & Short‑Lived Credentials](#impersonation--shortlived-credentials)
9. [Workload Identity Federation (OIDC)](#workload-identity-federation-oidc)
10. [Data‑plane vs Control‑plane Permissions](#dataplane-vs-controlplane-permissions)
11. [Organization Policies & Deny Policies](#organization-policies--deny-policies)
12. [Audit Logs, Policy Troubleshooter, Recommender](#audit-logs-policy-troubleshooter-recommender)
13. [Best Practices Checklist](#best-practices-checklist)
14. [Glossary](#glossary)

---

## Prerequisites & Setup

- A Google Cloud project where you have **Project Owner** (for learning) or appropriate admin rights.
- **gcloud CLI** installed and authenticated.

### Install & Authenticate
```bash
# Install (examples)
# macOS (Homebrew)
brew install --cask google-cloud-sdk
# Debian/Ubuntu
sudo apt-get update && sudo apt-get install google-cloud-sdk -y

# Login & set project
gcloud auth login
PROJECT_ID=<your-project-id>
gcloud config set project $PROJECT_ID

# Optional: Application Default Credentials for local SDKs/tools
gcloud auth application-default login
```
**Expected output:**
- Browser login flow, followed by CLI messages confirming authentication and active project.

---

## IAM Mental Model & Resource Hierarchy

**Mental Model**
- **Who are you?** → **Principal/Member** (user, service account, group, domain, etc.)
- **What can you do?** → **Role** (collection of permissions) assigned at a **scope**.
- **How is access decided?** → Policies are **allow‑lists**; no allow → no access. Some features (deny policies/constraints) can block.

**Resource Hierarchy**
- **Organization** → **Folder(s)** → **Project(s)** → **Resource(s)** (e.g., buckets, topics, instances).
- IAM **inherits down** the hierarchy.

**Console Steps**
- Open **IAM & Admin → Settings** to view **Organization**.
- Explore **IAM & Admin → IAM** to see **principals** and their **roles**.

**gcloud Steps**
```bash
# See current project
gcloud config get-value project
# List projects you can access
gcloud projects list --format="table(projectId,name,projectNumber)"
```
**Expected output:**
- Tables of project information.

---

## Principals (Members) & Identities

**Types**
- **user:** `user:alice@example.com`
- **serviceAccount:** `serviceAccount:sa-name@PROJECT_ID.iam.gserviceaccount.com`
- **group:** `group:datareaders@example.com`
- **domain:** `domain:example.com` (be careful; very broad)
- **allUsers/allAuthenticatedUsers:** public access (rarely appropriate)

**Console Steps**
- **IAM & Admin → IAM → Grant access**; pick a principal type and add roles.

**gcloud Steps**
```bash
# Add a role to a user at project scope
USER=alice@example.com
ROLE=roles/viewer
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=user:$USER \
  --role=$ROLE

# Remove the same binding later
gcloud projects remove-iam-policy-binding $PROJECT_ID \
  --member=user:$USER \
  --role=$ROLE
```
**Expected output:**
- Updated policy JSON printed with the new/removed binding.

---

## Roles: Basic, Predefined, Custom

**Theory**
- **Basic roles:** `roles/owner`, `roles/editor`, `roles/viewer` (too broad for production).
- **Predefined roles:** Service‑specific least‑privilege roles (e.g., `roles/storage.objectViewer`).
- **Custom roles:** Your own role definitions at **project** or **organization** level.

**Console Steps**
- **IAM & Admin → Roles**: browse predefined roles, create a **Custom role** with selected permissions.

**gcloud Steps: Create a Custom Role**
```bash
# List permissions matching storage (example)
gcloud iam permissions list --filter="storage.objects" --format="table(name,title)"

# Create a custom role at project level
ROLE_ID=objectViewerLimited
PERMS=storage.objects.get,storage.objects.list

gcloud iam roles create $ROLE_ID \
  --project $PROJECT_ID \
  --title "Object Viewer Limited" \
  --description "Read-only on GCS objects" \
  --permissions $PERMS \
  --stage GA

# Describe it
gcloud iam roles describe $ROLE_ID --project $PROJECT_ID
```
**Expected output:**
- JSON with `includedPermissions`, `stage`, and role `name`.

---

## IAM Policies (Bindings) & Inheritance

**Theory**
- An **IAM policy** is a set of **bindings**: `{role → [members...]}` at a **scope** (org/folder/project/resource).
- Children inherit parent bindings; more specific scopes can add more bindings.

**Console Steps**
- **IAM & Admin → IAM → Grant access** at **Project** or **Resource** (e.g., Storage bucket **Permissions** tab).

**gcloud Steps**
```bash
# View the project policy
gcloud projects get-iam-policy $PROJECT_ID --format=json > /tmp/policy.json

# Add a binding (Viewer to a group)
GROUP=datareaders@example.com
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=group:$GROUP \
  --role=roles/viewer
```
**Expected output:**
- Policy JSON before/after with updated `bindings`.

---

## IAM Conditions (ABAC)

**Theory**
- Conditional bindings add **expressions** using **CEL** (Common Expression Language). Examples:
  - **Resource attributes:** `resource.name`, `resource.type`
  - **Request time/IP:** `request.time`, `request.auth.claims`
  - **Principal attributes:** `principal.email`, `principal.groups`

**Use case:** Allow `Storage Object Viewer` only for objects under `gs://my-bucket/reports/`.

**Console Steps**
- Storage → Bucket → **Permissions → Grant access** → **Add condition** → **Title/Desc** → Expression:
  `resource.name.startsWith("projects/_/buckets/my-bucket/objects/reports/")`

**gcloud Steps**
```bash
BUCKET=my-bucket
GROUP=datareaders@example.com

# Add a conditional binding at the project scope limiting to one bucket path
# (Often you would bind at the bucket scope; shown here for demo.)

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=group:$GROUP \
  --role=roles/storage.objectViewer \
  --condition=expression="resource.name.startsWith('projects/_/buckets/${BUCKET}/objects/reports/')",title="LimitToReportsPrefix",description="Allow read under reports/ only"
```
**Expected output:**
- Policy JSON with a `condition` block inside the new binding.

> **Tip:** For bucket‑level binding with conditions, prefer **bucket scope** (see lab below).

---

## Service Accounts

**Theory**
- **Service accounts (SAs)** are identities for apps/compute. Prefer SAs over user keys. Avoid SA **key files** where possible; use **Workload Identity Federation** or **Managed/Compute identities**.

**Console Steps**
1. **IAM & Admin → Service Accounts → Create**: Name `analytics-reader`.
2. Grant a minimal predefined role (later we’ll do bucket‑level role).

**gcloud Steps**
```bash
SA_NAME=analytics-reader
SA_EMAIL=${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com

# Create service account
gcloud iam service-accounts create $SA_NAME \
  --display-name "Analytics Reader"

# (Avoid keys; shown here only for demos)
# gcloud iam service-accounts keys create key.json --iam-account $SA_EMAIL
```
**Expected output:**
- Confirmation of SA creation; key file produced if you ran the keys command (store securely; prefer not to use).

---

## Impersonation & Short‑Lived Credentials

**Theory**
- Humans/tools should **impersonate** SAs instead of using keys.
- Grant **Token Creator** on the SA to the human/group that will impersonate:
  `roles/iam.serviceAccountTokenCreator`.

**gcloud Steps**
```bash
USER=alice@example.com
SA_EMAIL=${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com

# Allow the user to impersonate the SA (create OAuth2 access tokens)
gcloud iam service-accounts add-iam-policy-binding $SA_EMAIL \
  --member=user:$USER \
  --role=roles/iam.serviceAccountTokenCreator

# Use impersonation for commands
gcloud auth print-access-token \
  --impersonate-service-account=$SA_EMAIL

# Example: list buckets using the SA identity
gcloud storage buckets list \
  --impersonate-service-account=$SA_EMAIL
```
**Expected output:**
- An access token printed, and a list of buckets per the SA’s permissions.

---

## Workload Identity Federation (OIDC)

**Theory**
- Federate external identities (e.g., **GitHub Actions**, **AWS**, **OIDC** providers) to **assume** a Google SA **without keys**.

**High‑level gcloud Steps (GitHub OIDC example)**
```bash
POOL_ID=github-pool
PROVIDER_ID=github
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')

# 1) Create a pool
gcloud iam workload-identity-pools create $POOL_ID \
  --location=global \
  --display-name="GitHub OIDC"

# 2) Create an OIDC provider
gcloud iam workload-identity-pools providers create-oidc $PROVIDER_ID \
  --location=global \
  --workload-identity-pool=$POOL_ID \
  --display-name="GitHub" \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository"

# 3) Allow identities in the pool to impersonate the SA (limit by attribute)
SA_EMAIL=${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
BINDING_MEMBER="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/${POOL_ID}/attribute.repository/ORG/REPO"

gcloud iam service-accounts add-iam-policy-binding $SA_EMAIL \
  --member=$BINDING_MEMBER \
  --role=roles/iam.workloadIdentityUser
```
**Expected output:**
- Pool/provider created; SA policy updated with `workloadIdentityUser` binding.

---

## Data‑plane vs Control‑plane Permissions

**Theory**
- **Control plane** (manage resources) vs **Data plane** (read/write data inside resources).
- Example (Cloud Storage): `roles/viewer` (control) **does not** grant `storage.objects.get` (data). Use `roles/storage.objectViewer` for object reads.

**gcloud Steps**
```bash
# Grant data-plane role at bucket scope (preferred)
BUCKET=my-analytics-bucket
SA_EMAIL=${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com

gcloud storage buckets add-iam-policy-binding gs://$BUCKET \
  --member=serviceAccount:$SA_EMAIL \
  --role=roles/storage.objectViewer
```
**Expected output:**
- Updated IAM on the bucket including the new binding.

---

## Organization Policies & Deny Policies

**Theory**
- **Organization Policies** (Constraints) set guardrails (e.g., disallow SA key creation, restrict regions).
- **IAM Deny policies** can **explicitly block** actions regardless of allows.

**Console Steps**
- **Organization → Policies**: Search `iam.disableServiceAccountKeyCreation` → **Enforce**.
- **Deny policies**: **IAM & Admin → Deny** (if enabled) → define deny rules.

**(Optional) gcloud Steps: Enforce SA key creation deny**
```yaml
# file: sa-key-deny.yaml
name: projects/PROJECT_NUMBER/policies/iam.disableServiceAccountKeyCreation
spec:
  rules:
  - enforce: true
```
```bash
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
sed -e "s/PROJECT_NUMBER/$PROJECT_NUMBER/" sa-key-deny.yaml > /tmp/policy.yaml

# Apply org policy (requires org/project policy admin)
gcloud org-policies set-policy /tmp/policy.yaml
```
**Expected output:**
- Policy updated; attempts to create SA keys now fail.

---

## Audit Logs, Policy Troubleshooter, Recommender

**Audit Logs**
- **Admin Activity**: Always on (no charge).
- **Data Access**: Off by default for many services; enable if needed (may have cost).

**Console Steps**
- **Logging → Logs Explorer**: query `protoPayload.authenticationInfo.principalEmail` or method names.

**gcloud Steps**
```bash
# Recent successful bucket list operations (example)
gcloud logging read \
  "resource.type=gcs_bucket AND protoPayload.methodName=storage.buckets.list" \
  --limit=10 --format=json
```
**Policy Troubleshooter**
```bash
# Why can/can't a principal access a resource?
# Provide principal, full resource name, and permission.
PRINCIPAL=alice@example.com
PERM=storage.objects.get
FULL_RES="//storage.googleapis.com/projects/_/buckets/$BUCKET/objects/reports/sample.csv"

gcloud policy-troubleshooter iam troubleshoot \
  --principal-email=$PRINCIPAL \
  --permission=$PERM \
  --resource=$FULL_RES
```
**Expected output:**
- An explanation showing allow/deny decision path.

**Recommender (IAM)**
- In Console: **Policy Intelligence → IAM Recommender** suggests role right‑sizing.
- CLI varies by recommender and location; use console for first review.

---

## Best Practices Checklist

- [ ] Prefer **predefined** and **custom least‑privilege** roles; avoid **basic roles**.
- [ ] Grant at the **lowest viable scope** (bucket/resource > project), and use **groups**.
- [ ] Use **service accounts** for workloads; **impersonate** instead of using keys.
- [ ] If you must use keys, restrict & **rotate**, and consider **org policy** to block key creation.
- [ ] Use **IAM Conditions** for attribute/time/IP‑based restrictions.
- [ ] Use **Workload Identity Federation** for CI/CD (no long‑lived keys).
- [ ] Enable **Audit Logs**; export to **Log Analytics / BigQuery / SIEM**.
- [ ] Periodically run **Policy Troubleshooter** and review **Recommender**.
- [ ] Keep bindings tidy; avoid `allUsers`/`allAuthenticatedUsers` unless truly public.
- [ ] Document and automate IAM with infra‑as‑code (Terraform, YAML policies).

---

## Glossary
- **Principal/Member:** Identity making a request (user, group, SA, domain).
- **Role:** Collection of permissions; can be basic, predefined, or custom.
- **Binding:** A role + a list of members (optionally with a **condition**).
- **Service Account:** Identity for workloads; prefer impersonation over keys.
- **Workload Identity Federation:** Keyless federation from external IdPs (OIDC/SAML).
- **Organization Policy (Constraint):** Guardrail enforced at org/folder/project.
- **Deny Policy:** Explicitly blocks permissions regardless of allows.
- **Audit Logs:** Admin/Data access records in Cloud Logging.
