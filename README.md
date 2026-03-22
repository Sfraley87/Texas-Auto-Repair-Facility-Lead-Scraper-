# Texas Auto Repair — Daily Lead Scraper
### Automated B2B lead generation pipeline built in n8n

![n8n](https://img.shields.io/badge/Built%20with-n8n-orange?style=flat-square)
![Google Places](https://img.shields.io/badge/API-Google%20Places%20(New)-blue?style=flat-square)
![Google Sheets](https://img.shields.io/badge/Storage-Google%20Sheets-34A853?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

---

## What This Does

This workflow runs every morning at **5:00 AM** and automatically builds a clean, deduplicated database of auto repair shops across Texas — without touching a browser or spreadsheet manually.

It queries the **Google Places API (New)** across **50+ zip codes** in all major Texas metros, normalizes the results, checks for existing records, and appends only net-new leads to a master Google Sheet. By the time you open your laptop, the list has already grown overnight.

**Built as a real outreach tool — not a demo.**

---

## The Problem It Solves

Manual lead research for local B2B sales is slow, inconsistent, and doesn't scale. A sales team targeting auto repair shops across Texas would typically spend hours per week scraping Google Maps, copy-pasting into spreadsheets, and deduplicating by hand.

This pipeline eliminates that entirely:

- **Automated** — zero human input after setup
- **Deduplicated** — `place_id` used as unique key; no record ever appears twice
- **Structured** — consistent schema across all 8 Texas regions
- **Extensible** — swap the zip list or `includedType` to target any vertical in any state

---

## Architecture

```
Schedule Trigger (5 AM daily)
        │
        ▼
Zip Code Seed List           50+ zips across 8 Texas regions
        │
        ▼
Google Places API (New)      POST /v1/places:searchText
                             includedType: car_repair | max: 20 results/zip
        │
        ▼
Normalize Results            Flatten API response → consistent lead schema
        │
        ▼
Loop Over Shops              Batch iterator — one shop at a time
        │
        ▼
Duplicate Check              Lookup place_id in Google Sheets
        │
        ├─── NEW ──────────► Append to Leads sheet
        └─── EXISTS ───────► Skip silently
```

---

## Coverage

| Region | Cities |
|--------|--------|
| Houston Metro | Houston, Pasadena, Sugar Land |
| DFW Metro | Dallas, Fort Worth, Arlington, Garland, Irving, Plano |
| Austin Metro | Austin, Round Rock, Cedar Park |
| San Antonio Metro | San Antonio, New Braunfels |
| El Paso | El Paso |
| West Texas | Lubbock, Amarillo, Midland, Odessa, Abilene |
| Coastal & South Texas | Corpus Christi, McAllen, Brownsville, Laredo |
| Central & East Texas | Waco, Killeen, Beaumont, Tyler |

---

## Lead Schema

Every record written to Google Sheets:

| Field | Source |
|-------|--------|
| `place_id` | Google Places — deduplication key |
| `name` | Business display name |
| `address` | Full formatted address |
| `city` / `zip` / `region` | Zip seed metadata |
| `rating` | Google star rating |
| `review_count` | Total user reviews |
| `phone_number` | Places API |
| `website` | Places API |
| `status` | Defaults to `New` |
| `scraped_at` | ISO date of scrape |

---

## Stack

- **n8n** — workflow orchestration and scheduling
- **Google Places API (New)** — business data source
- **Google Sheets** — lead storage and deduplication lookup
- **JavaScript (Code nodes)** — data normalization and zip seed generation

---

## Setup

### Prerequisites
- n8n instance (self-hosted or cloud)
- Google Cloud project with **Places API (New)** enabled
- Google Sheet with a `Leads` tab and column headers matching the schema above

### Credentials Needed

| Credential | Used By |
|------------|---------|
| Google Places API Key | `X-Goog-Api-Key` header on HTTP Request node |
| Google Workspace Admin OAuth2 | Google Places API node auth |
| Google Sheets OAuth2 | Duplicate check + append nodes |

### Import Steps
1. Download `Texas_Auto_Repair_Daily_Lead_Scraper.json`
2. In n8n → **Workflows → Import from File**
3. Connect credentials to each node
4. Update the Google Sheet ID in both Sheets nodes if using your own sheet
5. Activate

---

## Extending This

This workflow was designed to be a template, not a one-off:

- **Different vertical** — change `car_repair` to any Google Places `includedType` (HVAC, dental, restaurants, etc.)
- **Different geography** — swap the zip seed list for any state or metro
- **Add enrichment** — pipe new leads into Hunter.io, Apollo, or a CRM after append
- **Add alerts** — connect a Slack or WhatsApp node to get a daily "X new leads added" digest
- **Scale up** — increase `maxResultCount` or add more zips to expand coverage

---

## Notes

- `phone_number` and `website` are fetched from the API but not currently mapped to sheet columns in the append node — easy fix if you need them for outreach sequences
- A `Wait` node between zip iterations is recommended at scale to avoid Places API quota spikes
- Error handling is wired to a separate error workflow

---

## About

Built by **Shaun** — AI Automation Architect at [Synelium](https://synelium.com)

I design and deploy AI-powered automation systems for mid-market businesses. This workflow is part of a broader lead generation and outreach infrastructure I build for clients in sales-intensive industries.

→ More projects: [github.com/yourusername](https://github.com/yourusername)  
→ Connect: [linkedin.com/in/yourprofile](https://linkedin.com/in/yourprofile)
