---
name: vestmap-om-pages
description: Render a property Offering Memorandum (OM) page for any US address — a visual, page-oriented PDF (default output) showing demographics, income, housing, workforce, and safety at Block / ZIP / City / County scales with explicit cross-scale comparisons. Use when the user asks for an "OM", "one-pager", "investor page", "property page", "marketing page", or a rendered / laid-out visual for a property. The page is market-agnostic — no city-specific wording, no carve-outs. Block-level data is added by default. Optional modules (Risk, Schools, Education detail, HPI, full income distribution, all 13 occupations, Business/MSA) are NOT rendered by default; they appear only on explicit request. Delegates all Block / Tract / ZIP / County data acquisition to the `vestmap` skill and inherits its R7 / R9 / R10 / R11 / R13 rules.
user_invocable: true
---

# VestMap OM Pages Skill

Render Offering Memorandum pages for US addresses. Single-address, page-oriented, PDF-first output.

This skill is a **rendering + data-curation layer on top of the `vestmap` skill**. It defines exactly which fields appear by default, how values flow from source data into the page, and how the page is laid out so comparisons across scales are legible. It does NOT define new data rules — those live in the parent `vestmap` skill.

## When to use this skill

Trigger on any of:
- User asks for an "OM", "offering memorandum", "one-pager", "investor page", "marketing page", "property page", "property brief page"
- User asks to "render" or "lay out" demographic / market data for a property as a page
- User asks for a rendered / formatted page for any US address
- User asks for a comparison page across Block / ZIP / City / County

Do NOT trigger for: plain ranking tables, CSV dumps, one-liners, "what's the median income here", "which ZIP has…". Those stay on the base `vestmap` skill.

## Inheritance from `vestmap`

Everything the parent skill enforces still applies. In particular:

- **R5 (never use Tapestry for hard metrics)** — OM numbers come from `get_section_data("income"|"expansion"|"crime"|"schools")` and `query_gis_field` Tier 1 fields. NEVER from `get_section_data("demographics")`. NO Tapestry segment names on the page (no "Tapestry Grade", no lifestyle pill).
- **R7 (explicit quantitative cross-scale comparisons)** — every multi-scale metric must show deltas/ratios, not just values. The 4-column + chip layout enforces this.
- **R9 (blank fields are omitted, never filled)** — if a VestMap call returned null, the cell disappears. Never print "N/A", "—", "data unavailable", etc.
- **R10 (no qualitative analysis beyond the numbers)** — no "desirable", "up-and-coming", "affluent". Numbers only.
- **R11 (computed metrics are precondition-gated)** — no delta shown when a component is null.
- **R13 (verify discovery-vs-field for decision-grade metrics)** — forecasted growth rates cross-check the canonical `_CY` / `_FY` field before rendering.

Always read `vestmap/SKILL.md`, `vestmap/references/fields.md`, and `vestmap/references/gotchas.md` before acquiring data. Do not duplicate those rules here — follow them.

## Reference Files

| File | When to load |
|---|---|
| `references/data-spec.md` | Always. Defines which fields are on the OM by default, their data source at each scale, and the opt-in modules. |
| `references/layout.md` | Always. Defines section order, comparison viz, typography, styling tokens, PDF export, and what NOT to do. |
| `templates/om-page.html` | Always. Self-contained HTML+CSS template with named slots. |

## Output Rules (hard constraints)

**O1. PDF is the default output.** Write the HTML to a temp path, then convert to PDF and print the PDF path to the user. Do NOT stream HTML in chat. Do NOT offer HTML by default. Only provide HTML instead of PDF if the user explicitly says "HTML" or "html only".

See `layout.md §9 PDF export` for the exact headless-Chrome command.

**O2. The rendered page NEVER mentions missing data, failed calls, or dropped modules.** This is the strictest output rule in the skill. The page does not contain:
- "N/A", "—", "No data", "data unavailable", "not available", "unknown"
- "row dropped", "omitted", "skipped", "conserve layer calls"
- R13 divergence explanations
- Lists of which optional modules were added / excluded
- Tool-call names, layer IDs, field names

If a value is missing, the cell is invisible (CSS `display:none`). If a row has fewer than 2 non-null cells, the row is removed from the DOM. If a section has fewer than 2 rows, the section is removed. Everything silently gets smaller.

**O3. Never fabricate a value to fill a cell.** Skip vs. make-up: always skip.

**O4. NO Tapestry anywhere on the page.** Per R5. No "Tapestry Grade: X" pill, no segment name, no "lifestyle" callout. Tapestry is forbidden from the rendered output even as decorative flavor.

**O5. NO market-specific / city-specific hardcoded wording.** The rendered page reads identically regardless of market. The only places a market name appears are the subject address, the locality line, and the MSA callout — all three data-driven from the geocoded subject. No template flavor text naming any city, no region-specific submarket taxonomies as headings.

**O6. The header does NOT say "Offering Memorandum".** No eyebrow text above the address. No document-type label. Just the address as the H1. The user wants this. Do not add it back.

**O7. The context band (3 big-number callouts at top of page 1) is FIXED in content:**
1. **Median HHI at Block** (label: "Median HHI · Block"; value: from `get_section_data("income").median_household_income.block`)
2. **Population growth % at ZIP** (label: "Pop Growth · ZIP, 5-yr CAGR"; value: from `get_section_data("expansion").zip`)
3. **MSA name** (label: "MSA"; value: MSA/CBSA name from `search_real_estate_data("CBSA")` or `Enriched_USA_Metropolitan_Statistical_Areas/FeatureServer/0`)

Do NOT substitute. Do NOT add a crime / safety / risk callout to the header. Do NOT use ZIP-level HHI if Block is missing (drop the whole callout cell instead — O2 + R9).

**O8. Map is default-on.** Every OM has the Leaflet+ESRI hero map rendered unless geocoding literally fails (in which case the `.hero__map` div is removed). Do not make the user opt in to the map.

**O9. Self-contained HTML before PDF conversion.** All CSS inline. The only external asset is the Leaflet CDN + ESRI tile layer. Before you open the HTML in headless Chrome to export the PDF, the HTML must be able to open standalone in a browser.

**O10. Chat response after generation is minimal.** After writing the PDF:
- Print the PDF path
- One sentence stating the subject address
- Stop. Do not list which sections rendered vs dropped. Do not apologize for missing data. Do not announce which opt-in modules were / weren't added unless the user explicitly asked for specific modules and one genuinely didn't materialize.

## Default sections (always rendered when data is present)

1. **Header** — full-bleed dark-green page-header strip with address (see `layout.md §Styling`), followed by a two-up row: summary table + map.
2. **Context band** — 3 big-number callouts per O7 (HHI Block · Pop Growth ZIP · MSA).
3. **Population** — 4-scale comparison: total pop, median age, density, 5-yr pop growth, avg HH size.
4. **Income** — 4-scale: median HHI, 2029 HHI forecast, 5-yr HHI growth, per-capita income, unemployment rate.
5. **Housing Values** — 4-scale: median home value, 2029 forecast, 5-yr growth.
6. **Rental Market** — 4-scale: median rent, renter units, owner units, 2029 renter units, renter share.
7. **Workforce** — 4-scale: 4 occupation buckets (Mgmt / Sales / Prod / Cons) + white/blue collar share + per-scale stacked bars.
8. **Safety** — Block Group raw crime counts from VestMap (3×3 grid). Per-capita line only when `TOTPOP_CY` at Block returned.

See `data-spec.md` for exact field-level data sources and `layout.md` for visual rules.

## Opt-in modules (never rendered by default)

Added only when the user explicitly names them:

| Trigger phrase | Module |
|---|---|
| "include risk", "FEMA", "hazards" | FEMA NRI (Tract only) |
| "include schools" | Nearest 3 schools + district |
| "education", "degree attainment" | 5-bucket education by scale |
| "income distribution", "HINC buckets" | 9-bucket income distribution by scale |
| "all occupations", "13 occupation buckets" | Full 13-bucket workforce |
| "businesses", "MSA data" | Business counts (CBSA) |
| "HPI" | House Price Index |

Details in `data-spec.md §Optional Modules`.

## Workflow

1. **Parse.** Extract subject address. Identify subject ZIP.
2. **Data acquisition** — run in parallel (see `data-spec.md §Block Acquisition`):
   - Required always: `get_section_data(address, "income" | "expansion" | "crime")`, Tier-1 `query_gis_field` batches at `/12` (Block), `/11` (Tract), `/9` (ZIP), `/7` (County).
   - MSA lookup: `search_real_estate_data("CBSA")` once per session → query CBSA name + N01_BUS/N01_EMP fields (the name is used in the context band; the business counts are only kept if the user opted in to the Business module).
   - Optional modules: only if triggered.
3. **Compute deltas.** For every metric with ≥2 non-null scale values, compute absolute + ratio/percentage deltas per `layout.md §2`.
4. **R9/R11 sweep.** Remove empty cells, empty rows, empty sections from the DOM before rendering.
5. **Render HTML** to a temp file.
6. **Convert to PDF** via headless Chrome (`layout.md §9`). Output path: `vestmap-om-{zip}-{YYYYMMDD-HHMMSS}.pdf` in the current working directory.
7. **Respond.** Print the PDF path, one sentence naming the subject address. Stop.

## Quick Reference

| User says | Do this |
|---|---|
| "OM for 123 Main St, Denver CO" | Default OM, PDF output. |
| "OM for <address>, include risk and schools" | Default + Risk + Schools modules. |
| "OM for <address>, HTML only" | HTML-only output, skip PDF conversion. |
| "Full OM for <address> with everything" | Default + all opt-in modules. |

## What this skill deliberately does NOT do

- Does not define new data rules.
- Does not publish ranking tables / CSVs / briefs / maps-only.
- Does not ask for confirmation before running (F6).
- Does not render Tapestry-derived values anywhere on the page.
- Does not mention missing data, failed calls, R13 divergences, or dropped sections in the rendered output OR the chat response.
- Does not use market-specific wording. The rendered page is layout-identical across every US market.
