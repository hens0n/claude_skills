---
name: sam-gov-search
description: Search SAM.gov (System for Award Management) for federal contract opportunities, assistance listings, entity information, and wage determinations using the browser. Use when the user wants to find government contracts, RFPs, RFIs, solicitations, or any federal procurement opportunities.
argument-hint: [search keywords] [optional: domain] [optional: notice type]
allowed-tools: mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_click, mcp__plugin_playwright_playwright__browser_type, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_press_key, mcp__plugin_playwright_playwright__browser_select_option, mcp__plugin_playwright_playwright__browser_wait_for, mcp__plugin_playwright_playwright__browser_close, mcp__plugin_playwright_playwright__browser_take_screenshot, mcp__plugin_playwright_playwright__browser_evaluate
---

# SAM.gov Search Skill

Search the federal government's System for Award Management (SAM.gov) for contract opportunities and other procurement data using the browser.

## How to Execute a Search

### Step 1: Parse User Input

Extract the following from `$ARGUMENTS`:
- **Keywords**: The search terms (required)
- **Domain**: Which SAM.gov domain to search (default: Contract Opportunities)
- **Notice Type**: Filter by notice type if specified
- **Status**: Active (default) or Inactive

### Step 2: Navigate to SAM.gov Search

Open the browser and navigate to SAM.gov search:

```
https://sam.gov/search/?page=1&pageSize=25&sort=-modifiedDate&sfm%5BsimpleSearch%5D%5BkeywordRadio%5D=ALL&sfm%5Bstatus%5D%5Bis_active%5D=true
```

### Step 3: Select Domain

Click the "Select Domain" button and choose the appropriate domain from the dropdown:

| User Request | Domain to Select |
|---|---|
| contracts, opportunities, RFP, RFI, solicitation (default) | Contract Opportunities |
| grants, assistance, CFDA | Assistance Listings |
| entities, vendors, companies, registrations | Entity Information |
| federal hierarchy, agencies, organizations | Federal Hierarchy |
| wages, wage determination, labor rates | Wage Determinations |
| everything, all | All Domains |

### Step 4: Enter Keywords

1. Find the keyword search textbox (labeled "keyword-text")
2. Type the search keywords
3. Press Enter to submit

### Step 5: Apply Additional Filters (if requested)

**Notice Type** (Contract Opportunities domain only) - click "Notice Type" filter button, then select from:
- Special Notice
- Sources Sought
- Presolicitation
- Consolidate/(Substantially) Bundle
- Solicitation
- Combined Synopsis/Solicitation
- Award Notice
- Justification
- Sale of Surplus Property

**Status** - checkboxes for Active and/or Inactive

### Step 6: Wait for and Extract Results

After search executes, take a snapshot and extract results. For each result, capture:

- **Title** (with link URL)
- **Notice ID**
- **Description snippet**
- **Department/Agency**
- **Subtier organization**
- **Office**
- **Response/Offers Due Date**
- **Notice Type**
- **Updated Date**
- **Published Date**

### Step 7: Present Results

Present results in a clean, structured format:

```
## SAM.gov Search Results: "[keywords]"
**Domain:** Contract Opportunities | **Results:** X total | **Showing:** 1-25

---

### 1. [Title]
- **Notice ID:** XXXXX
- **Type:** Sources Sought
- **Agency:** Department of Defense > Navy > NAVSUP
- **Due Date:** March 15, 2026
- **Published:** Feb 6, 2026
- **Description:** [snippet...]
- **Link:** https://sam.gov/workspace/contract/opp/[id]/view

---
```

### Step 8: Offer Follow-up Actions

After presenting results, offer:
1. **View details** - Click into a specific result to get full description, contact info, attachments
2. **Next page** - Navigate to the next page of results
3. **Refine search** - Add/change filters
4. **Download attachments** - View available documents on a detail page

## Detail Page Navigation

When the user wants to see details for a specific result, click its title link. On the detail page, extract:

- **Full description** text
- **Classification**: Set Aside, Product Service Code, NAICS Code, Place of Performance
- **Contact Information**: Primary and Alternative POCs (name, email, phone)
- **Contracting Office Address**
- **Attachments**: List of available documents with file sizes
- **Response/Due Date**

## Tips for Better Results

- Use "All Words" mode (default) for precise matches
- The site sorts by "Updated Date" by default - can change via the Sort dropdown
- Active status is checked by default
- The keyword search supports Boolean-style operators when using "Search Editor" tab
- Contract Opportunities is the most commonly searched domain

## Important Notes

- SAM.gov is a U.S. government website - treat all data as public information
- Some attachments may require sign-in to access (marked as "Controlled Unclassified")
- The site may show maintenance alerts - these can be dismissed
- Results show 25 per page by default (can be changed to 50 or 100)
