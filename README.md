# PO-to-Fulfillment Control Tower | Built in Palantir Foundry + AIP

> A practical, line-level control tower that stitches planning → buying → procurement → logistics → receipt into a single operational view, with an actionable Workshop App and an AIP automation agent for guided decisions.

[![Built with Palantir Foundry](https://img.shields.io/badge/built%20with-Palantir%20Foundry-001a72)](#)

---

## Overview
This project demonstrates an end-to-end **PO-to-Fulfillment Control Tower workflow** built entirely within **Palantir Foundry**, powered by **AIP Logic**.

It unifies fragmented planning and execution data—from demand forecasts and purchase orders to shipments, receipts, and allocations—into a single ontology and applies AIP reasoning to generate real-time operational recommendations.  
**PO-to-Fulfillment Control Tower** treats each *PO line* as the atomic “case” to track end-to-end. Every upstream system tells part of the story; Foundry translates them into a single language (ontology) and surfaces **actionable** insights in a **Workshop App**, while an **AIP automation agent** proposes next-best-actions based on live context and user inputs.

The goal: enable planners and sourcing managers to move from *reactive monitoring* to *proactive decision-making*.

---

## Demo
Watch the full 4-minute demo here:  
<a href="https://www.youtube.com/watch?v=44KYtZuDBko"> <img alt="Watch the Demo on YouTube" src="https://img.shields.io/badge/Watch-Demo-red?logo=youtube"> </a><br> <a href="https://www.youtube.com/watch?v=44KYtZuDBko"> <img alt="Demo Thumbnail" src="https://img.youtube.com/vi/44KYtZuDBko/hqdefault.jpg" width="320"> </a>

---

## Business Context
In complex global supply chains, every Purchase Order travels across multiple systems—ERP, WMS, freight tracking, and allocation engines—each producing its own version of the truth.  
The result: fragmented visibility, inconsistent timestamps, and delayed decisions.

This workflow re-imagines that process in Foundry:
- **Plan:** Capture demand forecasts by SKU, DC, and week.  
- **Buy:** Issue purchase orders and normalise them as standard `PurchaseOrderLine` objects.  
- **Execute:** Track the flow from vendor shipment to warehouse receipt and final allocation.  
- **Decide:** Use AIP Logic to recommend corrective actions such as *expedite*, *reallocate*, or *escalate*.

---

## Architecture Overview


```mermaid
flowchart LR
  A["Source Systems (Planning, ERP, WMS, TMS, ASN, Carrier)"] --> B["Foundry Ingest"]
  B --> C["Raw Datasets"]
  C --> D["Standardise & Conform"]
  D --> E["Ontology Mapping (Object Types & Links)"]
  E --> F["Metrics Build (SQL)"]
  F --> G["Workshop App (Process Map • Cards • Actions)"]
  F --> H["AIP Automation Agent (Suggestions & Rationale)"]
```
Key Layers:

| Layer                           | Description                                                                                                                                                                                                                            |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Data Ingestion**           | Source data from Planning, ERP, WMS, and Allocation systems covering Purchase Orders, Shipments, Receipts, and Allocations.                                                                                                            |
| **2. Transformation Pipelines** | Standardize and clean each domain dataset, cast timestamps to UTC, compute durations (Ex-Factory→Hub, Hub→DC, Ex-Factory→DC), and flag SLA breaches.                                                                                   |
| **3. Event Log Construction**   | Build a single “movie reel” per PO line with schema: `event_id`, `po_line_id`, `event_type`, `event_ts`, `source_system`, `is_authoritative`, `note`. Apply precedence + dedupe rules (e.g., Receipt > Shipment for DC arrival).       |
| **4. Metrics Object**           | SQL transformations producing per-PO-line KPIs: `lead_time_days`, `cycle_time_days`, `is_late`, `sla_risk_score`, `value_at_risk`, `current_stage`, `most_recent_event_type/date`. This object feeds both the dashboard and AIP Logic. |
| **5. Ontology Modeling**        | Central object `PurchaseOrderLine` linked to `MetricsTable`, `EventLog`, `Shipment`, `Receipt`, and `Allocation`. Configure **filter propagation** from PO Line to linked datasets.                                                    |
| **6. AIP Logic Function**       | A reasoning agent that consumes a `po_line_id`, fetches context via the ontology, and returns a structured recommendation (`recommended_action`, `rationale_md`, `confidence`, `expected_impact_value`).                               |
| **7. Workshop App**             | One-page control tower: KPI cards, process map, PO table, and an AIP panel. Row click sets `selectedPOLineID` → passed to AIP Logic → recommendation displays inline.                                                                  |
