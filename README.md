# Google Cloud (GCP) Training

© TINITIATE.COM

---

### Phase A — Cloud / Foundation (Beginner → Associate)

1. **Foundations & Account Setup**

   * Resource hierarchy: Organization → Folders → Projects → Resources.
   * Regions vs zones; multi‑region services.
   * IAM roles, bindings, service accounts, IAM Conditions.
   * Labels, tags, quota management, billing setup.
   * **Lab:** Configure default project, enable billing alerts.

2. **Landing Zone (Cloud Foundation)**

   * Enterprise resource framework (folders, policies, networking).
   * Organization Policy constraints: allowed regions, public IP control, CMEK requirement.
   * Shared VPC, DNS, Private Service Connect.
   * IaC with Terraform or Infrastructure Manager.
   * **Lab:** Scaffold folders, baseline projects, apply org policies.

3. **Identity & Governance**

   * Roles, custom roles, IAM Recommender.
   * Service accounts & Workload Identity Federation.
   * Secrets in Secret Manager; CMEK with KMS.
   * Asset Inventory; Access Context Manager.
   * **Lab:** Create service account with least privilege.

4. **Networking (VPC)**

   * Custom VPCs & subnets; firewall rules; shared VPC.
   * Cloud NAT, Cloud Router, Private Google Access.
   * Load balancers: HTTP(S), TCP/UDP proxy, internal LB.
   * Hybrid: VPN, Cloud Interconnect.
   * **Lab:** Create 2‑tier VPC topology.

5. **Compute (Compute Engine)**

   * VM families, instance templates, MIG autoscaling.
   * Shielded VMs, OS patching, workload identity.
   * **Lab:** Deploy MIG behind HTTP load balancer.

6. **Serverless (Cloud Run & Cloud Functions)**

   * Cloud Run (revisioning, concurrency, VPC access).
   * Cloud Functions 2nd Gen with Eventarc triggers.
   * Workflows orchestration patterns.
   * **Lab:** Deploy Cloud Run API + Event‑triggered function.

7. **Containers (GKE & Autopilot)**

   * GKE Standard vs Autopilot, node pools, auto‑scaling.
   * Artifact Registry; Binary Authorization policies.
   * Config Sync & GitOps.
   * **Lab:** Deploy app to Autopilot cluster.

---

### Phase B — Platform & Services

8. **App Integration (Messaging/Eventing)**

   * Pub/Sub (push/pull & ordering).
   * Eventarc for event routing.
   * Workflow patterns for event‑driven architecture.
   * **Lab:** Pub/Sub producer → Cloud Run subscriber.

9. **Storage & Databases**

   * Cloud Storage tiers & lifecycle.
   * Relational: Cloud SQL, AlloyDB, Spanner.
   * NoSQL: Bigtable, Firestore.
   * Memorystore caching.
   * **Lab:** Deploy Cloud SQL private IP + Secret Manager rotation.

10. **Data & Analytics**

    * BigQuery (slots, partitions, clustering).
    * ETL/ELT with Dataflow (Beam) or Dataproc.
    * Dataplex, Data Catalog, Dataform.
    * Looker Studio / Looker BI.
    * **Lab:** Pipe streaming data into BigQuery + BI dashboard.

11. **Observability (Cloud Operations)**

    * Monitoring dashboards, SLOs, log‑based metrics.
    * Error Reporting, Trace, Profiler.
    * Central logging to BigQuery or Storage.
    * **Lab:** Create SLO for Cloud Run + alert routing.

12. **Security & Governance**

    * Security Command Center posture.
    * Cloud Armor WAF, IDS, reCAPTCHA.
    * VPC Service Controls, Access Context Manager.
    * Secret Manager, KMS, Binary Authorization.
    * **Lab:** Build VPC SC perimeter.

13. **DevOps: CI/CD**

    * Cloud Build pipelines and triggers.
    * Cloud Deploy for progressive rollout.
    * SBOM, scanning, artifact analysis.
    * **Lab:** Build → Push → Deploy to Cloud Run across environments.

14. **FinOps (Billing & Optimization)**

    * Billing export to BigQuery, anomaly detection.
    * Sustained Use & Committed Use Discounts.
    * Chargeback using labels and tags.
    * **Lab:** Build Looker Studio cost dashboard.

---

### Phase C — Advanced Cloud Architecture

15. **Architecture & SRE**

    * DR strategies, chaos testing, runbooks.
    * Global load balancing & CDN edge.
    * SRE: SLIs, SLOs, burnout protection.
    * **Lab:** Deploy multi‑region active/passive design.

16. **AI & ML (Vertex AI & Gemini)**

    * Vertex AI training, pipelines, model registry.
    * Gemini + grounding using Search/BigQuery/vector stores.
    * Governance, guardrails, model security.
    * **Lab:** Build RAG microservice + Cloud Run endpoint.

17. **Hybrid & Multi‑Cloud (GKE Enterprise)**

    * Fleet management for clusters.
    * Config Sync policy enforcement.
    * Multi‑cloud mesh & GitOps compliance.
    * **Lab:** Register hybrid cluster + enforce policy.

---

| © TINITIATE.COM |
