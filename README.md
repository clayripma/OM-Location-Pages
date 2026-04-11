# OM Location Pages

A Claude Code skill that generates real estate Offering Memorandum (OM) demographic and market summary pages using VestMap data.

## What it does

Generates self-contained HTML pages with:

- **Demographics summary** — Population, households, income, age at Block Group / Tract / ZIP levels
- **Interactive map** — Leaflet with ESRI World Street Map tiles and property pin
- **Education breakdown** — Attainment levels with percentage distributions
- **Employment data** — White collar, blue collar, services split with unemployment rate
- **Income distribution** — Household income brackets with county comparison
- **Crime & schools** — Crime indices and nearby school ratings
- **Natural hazards** — FEMA National Risk Index data
- **Business data** — CBSA-level business and employee counts

All data is queried live from VestMap / ESRI Demographics 2024. No fabricated numbers.

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- VestMap MCP server configured in your Claude Code settings

## Usage

```
/om-page
```

Provide an address and optionally specify which sections to include, a design to match, or custom data topics.

### Example

```
Address: 112-120 E 43rd St, Kansas City, MO 64111
Sections: all
Design: vestmap
```

## Methodology

Uses **Block Group / Tract / ZIP Code** geographic levels for all comparisons — never 1/3/5 mile radius. County and State can be added as extra columns on request.

## Installation

Clone this repo and the skill will be available when running Claude Code from this directory:

```bash
git clone https://github.com/clayripma/om-location-pages.git
cd om-location-pages
claude
```

Then use `/om-page` to invoke the skill.
