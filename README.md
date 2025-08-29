# Power BI Lead & Sales Analytics

This case study demonstrates how I built a **production-style Power BI solution** on top of a SQL-backed pipeline to turn messy lead drops into **clean, de-duplicated, decision-ready** insights. The model uses **DAX measures** for explainable KPIs (conversion, follow-up rate, revenue, average days to convert), and runs with **scheduled refresh** so sales/ops always see the latest truth.

---

## Overview (What this solves)
Lead data used to live in multiple Excel files, creating duplicates and delayed follow-ups. I connected Power BI directly to **SQL Server** tables produced by my ETL (staging → cleaned/rejected → master → reporting). This eliminated manual exports and made the dashboard **reliable, repeatable, and always in sync**.

**Key outcomes**
- Trustworthy KPIs from a **stable master layer** (duplicates suppressed upstream).
- Faster decisions via **slicers, drillthrough**, and **tooltips** instead of cluttered pages.
- Hands-off operations with **Power BI scheduled refresh**.

---

## Business questions answered
- Which **sources** (e.g., MagicBricks, Instagram, Referrals) convert best?
- Where do leads **leak** in the funnel (New → Contacted → Qualified → Closed)?
- Which **agents/areas/property types** drive the most impact?


---

## Core measures (DAX highlights)
DAX
Total Leads = COUNTROWS(factLeads)

Closed Leads = CALCULATE(
    COUNTROWS('master_leads'), 
    'master_leads'[lead_status] = "closed")

Conversion % =
DIVIDE([Converted Leads], [Total Leads], 0)

Days to Convert = AVERAGE(master_leads[Days to Convert])

Follow-up Rate = CALCULATE(
    COUNTROWS('master_leads'),
    master_leads[lead_status] = "followup"
) / [Total Leads]

REVENUE = CALCULATE(
    SUM(master_leads[budget]),
    master_leads[lead_status] = "closed"
)

Lost-leads Rate = CALCULATE(
    COUNTROWS('master_leads'),
    master_leads[lead_status] = "lost"
) / [Total Leads]
