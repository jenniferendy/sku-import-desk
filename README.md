# SKU Import Desk

A drag-and-drop converter that turns BB&B source files into import-ready files for **Shopify**, **Matrixify**, and **Fulfil (FFIO)** — units converted, variants grouped, categories coded, gaps flagged. No spreadsheet formulas, no re-keying.

**Live app:** `https://jenniferendy.github.io/sku-import-desk/`

---

## How it works — three steps

1. **Drop a source file.** The app reads it, auto-detects the format, groups variants into products, and shows a summary plus a row-by-row review table with any gaps flagged.
2. **Fill any gaps (optional).** A second panel appears with an enrichment drop box and *only* the manual fields the file is actually missing (vendor name, brand, or SKU prefix).
3. **Download the import files.** Three buttons, available as soon as a file loads.

---

## Step 1 — one box, many formats

Drop any of these into the single box (either `.xlsx` or `.csv`) and the app figures out which it is:

| Source file | Detected by | Notes |
|---|---|---|
| Vendor Quote Sheet — Kitchen & Dining | "Kitchen & Dining" tab | `.xlsx` only (needs the named tab) |
| Vendor Quote Sheet — General | "Categories" tab | `.xlsx` only (needs the named tab) |
| Master SKU Register | BBB SKU ID + Vendor SKU columns (row 3) | full standalone export |
| SKU List (Ready for Fulfil) | same keys, any sheet name | full standalone export |
| Assortment Plan | Vendor + Description + First Cost header | early-stage; cost/retail/category only |
| Filled-out Fulfil export | `code` + `template_name`/`category_name` (row 1) | uses the file's own values; BBB defaults fill blanks |

`.csv` works for every format that detects by header content (register, SKU List, plan, Fulfil export). The two vendor quote sheets need `.xlsx` because they're detected by a named tab. If a file isn't recognized, the app says so and lists the accepted formats.

## Product grouping (automatic)

Rows that share a **BBB Product ID** are grouped into one product, with **colour and/or size becoming variants**. The shared product title is derived by stripping the known size/colour values out of the product name (falling back to the common prefix of the variant names, then the full name). This means a set like "…Snack Bowl - Cream / Blue / Sage" imports as **one product with colour variants**, not four separate products.

**Mis-group check.** If variants under one BBB Product ID have mismatched category (different L4 / Class), the app flags them — an amber banner lists the products to check, and a ⚠ appears next to the product in the review table. These usually mean two different products were given the same Product ID in the source; fix the source and re-drop.

## Step 2 — filling gaps

**Enrichment file (optional).** Drop a second file to add data before exporting. Three kinds are auto-detected:
- **A plain file with a SKU column** — columns matching Shopify/Matrixify fields (SEO title, Tags, etc.) fill in or override; `Metafield:` columns pass through.
- **A filled-out Fulfil export** — matched on `code`, translating title, description, price, cost, barcode, options, and weight (kg/lb → grams).
- **An image sheet** — Product ID / SKU ID / image URL / position, one image per row. Product ID builds the product's gallery; a row with a SKU ID also sets that variant's single image.

The merge report shows how many SKUs/images matched, which columns are being filled, and which were ignored. Rows whose gaps are filled by enrichment lose their flags automatically.

**Manual fields** appear only when the file needs them:
- **Vendor name** — the supplier. Shows when the file carries no supplier name. For drop-ship brands (e.g. Krinkle) the brand is used as the vendor automatically; the assortment plan auto-fills this from its own Vendor column.
- **Brand** — forces all rows to one brand. Shows only if rows are missing a brand.
- **SKU prefix** — auto-numbers rows with no SKU (e.g. `26BBB-` → `26BBB-001`). Shows only when some rows have blank SKUs (typical for the assortment plan).

## Step 3 — downloads

| Button | File | Use |
|---|---|---|
| Shopify import CSV | `shopify_import_YYYY-MM-DD.csv` | Shopify admin → Products → Import (data-rich product creation) |
| Matrixify import CSV | `matrixify_import_YYYY-MM-DD.csv` | Matrixify import (native format: ID / Handle / Command=MERGE / variant + image rows; imports as Draft, unpublished) |
| FFIO update CSV | `ffio_update_YYYY-MM-DD.csv` | Fulfil — lean updater keyed by SKU (see below) |

Rows the vendor didn't mark **✓ COMPLETE** are skipped by default; tick *"Include rows not marked ✓ COMPLETE"* to pull them in.

### Division of labour: Shopify vs FFIO

Shopify is the **primary, data-rich import** — it creates the products with everything customer-facing plus everything Shopify accepts (title, options, price, barcode, weight in grams, etc.). The **FFIO file is a lean updater**, not a full product import: it carries the SKU as the join key plus only the operational fields Shopify can't hold, so Fulfil tops up the matching products afterward. The FFIO file contains:

- `code` — the SKU (join key back to the Shopify product)
- `brand` = BBB (fixed)
- `is_purchasable` = TRUE (fixed)
- weight + weight uom, length / width / height + dimensions uom
- barcode routed by digit count: **12 digits → `ean` (GTIN)**, fewer than 12 → `upc`
- *(supplier / `supplying_company` — pending the confirmed column name from FFIO)*

### The Matrixify image pattern

When an image sheet is loaded, each product emits `max(variants, images)` rows: variant data on the first rows, gallery images stacked below via `Image Src` + `Image Position`, all tied together by the shared `ID` (BBB Product ID) and `Handle`. Product-level fields (Title, Body HTML, etc.) appear on the first row of each product only, as Matrixify requires.

---

## What it does automatically

- **Unit conversion:** kg → lb, cm → inches (splits `L×W×H` text into three fields), and → grams for Shopify/Matrixify weight.
- **Category coding:** matches each row's category against the embedded BB&B merch hierarchy (592 lines), case-insensitive, at either the L4 line or Class level, to fill the code.
- **Variant grouping** by BBB Product ID (see above).
- **Barcode routing** into upc vs ean by digit count (FFIO).
- **UTF-8 output** so accents (é) and symbols (×) export correctly.
- **Sensible defaults** for each system (UOM = Unit, Shopify status = Draft, inventory tracked, tax on, etc.).

## What it flags but can't fix

A row shows **gaps** when missing any of: vendor, brand, SKU, cost, MSRP, fulfil strategy, UPC, or a category that matches the hierarchy. Quote sheets and the assortment plan don't collect **image URLs** — those come from the image sheet or the PIM downstream. If a category won't resolve, the source sheet's spelling doesn't match the hierarchy (e.g. "Mattressess", "Duvet & Comforters"); fix the source cell and it resolves.

**Category on Shopify/Matrixify is intentionally left blank** — Shopify's taxonomy validation rejects internal BB&B category paths, so the Shopify `Product category` and Matrixify `Category` columns are left empty and set downstream in the PIM.

---

## Privacy

Everything runs **in the browser**. Files are never uploaded to any server — the page reads them locally and generates CSVs locally. Safe to use with confidential pricing.

## Updating the app

`index.html` is the only file that runs the app; this README is just documentation. Datasets baked into `index.html`:

- Fulfil importer column order (75 columns)
- Shopify product CSV column order (57 columns)
- Matrixify column order (native Matrixify format)
- BB&B merch hierarchy (line/class → code + category path)

If Fulfil, Shopify, or Matrixify change their templates, or merch revises the hierarchy, the file must be regenerated from the new source files — then replace `index.html` in this repo and the link stays the same for everyone. After committing, wait for the Pages build (Actions tab) and hard-refresh (Ctrl+Shift+R).

## How it fits the pipeline

```
Source file (quote sheet / register / SKU list / plan / Fulfil export)
        |
        v
  SKU Import Desk
        |
        +-->  Shopify import CSV   --> Shopify product creation (primary, data-rich)
        +-->  Matrixify CSV        --> Matrixify (products + variants + images)
        +-->  FFIO update CSV      --> Fulfil (lean updater, keyed by SKU)
```

The PIM owns content enrichment (descriptions, imagery, final categories). This app removes the manual conversion work in the middle.
