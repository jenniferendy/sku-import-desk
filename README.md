# SKU Import Desk

A drag-and-drop converter that turns BB&B source files into import-ready files for **Fulfil**, **Shopify**, and **Matrixify** — units converted, categories coded, gaps flagged. No spreadsheet formulas, no re-keying.

**Live app:** `https://jenniferendy.github.io/sku-import-desk/`

---

## How it works — three steps

1. **Drop a source file.** The app reads it, auto-detects the format, and shows a summary plus a row-by-row review table with any gaps flagged.
2. **Fill any gaps (optional).** A second panel appears with an enrichment drop box and *only* the manual fields the file is actually missing (vendor name, brand, or SKU prefix).
3. **Download the import files.** Four buttons, available as soon as a file loads.

---

## Step 1 — one box, five formats

Drop any of these into the single box and the app figures out which it is:

| Source file | Detected by | Notes |
|---|---|---|
| Vendor Quote Sheet — Kitchen & Dining | "Kitchen & Dining" tab | Colour variants |
| Vendor Quote Sheet — General | "Categories" tab | Size + colour variants |
| Master SKU Register | BBB SKU ID + Vendor SKU columns (row 3) | Full standalone export |
| SKU List (Ready for Fulfil) | same keys, any sheet name | Full standalone export |
| Assortment Plan | Vendor + Description + First Cost header | Early-stage; cost/retail/category only |

If a file isn't recognized, the app says so and lists the accepted formats.

## Step 2 — filling gaps

**Enrichment file (optional).** Drop a second file to merge extra data onto the loaded rows by **SKU**. Two kinds are accepted:
- **A plain file with a SKU column** — columns matching Matrixify fields (SEO title, Tags, Product image URL, etc.) fill in or override; `Metafield:` columns pass through as-is.
- **A filled-out Fulfil export** — matched on the `code` column, translating title, description, price, cost, barcode, options, and weight (kg/lb -> grams) into the Shopify/Matrixify output.

The merge report shows how many SKUs matched, how many were in the enrichment file but not the loaded sheet, which columns are being filled, and which were ignored (no matching field). Rows whose gaps are filled by enrichment lose their flags automatically.

**Manual fields** appear only when the file needs them:
- **Vendor name** — stamped onto every row (Fulfil `supplying_company`). Shows when the file carries no supplier name. The assortment plan auto-fills this from its own Vendor column.
- **Brand** — forces all rows to one brand. Shows only if rows are missing a brand; otherwise each row uses its own Brand column.
- **SKU prefix** — auto-numbers rows with no SKU (e.g. `26BBB-` -> `26BBB-001`). Shows only when some rows have blank SKUs (typical for the assortment plan; the register and SKU List already have issued IDs).

## Step 3 — downloads

| Button | File | Use |
|---|---|---|
| Fulfil import CSV | `fulfil_import_YYYY-MM-DD.csv` | Fulfil product importer |
| Shopify import CSV | `shopify_import_YYYY-MM-DD.csv` | Shopify admin -> Products -> Import |
| Matrixify import CSV | `matrixify_import_YYYY-MM-DD.csv` | Matrixify import (native format: Handle / Command=MERGE / Variant-level fields; imports as Draft, unpublished) |
| Tracker rows CSV | `tracker_rows_YYYY-MM-DD.csv` | Paste into the SKU Intake Tracker |

The enrichment panel has its own **"Download enriched Matrixify CSV"** button — identical to the main Matrixify export but with any enrichment file merged in.

Rows the vendor didn't mark **COMPLETE** are skipped by default; tick *"Include rows not marked COMPLETE"* to pull them in.

---

## What it does automatically

- **Unit conversion:** kg -> lb, cm -> inches (splits `LxWxH` text into three fields), lb/kg -> grams for Shopify.
- **Category coding:** matches each row's category against the embedded BB&B merch hierarchy (592 lines) — case-insensitive, and matches at either the L4 line or the Class level — to fill the code and build the Shopify category path.
- **Fulfillment strategy:** Drop Ship Y/N -> `Dropship` / `Ship from stock`, with the matching lead time.
- **Variant grouping:** where a BBB Product ID is present, variants group under one Fulfil product template while each keeps its own SKU.
- **Sensible defaults** for both systems (cost method, UOM = Unit, Shopify status = draft, inventory tracker, tax, etc.).

## What it flags but can't fix

A row shows **gaps** when missing any of: vendor, brand, SKU, cost, MSRP, fulfil strategy, UPC, or a category that matches the hierarchy. Quote sheets and the assortment plan don't collect **image URLs** — those come from the PIM downstream. If a category won't resolve, the source sheet's spelling doesn't match the hierarchy (e.g. "Mattressess", "Duvet & Comforters"); fix the source cell and it resolves.

---

## Privacy

Everything runs **in the browser**. Files are never uploaded to any server — the page reads them locally and generates CSVs locally. Safe to use with confidential pricing.

## Updating the app

`index.html` is the only file that runs the app; this README is just documentation. Datasets baked into `index.html`:

- Fulfil importer column order (75 columns)
- Shopify product CSV column order (57 columns)
- Matrixify column order (58 columns, native Matrixify format)
- BB&B merch hierarchy (line/class -> code + category path)

If Fulfil, Shopify, or Matrixify change their templates, or merch revises the hierarchy, the file must be regenerated from the new source files — then replace `index.html` in this repo and the link stays the same for everyone. After committing, wait for the Pages build (Actions tab) and hard-refresh (Ctrl+Shift+R).

## How it fits the pipeline

```
Source file (quote sheet / register / SKU list / plan)
        |
        v
  SKU Import Desk  -->  tracker rows CSV --> SKU Intake Tracker (status, flags, stuck detection)
        |
        +-->  Fulfil import CSV   --> Fulfil product importer
        +-->  Shopify import CSV  --> Shopify product import
        +-->  Matrixify CSV       --> Matrixify (enrich with a 2nd file first if needed)
```

The tracker stays the system of record for *status*; the PIM owns content enrichment (descriptions, imagery). This app removes the manual conversion work in the middle.
