# Dynamic Agronomic Supply Chain & Market Arbitrage Router

Welcome to the comprehensive technical documentation and developer README for the **Dynamic Agronomic Supply Chain & Market Arbitrage Router**. This document covers the system architecture, data pipelines, error-handling routines, logistics baselines, design deliberations, presentation deck structure, testing protocols, master ledger tracking, and the technical mechanics of the JavaScript Arbitrage Engine.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [The Problem We're Solving](#the-problem-were-solving)
- [System Architecture & UI/UX Presentation Deck](#system-architecture--uiux-presentation-deck)
- [Tool Stack](#tool-stack)
- [Workflow Structure & Node Breakdown](#workflow-structure--node-breakdown)
- [JavaScript Arbitrage Engine & Mathematical Formulas](#javascript-arbitrage-engine--mathematical-formulas)
- [Data Sources & Logistics Baseline](#data-sources--logistics-baseline)
- [Notification Master Ledger & Routing Rules](#notification-master-ledger--routing-rules)
- [Setup Instructions](#setup-instructions)
- [Testing Protocol](#testing-protocol)
- [Challenges, Engineering Decisions & Design Deliberations](#challenges-engineering-decisions--design-deliberations)
- [Known Limitations](#known-limitations)
- [Future Roadmap](#future-roadmap)

---

## Overview

The Dynamic Agronomic Supply Chain & Market Arbitrage Router is an automated data aggregation, processing, and profitability calculation pipeline designed for Nigerian agricultural commodity markets.

Operating on an automated 8-hour schedule, the system fetches live market price data across target regional hubs, normalizes unstructured schemas into a unified format, factors in standard truck capacities (5 tons per truck), handles unit conversions (converting kg to tons and routing non-conforming items via an IF node to a separate tracking sheet), logs historical tonnage records into a dedicated spreadsheet, merges transport overheads, evaluates net profitability via an internal JavaScript arbitrage engine, splits routes via an IF node to alert administrators of unprofitable paths while looping profitable routes through an AI agent to generate shipping manifests, and logs final results to the Notion master ledger and Slack channels.

### Our Solution: The Intelligent End-to-End Pipeline

1. **Automated Data Ingestion & Append** — Multi-API pulls every 8 hours across regional market endpoints, merged in append mode.
2. **Schema Normalization & Capacity Assignment** — Centralized JS parsing node unifies disparate JSON payloads and assigns a standardized truck capacity of 5 tons per truck.
3. **Unit Validation & Fallback Routing** — An IF node inspects tonnage; items measured in non-kg derica units resulting in 0 tons are routed to a separate spreadsheet, while standardized tonnage data is appended to the main records spreadsheet.
4. **Logistics & Overhead Integration** — Tonnage data spreadsheet is merged with logistics cost sheets before passing through the JavaScript Arbitrage Engine.
5. **Profitability Split & Automated Dispatch** — An IF node splits routes into profitable (`True`) and non-profitable (`False`).
   - Non-profitable routes report directly to the Admin Slack channel.
   - Profitable routes pass through an AI agent to build shipping manifests, dispatching updates to the Notion Master Ledger and alerting the Logistics Admin via Slack.

---

## Prerequisites

- **Hosting Server:** Self-hosted n8n instance on a Contabo VPS (Ubuntu 20.04/22.04 LTS).
- **Runtime Environment:** Node.js (supporting native n8n JavaScript code nodes).
- **Google Cloud Workspace:** Dedicated project Google Account with OAuth2 credentials for Google Sheets, Google Docs, and Google Drive API.
- **Notion Workspace:** Dedicated Notion integration token and database setup for the Master Ledger to track system outputs, shipping manifests, and routing parameters.
- **Communication Channels:** Slack Workspace & Bot Application
  - Created as a Blank App in Slack (to prevent `invalid_team_for_non_distributed_app` authorization errors).
  - Generated credentials: Client ID, Client Secret, Signing Secret, User OAuth Token, and Bot Token.
  - Configured channels: `#logistics-alerts`, `#admin-alerts`, `#daily-summary`.
- **Market API Access:** Endpoint credentials/URLs for the 4 primary Market APIs:
  - HARVESTIQ API
  - Ọla of North API
  - Food Prices API
  - PriceNinja API
- **Presentation Software:** Figma Slides account for deck editing and stakeholder presentations.

---

## The Problem We're Solving

### Core Business Challenges

- **Rapid Price Volatility:** Agricultural commodity prices swing sharply due to seasonal shocks, regional insecurity, and localized supply-demand imbalances.
- **Manual Decision-Making:** Relying on phone calls and manual spreadsheets causes severe delays, leading to missed arbitrage opportunities and food spoilage.
- **Fluctuating & Opaque Logistics Costs:** Commercial freight pricing is rarely transparent, making it difficult to calculate true net margins quickly.
- **Fragmented Market Information & Unit Inconsistencies:** Disparate market sources publish data using inconsistent schemas, key names, and varying local volume units (such as derica measurements instead of standard kilograms or tons).

### Impact on Food Security in Nigeria

Agricultural produce often rots at farm-gate markets in northern producing hubs (e.g., Kano) while southern urban centers (e.g., Lagos) suffer from artificial scarcity and high food inflation. By delivering automated market intelligence every 8 hours, this workflow bridges regional supply gaps, accelerates distribution, and ensures commodities flow to high-demand areas efficiently.

---

## System Architecture & UI/UX Presentation Deck

### End-to-End System Architecture

**Figma Slides Presentation Link:** [View Presentation Deck](#)

### UI/UX Design Strategy & Figma Slides Deck Architecture

The UI/UX design team evaluated multiple presentation environments (Figma vs. FigJam vs. Figma Slides):

- **Figma:** Evaluated but determined to be tailored primarily for screen/database UI component design rather than structured deck delivery.
- **FigJam:** Considered for flowchart wireframing and freeform brainstorming, but lacked formal page-by-page presentation controls.
- **Figma Slides (Chosen Platform):** Selected as the final tool because it provides structured page-by-page slide formatting, professional design controls, and formal presenter features suitable for classroom and stakeholder demonstrations.

### 9-Slide Presentation Structure

| # | Slide | Description |
|---|-------|-------------|
| 1 | Cover Slide | Introduction to the workflow. Features project title, description, key pillars, and an end-to-end flowchart. |
| 2 | Problem Statement Slide | Highlights business impacts of price volatility, manual decisions, fluctuating logistics, fragmented data, and distribution delays. |
| 3 | Our Solution Slide | Details the 5-step intelligent pipeline addressing market inefficiencies. |
| 4 | Our n8n Workflow Slide | Displays the main n8n workflow diagram. |
| 5 | Dataflow Slide | Maps the visual data journey from ingestion sources to the final Slack message. |
| 6 | Error Handler Workflow Slide | Showcases the structure of the dynamic error handler, nodes used, information captured, and system resilience benefits. |
| 7 | Node Explanation Slide | Comprehensive functional breakdown of all nodes utilized across both workflows. |
| 8 | Business Benefits Slide | Details operational value: faster market intelligence, smarter profit decisions, higher margin potential, reduced logistics waste, and rapid execution. |
| 9 | Concluding Slide | Summarizes what was built, why it matters, current limitations, and immediate next steps. |

---

## Tool Stack

| Category | Tool | Purpose |
|----------|------|---------|
| Workflow Automation | n8n (v1.x+) | Orchestrates schedule triggers, API requests, data merging, conditional branching, and error handling. |
| Server Infrastructure | Contabo VPS | Low-cost Linux hosting for the n8n server environment. |
| Data Ingestion | 4 HTTP APIs | Live price feeds (HARVESTIQ, Ọla of North, Food Prices, PriceNinja). |
| Database & Ledger Layer | Google Sheets & Notion | Decoupled spreadsheet tracking for raw appends, unit exceptions, and transport data, paired with Notion as the Master Ledger for shipping manifests and notification logging. |
| Analytics & AI Engine | JavaScript Code Node (n8n Native) & AI Agent Node | Processes net profit matrices, evaluates arbitrage route pairings, and auto-generates shipping manifests. |
| Messaging & Notifications | Slack API (Blank App) | Multi-channel dispatch (`#logistics-alerts`, `#admin-alerts`, `#daily-summary`). |
| UI/UX & Deck Design | Figma Slides | Primary presentation deck design. |

---

## Workflow Structure & Node Breakdown

### 1. Main Ingestion & Execution Pipeline

| # | Node | Purpose |
|---|---|---|
| 1 | **Schedule Trigger Node** | Triggers every 8 hours (3 times daily) to pull fresh pricing. |
| 2 | **HTTP Request Nodes (x4)** | Concurrently fetch price payloads from HARVESTIQ, Ọla of North, Food Prices, and PriceNinja. |
| 3 | **Merge Node (Append)** | Unifies incoming JSON arrays from all 4 API endpoints into a single data stream. |
| 4 | **Consolidated Code Node** | Runs custom JavaScript to dynamically parse varying object keys across APIs, normalize records into a uniform schema, and assign a standardized truck capacity of 5 tons per truck. |
| 5 | **IF Node (Unit Validation)** | Evaluates incoming items to check tonnage. Commodities measured in local volume units (derica) resulting in 0 tons are filtered out and routed to a separate unit exception spreadsheet. Items with valid tonnage are appended to the main tonnage spreadsheet. |
| 6 | **Merge Node (Logistics Integration)** | Combines rows from the main tonnage spreadsheet with data from the logistics cost spreadsheet. |
| 7 | **JavaScript Arbitrage Engine Code Node** | Evaluates pairwise market comparisons against transport overheads to compute net profit spreads and operational rankings. |
| 8 | **IF Node (Profitability Split)** | Routes profitable routes to `True` and non-profitable routes to `False`. **False Branch:** sends a report of non-profitable routes directly to the Admin Slack channel. **True Branch:** loops profitable routes through an AI Agent Node to generate a structured shipping manifest. |
| 9 | **Notion Master Ledger Node** | Ingests the AI-generated shipping manifest and logs master tracking records. |
| 10 | **Slack Alert Node** | Alerts the Logistics Admin in the designated Slack channel regarding confirmed profitable routes. |

### 2. Error Handler Workflow

| # | Node | Purpose |
|---|---|---|
| 1 | **Error Trigger Node** | Captures execution failures across any node in the main workflow. |
| 2 | **Set/Edit Fields Node** | Pulls workflow name, failed node, error message, and timestamp. |
| 3 | **Slack Alert Node** | Sends immediate failure warnings to the team Slack channel and dispatches formal incident reports to system administrators. |

---

## JavaScript Arbitrage Engine & Mathematical Formulas

The system relies on a unified JavaScript Code Node executing in **"Run Once for All Items"** mode. It ingests a merged stream of market commodity prices and logistics routes, automatically parsing and processing them through a multi-step intelligence pipeline:

### Engine Mechanics & Processing Steps

1. **Input Auto-Sorting:** Uses case-insensitive key inspection (`getField`) to separate incoming records into a Route Pile (matching origin/destination) and a Market Pile (matching commodity/market).
2. **Fuzzy Logistics Matching:** Because market names vary across sources (e.g., "Mile 12 Market" vs. "Lagos (Mile 12)"), the engine builds exact and fuzzy lookup maps using normalized name variants to successfully resolve transport costs.
3. **Market Consensus Building:** Groups records by commodity + market + unit to synthesize average pricing, source counts, confidence scores, and timestamps across multiple API pulls.
4. **Arbitrage Generation & Zero-Data Filtering:** Evaluates all pairwise market combinations for the same commodity. Any route lacking verified logistics cost data (`logistics_match === "none"`) is dropped entirely to prevent false positives.
5. **Scoring and Ranking:** Rates routes ("Excellent", "Very Good", "Good", "Fair", "Poor") based on a weighted scoring model incorporating net profit, margin percentage, and source confidence levels.

---

## Data Sources & Logistics Baseline

- **Target Markets & Geopolitical Pairings:** Data is aggregated across 6 key regional markets representing major Nigerian trade zones (Kano, Abuja, Lagos, Kaduna, Benin, Ibadan). Core trade corridors include Kano ↔ Abuja, Lagos ↔ Kano, and Abuja ↔ Lagos.
- **Fixed Logistics Parameters:**
  - **Diesel Rate:** Fixed at ₦1,620 / Liter (commercial freight vehicle benchmark).
  - **Truck Hiring Base:** Fixed flat-rate baseline quotes per market pair.
  - **Distance & Drive Time:** Fixed distance (km) and estimated drive hours per route.

---

## Notification Master Ledger & Routing Rules

Dispatch parameters are dynamically tracked, consolidated, and managed directly inside the Notion master ledger.

**Master Ledger Reference:** [View Notion Master Ledger](#)

### Channel Architecture & Routing Rules

| Channel | Receives |
|---|---|
| `#logistics-alerts` | High-priority Profitable Route Alerts when arbitrage spreads exceed logistics baselines, or Daily Market Reports when no opportunities are found under live market conditions. |
| `#admin-alerts` | System Error Alerts (API timeouts, node failures, database authorization drops) as well as Non-Profitable Route Reports routed from the arbitrage IF node. |
| `#daily-summary` | Daily Operations Summary detailing scan counts, uptime, and performance metrics. |

### Critical Membership & Workspace Rules

- **App Type Requirement:** Slack apps connecting to n8n must be created as a Blank App to avoid `invalid_team_for_non_distributed_app` errors.
- **Account Workspace Invitation:** Any account or bot responsible for connecting n8n to Slack must first be invited to the Slack workspace as an active member.
- **Channel Membership Requirement:** Bot accounts must be explicitly added as members to each destination Slack channel.

---

## Setup Instructions

1. **Provision VPS & Install n8n:** Set up a Contabo VPS running Ubuntu. Install Docker or Node.js to host n8n, ensuring execution limits and timezones are configured.
2. **Configure Credentials:** In n8n, configure credentials for Google OAuth2 (Sheets, Docs, Gmail), Notion API integration, Slack Webhook/OAuth, and header auth keys for the 4 Market APIs.
3. **Import Workflows:** Import the Main Ingestion Workflow and Error Handler Workflow `.json` files into n8n.
4. **Initialize Database Sheets & Notion:** Create Google Spreadsheets for raw ingress, unit exceptions (derica 0-ton routes), and logistics data, and set up the Notion Master Ledger database for shipping manifest and notification tracking.
5. **Deploy JavaScript Arbitrage Engine & Figma Slides:** Configure the native n8n JavaScript code node for processing arbitrage calculations with 5-ton truck capacities. Set up the 9-slide deck in Figma Slides for stakeholder presentation.

---

## Testing Protocol

To ensure full compliance and system operational readiness, execute the following mandatory test suite:

| # | Test | Objective | Execution |
|---|---|---|---|
| 1 | **Schema Ingestion, Append & Normalization Test** | Verify that the 4 API endpoints merge correctly in append mode, and that the Consolidated Code Node normalizes payloads and assigns the 5-ton truck capacity without throwing empty output (`[]`) errors. | Run ingestion nodes manually in n8n and inspect the output payload of the normalization code node to confirm uniform schema structure and capacity keys. |
| 2 | **Unit Inspection IF Node & Exception Routing Test** | Confirm that the IF node successfully separates non-kg derica measurements (resulting in 0 tons) into the derica exception spreadsheet, while passing valid tonnage records to the main tonnage spreadsheet. | Test with mixed unit payloads to verify proper branching into Google Sheets. |
| 3 | **Logistics Merge & JavaScript Arbitrage Engine Test** | Validate that the tonnage spreadsheet merges cleanly with the logistics cost spreadsheet, and that the native JavaScript Code Node correctly computes net arbitrage margins for 5-ton shipments using both live data inputs and simulation data overrides. | Test the code node with live payloads and simulation datasets to verify accurate calculation of fuel costs, truck hires, and net spreads. |
| 4 | **Profitability IF Node, AI Manifest Generation & Slack/Notion Dispatch Test** | Confirm that the profitability IF node correctly routes non-profitable routes to the Admin Slack channel while looping profitable routes through the AI Agent to generate shipping manifests, log to Notion, and alert the Logistics Admin via Slack. | Test manual dispatches across both IF branches to verify bot permissions, manifest formatting, and Notion database entry creation against the master ledger. |
| 5 | **Automated Error-Handling & Resilience Test** | Verify that the Error Handler Workflow correctly captures execution faults, extracts metadata, and alerts administrators. | **(Automated Rule)** Do not test manually (as manual runs suppress Error Triggers). Set the Schedule Trigger to run every 1 minute, intentionally inject a temporary error into an HTTP Request node, and verify that Slack failure warnings trigger automatically. |

---

## Challenges, Engineering Decisions & Design Deliberations

- **API Standardization:** Formally abandoned web scraping and reverted to 4 direct Market APIs to align with project requirements and maintain stable data pipelines.
- **Code Node Engine Architecture:** Replaced external Python execution scripts with a native JavaScript Code Node inside n8n. This simplifies deployment, eliminates external runtime dependencies on the VPS, and keeps the entire data transformation pipeline contained within n8n.
- **Unit Variance & Exception Handling:** Addressed local volume measurement discrepancies (e.g., derica vs. standard kilograms/tons) by implementing an explicit IF node routing rule that filters 0-ton anomalies into a dedicated tracking spreadsheet, protecting the main arbitrage engine from calculation errors.
- **Truck Capacity Standardization:** Standardized commercial hauling payloads to 5 tons per truck during initial normalization to ensure consistent logistics cost scaling across all market pairs.
- **Live Market Pricing vs. Demo Data:** Faced with live APIs returning tight markets with zero immediate arbitrage margin, we integrated a simulation/demo switch in the JavaScript processing node. This allows the team to demonstrate rich, actionable Slack alerts and visual routing tables during presentations without relying on artificially manipulated live API endpoints.
- **Error Trigger Testing:** Error triggers do not execute during manual n8n runs. Tested error pipelines using temporary 1-minute automated schedule triggers.

---

## Known Limitations

- **Inability to Find Profitable Routes with Purely Live API Data:** Relying exclusively on live market price feeds from public APIs frequently results in zero profitable arbitrage routes due to tight regional spot pricing and heavy diesel freight overheads. This necessitates the use of simulation switches during live demonstrations.
- **Fixed Logistics Parameters:** Freight quotes and diesel rates use static baselines due to the absence of public commercial trucking quote APIs.
- **Google Sheets Write Quotas:** High-frequency logging risks hitting API write limits (mitigated by the 8-hour execution interval).

---

## Future Roadmap

- **Dynamic Freight API Integration:** Partner with or integrate logistics APIs for live trucking quotes.
- **36-State Geographic Expansion:** Expand market tracking beyond the 6 core regional hubs to cover all 36 Nigerian states.
