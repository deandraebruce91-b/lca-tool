# How to Use the LCA Tool

A field-by-field guide organized by source document. Have your PO, commercial invoice, packing list, and freight invoice open when entering data.

## Source Documents

| Document | What You Pull From It |
|----------|-----------------------|
| **Purchase Order (PO)** | PO number, supplier code, item codes, quantities, unit costs, UOM, style numbers |
| **Commercial Invoice** | Confirms prices (may differ from PO), HS codes, bank/declaration fees, surcharges |
| **Packing List** | Carton-by-carton breakdown: item codes, quantities, bag counts, dimensions, weights |
| **Freight Invoice** | From logistics team: freight, brokerage, insurance, drayage, customs fees |

## Section: Shipment Details

Top-level info entered once per physical shipment.

| Field | Source | What to Enter |
|-------|--------|---------------|
| Shipment Ref | Freight Invoice | BOL number, container ID, or tracking reference |
| Date Shipped | Packing List | When goods left origin — usually on packing list or bill of lading |
| Date Received | Internal | When goods hit your warehouse |
| Origin Country | Commercial Invoice | Country of origin — affects duty rates |
| Destination | Internal | Your receiving warehouse |
| Shipment Mode | Freight Invoice | How it shipped: ocean FCL/LCL, air, parcel, trucking |
| Container Type | Freight Invoice | FCL only — 20', 40', 40' HC, 45' — for utilization calculation |
| Incoterms | PO | FOB, CIF, EXW, etc. — determines what's already in the unit price |
| Currency | Commercial Invoice | Currency of unit costs |
| FX Rate | Internal | Conversion multiplier — default 1.0 for USD-to-USD |

### Incoterms Warning

If you select CIF or DDP and then enter freight or insurance costs, the tool warns you. Under these terms, freight and insurance are typically included in the supplier's unit price. Adding them again would double-count.

## Section: Cost Components

Each row is a cost from the freight invoice. Each gets an allocation method.

| Field | What It Is |
|-------|------------|
| Type | The cost category — freight, brokerage, insurance, drayage, handling, customs exam, terminal, filing, other |
| Amount | Total cost for the entire shipment |
| Allocation Method | How to distribute across items — by weight, volume, unit count, or FOB value |
| Nature | Scalable (varies with shipment size) or Fixed (flat fee) — auto-set from cost type |

### Which Method to Use

| Cost Type | Best Method | Why |
|-----------|-------------|-----|
| Freight | By Weight | Carriers charge by weight |
| Insurance | By FOB Value | Premiums are on declared value |
| Brokerage | By Unit Count | Flat fee — split evenly |
| Customs Exam | By Unit Count | Flat fee |
| Drayage | By Weight | Follows freight logic |

## Section: Supplier-Level Costs

Some suppliers add fees on their commercial invoice: bank transfer fees, declaration fees. These aren't for a specific item — they apply to all items from that supplier.

| Field | What It Is |
|-------|------------|
| Supplier Code | Must match the supplier code on the item lines |
| Cost Type | Bank fees, declaration fees, or other |
| Amount | Total fee |

Distributed across that supplier's items proportional to FOB value.

## Section: Packages

One entry per carton or wrapped bundle from the packing list. Each package contains one or more item lines.

| Field | Source | What to Enter |
|-------|--------|---------------|
| Package # | Packing List | Carton number as printed on the packing list |
| L × W × H (cm) | Packing List | Outer dimensions in centimeters |
| Gross Weight (kg) | Packing List | Total weight including all contents and packaging |

### Shared Cartons

When two items share a carton, the tool splits weight and volume between them based on their individual item weights. This prevents double-counting.

## Section: Item Lines — Packing List

Each row within a package is one item from the packing list. Enter fields in this order:

| Field | Source | What to Enter |
|-------|--------|---------------|
| PO # | PO | Purchase order number — one package can hold items from multiple POs |
| Supplier | PO | Your internal supplier code |
| Item Code | Packing List | Trim or fabric code |
| Color | Packing List | Color variant |
| Quantity | Packing List | Actual shipped qty — not the PO qty |
| UOM | PO | Pieces, yards, meters, pairs, sets, or rolls |
| Description | PO / Packing List | Item description |
| Unit Weight | Sourcing | Weight per UOM — select grams, kg, oz, or lbs |
| Bags | Packing List | Number of inner bags |
| Bag Detail | Packing List | Text description (e.g., "2 × 150 pcs") |

## Section: Internal Allocation (Styles & Lots)

Collapsible section within each item. Links the trim to your finished products and production runs.

### Style Numbers

Which finished garment style(s) this trim is for. Click **+** to add more. One trim can be tagged to multiple styles.

### Production Lot Numbers

Which garment production lot(s) this trim will be used in. Click **+** to add more.

Both are reference-only — they don't affect the landed cost calculation. They're for traceability so you can see which styles and lots a shipment's costs are flowing into.

## Section: Item Lines — Commercial Invoice

Expand the "Commercial Invoice" toggle on each item to enter financial data.

| Field | Source | What to Enter |
|-------|--------|---------------|
| FOB / Unit | Commercial Invoice | Actual unit price — may differ from PO if prices changed |
| HS Code | Commercial Invoice | Harmonized tariff code — usually 10 digits |
| Duty Type | Internal | Ad Valorem (%), Specific ($/unit), or Compound (% + $) |
| Rate % | Commercial Invoice | For ad valorem or compound duties |
| Specific $/u | Commercial Invoice | For specific or compound duties |
| Duty Total | Customs Doc | If you have the total from the customs invoice — overrides rate |
| ADD/CVD % | Customs Doc | Anti-dumping or countervailing duty surcharge |
| FX Override | Internal | If this item's supplier invoices in a different currency |

### Surcharges

Click **+ Add Surcharge** within the Commercial Invoice section. Types: MOQ surcharge, dyeing charge, finish upcharge, other. Enter the total amount — the tool divides by quantity to get per-unit.

## Reading the Results

### Per-Item Table

| Column | Meaning |
|--------|---------|
| FOB/u | Original unit cost from supplier |
| Alloc/u | Allocated shipping costs per unit |
| Duty/u | Duty per unit |
| Srchg/u | Surcharges per unit |
| Landed/u | Total true cost per unit |
| Inflate % | How much higher landed cost is vs FOB |

### Shipment Summary Cards

| Metric | What It Tells You |
|--------|-------------------|
| Total FOB | What you paid the supplier |
| Total Ship Cost | Everything on top of FOB |
| Total Landed | True total cost of the shipment |
| Avg Inflation | Average cost increase across all items |
| Freight Factor | Ship cost as % of FOB — use to benchmark efficiency |
| Cost/CBM | Benchmark for forwarder rate negotiation |
| Cost/KG | Alternative rate benchmark |
| Container Util | FCL only — below 70% means wasted space |

### Reconciliation

Confirms every dollar entered as a cost has been fully allocated. Each cost component should show PASS. FAIL means a rounding discrepancy the tool should have auto-corrected.
