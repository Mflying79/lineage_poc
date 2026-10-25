# Technical Specification & PoC Blueprint: Banking Regulatory Data Lineage Studio

## 1. Executive Summary & Objective
This document outlines the architectural specifications and implementation blueprint for the **Banking Regulatory Data Lineage Studio**, a PoC tool designed for financial institutions to manage bidirectional column/attribute-level lineage across core banking systems, regulatory data models, transformation rules, aggregations, and final reporting schedules (specifically focusing on **Liquidity Coverage Ratio - LCR** and **Large Exposures - LE**).

---

## 2. Architecture & Data Model

### 2.1 Graph Data Structure
The application models metadata as a directed property graph $G = (V, E)$:
* **Nodes ($V$):** Represent entities across 5 architectural layers:
  1. `SOURCE`: Raw ingest schemas (`treasury.positions`, `core_banking.loan_master`, etc.)
  2. `DATA_MODEL`: Conformed data warehouse models (`dm_sovereign_holdings`, `dm_facility_exposure`)
  3. `RULE`: Business logic, eligibility filters, and translation functions (`RULE_HQLA_ELIGIBILITY`, `RULE_CCF_APPLICATION`)
  4. `AGGREGATION`: Group-by and rollup summaries (`AGG_LCR_L1`, `AGG_LE_GROUP`)
  5. `REPORT`: Final regulatory disclosure cells (`RPT_LCR_01`, `RPT_LARGE_EXPOSURES`)
* **Edges ($E$):** Directed dependencies classified into:
  * `DIRECT_DERIVATION`: Direct mapping or assignment.
  * `COMPUTATION`: Mathematical calculation involving multiple attributes and formulas.
  * `FILTER`: Conditional inclusion/exclusion logic (`WHERE` clauses).
  * `JOIN_KEY`: Relational join keys connecting disparate source tables.

---

## 3. Core Features & User Interface Specifications

### 3.1 Bidirectional Lineage Navigation
* **Upstream Trace (Report $ightarrow$ Source):** Backtracking from any final regulatory schedule cell to identify all contributing source attributes and filter criteria.
* **Downstream Impact (Source $ightarrow$ Report):** Forward propagation from any raw database attribute to evaluate which derived models, rules, and regulatory reports are impacted by schema changes or data anomalies.
* **Full Pipeline View:** End-to-end DAG visualization across all 5 tiers.

### 3.2 Collapsible & Expandable Entity Nodes
* Every table/stage card can be toggled individually to minimize visual clutter.
* Global **"Expand All"** and **"Collapse All"** controls in the toolbar instantly adjust information density across the entire workspace.

### 3.3 Interactive Visuals & Light Theme Styling
* Designed with a clean corporate **Light Theme** (slate/indigo accents, crisp card containers, high contrast text).
* Dynamic SVG cubic bezier connectors with neon blue active path highlighting and automatic dimming of non-participating nodes.
* Comprehensive metadata inspection drawer displaying verbatim SQL expressions, data types, and direct links to adjacent graph nodes.

---

## 4. PoC Mock Scenarios

### Scenario A: Liquidity Coverage Ratio (LCR - HQLA Level 1)
* **Target Metric:** `RPT_LCR.HQLA_L1_Sovereign` ($SUM(market\_val\_eur 	imes (1 - haircut\_rate))$)
* **Key Attributes:** `market_val_eur`, `encumbrance_status`, `issuer_type`, `credit_rating`.

### Scenario B: Large Exposures (LE - Net Exposure)
* **Target Metric:** `RPT_LE.Total_Net_Exposure` ($SUM(drawn\_bal + (undrawn\_bal 	imes ccf) - (allocated\_val 	imes (1 - haircut)))$)
* **Key Attributes:** `drawn_bal`, `undrawn_bal`, `facility_type`, `parent_group_id`.
