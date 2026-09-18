# SKU Import Desk

A drag-and-drop tool that turns the BB&B Master SKU Register into import-ready files for **Shopify** and **Fulfil (FFIO)** — units converted, vendors mapped to their exact Fulfil names, variants grouped, gaps flagged. Add a sourcing assortment file and it also builds a **Purchase Order import**. No spreadsheet formulas, no re-keying.

**Live app:** `https://jenniferendy.github.io/sku-import-desk/`

---

## Two inputs

1. **Master SKU Register** — the source of truth for all product data. Drop this to unlock every output except the PO.
2. **Sourcing assortment file** (optional) — a list with a `SKU` column and a `Unit QTY` column. Supplies the order quantities for the PO import. The register provides the product and supplier data; the assortment provides *how many* to order.

Both can be `.xlsx` or `.csv`. Drop the register first, then the assortment file if you need a PO.

---

## How it works — three steps

1. **Drop the Master SKU Register.** The app reads it, groups variants into products, maps vendors to their Fulfil names, and shows a summary plus a row-by-row review table with any gaps flagged.
2. **Add the assortment file (for PO quantities) and fill any gaps.** A vendor include/exclude panel appears, plus an optional enrichment box and only the manual fields the file is actually missing.
3. **Download the import files.**

---

## Outputs

All generated from the register (the PO also needs the assortment file):

| Button | File | What it is |
|---|---|---|
| Shopify import CSV | `shopify_import_YYYY-MM-DD.csv` | Data-rich Shopify product creation — title, options, price, barcode, weight (grams), grouped variants |
| FFIO update CSV | `ffio_update_YYYY-MM-DD.csv` | Lean Fulfil updater keyed by SKU — the operational fields Shopify can't hold |
| FFIO enrichment CSV | `ffio_enrichment_YYYY-MM-DD.csv` | Rich Fulfil file — full product data from the register (category, price, cost, supplier, weight, dims, description, country, lead time, attributes) |
| FFIO supplier mapping CSV | `ffio_supplier_mapping_YYYY-MM-DD.csv` | Product Suppliers import — maps each SKU to its exact Fulfil supplier with cost, lead time, MOQ |
| PO import CSV | `ffio_po_import_YYYY-MM-DD.csv` | Purchase Order import — appears once an assortment file is loaded; product + quantity + unit price |
| Missing-info report | `missing_info_report_YYYY-MM-DD.csv` | One row per SKU with gaps, listing exactly which fields are missing |
| Check-these report | `check_these_report_YYYY-MM-DD.csv` | SKUs in products flagged for mismatched category (possible mis-grouping) |

**Matrixify** is still available as a small secondary button, kept for occasional use.

### Shopify vs the Fulfil files

Shopify is the **primary, data-rich import** — it creates the products with everything customer-facing plus everything Shopify accepts. The **FFIO update** is a lean updater (SKU as join key + the fields Shopify can't hold). The **FFIO enrichment** is the fuller version when you want as much register data in Fulfil as possible. The **supplier mapping** links each SKU to its purchasing supplier. All are keyed by SKU so Fulfil updates the matching products.

### Barcodes

Routed by digit count: **12 digits → `ean` (GTIN)**, fewer than 12 → `upc`.

---

## Vendor name mapping

Fulfil expects supplier names spelled exactly its way ("MODA", "Beco Home", "Lifetime Brands Canada Group"), while the register uses shorthand ("Moda at Home", "Beco", "Lifetime Brands"). The app has a **built-in mapping** from register vendor names to their exact Fulfil names, applied automatically to the FFIO update, enrichment, and supplier-mapping files (the `supplying_company` / `supplier_name` fields).

If a vendor has no Fulfil match, it shows a ⚠ note in the vendor panel and is skipped from the supplier-mapping file. To add a new vendor, extend the `VENDOR_MAP` list in `index.html`.

## Vendor include / exclude

After the register loads, a panel lists every vendor found, with counts and a checkbox each. Untick any vendor to exclude all its SKUs from every export (e.g. Silk & Snow). Vendors with no Fulfil match are flagged so you can see them at a glance.

## Product grouping (automatic)

Rows sharing a **BBB Product ID** are grouped into one product, with colour and/or size as variants. The shared title is derived by stripping the known size/colour values out of the product name (falling back to the common prefix of the variant names). If variants under one Product ID have mismatched category (different L4/Class), the app flags them — an amber banner and a ⚠ in the table — since that usually means two different products were given the same Product ID.

## Enrichment file (Layer 2, optional)

A separate drop box that merges extra data onto the loaded rows by SKU. Auto-detects three kinds: a plain file with a SKU column (Shopify/Matrixify fields fill in; `Metafield:` columns pass through); a filled-out Fulfil export (matched on `code`); or an image sheet (Product ID / SKU ID / image URL / position) that builds each product's gallery in the Matrixify export.

---

## What it does automatically

- **Unit conversion:** kg → lb, cm → inches (splits `L×W×H`), and → grams for Shopify.
- **Vendor mapping:** register vendor → exact Fulfil supplier name.
- **Variant grouping:** by BBB Product ID.
- **Barcode routing:** upc vs ean by digit count.
- **Category coding:** matches against the embedded BB&B merch hierarchy (592 lines), case-insensitive.
- **UTF-8 output:** accents (é) and symbols (×) export correctly.
- **Sensible defaults** for each system (UOM = Unit, Shopify status = Draft, inventory tracked, tax on, etc.).

## What it flags but can't fix

A row shows **gaps** when missing vendor, brand, SKU, cost, MSRP, fulfil strategy, UPC, or a category that matches the hierarchy — see the Missing-info report for the full list per SKU. If a category won't resolve, the register's spelling doesn't match the hierarchy; fix the source cell and it resolves.

**Category on Shopify is intentionally left blank** — Shopify's taxonomy validation rejects internal BB&B category paths, so the category is set downstream in the PIM.

**Mismatched variant options** (e.g. one variant has a Size, its product-mates don't) will fail Shopify's import; the Check-these report surfaces the products to review.

---

## Privacy

Everything runs **in the browser**. Files are never uploaded to any server — the page reads them locally and generates CSVs locally. Safe to use with confidential pricing.

## Updating the app

`index.html` is the only file that runs the app; this README is just documentation. Baked into `index.html`:

- Fulfil importer column order (75 columns), Shopify column order (57 columns), Matrixify column order
- Fulfil supplier-import and PO-import column orders
- BB&B merch hierarchy (line/class → code + category path)
- The official Fulfil vendor list and the register→Fulfil vendor name mapping

If Fulfil or Shopify change their templates, the hierarchy is revised, or vendors are added/renamed, update the relevant list in `index.html`, replace the file in this repo, wait for the Pages build (Actions tab), and hard-refresh (Ctrl+Shift+R).

## How it fits the pipeline

```
Master SKU Register  (+ assortment file for quantities)
        |
        v
  SKU Import Desk
        |
        +-->  Shopify import CSV        --> Shopify product creation (primary)
        +-->  FFIO update CSV           --> Fulfil (lean updater, keyed by SKU)
        +-->  FFIO enrichment CSV       --> Fulfil (full product data)
        +-->  FFIO supplier mapping CSV --> Fulfil (SKU → supplier, price, lead time)
        +-->  PO import CSV             --> Fulfil (purchase orders, from assortment qty)
```

The PIM owns content enrichment (descriptions, imagery, final categories). This app removes the manual conversion and vendor-mapping work in the middle.
