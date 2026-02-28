# Formulary Data - Reference Guide

## Files Overview

| File | Records | Description |
|------|---------|-------------|
| `horizon_formulary.json` | ~1,440 | Horizon BCBSNJ Classic Formulary (Oct 2025), keyed by normalized drug name |
| `redirect_formulary.json` | ~523 | Redirect Health Master Rx Formulary (Jan 2026), keyed by normalized drug name |
| `unified_formulary.json` | ~1,784 | Merged dataset with both plans side-by-side, keyed by normalized drug name |
| `app_data.json` | ~1,784 | Compact version of unified data optimized for frontend embedding (short keys) |

---

## horizon_formulary.json

Dictionary keyed by normalized drug name (lowercase, salt forms stripped).

```json
{
  "pantoprazole": {
    "display_name": "pantoprazole sodium ec tab",
    "tier": "1",
    "tier_label": "Generic",
    "copay": 15
  },
  "vyvanse": {
    "display_name": "Vyvanse (lisdexamfetamine)",
    "tier": "1",
    "tier_label": "Generic",
    "copay": 15
  }
}
```

### Schema

| Field | Type | Description |
|-------|------|-------------|
| `display_name` | string | Human-readable drug name with dosage/form info |
| `tier` | "1" \| "2" \| "3" | Formulary tier |
| `tier_label` | string | "Generic", "Preferred Brand", or "Non-Preferred Brand" |
| `copay` | int | Monthly copay in dollars |

### Horizon Copay Schedule (Earle Companies)

| Tier | Label | Retail (30-day) | Mail Order (90-day) |
|------|-------|-----------------|---------------------|
| 1 | Generic | $15 | $30 |
| 2 | Preferred Brand | $50 | $100 |
| 3 | Non-Preferred Brand | $75 | $150 |

Source: Horizon BCBSNJ Classic Plan Formulary, October 2025 (128-page PDF)

---

## redirect_formulary.json

Dictionary keyed by normalized drug name.

```json
{
  "pantoprazole sodium": {
    "display_name": "Pantoprazole Sodium",
    "copay": 25,
    "covered": true
  },
  "ozempic (1 mg/dose)": {
    "display_name": "Ozempic (1 Mg/Dose)",
    "copay": null,
    "covered": false,
    "note": "Listed but 100% member responsibility under Hospital plan"
  }
}
```

### Schema

| Field | Type | Description |
|-------|------|-------------|
| `display_name` | string | Drug name as listed on formulary |
| `copay` | int \| null | Monthly copay (null if not covered) |
| `covered` | bool | true = plan pays something, false = 100% member responsibility |
| `note` | string? | Explanation when not covered |

### Redirect Copay Tiers (EverydayCARE with Hospital)

| Copay | Count | Examples |
|-------|-------|---------|
| $0 | 63 | Contraceptives |
| $10 | 34 | Common generics (allopurinol, amitriptyline) |
| $25 | 223 | Most generics |
| $50 | 52 | Some generics, preferred brands |
| $100 | 123 | Specialty/brand drugs |
| 100% member resp | 28 | Listed but fully excluded (Ozempic, Humira, Eliquis, Paxlovid, Slynd, etc.) |

**Critical:** "100% Member Responsibility" drugs are listed on the formulary but NOT covered. Costs do NOT count toward the $6,000 out-of-pocket maximum.

**Changes from Jan 2023 → Jan 2026:** Paxlovid ($0 → 100% MR), Slynd ($100 → 100% MR). All other copays unchanged.

Source: Redirect Health Master Rx Formulary, January 1, 2026

---

## unified_formulary.json

Array of merged entries. Each entry has data from both plans.

```json
{
  "id": "pantoprazole",
  "name": "Pantoprazole Sodium Ec Tab",
  "search": "pantoprazole protonix",
  "horizon": {
    "copay": 15,
    "tier": "1",
    "tierLabel": "Generic",
    "covered": true
  },
  "redirect": {
    "copay": 25,
    "covered": true
  }
}
```

### Schema

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Normalized key (lowercase, base drug name) |
| `name` | string | Best available display name |
| `search` | string | Space-separated search terms including brand/generic aliases |
| `horizon` | object | Horizon plan data |
| `horizon.covered` | bool | Whether drug is on formulary |
| `horizon.copay` | int? | Monthly copay |
| `horizon.tier` | "1"\|"2"\|"3" | Tier assignment |
| `horizon.tierLabel` | string | Tier description |
| `redirect` | object | Redirect Health plan data |
| `redirect.covered` | bool | Whether drug is covered (not just listed) |
| `redirect.copay` | int? | Monthly copay if covered |
| `redirect.note` | string? | Explanation when not covered |

---

## app_data.json

Compact version of unified data. Short keys to minimize file size (~160 KB).

```json
{
  "id": "pantoprazole",
  "n": "Pantoprazole Sodium Ec Tab",
  "s": "pantoprazole protonix",
  "h": [1, 15, 1],
  "r": [1, 25]
}
```

### Compact Schema

| Field | Expands To | Format |
|-------|-----------|--------|
| `id` | drug identifier | normalized string |
| `n` | display name | string |
| `s` | search terms | space-separated string |
| `h` | horizon data | `[covered, copay, tier]` or `[0]` if not found |
| `r` | redirect data | `[1, copay]` if covered, `[0]` if not on formulary, `[2]` if 100% member resp |

---

## Brand-Generic Cross-References

The search index includes these mappings (partial list):

| Brand | Generic |
|-------|---------|
| Adderall / Adderall XR | amphetamine-dextroamphetamine |
| Vyvanse | lisdexamfetamine |
| Protonix | pantoprazole |
| Valtrex | valacyclovir |
| Synthroid | levothyroxine |
| Colestid | colestipol |
| Concerta / Ritalin | methylphenidate |
| Ozempic / Wegovy | semaglutide |
| Mounjaro / Zepbound | tirzepatide |
| Lexapro | escitalopram |
| Zoloft | sertraline |
| Wellbutrin | bupropion |
| Xanax | alprazolam |
| Intuniv / Tenex | guanfacine |

Full mapping is embedded in the app's search logic.

---

## Key Drugs Verified

| Drug | Horizon | Redirect | Delta |
|------|---------|----------|-------|
| Pantoprazole (Protonix) | T1 $15 | $25 | +$10/mo |
| Valacyclovir (Valtrex) | T1 $15 | $50 | +$35/mo |
| Levothyroxine (Synthroid) | T1 $15 | $100 | +$85/mo |
| Lisdexamfetamine (Vyvanse) | T1 $15 | NOT ON FORMULARY | +$700-800 cash |
| Colestipol (Colestid) | T1 $15 | NOT ON FORMULARY | +cash |
| Amphetamine-Dextro IR (Adderall) | T1 $15 | $25 | +$10/mo |
| Adderall XR (ER formulation) | T1 $15 | Not explicitly listed | Unclear |

---

## Parsing Notes

- Horizon PDF (128 pages) was parsed with pdfplumber. The main drug listing uses a tier-number-first layout that wraps across lines. Tier assignments were extracted from both the main listings and the index section, with manual verification for key medications.
- Redirect PDF (Jan 2026, 8 pages) was parsed using PyMuPDF with coordinate-based column mapping from a 5-column table layout (iEverydayCARE / MEC / EverydayCARE / with Hospital / with Hospital PLUS). The "with Hospital" column (column 4) was used as the relevant plan tier for The Earle Companies' plan.
- Drug name normalization strips salt forms (hcl, sodium, sulfate, etc.), dosage forms (tab, cap, soln, etc.), and strength info to create base-name keys for cross-referencing.
- Some Horizon entries had parsing artifacts from multi-line wrapping. Manual entries were added for verified drugs that the parser missed or mangled.
- The Jan 2026 Redirect formulary has the same 523 drug entries as Jan 2023, with only 2 coverage changes (Paxlovid and Slynd moved to 100% Member Responsibility).
