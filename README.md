# OM Location Pages

Generate presentation-ready Offering Memorandum (OM) location pages from any US address using Claude Code and VestMap.

One command. Real data. A polished, self-contained HTML page you can drop into any deal package.

## What you get

Each page pulls live data from 1,000+ ESRI, AGS, FHFA, FEMA, and US Census GIS fields and renders it into a single HTML file with:

- **Demographics** — Population, households, income, median age, and net worth compared across Block Group, Tract, and ZIP
- **Interactive map** — Leaflet map with ESRI World Street Map tiles pinned to the property
- **Education** — Attainment distribution from no diploma through graduate degree
- **Employment** — White collar / blue collar / services breakdown with unemployment rate
- **Income distribution** — Household income brackets benchmarked against the county
- **Crime & schools** — AGS Crime Indexes and nearby school ratings
- **Natural hazards** — FEMA National Risk Index scores for flood, wildfire, earthquake, hurricane, tornado, and 14 other hazard types
- **Business activity** — CBSA-level business and employee counts

Every number comes from a live VestMap query. Nothing is estimated or fabricated.

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- A VestMap account — [sign up here](https://app.vestmap.com/mcp)
- VestMap MCP server connected in your Claude Code settings

## Usage

```
/vestmap-om-pages
```

Provide a US address. Optionally specify which sections to include, a design to match, or custom data topics.

```
Address: 112-120 E 43rd St, Kansas City, MO 64111
```

## Methodology

All comparisons use **Block Group / Tract / ZIP Code** geographic levels — not 1/3/5 mile radius. County and State columns can be added on request.

## Installation

```bash
git clone https://github.com/clayripma/om-location-pages.git
cd om-location-pages
claude
```

Then run `/vestmap-om-pages` to generate a page.

## About VestMap

VestMap brings nationwide location intelligence to AI agents. Turn any US address into a complete picture of the people, economy, housing market, and risk profile around it — demographics, income and wealth, employment, median rent, home values, FHFA House Price Index trends, crime, schools, growth projections, lifestyle segments, and FEMA natural hazard exposure.

Data is sourced from ESRI, AGS, FHFA, FEMA, and the US Census at every geographic level from block group to national. Use it to build OMs, run site selection, underwrite investments, compare markets, assess climate risk, or answer any location question grounded in current, authoritative data.

[Get a VestMap account](https://app.vestmap.com/mcp)
