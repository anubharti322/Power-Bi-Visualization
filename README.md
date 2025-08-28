# Power BI Lead & Sales Analytics

This case study demonstrates how I built a **production-style Power BI solution** on top of a SQL-backed pipeline to turn messy lead drops into **clean, de-duplicated, decision-ready** insights. The model follows a **star schema**, uses **DAX measures** for explainable KPIs (conversion, first-response time, SLA compliance, duplicate rate), and runs with **scheduled refresh** so sales/ops always see the latest truth.

---

## Overview (What this solves)
Lead data used to live in multiple Excel files, creating duplicates and delayed follow-ups. I connected Power BI directly to **SQL Server** tables produced by my ETL (staging → cleaned/rejected → master → reporting). This eliminated manual exports and made the dashboard **reliable, repeatable, and always in sync**.

**Key outcomes**
- Trustworthy KPIs from a **stable master layer** (duplicates suppressed upstream).
- Faster decisions via **slicers, drillthrough**, and **tooltips** instead of cluttered pages.
- Hands-off operations with **Power BI scheduled refresh** and optional **RLS**.

---

## Business questions answered
- Which **sources** (e.g., MagicBricks, Instagram, Referrals) convert best?
- Where do leads **leak** in the funnel (New → Contacted → Qualified → Closed)?
- Which **agents/areas/property types** drive the most impact?
- Are we meeting **SLA targets** (first-response, follow-ups)? What’s at risk *this week*?

---

## Data model (Star)
- **Facts:** `factLeads` (one row per lead), `factInteractions` (calls/WhatsApp), `factDeals` (won)
- **Dimensions:** `dimDate`, `dimAgent`, `dimSource`, `dimProperty` (type, BHK, area, price band)
- **Relationships:** single-direction, 1:* from dims → facts; surrogate integer keys
- **Design goal:** keep the semantic layer **simple, explainable, and fast** for DAX

---

## Core measures (DAX highlights)
```DAX
Total Leads = COUNTROWS(factLeads)

Converted Leads =
CALCULATE([Total Leads], factLeads[Stage] = "Closed Won")

Conversion % =
DIVIDE([Converted Leads], [Total Leads], 0)

Avg First Response (hrs) =
AVERAGEX(
  VALUES(factLeads[LeadID]),
  VAR t0 = MIN(factLeads[CreatedAt])
  VAR t1 = MIN(factInteractions[ContactedAt])
  RETURN DIVIDE(DATEDIFF(t0, t1, HOUR), 1.0)
)

Duplicate Rate % =
VAR total = COUNTROWS(factLeads)
VAR distinctIds = DISTINCTCOUNT(factLeads[LeadID])
RETURN DIVIDE(total - distinctIds, total, 0)
