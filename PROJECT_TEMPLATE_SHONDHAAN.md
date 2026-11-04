# Shondhaan Cross-Functional Project Architecture & Template
### *Web Platform, Backend API Mesh & Flutter Mobile App Synchronization*

> **Objective:** Synchronize feature delivery across the Web, Backend API, and Flutter Mobile teams while eliminating **API contract drift**, preventing breaking production changes, and coordinating mobile client release cycles.

---

## 🎯 The Real-World Engineering Challenge

In the Shondhaan ecosystem:
1. **The Backend API Mesh** serves live traffic across distinct microservice domains:
   - `backend-yservice.shondhaan.com` (Home Services & Booking Engine)
   - `backend-ymart.shondhaan.com` (Grocery Mart & Dynamic Inventory)
   - `backend-deal.shondhaan.com` (Classifieds & Buyer/Seller Chat)
   - `backend-yjob.shondhaan.com` (Career Hub & Digital Resume Engine)
   - `backend-exam.shondhaan.com` (Timed MCQ Assessments)
   - `backend-parcel.shondhaan.com` & `backend-ride.shondhaan.com` (Logistics & Mobility)
2. **The Web Team** can ship continuous deployments (CD) directly to web servers and Cloudflare edge workers in minutes.
3. **The Flutter Mobile App** (Android & iOS) cannot be hot-patched instantly:
   - Client binaries are installed on end-user phones.
   - App Store and Google Play reviews introduce release delays (24–72 hours).
   - Users may delay updating their apps for days or weeks.
   - Dart's strict serialisation (`fromJson` / `toJson` DTO models) crashes or throws unhandled `FormatException` / `TypeError` if a backend field changes types (e.g. `int` to `String`), changes names, or becomes non-nullable without notice.

**Conclusion:** Web and backend changes *cannot* be deployed unilaterally without explicit contract governance and deprecation windows.

---

## 🏗️ GitHub Projects v2 Template Architecture

### 1. Custom Fields & Taxonomies

| Field Name | Type | Options / Values | Purpose |
| :--- | :--- | :--- | :--- |
| **`Platform / Layer`** | Single Select | • `Backend / API Mesh`<br/>• `Web Platform`<br/>• `Mobile (Flutter)`<br/>• `Full-Stack Feature`<br/>• `DevOps & Infra` | Categorizes which engineering team owns the task slice. |
| **`Contract Impact`** | Single Select | • 🟢 `None (Internal/UI Only)`<br/>• 🟡 `Additive (Safe / Backwards-Compatible)`<br/>• 🔴 `Breaking (Drift Risk / Requires Versioning)`<br/>• 📐 `RFC / Contract Review` | **The Core Guardrail:** Flags changes that alter payload shapes, endpoint paths, or status codes. |
| **`Subsystem / Domain`** | Single Select | • `Core Identity & Auth`<br/>• `YessService (Home Services)`<br/>• `YessMart (Grocery Store)`<br/>• `YessDeal (Classifieds)`<br/>• `YessJob (Career Hub)`<br/>• `YessExam (Skill Assessment)`<br/>• `Logistics (Parcel & Ride)`<br/>• `Payments & Wallet` | Aligns tickets to the specific microservice backend and Flutter feature module. |
| **`Status`** | Single Select | • `📋 Backlog`<br/>• `📐 Contract Spec & RFC`<br/>• `⚙️ In Progress (Backend/Web)`<br/>• `📱 In Progress (Flutter Mobile)`<br/>• `🧪 Contract Testing & Staging`<br/>• `🚀 Deployed & Verified` | State machine governing parallel development and verification. |
| **`Priority`** | Single Select | • `P0 - Blocker / Breaking`<br/>• `P1 - High (Core Sprint)`<br/>• `P2 - Standard`<br/>• `P3 - Low / Nice-to-have` | Urgency and triage order. |
| **`Target Release`** | Text / Iteration | • `Sprint 26.1`<br/>• `Mobile v1.0.0-rc1`<br/>• `Hotfix` | Aligns code to mobile binary cutoffs and web sprints. |

---

## 📊 Standard Project Views

### View 1: 🚦 Cross-Team Sprint Kanban (Board View)
* **Grouping:** `Status` (`Backlog` ➔ `Contract Spec` ➔ `In Progress` ➔ `Contract Testing` ➔ `Deployed`)
* **Card Badges:** Displays `Platform / Layer`, `Contract Impact`, and `Subsystem`.
* **Purpose:** Daily standup view where backend, web, and mobile engineers see cross-dependencies side-by-side.

### View 2: 🛡️ API Contract Watchtower (Table View)
* **Filter:** `Contract Impact` is NOT `None (Internal/UI Only)`.
* **Grouping:** `Subsystem / Domain`
* **Columns:** `Title`, `Contract Impact`, `Platform / Layer`, `Assignees`, `Target Release`, `Status`.
* **Purpose:** Mobile leads monitor this table daily. Any card marked 🔴 `Breaking` or 📐 `RFC` triggers contract alignment before backend engineers write production database migrations or endpoint modifications.

### View 3: 📱 Mobile Release Readiness (Board View)
* **Filter:** `Platform / Layer` is `Mobile (Flutter)` or `Full-Stack Feature`.
* **Grouping:** `Status`
* **Purpose:** Tracks Flutter feature progress and validates that backend staging endpoints are certified before the mobile release binary is built.

### View 4: 🗺️ Subsystem Architecture Matrix (Table View)
* **Grouping:** `Subsystem / Domain`
* **Purpose:** High-level overview of which microservices are receiving updates in the current release cycle.

---

## 🛡️ Anti-Drift Workflow & Engineering Standard Operating Procedure (SOP)

```
[Feature Request / API Modification]
                 │
                 ▼
     Phase 1: Contract First (RFC)
     • Define OpenAPI 3.x payload in docs/spec PR
     • Tag card: Contract Impact = 📐 RFC
     • Review: Backend Lead + Flutter Lead sign off
                 │
                 ▼
     Phase 2: Parallel Implementation
     • Backend: Implements using Expand & Contract pattern
     • Flutter: Generates mock DTO models & tests UI against Prism mock server
     • Tag card: Status = ⚙️ In Progress
                 │
                 ▼
     Phase 3: Automated Contract Validation
     • CI runs schema diff (oasdiff) on backend PR
     • Staging endpoint deployed with backwards compatibility
     • Flutter runs against live staging server
     • Tag card: Status = 🧪 Contract Testing
                 │
                 ▼
     Phase 4: Coordinated Production Rollout
     • Backend deploys (Old fields maintained for 30+ days)
     • Flutter releases binary to Google Play / App Store
     • Legacy fields deprecated only after 95%+ users update
```

### The "Expand & Contract" Rule for Shondhaan Backend Engineers
* **Never delete or rename an existing response key in a single deployment.**
* **Adding new data:** Add the new key as nullable or with default values.
* **Modifying a field:** 
  1. Add the new field alongside the old field (Dual Output).
  2. Mobile team migrates to read the new field.
  3. Once mobile app store telemetry confirms minimum adoption, remove the deprecated field in a future cycle.
