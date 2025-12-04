# GCP — Overview

## Table of Contents
1. [What GCP Is](#1-what-gcp-is)  
2. [Global Anatomy & Resource Hierarchy](#2-global-anatomy--resource-hierarchy)  
3. [IAM (Projects & Roles)](#3-iam-projects--roles)  
4. [Networking Basics (VPC)](#4-networking-basics-vpc)  
5. [Compute Options](#5-compute-options)  
6. [Storage Building Blocks](#6-storage-building-blocks)  
7. [Databases & Analytics (Essentials)](#7-databases--analytics-essentials)  
8. [Messaging & Eventing](#8-messaging--eventing)  
9. [Observability & Operations](#9-observability--operations)  
10. [Security & Shared Responsibility](#10-security--shared-responsibility)  
11. [Cost Management](#11-cost-management)  
12. [Naming, Labels & Environments](#12-naming-labels--environments)

---

## 1) What GCP Is
Google Cloud Platform provides **global-scale** infrastructure with strong serverless and data analytics primitives (e.g., **Cloud Run**, **BigQuery**).

---

## 2) Global Anatomy & Resource Hierarchy
- **Organization** → top container tied to your domain.
- **Folders** (optional) → group projects by dept/env.
- **Project** → **billing & isolation** unit; resources live here.
- **Region** & **Zones** for placement.  
- GCP **VPC is global**; **subnets are regional**.

---

## 3) IAM (Projects & Roles)
- **Principals**: users, service accounts, groups.
- **Roles**: basic (Owner/Editor/Viewer), **predefined**, and custom. Prefer **predefined**.
- Bind roles at **Org/Folder/Project/Resource** scopes; embrace **least privilege**.
- Use **Workload Identity Federation** to avoid long-lived keys.

---

## 4) Networking Basics (VPC)
- **VPC (global)** with **regional subnets**. Plan CIDR; enable **Private Google Access**.
- **Firewall rules** at VPC level; **Cloud NAT** for private egress; **Cloud Router** for hybrid.
- **Private Service Connect** to access Google/partner services privately.
- **Load Balancing** is **global** for HTTP(S) (anycast).

---

## 5) Compute Options
- **Compute Engine** (VMs) with **Managed Instance Groups** (autoscale).
- **GKE** (Kubernetes), **Cloud Run** (serverless containers), **Cloud Functions** (2nd gen on Cloud Run).
- Prefer **Cloud Run** for stateless web/APIs; scale-to-zero, fast deploys.

---

## 6) Storage Building Blocks
- **Cloud Storage** (object): classes **Standard/Nearline/Coldline/Archive**; lifecycle rules.
- **Persistent Disk** (block) for VMs; **Filestore** (file).
- Encryption at rest; CMEK via **Cloud KMS** if needed.

---

## 7) Databases & Analytics (Essentials)
- **Cloud SQL** (MySQL/Postgres/SQL Server), **AlloyDB** (Postgres-compatible, high perf).
- **Spanner** (global relational), **Firestore** (NoSQL doc), **Bigtable** (wide-column).
- **BigQuery** (serverless DW) + **Dataflow** (Apache Beam) for ETL.

---

## 8) Messaging & Eventing
- **Pub/Sub** (topics/subscriptions), **Eventarc** (event routing), **Cloud Tasks** (async jobs).

---

## 9) Observability & Operations
- **Cloud Logging & Monitoring** (Operations Suite), **Cloud Trace/Profiler**, **Error Reporting**.
- **Audit Logs** (Admin, Data Access) per service/project.

---

## 10) Security & Shared Responsibility
- **Cloud KMS**, **Secret Manager**, **Security Command Center**, **VPC Service Controls** (data exfiltration guard).
- Your side: IAM, network design, data protection, app security.

---

## 11) Cost Management
- **Budgets & alerts**, **Cost table** and **Reports**.
- **Committed Use Discounts** (CUDs), **Sustained Use Discounts** (automatic).
- Recommenders for rightsizing and idle resource cleanup.

---

## 12) Naming, Labels & Environments
- Project IDs are globally unique: plan `tinitiate-dev-core-001`.
- **Labels**: `env`, `owner`, `cost_center`, `app`, `data_class`, `lifecycle`.
- Separate `dev`/`stg`/`prod` by **Projects** (and often **Folders**).
