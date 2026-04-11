---
name: om-page
description: Generate real estate Offering Memorandum (OM) demographic/market summary pages using VestMap data. Creates styled HTML pages with interactive maps, charts, and Block/Tract/ZIP geographic comparisons.
user_invocable: true
---

# VestMap OM Page Generator

You are generating a real estate Offering Memorandum (OM) demographics/market summary page. You will query VestMap tools for all data and produce a single self-contained HTML file.

## User Input

The user provides:
- **Address** (required): A US property address (e.g., "112-120 E 43rd St, Kansas City, MO 64111")
- **Sections** (optional): Which sections to include. Default is ALL sections.
- **Design** (optional): A screenshot to match, or "vestmap" (default) for VestMap branding.
- **Custom data requests** (optional): Any additional data topics to include (e.g., "veteran population", "commute times", "poverty rates").
- **Additional geographic levels** (optional): County or State columns in addition to Block/Tract/ZIP.

## Core Rules

1. **Always Block/Tract/ZIP** — never 1/3/5 mile radius. If the user provides a source OM that uses 1/3/5 mile radius, translate that to Block Group / Tract / ZIP Code. County and State can be added as extra columns if requested.
2. **Always include a map** — Leaflet with ESRI World Street Map tiles. Every OM must have a map.
3. **Query fields ONE AT A TIME** — multi-field queries to `query_gis_field` often return "No data found". Always query each field individually.
4. **Never use Tapestry for specific metrics** — Tapestry is for lifestyle segmentation only. Median age, income, population, etc. must come from direct field queries (e.g., MEDAGE_CY, MEDHINC_CY, TOTPOP_CY).
5. **Never run DISCERN** — do NOT call `generate_vestmap_report`. Only query the specific data you need via `query_gis_field` and `get_section_data`.
6. **VestMap branding by default** — use the VestMap color palette and Inter font unless the user provides a different design to match.
7. **All data must be real** — every number comes from VestMap queries. Never fabricate or estimate data.
8. **Key Facts use Block/Tract/ZIP** — the Key Facts panel shows data at all three geographic levels for comparison, not just one level.
9. **Flexible data discovery** — if the user asks for data not in the predefined sections, use `search_real_estate_data` to find field names, then query at Block/Tract/ZIP.
10. **County comparison for income distribution** — the "Diff" column compares block-level income distribution to county-level data.

## ESRI Layer URLs

These are the primary layers for the Block/Tract/ZIP methodology:

| Level | Layer URL | MapServer |
|-------|----------|-----------|
| Block Group | `https://demographics5.arcgis.com/arcgis/rest/services/USA_Demographics_and_Boundaries_2024/MapServer/12` | /12 |
| Tract | `https://demographics5.arcgis.com/arcgis/rest/services/USA_Demographics_and_Boundaries_2024/MapServer/11` | /11 |
| ZIP Code | `https://demographics5.arcgis.com/arcgis/rest/services/USA_Demographics_and_Boundaries_2024/MapServer/9` | /9 |
| County | `https://demographics5.arcgis.com/arcgis/rest/services/USA_Demographics_and_Boundaries_2024/MapServer/7` | /7 |
| State | `https://demographics5.arcgis.com/arcgis/rest/services/USA_Demographics_and_Boundaries_2024/MapServer/3` | /3 |
| CBSA (MSA) | `https://services5.arcgis.com/9fQmObndozAJu9f5/arcgis/rest/services/Enriched_USA_Metropolitan_Statistical_Areas/FeatureServer/0` | N/A |
| FEMA NRI (Tract) | `https://services.arcgis.com/XG15cJAlne2vxtgt/arcgis/rest/services/National_Risk_Index_Census_Tracts/FeatureServer/0` | N/A |
| Crime (Block Group) | `https://demographics5.arcgis.com/arcgis/rest/services/USA_Crime_2024/MapServer/12` | /12 |

## Data Gathering Process

### Step 1: Query Block Group Data (MapServer/12)

Query each field individually using `query_gis_field`. Batch parallel calls where possible.

**Summary Table fields:**
- `TOTPOP_CY` — Total Population
- `TOTHH_CY` — Total Households
- `FAMHH_CY` — Family Households
- `AVGHHSZ_CY` — Average Household Size
- `OWNER_CY` — Owner Occupied Housing Units
- `RENTER_CY` — Renter Occupied Housing Units
- `MEDAGE_CY` — Median Age
- `MEDHINC_CY` — Median Household Income
- `AVGHINC_CY` — Average Household Income
- `PCI_CY` — Per Capita Income
- `MEDNW_CY` — Median Net Worth
- `MALES_CY` — Male Population
- `FEMALES_CY` — Female Population

**Employment fields:**
- `EMP_CY` — Employed Civilian Pop 16+
- `UNEMP_CY` — Unemployed Pop
- White Collar: `OCCMGMT_CY`, `OCCSALE_CY`, `OCCCOMP_CY`, `OCCEDUC_CY`, `OCCHLTH_CY`
- Services: `OCCFOOD_CY`, `OCCPROT_CY`, `OCCPERS_CY`, `OCCBLDG_CY`
- Blue Collar: `OCCCONS_CY`, `OCCFARM_CY`, `OCCPROD_CY`, `OCCTRAN_CY`

**Education fields (Pop 25+):**
- `NOHS_CY` — Less than 9th Grade
- `SOMEHS_CY` — 9th-12th Grade, No Diploma
- `HSGRAD_CY` — High School Diploma
- `SMCOLL_CY` — Some College, No Degree
- `ASSCDEG_CY` — Associate Degree
- `BACHDEG_CY` — Bachelor's Degree
- `GRADDEG_CY` — Graduate/Professional Degree

**Income Distribution fields (Household counts):**
- `HINC0_CY` — <$15,000
- `HINC15_CY` — $15,000-$24,999
- `HINC25_CY` — $25,000-$34,999
- `HINC35_CY` — $35,000-$49,999
- `HINC50_CY` — $50,000-$74,999
- `HINC75_CY` — $75,000-$99,999
- `HINC100_CY` — $100,000-$149,999
- `HINC150_CY` — $150,000-$199,999
- `HINC200_CY` — $200,000+

**Other fields:**
- `DIVINDX_CY` — Diversity Index
- `POPGRWCYFY` — Population Growth Rate (CAGR 2024-2029)

### Step 2: Query Tract Data (MapServer/11)

Same summary table fields as Block Group. Query each one individually.

### Step 3: Query ZIP Data (MapServer/9)

Same summary table fields as Block Group. Query each one individually.

### Step 4: Get Crime & Schools Data

Use `get_section_data`:
```
get_section_data(address, "crime")   → Total, Personal, Property crime indices + specific offenses
get_section_data(address, "schools") → Top 3 nearest schools with GreatSchools ratings + URLs
```

### Step 5: Query County Data for Income Distribution Comparison

Query income distribution at County level (MapServer/7) to compute the "Diff" column:
- `HINC0_CY` through `HINC200_CY` + `TOTHH_CY`

### Step 6: Query FEMA NRI (if Natural Hazards section requested)

Layer: `https://services.arcgis.com/XG15cJAlne2vxtgt/arcgis/rest/services/National_Risk_Index_Census_Tracts/FeatureServer/0`

Fields (each has _RISKV value, _RISKS score, _RISKR rating):
- `ERQK_RISKR` — Earthquake
- `HRCN_RISKR` — Hurricane
- `TRND_RISKR` — Tornado
- `DRGT_RISKR` — Drought
- `WFIR_RISKR` — Wildfire
- `RFLD_RISKR` — Riverine Flooding
- `CFLD_RISKR` — Coastal Flooding
- `AVLN_RISKR` — Avalanche
- `LNDS_RISKR` — Landslide
- `TSUN_RISKR` — Tsunami
- `HWAV_RISKR` — Heat Wave
- `CWAV_RISKR` — Cold Wave
- `ISTM_RISKR` — Ice Storm
- `LTNG_RISKR` — Lightning
- `HAIL_RISKR` — Hail
- `SWND_RISKR` — Strong Wind
- `VLCN_RISKR` — Volcanic Activity
- `WNTW_RISKR` — Winter Weather
- `SOVI_SCORE` — Social Vulnerability Score
- `RESL_SCORE` — Community Resilience Score
- `RISK_SCORE` — Overall Risk Score
- `RISK_RATNG` — Overall Risk Rating

### Step 7: Query Business Data (if Business section requested)

Layer: `https://services5.arcgis.com/9fQmObndozAJu9f5/arcgis/rest/services/Enriched_USA_Metropolitan_Statistical_Areas/FeatureServer/0`
- `N01_BUS` — Total Businesses
- `N01_EMP` — Total Employees

Note: Business data is only available at CBSA (metro area) level.

### Step 8: Flexible Data Discovery

If the user requests ANY data topic not covered above:
1. Call `search_real_estate_data(query="<user's topic keywords>", max_results=15)`
2. From the results, identify the field name and layer URL
3. Query it at Block Group (/12), Tract (/11), and ZIP (/9) levels
4. If a field isn't available at a given level, check the search results for alternative layers at that level
5. Present the data in a new panel or add rows to the summary table

## Map Integration

Every OM page MUST include an interactive Leaflet map with ESRI World Street Map tiles.

### Geocoding

Use the Nominatim API to geocode the address. Include this in the HTML:
```javascript
// Geocode and initialize map
fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(ADDRESS)}`)
  .then(r => r.json())
  .then(data => {
    if (data.length > 0) {
      const lat = parseFloat(data[0].lat);
      const lng = parseFloat(data[0].lon);
      initMap(lat, lng);
    }
  });
```

OR if you know the coordinates, hardcode them directly for reliability.

### Map Configuration
```javascript
const map = L.map('map', {
  zoomControl: false,
  attributionControl: true,
  scrollWheelZoom: false,
  dragging: false
}).setView([lat, lng], 13);

// ESRI World Street Map tiles
L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}', {
  attribution: 'Tiles &copy; Esri &mdash; Source: Esri, HERE, Garmin, USGS, NGA',
  maxZoom: 19
}).addTo(map);

// Custom teal pin marker
const pinIcon = L.divIcon({
  html: `<svg xmlns="http://www.w3.org/2000/svg" width="36" height="50" viewBox="0 0 36 50">
    <path d="M18 0C8.06 0 0 8.06 0 18c0 13.5 18 32 18 32s18-18.5 18-32C36 8.06 27.94 0 18 0z" fill="#0D4F47"/>
    <circle cx="18" cy="18" r="8" fill="#fff"/>
  </svg>`,
  iconSize: [36, 50],
  iconAnchor: [18, 50],
  className: ''
});

L.marker([lat, lng], { icon: pinIcon }).addTo(map);
```

### Leaflet Dependencies
```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

## VestMap Default Branding

Use this color palette and typography when no alternative design is specified:

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap');

:root {
  --primary: #0D4F47;        /* Deep Teal - headers, CTAs, hero backgrounds */
  --accent: #1A7A6E;         /* Mid Teal - subheadings, icons */
  --interactive: #3CB8A8;    /* Interactive Teal - buttons, hover states, links */
  --bg-dark: #0F1A19;        /* Dark sections, hero overlays */
  --bg-light: #EDF2F7;       /* Alternating sections, light backgrounds */
  --white: #FFFFFF;          /* Default background */
  --text-body: #2D3748;      /* Body text */
  --text-secondary: #718096; /* Secondary text, labels */
  --success: #2ECC71;        /* Success/CTA accent, positive trends */
  --negative: #c0392b;       /* Negative trends, decline */
  --border: #d6dfe8;         /* Panel borders, dividers */
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  -webkit-font-smoothing: antialiased;
}
```

**Typography scale:**
- Page title: 32px, weight 900, uppercase, letter-spacing 2px
- Address: 15px, weight 400, rgba(255,255,255,0.8)
- Panel titles: 13px, weight 800, uppercase, letter-spacing 1.5px
- Table headers: 11px, weight 700, uppercase, letter-spacing 0.8px
- Table body: 13px, weight 500, tabular-nums
- Metric values: 28px, weight 800
- Labels: 10px, weight 600, uppercase, letter-spacing 0.5px
- Footer: 10px, rgba(255,255,255,0.6)

## HTML Template Structure

The page uses a fixed 1020px container for print-friendliness:

```
HEADER (--primary background)
  H1: "DEMOGRAPHICS" (or custom title)
  Address text

TOP SECTION (CSS Grid: 480px | 1fr)
  LEFT: Summary Table
    Columns: [Metric] [Block] [Tract] [Zip] (+ County/State if requested)
    Rows: Population, Households, Families, Avg HH Size, Owner Occ, Renter Occ,
          Median Age, Median HH Income, Avg HH Income, Per Capita Income, Median Net Worth
  RIGHT: Leaflet Map (min-height 380px)

PANELS (CSS Grid: 1fr 1fr, with borders)
  Panel 1: KEY FACTS
    - 2x2 grid showing key metrics at Block/Tract/ZIP
    - Circle badges for featured values (median age, unemployment)
    
  Panel 2: EDUCATION
    - 2x2 grid: No HS Diploma %, HS Graduate %, Some College/Assoc %, Bachelor's+ %
    
  Panel 3: BUSINESS (CBSA-level)
    - Total Businesses | Total Employees
    
  Panel 4: EMPLOYMENT
    - Icon rows: White Collar %, Blue Collar %, Services %
    - Circle badge: Unemployment Rate %

  Panel 5: INCOME (full width, grid-column: 1 / -1)
    LEFT: Income highlights (Median HH Income, Per Capita Income, Median Net Worth)
    RIGHT: Income Distribution table
      - Columns: Income Range | % | Diff from County
      - Color bars for positive/negative deviation

ADDITIONAL PANELS (if requested):
  Crime | Schools
  Natural Hazards | Pop & Housing Growth
  Housing Market | Diversity

FOOTER (--primary background)
  "Data Source: VestMap / Esri Demographics 2024"
  "Block / Tract / Zip methodology"
```

## Occupation Category Mapping

White Collar = Management/Business/Financial (OCCMGMT) + Sales (OCCSALE) + Computer/Engineering/Science (OCCCOMP) + Education/Legal/Arts (OCCEDUC) + Healthcare (OCCHLTH)

Services = Food Prep/Serving (OCCFOOD) + Protective Service (OCCPROT) + Personal Care (OCCPERS) + Building/Grounds (OCCBLDG)

Blue Collar = Construction/Extraction (OCCCONS) + Farming/Fishing/Forestry (OCCFARM) + Production (OCCPROD) + Transportation/Moving (OCCTRAN)

Unemployment Rate = UNEMP_CY / (EMP_CY + UNEMP_CY) * 100

## Education Category Mapping

No HS Diploma = NOHS_CY + SOMEHS_CY
HS Graduate = HSGRAD_CY
Some College / Associate = SMCOLL_CY + ASSCDEG_CY
Bachelor's & Higher = BACHDEG_CY + GRADDEG_CY
Total 25+ = sum of all education fields

## Output

Save the generated HTML file to the current working directory with a descriptive name:
`demographics-{address-slug}.html` or `om-{address-slug}.html`

Then serve it via the preview server and verify it renders correctly with:
- Map displaying with correct pin location
- All data populated from VestMap queries
- Charts/visualizations rendering
- VestMap branding applied
- No console errors
