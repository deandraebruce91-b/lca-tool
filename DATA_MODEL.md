# Data Model

Complete specification of every entity, field, and relationship in the LCA tool.

## Entity Hierarchy

```
Shipment
├── Cost Components[]          (shipment-level: freight, brokerage, etc.)
├── Supplier Costs[]           (per-supplier: bank fees, declaration fees)
└── Packages[]                 (cartons / wraps)
    └── Items[]                (trim/fabric lines within each package)
        ├── Surcharges[]       (MOQ, dyeing, finish upcharges)
        ├── Style Numbers[]    (internal allocation references)
        └── Lot Numbers[]      (production lot references)
```

## Shipment

One record per physical shipment.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| shipment_ref | String | No | BOL number, container ID, or tracking reference |
| date_shipped | Date | No | Date goods left origin |
| date_received | Date | No | Date goods arrived at warehouse |
| origin_country | String | No | Country of origin |
| destination | String | No | Receiving warehouse or location |
| shipment_mode | Enum | Yes | ocean_fcl, ocean_lcl, air, parcel, ltl, ftl |
| container_type | Enum | Cond. | 20ft, 40ft, 40hc, 45hc — only when shipment_mode = ocean_fcl |
| incoterms | Enum | Yes | EXW, FOB, CIF, CFR, DDP, DAP |
| currency | String | Yes | Default: USD |
| fx_rate | Decimal | Yes | Default: 1.0 — multiplier for currency conversion |

## Cost Component

Shipment-level costs from the freight invoice. Each has its own allocation method.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| cost_type | Enum | Yes | freight, brokerage, insurance, drayage, handling, customs_exam, terminal, filing, other |
| amount | Decimal | Yes | Total cost in shipment currency |
| allocation_method | Enum | Yes | by_weight, by_volume, by_unit_count, by_fob_value |
| nature | Enum | Auto | scalable or fixed — set from cost_type defaults |

### Default Allocation Methods

| Cost Type | Default Method | Nature | Rationale |
|-----------|----------------|--------|-----------|
| Freight | By Weight | Scalable | Carriers charge by weight/volume |
| Brokerage | By Unit Count | Fixed | Flat service fee |
| Insurance | By FOB Value | Scalable | Premiums are on declared value |
| Drayage | By Weight | Scalable | Follows freight logic |
| Warehouse Handling | By Unit Count | Fixed | Receiving is per-unit labor |
| Customs Exam | By Unit Count | Fixed | Flat fee |
| Terminal Handling | By Unit Count | Fixed | Flat fee |
| AMS/ISF Filing | By Unit Count | Fixed | Flat fee |
| Other | By FOB Value | Scalable | Safe default |

## Supplier Cost

Fees on a supplier's commercial invoice that apply to all items from that supplier.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| supplier_code | String | Yes | Must match supplier_code on item lines |
| cost_type | Enum | Yes | bank_fees, declaration_fees, supplier_other |
| label | String | No | Description |
| amount | Decimal | Yes | Total fee amount |

Allocated across matching items proportional to FOB value.

## Package

One record per physical carton or wrapped bundle. Maps directly to packing list carton lines.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| package_number | String | No | Carton number as printed on packing list |
| length_cm | Decimal | No | Outer length in centimeters |
| width_cm | Decimal | No | Outer width in centimeters |
| height_cm | Decimal | No | Outer height in centimeters |
| gross_weight_kg | Decimal | No | Total package weight including contents and packaging |

## Item

One record per item line within a package. This is where PO data, packing list data, and commercial invoice data converge.

### Packing List Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| po_number | String | No | Purchase order reference |
| supplier_code | String | No | Internal supplier code — used for filtering and supplier-level fee allocation |
| item_code | String | No | Trim or fabric item code |
| color_code | String | No | Color variant |
| quantity | Decimal | Yes | Actual shipped quantity (not PO quantity) |
| uom | Enum | Yes | pcs, yds, m, prs, sets, rolls |
| description | String | No | Item description |
| unit_weight | Decimal | Cond. | Weight per unit of measure — required for weight-based allocation |
| weight_uom | Enum | Yes | g, kg, oz, lb — default: g |
| bag_count | Integer | No | Number of inner bags in the carton |
| bag_detail | String | No | Descriptive text (e.g., "2 × 150 pcs") |

### Internal Allocation Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| styles[] | Array | No | List of style number strings — links trim to finished product styles |
| lots[] | Array | No | List of production lot number strings — links trim to garment production lots |

### Commercial Invoice Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| unit_cost_fob | Decimal | Yes | FOB unit cost from commercial invoice |
| hs_code | String | No | Harmonized tariff code |
| duty_type | Enum | Yes | ad_valorem, specific, compound |
| duty_rate_pct | Decimal | Cond. | Percentage for ad valorem or compound |
| duty_specific | Decimal | Cond. | Per-unit amount for specific or compound |
| duty_amount_total | Decimal | No | Direct total — overrides rate-based calculation |
| duty_surcharge_pct | Decimal | No | ADD/CVD surcharge percentage |
| fx_rate | Decimal | No | Per-item FX override — when empty, shipment-level rate applies |

## Surcharge

Item-level costs outside the base unit price.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| type | Enum | Yes | moq, dyeing, finish, other_surcharge |
| amount | Decimal | Yes | Total surcharge amount for this item line |

Divided across the item's units: surchargePerUnit = totalAmount / quantity.

## Style Number

Internal reference linking a trim to a finished product.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| style | String | Yes | Internal style number |

## Lot Number

Internal reference linking a trim to a garment production lot.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | String | Auto | Unique identifier |
| lot | String | Yes | Internal production lot number |

## Storage

### Item Master

Persistent lookup stored in browser localStorage under key `lca3_master`.

| Field | Type | Description |
|-------|------|-------------|
| [item_code] | Object key | Primary key |
| desc | String | Description |
| wt | String | Default unit weight |
| wtu | String | Default weight UOM |
| uom | String | Default item UOM |
| hs | String | Default HS code |
| dr | String | Default duty rate |
| upd | ISO Date | Last updated timestamp |

Auto-saved on first calculation for new item codes. Not overwritten for existing codes.

### Shipment History

Stored in browser localStorage under key `lca3_hist`. Array of up to 50 entries, most recent first.

| Field | Type | Description |
|-------|------|-------------|
| id | String | Unique identifier |
| date | ISO Date | Calculation timestamp |
| ref | String | Shipment reference |
| ship | Object | Complete shipment input data |
| result | Object | Complete calculation output |
