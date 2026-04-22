# OM Data Specification

The authoritative list of what data goes on the OM, where each value comes from, and how it's labeled. If a field isn't listed here, it does not appear on a default OM.

Scale convention: **Block · ZIP · City · County** (CSV-backed ZIPs) or **Block · Tract · ZIP · County** (VestMap-direct, when no CSV row exists).

**Render modes are invisible to the reader.** The page layout, typography, and colors are identical regardless of whether the ZIP had a CSV row. Never label the output as "Mode A / B", never mention whether a CSV was found, never cite specific markets. See `SKILL.md §O2, O5`.

---

## Section 0 — Hero (identifiers)

Top of page. No cross-scale comparison.

| OM label | CSV column | VestMap-direct source | Notes |
|---|---|---|---|
| Address (H1) | user-supplied | user-supplied | Exactly as given. NO eyebrow label above (no "Offering Memorandum", no "OM"). |
| City, State · County · ZIP (locality line) | `city`, `county`, `zip_code` | geocode lookup | |
| Submarket pill | `submarket` | — | Render only if CSV row has a non-null submarket. Never for non-CSV ZIPs. |
| Map | — | Leaflet + ESRI tile layer | **Default-on.** Remove the `.hero__map` div only if geocoding fails. |

Not on the hero: Tapestry pill, opportunity score badge, SV rank badge, safety pill. None of these are in the header by default (per user spec).

---

## Context band (3 big-number callouts)

Hard-coded slot contents — see `SKILL.md §O7`. Do NOT substitute.

| Slot | Label | Value | Source |
|---|---|---|---|
| 1 | `Median HHI · Block` | `${{HHI_BLOCK}}` | `get_section_data(address, "income").median_household_income.block` |
| 2 | `Pop Growth · ZIP, 5-yr CAGR` | `{{POPGR_ZIP}}%` | `get_section_data(address, "expansion").zip` |
| 3 | `MSA` | `{{MSA_NAME}}` (verbatim) | See §MSA Lookup below |

**R9 per-slot:** if Block HHI returned null, remove slot 1 entirely (do NOT fall back to ZIP HHI). If ZIP pop growth returned null, remove slot 2. If MSA lookup fails, remove slot 3. The context-band grid simply becomes 2 or 1 columns. Never say "data unavailable".

### MSA Lookup

**Layer:** `https://services5.arcgis.com/9fQmObndozAJu9f5/arcgis/rest/services/Enriched_USA_Metropolitan_Statistical_Areas/FeatureServer/0`

**Discovery:** `search_real_estate_data("Metropolitan Statistical Area")` once per session to confirm the layer URL and the MSA-name field (typical field names: `NAME`, `NAMELSAD`, or `MSA_NAME` — verify which one returns the full name string).

**Query:** `query_gis_field(address, <layer>, ["NAME"])` (or whichever field the discovery call identifies). Use the returned string verbatim — do not truncate, do not rename, do not abbreviate. Examples of MSA names the user expects to see as-is: `"Kansas City, MO-KS"`, `"San Jose-Sunnyvale-Santa Clara, CA"`, `"Denver-Aurora-Centennial, CO"`.

If the address is not inside any CBSA / MSA polygon (rural, outside metros), the slot is dropped (R9).

---

## Section 1 — Market Demand (CSV-backed ZIPs only)

Single column — no cross-scale comparison. These are external SEO/demand signals, ZIP-level.

**If subject ZIP has no CSV row: OMIT the entire section silently.** No message, no placeholder.

| OM label | CSV column |
|---|---|
| "real estate" SV / mo | `sv_real_estate` |
| "apartments for sale" SV | `sv_apartments_for_sale` |
| "homes for sale" SV | `sv_homes_for_sale` |
| Total monthly SV | `total_monthly_sv` |
| ZIP demand rank | `sv_rank` |
| Multifamily opportunity score | `multifamily_opportunity_score` |

**Rank display is generic.** The template says `#{{SV_RANK}}` — no "of N", no "in KC". If the CSV grows or shrinks, nothing breaks.

**If all SV columns are 0:** collapse to one line: `No significant organic search demand for this ZIP.` Keep the opportunity-score gauge.

---

## Section 2 — Population Profile

4 columns.

| OM label | CSV columns | Block source | VestMap-direct non-Block source |
|---|---|---|---|
| Total population | `total_population_{zip,city,county}` | `query_gis_field(TOTPOP_CY, /12)` — Tier 2 | `query_gis_field(TOTPOP_CY, /11 /9 /7)` |
| Median age | `median_age_{zip,city,county}` | `query_gis_field(MEDAGE_CY, /12)` — Tier 3 (drop cell if null) | same at `/11 /9 /7` |
| Density (per sq mi) | `population_density_{zip,city,county}` | Block blank (R9) | non-CSV scales blank unless `search_real_estate_data("density")` surfaces a queryable field |
| 5-yr population growth (CAGR) | `population_growth_pct_{zip,city,county}` | `get_section_data(address, "expansion").block` | `get_section_data(..., "expansion").tract/.zip/.county` |
| Avg household size | `avg_household_size_{zip,city,county}` | `query_gis_field(AVGHHSZ_CY, /12)` — Tier 3 (drop if null) | same at `/11 /9 /7` |

---

## Section 3 — Income Profile

4 columns.

| OM label | CSV columns | Block source | VestMap-direct non-Block source |
|---|---|---|---|
| Median HHI | `median_hhi_{zip,city,county}` | `get_section_data("income").median_household_income.block` | `.tract/.zip/.county` |
| HHI 2029 (forecast) | `median_hhi_2029_{zip,city,county}` | `query_gis_field(MEDHINC_FY, /12)` — Tier X, drop if null | same at `/11 /9 /7` |
| 5-yr HHI growth | `hhi_growth_5yr_pct_{zip,city,county}` | `query_gis_field(MHIGRWCYFY, /12)` — R13 mandatory. NEVER use `get_section_data("income").annual_forecasted_median_income_growth` per `vestmap/references/gotchas.md`. | `query_gis_field(MHIGRWCYFY, /11 /9 /7)` |
| Per capita income | `per_capita_income_{zip,city,county}` | `query_gis_field(PCI_CY, /12)` — Tier 3, drop if null | same at `/11 /9 /7` |
| Unemployment rate | `unemployment_rate_{zip,city,county}` | `query_gis_field(UNEMPRT_CY, /12)` Tier 2, fallback compute from `UNEMP_CY / (EMP_CY + UNEMP_CY)` | same |

---

## Section 4 — Housing Values

4 columns.

| OM label | CSV columns | Block source | VestMap-direct non-Block |
|---|---|---|---|
| Median home value | `median_home_value_{zip,city,county}` | `get_section_data("income").median_home_value.block` | `.tract/.zip`; `query_gis_field(MEDVAL_CY, /7)` for County |
| HV 2029 (forecast) | `median_home_value_2029_{zip,city,county}` | `query_gis_field(MEDVAL_FY, /12)` — Tier X | same at `/11 /9 /7` |
| 5-yr HV growth | `home_value_growth_5yr_pct_{zip,city,county}` | compute `((MEDVAL_FY/MEDVAL_CY)^(1/5) − 1) × 100` — R11 gated | same at `/11 /9 /7` |

If `median_home_value_zip` is 0 in the CSV (industrial ZIPs with no residential stock), omit the row. No message.

---

## Section 5 — Rental Market

4 columns.

| OM label | CSV columns | Block source | VestMap-direct non-Block |
|---|---|---|---|
| Median rent | `median_rent_{zip,city,county}` | `query_gis_field(MEDRENT_CY, /12)` — Tier X. Alt: `MEDCRNT_CY`. Drop if both null. | same at `/11 /9 /7` |
| Renter units | `renter_units_{zip,city,county}` | `query_gis_field(RENTER_CY, /12)` Tier 3, drop if null | same at `/11 /9 /7` |
| Owner units | `owner_units_{zip,city,county}` | `query_gis_field(OWNER_CY, /12)` Tier 3, drop if null | same at `/11 /9 /7` |
| 2029 renter units | `renter_units_2029_{zip,city,county}` | `query_gis_field(RENTER_FY, /12)` Tier X | same at `/11 /9 /7` |
| Renter share | `renter_share_pct_{zip,city,county}` | compute `RENTER_CY / (RENTER_CY + OWNER_CY) × 100` — R11 gated | same at `/11 /9 /7` |

---

## Section 6 — Workforce

4 columns. Default renders 4 occupation buckets + collar shares. Full 13-bucket is opt-in.

| OM label | CSV columns | Block source | VestMap-direct non-Block |
|---|---|---|---|
| Management | `mgmt_occupations_{zip,city,county}` | `query_gis_field(OCCMGMT_CY, /12)` Tier 1 | same at `/11 /9 /7` |
| Sales | `sales_occupations_{zip,city,county}` | `query_gis_field(OCCSALE_CY, /12)` Tier 1 | same |
| Production | `production_occupations_{zip,city,county}` | `query_gis_field(OCCPROD_CY, /12)` Tier 1 | same |
| Construction | `construction_occupations_{zip,city,county}` | `query_gis_field(OCCCONS_CY, /12)` Tier 1 | same |
| White-collar share | `white_collar_share_pct_{zip,city,county}` | compute from 13 OCC fields | same |
| Blue-collar share | `blue_collar_share_pct_{zip,city,county}` | compute from 13 OCC fields | same |

**Collar share formula:** White = MGMT+SALE+COMP+EDUC+HLTH. Blue = CONS+FARM+PROD+TRAN. Services = FOOD+PROT+PERS+BLDG. Share% = bucket / (WC+BC+Services) × 100. R11 gated at each scale.

---

## Section 7 — Safety / Crime

Two representations, NOT merged:

### 7a. Crime Index (ZIP · City · State)

Only from CSV. If `crime_total_index_zip` null → OMIT entire sub-block.

| OM label | CSV column |
|---|---|
| Total (ZIP) | `crime_total_index_zip` |
| Personal (ZIP) | `crime_personal_index_zip` |
| Property (ZIP) | `crime_property_index_zip` |
| City total | `crime_total_index_city` |
| State total | `crime_total_index_state` |
| ZIP vs City ratio chip | `crime_zip_vs_city_ratio` |
| ZIP vs State ratio chip | `crime_zip_vs_state_ratio` |

**No safety label pill in the hero.** The CSV safety_label column is not rendered by default.

### 7b. Block Group raw counts

From `get_section_data(address, "crime")` → 3×3 grid of raw counts. Hidden cells if null. Per-capita line only if `TOTPOP_CY` at Block returned (R11).

---

## Optional Modules (opt-in only — never rendered by default)

### Natural Hazard Risk (FEMA NRI)
- **Trigger:** "include risk", "FEMA", "hazards"
- **Source:** `search_real_estate_data("National Risk Index")` → Tract FeatureServer
- **Payload:** `RISK_SCORE`, `RISK_RATNG`, `SOVI_SCORE`, `RESL_SCORE`, top 5 hazards by `_RISKS` score. Omit null-rated hazards (R9).
- **Scale:** Tract only.

### Schools
- **Trigger:** "include schools"
- **Source:** `get_section_data(address, "schools")`
- **Payload:** district + 3 nearest schools with rating + URL if returned.

### Education breakdown (5 buckets)
- **Trigger:** "education", "degree attainment"
- **Source:** `query_gis_field` on `NOHS_CY`, `HSGRAD_CY`, `SMCOLL_CY`, `BACHDEG_CY`, `GRADDEG_CY` at `/12 /11 /9 /7`
- **Layout:** 5-segment stacked bar per scale; R11 gated.

### Full Income Distribution (9 HINC buckets)
- **Trigger:** "income distribution", "HINC buckets"
- **Source:** 9 `HINC*_CY` + `TOTHH_CY` at `/12 /11 /9 /7`
- **Layout:** 9-segment stacked bar per scale; R11 gated.

### All 13 occupation buckets
- **Trigger:** "all occupations", "13 occupation buckets"
- **Source:** all 13 `OCC*_CY` at `/12 /11 /9 /7`
- **Layout:** replaces the 4-bar Workforce chart with 13-segment.

### Business / MSA
- **Trigger:** "businesses", "business count"
- **Source:** `N01_BUS`, `N01_EMP` on the MSA FeatureServer (same layer used for the context-band MSA name).
- **Layout:** single-row callout. No scale comparison.

### HPI
- **Trigger:** "HPI", "house price index"
- **Source:** `get_section_data(address, "hpi")` — ~90% reliable.

---

## Block Acquisition — one-shot parallel batch

For every OM (CSV-backed or VestMap-direct), fire these in parallel (single turn):

1. `get_section_data(address, "income")` — Block HHI, home value, forecasted income growth (discovery)
2. `get_section_data(address, "expansion")` — Block pop growth CAGR
3. `get_section_data(address, "crime")` — Block Group crime counts
4. `search_real_estate_data("Metropolitan Statistical Area")` — MSA layer + name field (once per session; cache)
5. `query_gis_field(address, <MSA layer>, ["NAME"])` — MSA name for context band
6. `query_gis_field(OCCMGMT_CY, OCCSALE_CY, OCCCOMP_CY, /12)`
7. `query_gis_field(OCCEDUC_CY, OCCHLTH_CY, OCCCONS_CY, /12)`
8. `query_gis_field(OCCFARM_CY, OCCPROD_CY, OCCTRAN_CY, /12)`
9. `query_gis_field(OCCFOOD_CY, OCCPROT_CY, OCCPERS_CY, /12)`
10. `query_gis_field(OCCBLDG_CY, EMP_CY, UNEMP_CY, /12)`
11. `query_gis_field(TOTPOP_CY, TOTHH_CY, UNEMPRT_CY, /12)`
12. `query_gis_field(MEDAGE_CY, AVGHHSZ_CY, PCI_CY, /12)`
13. `query_gis_field(MEDHINC_FY, MEDVAL_FY, RENTER_FY, /12)`
14. `query_gis_field(MHIGRWCYFY, /12)` — R13 verification of income growth CAGR
15. `query_gis_field(RENTER_CY, OWNER_CY, MEDRENT_CY, /12)`

**VestMap-direct OMs** (no CSV row for the subject ZIP): also run queries 6–15 at `/11`, `/9`, `/7` (parallel). That's ~40 calls in one turn — fire them all. F6: VestMap is unlimited.

Opt-in modules: add their specified queries to the parallel batch.

**Failure handling:** if any of these returns null, skip that cell silently. Never mention which ones missed.
