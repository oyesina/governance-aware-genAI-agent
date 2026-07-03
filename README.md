# Intelligent Data Governance with GenAI & Dataplex

This repository demonstrates how to build **Governance-Aware GenAI Agents** that utilize Google Cloud **Dataplex Knowledge Catalog** metadata as a strict context boundary.

Instead of relying on broad table searches or potentially hallucinated schema names, the agent in this project follows a deterministic **3-Phase Reasoning Loop** to discover, search, and verify data assets based on official governance "Aspects."

---

## 🏗 Architecture & Core Concepts

The project implements a "Governance-First" retrieval pattern. The agent doesn't just "find data"; it **discovers the rules** before it **searches for the data**.

### 1. The Reasoning Loop
1.  **Metadata Discovery:** The agent queries the Dataplex Aspect Type definition (`official-data-product-spec`) to understand the available governance fields (Enums, Booleans, Strings).
2.  **Strict Search Execution:** The agent maps natural language (e.g., "Certified Finance data") to strict Dataplex search syntax (e.g., `is_certified=true data_domain=FINANCE`).
3.  **Verification:** Before recommending a table, the agent performs a lookup to ensure the specific asset's metadata actually satisfies the policy.

### 2. Implementation Tiers
*   **Local Tier:** Prototyping using the **Gemini CLI** and `GEMINI.md` system instructions.
*   **Production Tier:** An enterprise-grade agent built with **Google's Agent Development Kit (ADK)** and the **Model Context Protocol (MCP)**, deployable to Cloud Run.

---

## 📁 Repository Structure

```text
├── terraform/                # Infrastructure as Code (BQ + Dataplex)
├── aspect_payloads/          # Generated YAML metadata for Dataplex
├── mcp_server/               # Production agent (ADK, Python, MCP)
├── apply_governance.sh       # Script to attach metadata to tables
├── generate_payloads.sh      # Script to generate YAML from schema
├── GEMINI.md                 # System instructions for local prototyping
└── README.md                 # You are here
```

---

## 🚀 Getting Started

### 1. Provision Infrastructure
Deploy the BigQuery data lake and the Dataplex Aspect Type.

```bash
cd terraform
terraform init
terraform apply -var="project_id=$(gcloud config get-value project)" -var="region=us-central1"
```

### 2. Generate and Apply Governance
Attach the governance "stamps" (Aspects) to the BigQuery tables.

```bash
# Generate the YAML payloads dynamically
./generate_payloads.sh

# Apply them to Dataplex entries
./apply_governance.sh
```

---

## 🤖 Interacting with the Agent

### Option A: Local Prototyping (Gemini CLI)
The `GEMINI.md` file defines the agent's identity and the 3-phase algorithm.

1. Install the Gemini CLI.
2. Start the CLI in the project root.
3. Ask: *"Show me certified finance data for internal use."*

### Option B: Production Agent (MCP + ADK)
The `mcp_server/` directory contains a production-ready agent.

*   **`agent.py`**: Uses a `SequentialAgent` workflow (Researcher -> Formatter).
*   **`tools.yaml`**: Exposes Dataplex tools via the GenAI Toolbox.

**Key Tools Used:**
*   `search_aspect_types`: To learn the governance schema.
*   `search_entries`: To find assets using aspect-based filtering.
*   `lookup_entry`: To verify specific asset details.

---

## 🛡 Governance Scenarios (The Test Suite)

The project includes four distinct scenarios to test the agent's compliance:

| Scenario | Dataset.Table | Tier | Certified? |
| :--- | :--- | :--- | :--- |
| **Finance Internal** | `finance_mart.fin_monthly_closing_internal` | GOLD | ✅ Yes |
| **Finance Public** | `finance_mart.fin_quarterly_public_report` | GOLD | ✅ Yes |
| **Marketing Realtime** | `marketing_prod.mkt_realtime_campaign_performance` | SILVER | ✅ Yes |
| **Analyst Sandbox** | `analyst_sandbox.tmp_data_dump_v2_final_real` | BRONZE | ❌ No |

**The "Trap":** If you ask for "Certified Logistics data," the agent should find the `tmp_data_dump` table, see `is_certified=false`, and **refuse** to recommend it.

---

## 🛠 Prerequisites
*   Google Cloud Project with Billing Enabled.
*   Dataplex, BigQuery, and Cloud Run APIs enabled.
*   Terraform installed.
*   `gcloud` SDK configured.
