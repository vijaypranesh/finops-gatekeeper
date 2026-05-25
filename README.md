# FinOps Gatekeeper

An autonomous, hard-gate CLI tool designed to run in CI/CD pipelines to enforce strict FinOps budget limits on Terraform infrastructure deployments before they are applied.

## How it was Built with Antigravity

This entire tool was iteratively generated and orchestrated by **Antigravity**, an advanced agentic coding assistant. 

Throughout the session, Antigravity utilized the following agentic skills:
- **Architectural Design & Pivot:** Initially scaffolded with mock data, transitioned to an Infracost architecture, and ultimately pivoted to a custom-built, native GCP Pricing Engine purely based on user feedback.
- **Code Generation & Full-Stack Implementation:** Authored the core Python logic (`gatekeeper.py`), native Jinja2 HTML templates, and Weasyprint PDF generators.
- **Terminal Execution & Orchestration:** Managed Ubuntu system dependencies, instantiated and managed Python virtual environments, installed `pip` packages, and executed live testing workflows via background terminal tasks.
- **Terraform Scaffolding:** Synthesized real test fixtures spanning multiple directories (`bookmycourt`, `GymApp`, `Infra_Lab`) containing dynamic GKE clusters and Cloud SQL databases to simulate pipeline outcomes.
- **Self-Healing:** Dynamically caught, diagnosed, and resolved dependency mismatch errors (e.g., the `pydyf` and `weasyprint` library conflict) and handled complex GCP Catalog API timeout constraints natively in code.

---

## Detailed Application Logic

The Gatekeeper acts as a standalone binary in your deployment pipeline.

### 1. Terraform Plan Generation
The tool takes a directory path as a positional argument (or via the `TERRAFORM_DIR` environment variable). It automatically executes `terraform init`, `terraform plan`, and `terraform show -json` under the hood to generate a fully resolved, variable-expanded JSON plan of the proposed infrastructure.

### 2. Native GCP Pricing Engine
The engine scans the JSON plan for supported Google Cloud resources:
- `google_compute_instance` (Compute Engine VMs)
- `google_container_node_pool` (GKE Kubernetes Clusters)
- `google_sql_database_instance` (Cloud SQL Databases)

For each resource, it extracts the exact `machine_type`, `tier`, or `node_count`. It then queries the official **Google Cloud Catalog Billing API** dynamically to resolve the exact hourly SKU rates for the requested regions and calculates the projected 730-hour monthly run rate. (It falls back to a realistic mock pricing dictionary if the massive GCP Catalog API drops the request).

### 3. Budget Evaluation
The aggregated cost ($C_{proposed}$) is extracted and compared against the maximum allowable budget limit ($B_{limit}$) defined in your local `budget.json` configuration file.

### 4. Smart Routing & PDF Report Generation
The engine uses Jinja2 and Weasyprint to compile a production-ready, beautiful PDF report containing a metadata matrix, budget progress bar, itemized offending resources breakdown, and remediation steps.

- **Pipeline CONTINUED (Under Budget):** Exits with code `0`. It automatically creates a `Successful_Deployment/` directory, drops the timestamped PDF report inside, and generates a `mock_deployment_report.log` to simulate a downstream `terraform apply` success.
- **Pipeline BLOCKED (Over Budget):** Exits with code `1` (killing the CI/CD workflow). It automatically creates a `Failed_Deployment/` directory and drops the timestamped PDF violation report inside for audit review.
