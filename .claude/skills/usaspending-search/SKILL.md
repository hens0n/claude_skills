---
name: usaspending-search
description: Search and analyze federal government spending data from USAspending.gov. Find contracts, grants, loans by keyword, agency, recipient, location, and time period. Analyze spending trends, top recipients, and agency budgets.
allowed-tools: Bash, Read, Write, Edit, AskUserQuestion
argument-hint: [search query or question about federal spending]
---

# USAspending Search — Federal Spending Data Tool

You are an expert at searching and analyzing U.S. federal government spending data using the USAspending.gov API (https://api.usaspending.gov/api/v2/).

## How to Interpret User Requests

The user will ask natural language questions about federal spending. Your job is to:

1. Determine which API endpoint(s) to call
2. Build the correct request body with appropriate filters
3. Make the API call(s) using `curl` via the Bash tool
4. Present the results in a clear, well-formatted summary

## Core API Endpoints

### 1. Search Awards (contracts, grants, loans, etc.)

**Endpoint:** `POST /api/v2/search/spending_by_award/`

Use this when the user wants to find specific awards — contracts, grants, loans, or other federal awards.

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/search/spending_by_award/ \
  -H "Content-Type: application/json" \
  -d '{
    "filters": { <FILTERS_OBJECT> },
    "fields": ["Award ID", "Recipient Name", "Award Amount", "Description", "Start Date", "End Date", "Awarding Agency", "Awarding Sub Agency"],
    "page": 1,
    "limit": 10,
    "sort": "Award Amount",
    "order": "desc",
    "subawards": false
  }'
```

**Available fields for contracts:** `"Award ID"`, `"Recipient Name"`, `"Start Date"`, `"End Date"`, `"Award Amount"`, `"Total Outlays"`, `"Description"`, `"def_codes"`, `"COVID-19 Obligations"`, `"COVID-19 Outlays"`, `"Infrastructure Obligations"`, `"Infrastructure Outlays"`, `"Awarding Agency"`, `"Awarding Sub Agency"`, `"Contract Award Type"`, `"Funding Agency"`, `"Funding Sub Agency"`

**Available fields for grants/other assistance:** `"Award ID"`, `"Recipient Name"`, `"Start Date"`, `"End Date"`, `"Award Amount"`, `"Total Outlays"`, `"Description"`, `"def_codes"`, `"COVID-19 Obligations"`, `"COVID-19 Outlays"`, `"Infrastructure Obligations"`, `"Infrastructure Outlays"`, `"Awarding Agency"`, `"Awarding Sub Agency"`, `"Funding Agency"`, `"Funding Sub Agency"`, `"Assistance Listing"`

**Award type codes:**
- Contracts: `"A"` (BPA Call), `"B"` (Purchase Order), `"C"` (Delivery Order), `"D"` (Definitive Contract)
- Grants: `"02"` (Block Grant), `"03"` (Formula Grant), `"04"` (Project Grant), `"05"` (Cooperative Agreement)
- Direct Payments: `"06"` (Direct Payment with Unrestricted Use), `"10"` (Direct Payment as Specified by Law)
- Loans: `"07"` (Direct Loan), `"08"` (Guaranteed/Insured Loan)
- Other: `"09"` (Insurance), `"11"` (Other Financial Assistance)
- IDVs: `"IDV_A"`, `"IDV_B"`, `"IDV_B_A"`, `"IDV_B_B"`, `"IDV_B_C"`, `"IDV_C"`, `"IDV_D"`, `"IDV_E"`

### 2. Award Counts by Type

**Endpoint:** `POST /api/v2/search/spending_by_award_count/`

Use this to get a quick count of how many awards match filters, broken down by type (contracts, grants, loans, etc.).

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/search/spending_by_award_count/ \
  -H "Content-Type: application/json" \
  -d '{"filters": { <FILTERS_OBJECT> }}'
```

### 3. Spending Over Time

**Endpoint:** `POST /api/v2/search/spending_over_time/`

Use this to show spending trends over fiscal years or months.

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/search/spending_over_time/ \
  -H "Content-Type: application/json" \
  -d '{
    "filters": { <FILTERS_OBJECT> },
    "group": "fiscal_year"
  }'
```

`group` can be `"fiscal_year"`, `"quarter"`, or `"month"`.

### 4. Spending by Category

**Endpoint:** `POST /api/v2/search/spending_by_category/<CATEGORY>/`

Use this to break down spending by a dimension: who's spending, who's receiving, what industry, etc.

Available categories: `awarding_agency`, `awarding_subagency`, `funding_agency`, `funding_subagency`, `recipient`, `cfda` (assistance listing), `naics`, `psc`, `federal_account`, `country`, `county`, `district`, `state_territory`

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/search/spending_by_category/awarding_agency/ \
  -H "Content-Type: application/json" \
  -d '{
    "filters": { <FILTERS_OBJECT> },
    "category": "awarding_agency",
    "limit": 10,
    "page": 1
  }'
```

### 5. Spending by Geography

**Endpoint:** `POST /api/v2/search/spending_by_geography/`

Use this for geographic analysis of spending by state, county, or congressional district.

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/search/spending_by_geography/ \
  -H "Content-Type: application/json" \
  -d '{
    "filters": { <FILTERS_OBJECT> },
    "scope": "place_of_performance",
    "geo_layer": "state",
    "spending_type": "obligation"
  }'
```

- `scope`: `"place_of_performance"` or `"recipient_location"`
- `geo_layer`: `"state"`, `"county"`, or `"district"`
- `spending_type`: `"obligation"` or `"outlay"`
- NOTE: Do NOT include `"geo_layer_filters"` unless filtering to specific states/counties. If included, it must be non-empty.

### 6. Individual Award Detail

**Endpoint:** `GET /api/v2/awards/<GENERATED_UNIQUE_AWARD_ID>/`

Use this when the user wants full details on a specific award (after finding it via search).

```bash
curl -s https://api.usaspending.gov/api/v2/awards/<GENERATED_UNIQUE_AWARD_ID>/
```

The `generated_internal_id` from search results is the ID to use here.

### 7. Agency Information

**Endpoint:** `GET /api/v2/agency/<TOPTIER_AGENCY_CODE>/`

```bash
curl -s https://api.usaspending.gov/api/v2/agency/<TOPTIER_AGENCY_CODE>/
```

**List all agencies:** `GET /api/v2/references/toptier_agencies/`

### 8. Recipient Search (Autocomplete)

**Endpoint:** `POST /api/v2/autocomplete/recipient/`

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/autocomplete/recipient/ \
  -H "Content-Type: application/json" \
  -d '{"search_text":"<NAME>","limit":10}'
```

### 9. Recipient Profile

**Endpoint:** `GET /api/v2/recipient/<RECIPIENT_HASH>/`

Use the `recipient_id` from spending_by_category/recipient results.

### 10. Transaction-Level Search

**Endpoint:** `POST /api/v2/search/spending_by_transaction/`

For finding individual transactions (more granular than award-level search). Uses the same filters object.

```bash
curl -s -X POST https://api.usaspending.gov/api/v2/search/spending_by_transaction/ \
  -H "Content-Type: application/json" \
  -d '{
    "filters": { <FILTERS_OBJECT> },
    "fields": ["Award ID", "Recipient Name", "Action Date", "Transaction Amount", "Awarding Agency", "Awarding Sub Agency", "Award Type"],
    "page": 1,
    "limit": 10,
    "sort": "Transaction Amount",
    "order": "desc"
  }'
```

## The Filters Object

The `filters` object is shared across all search endpoints. Build it based on the user's query. All fields are optional — include only what's relevant.

```json
{
  "keywords": ["search terms here"],
  "time_period": [
    {"start_date": "2024-01-01", "end_date": "2025-12-31"}
  ],
  "award_type_codes": ["A", "B", "C", "D"],
  "agencies": [
    {
      "type": "awarding",
      "tier": "toptier",
      "name": "Department of Defense"
    }
  ],
  "recipient_search_text": ["Lockheed Martin"],
  "place_of_performance_locations": [
    {"country": "USA", "state": "CA"}
  ],
  "recipient_locations": [
    {"country": "USA", "state": "TX"}
  ],
  "award_amounts": [
    {"lower_bound": 1000000, "upper_bound": 10000000}
  ],
  "naics_codes": {"require": ["541511"]},
  "psc_codes": {"require": ["D306"]},
  "award_ids": ["CONT_AWD_...", "W911NF..."]
}
```

### Filter Field Reference

| Filter | Type | Description |
|--------|------|-------------|
| `keywords` | `string[]` | Free-text search across award descriptions, recipient names, NAICS, PSC |
| `time_period` | `object[]` | Date ranges with `start_date` and `end_date` (YYYY-MM-DD). Earliest: 2007-10-01 |
| `award_type_codes` | `string[]` | See award type codes above |
| `agencies` | `object[]` | Filter by agency. Each: `{type, tier, name}`. type: `"awarding"` or `"funding"`. tier: `"toptier"` or `"subtier"` |
| `recipient_search_text` | `string[]` | Search recipient names |
| `place_of_performance_locations` | `object[]` | Where work is performed. `{country, state, county, city, zip, district}` |
| `recipient_locations` | `object[]` | Where recipient is located. Same fields as above |
| `award_amounts` | `object[]` | Amount ranges: `{lower_bound, upper_bound}`. Omit either for one-sided range |
| `naics_codes` | `object` | `{require: ["code1", "code2"]}` — filter by NAICS industry codes |
| `psc_codes` | `object` | `{require: ["code1"]}` — filter by Product/Service codes |
| `award_ids` | `string[]` | Search by specific award IDs (PIID, FAIN, URI) |
| `program_numbers` | `string[]` | CFDA/Assistance Listing numbers |
| `def_codes` | `string[]` | Disaster Emergency Fund codes (e.g., `["L", "M", "N", "O", "P"]` for COVID) |

### State Codes for Location Filters

Use standard 2-letter state abbreviations: AL, AK, AZ, AR, CA, CO, CT, DE, FL, GA, HI, ID, IL, IN, IA, KS, KY, LA, ME, MD, MA, MI, MN, MS, MO, MT, NE, NV, NH, NJ, NM, NY, NC, ND, OH, OK, OR, PA, RI, SC, SD, TN, TX, UT, VT, VA, WA, WV, WI, WY, DC, PR, VI, GU, AS, MP

## Workflow

1. **Parse the user's question** to determine what data they need
2. **Choose the right endpoint(s)** — often you'll combine multiple calls for a complete answer
3. **Build the filters object** with only the relevant fields
4. **Make the API call(s)** using curl via Bash. Parse the JSON response with `python3 -m json.tool` for readability
5. **Present results clearly** — use tables, bullet points, and summaries. Always include:
   - Total amounts in human-readable format (e.g., "$1.2 billion" not "1200000000")
   - Award counts when relevant
   - Source context (time period, filters applied)
6. **Offer follow-up options** — suggest ways to drill deeper (e.g., "Want to see the top recipients?" or "Should I break this down by state?")

## Common Query Patterns

### "How much has been spent on X?"
1. Call `spending_by_award_count` to get totals by type
2. Call `spending_over_time` to show trend
3. Call `spending_by_category/awarding_agency` to show which agencies

### "Who are the top contractors/recipients for X?"
1. Call `spending_by_category/recipient` with keyword filters
2. Optionally call `spending_by_award` to show top individual awards

### "What contracts has Company X received?"
1. Use `spending_by_award` with `recipient_search_text` filter
2. Set `award_type_codes` to contracts: `["A","B","C","D"]`

### "Show me spending in State X" or "What federal money goes to State X?"
1. Call `spending_by_geography` with the state filter
2. Call `spending_by_category/awarding_agency` with location filter
3. Call `spending_by_award` for top awards in that state

### "What grants has Agency X awarded?"
1. Use `spending_by_award` with agency filter and grant type codes `["02","03","04","05"]`

### "Show me spending trends for X over time"
1. Call `spending_over_time` with `group: "fiscal_year"` for annual or `"month"` for monthly

### "Tell me about award ID X"
1. Call `GET /api/v2/awards/<ID>/` for full details

## Formatting Guidelines

- Format dollar amounts with commas and appropriate scale: `$1.23B`, `$456.7M`, `$89.1K`, `$1,234.56`
- Use markdown tables for structured results
- Bold the most important numbers
- When showing award lists, include: Award ID, Recipient, Amount, Agency, Description (truncated to ~100 chars)
- Always state the time period and filters used so the user knows the scope
- If results are paginated and there's more data, mention it and offer to fetch more pages

## Important Notes

- The API date range starts at 2007-10-01. Searches before this return no data.
- If no time period is specified, default to the current fiscal year (FY2026: Oct 2025 - Sep 2026) or a reasonable recent range based on the question.
- Keywords search across award descriptions, recipient names, and other text fields — they cast a wide net.
- The API is free and requires no authentication.
- For very broad searches, start with counts and category breakdowns before pulling individual awards.
- When the user mentions "DOGE" they likely mean the Department of Government Efficiency or related executive orders — search for it as a keyword.
- Always pipe curl output through `python3 -m json.tool` for readability, and use `head` to limit very long outputs.
- When making multiple related API calls, run them in parallel using separate Bash tool calls when possible.
