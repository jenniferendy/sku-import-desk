# SKU Import Desk

A drag-and-drop converter that turns completed **BB&B vendor quote sheets** into import-ready files for **Fulfil** and **Shopify** — units converted, categories coded, gaps flagged. No spreadsheet formulas, no re-keying.

**Live app:** open `index.html`, or use the GitHub Pages link for this repo.

---

## How to use it

1. **Drop the vendor's quote sheet** (.xlsx) onto the page — or click to browse.
   Both templates are supported and auto-detected: *Kitchen & Dining* and *General*.
2. **Review the results.** You'll see a summary (rows found / ready / with gaps / skipped) and a table showing every product with its resolved L4 category code and exactly which fields are missing.
3. **Download what you need:**
   | Button | File | Use |
   |---|---|---|
   | Fulfil import CSV | `fulfil_import_YYYY-MM-DD.csv` | Upload via Fulfil's product importer |
   | Shopify import CSV | `shopify_import_YYYY-MM-DD.csv` | Products → Import in Shopify admin |
   | Tracker rows CSV | `tracker_rows_YYYY-MM-DD.csv` | Paste into the SKU Intake Tracker for status monitoring |

### Optional inputs

- **Vendor name** — stamped into every row (Fulfil `supplying_company`).
- **Brand override** — leave blank to use the sheet's own Brand Name column.
- **SKU prefix** — if BB&B SKUs haven't been assigned yet, enter a prefix (e.g. `26BBB-`) and blank SKUs are auto-numbered `26BBB-001, -002…`

### What counts as a row

Rows with a Product Name are read starting from the first data row. Rows the vendor didn't mark **✓ COMPLETE** are skipped by default — tick *"Include rows not marked ✓ COMPLETE"* to pull them in anyway.

---

## What it does automatically

- **Unit conversion:** kg → lb, cm → inches (splits `L×W×H` text like `112 × 46 × 46 cm` into three numeric fields), lb → grams for Shopify.
- **Category coding:** looks up each row's *L4 Product Type* against the embedded BB&B merch hierarchy (592 lines) to fill the L4 code and build the Shopify category path (`Category > Sub-Category > Class`).
- **Fulfillment strategy:** Drop Ship Y/N → `Dropship` / `Ship from stock`, and picks the matching lead time (dropship vs. bulk).
- **Pallet math:** cases per layer × layers per pallet → cases per pallet.
- **Sensible defaults** baked in for both systems (cost method, sellable/purchasable, UOM = Unit, Shopify status = draft, inventory tracker, tax on, etc.).

## What it flags but can't fix

A row shows **gaps** when it's missing any of: vendor, brand, SKU, cost, MSRP, fulfil strategy, UPC, or an L4 type that matches the hierarchy. The quote sheet template doesn't collect **HS codes, customs values, or image URLs** — chase those separately if the product needs them.

---

## Privacy

Everything runs **in the browser**. Vendor files are never uploaded to any server — the page reads the .xlsx locally and generates CSVs locally. It's safe to use with confidential pricing.

## Updating the app

The app is a single `index.html` with three datasets baked in:

- Fulfil importer column order (75 columns)
- Shopify product CSV column order (57 columns)
- BB&B merch hierarchy (Line → L4 code / category path)

If Fulfil or Shopify change their import templates, or merch revises the hierarchy, the file needs to be regenerated with the new source files — then replace `index.html` in this repo and the link stays the same for everyone.

**If an L4 code comes up blank:** the vendor's *L4 Product Type* doesn't exactly match a hierarchy *Line* name. Fix the value in the vendor sheet (or update the hierarchy) and re-drop the file.

## How it fits the pipeline

```
Vendor fills quote sheet
        │
        ▼
  SKU Import Desk  ──►  tracker rows CSV ──► SKU Intake Tracker (status, flags, stuck detection)
        │
        ├──►  Fulfil import CSV  ──► Fulfil product importer
        └──►  Shopify import CSV ──► Shopify product import
```

The tracker remains the system of record for *status* (New → Info Complete → Pushed → Live); this app removes the manual conversion work in the middle. Content enrichment (descriptions, imagery, features) is owned by the PIM downstream.
