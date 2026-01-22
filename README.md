# Fexcon AI Intern Task — LeadGen + Outreach Drafting with Human Approval (n8n + OSM + Ollama)

This project implements an end-to-end automation pipeline to:
1) Discover local business leads using OpenStreetMap APIs (Nominatim + Overpass),
2) Enrich + score + deduplicate leads,
3) Generate personalized outreach drafts (email + social post) using a **local LLM** (Ollama + `phi3:mini`),
4) Store and track everything in **Google Sheets**,
5) Require a **human approval gate** before any action (Approve/Reject buttons via webhook).

The entire solution is designed to run using **free tools** (no paid LLM API required).

---

## Architecture

**Workflow 1: Lead Intake + Draft Generation**
- Trigger (Webhook/Manual)
- Nominatim → Overpass → Parse leads → Lead scoring
- Deduplicate by `osm_id` against Google Sheets
- Generate drafts using local LLM (Ollama `phi3:mini`)
- Append/Update row in Google Sheets
- Send approval email with Approve/Reject buttons

**Workflow 2: Approval Webhook**
- Webhook endpoint `/webhook/lead-approval`
- Validates `osm_id` + `approval_token`
- Updates Google Sheet row (`draft_status = approved/rejected`)
- Returns an HTML confirmation page

> Note: For safety and ethics, no emails are sent automatically to leads without human approval.

---

## Tech Stack
- **n8n** (workflow automation)
- **Google Sheets** (lead database + status tracking)
- **OpenStreetMap**
  - Nominatim (geocoding / bounding box)
  - Overpass API (place search)
- **Ollama** + **phi3:mini** (local LLM drafting)
- **SMTP** (approval email notifications)

---

## Data Model (Google Sheet Columns)

Minimum recommended columns:
- `osm_id` (unique key, e.g., `node/1838419433`)
- `company_name`, `website`, `email`, `phone`, `city`, `country`, `industry`
- `summary`, `lead_score`
- Draft outputs: `email_subject`, `email_body`, `social_post`
- Approval: `draft_status` (`drafted` / `approved` / `rejected`)
- Security: `approval_token`
- Timestamps: `created_at`, `approved_at`, `rejected_at`

---

## Setup (Step-by-step)

### 1) Install and run n8n
Use any method you prefer (desktop/self-host). Ensure you can run and execute workflows locally.

### 2) Create the Google Sheet
Create a sheet with the headers listed above.

### 3) Install Ollama and the model
Install Ollama and pull the model:

```powershell
ollama pull phi3:mini
