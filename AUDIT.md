# Supply Chain Strategist Audit

Findings from auditing the LCA spec from the perspective of a senior supply chain strategist with carrier-side landed cost modeling experience. All findings in this document have been resolved and implemented in the current build.

## Critical Findings (All Resolved)

### C1: Missing Shipment Mode Field — RESOLVED

**Problem:** The volumetric weight formula referenced ocean and air divisors but the data model had no shipment mode field. Different modes have fundamentally different charging logic.

**Resolution:** Added `shipment_mode` enum: ocean_fcl, ocean_lcl, air, parcel, ltl, ftl. Volumetric formulas branch on mode. Ocean uses 1:1000 divisor, air/parcel uses 1:5000.

### C2: Ocean FCL Breaks the Allocation Model — RESOLVED

**Problem:** FCL shipments have flat container rates regardless of fill level. Per-unit freight doubles when a container is half-empty, but the tool didn't surface this.

**Resolution:** Added `container_type` field with capacity data. Container utilization % is calculated and displayed in results. Below 70% is flagged red.

### C3: No Handling of Flat/Minimum Charges — RESOLVED

**Problem:** Customs exam fees, filing fees, and chassis charges are flat costs that shouldn't be allocated by weight. Users could accidentally assign weight-based allocation to flat fees.

**Resolution:** Added `nature` enum (scalable vs fixed) to each cost component. Fixed costs default to unit-count allocation. UI displays the nature tag. User can still override.

### C4: Duty Model Incomplete — RESOLVED

**Problem:** Only ad valorem duties (percentage of value) were supported. Real tariff schedules include specific duties (flat per-unit), compound duties (both), and ADD/CVD surcharges.

**Resolution:** Added `duty_type` enum: ad_valorem, specific, compound. Separate fields for rate percentage and specific per-unit amount. ADD/CVD surcharge as a separate percentage field on top of regular duty.

### C5: No Incoterms Field — RESOLVED

**Problem:** No way to capture trade terms. CIF purchases include freight and insurance in unit price — entering freight costs on top would double-count.

**Resolution:** Added `incoterms` field with live warnings when cost components are entered that may already be included under the selected incoterm.

## Structural Findings (All Resolved)

### S1: Shipment Summary Metrics Too Thin — RESOLVED

**Problem:** Output focused on per-item costs but lacked shipment-level benchmarking metrics.

**Resolution:** Added freight factor (cost as % of FOB), cost per unit, cost per CBM, cost per KG, and container utilization to the results summary.

### S2: Results Not Persistable — RESOLVED

**Problem:** Shipment results existed only in the current session.

**Resolution:** Shipment history persisted to browser localStorage. Up to 50 calculations saved. Click any history entry to reload the full shipment and results.

### S3: FX Rate Must Support Per-Line Override — RESOLVED

**Problem:** Single FX rate at shipment level wouldn't work for multi-supplier consolidation with mixed currencies.

**Resolution:** Optional `fx_rate` field on each item line. When populated, overrides the shipment-level rate for that item. When empty, shipment rate applies.

### S4: Remainder Distribution Rule — RESOLVED

**Problem:** Remainder was applied to the line with largest quantity. Industry standard targets largest allocated amount.

**Resolution:** Changed to apply remainder to the line with the largest allocated dollar amount per cost component.

## Minor Findings (All Resolved)

### M1: Uniform Packing Assumption — RESOLVED

**Problem:** Data model assumed uniform carton packing. Real shipments have mixed cartons.

**Resolution:** Redesigned to carton-first data model. Packages are the parent entity with items nested inside. No uniform packing assumption — each item's actual quantity within each carton is entered directly.

### M2: Rounding Method Undefined — RESOLVED

**Problem:** No specification of rounding method.

**Resolution:** All monetary calculations use round half up to 2 decimal places, matching standard commercial accounting.

### M3: No Export in Prototype — RESOLVED

**Problem:** No way to get data out of the tool.

**Resolution:** CSV export button on the results table. Exports all per-item results with headers.

## Additional Findings from Domain Review

### Raw Materials Packing Complexity

The original spec assumed finished-goods packing (one SKU per carton). Raw materials have fundamentally different packing: mixed cartons with multiple items from multiple POs, inner bag packing, and non-standard containers (wrapped rolls vs boxes).

**Resolution:** Complete data model redesign to Package → Item hierarchy. Shared carton weight/volume distribution logic. Bag count and detail as reference fields.

### Unit of Measure Diversity

Raw materials use pieces, yards, meters, pairs, sets, and rolls — these aren't interchangeable for allocation purposes.

**Resolution:** UOM selector per item. Weight-based allocation is the only universally applicable method for mixed-UOM shipments.

### Item-Level Surcharges

MOQ surcharges, dyeing charges, and finish upcharges are common in trim sourcing and sit outside the line-level unit cost.

**Resolution:** Repeatable surcharge entries per item with type selector and amount. Divided across the item's units in calculation.

### Supplier-Level Fees

Some suppliers add bank fees and declaration fees on commercial invoices that apply to all their items, not a specific item.

**Resolution:** Separate supplier cost entity with supplier code matching. Allocated across matching items by FOB value.

### Internal Traceability

Buyers need to tag trims with the finished product styles and production lots they're destined for.

**Resolution:** Style number and lot number arrays per item, supporting multiple entries each. Reference-only fields that don't affect calculation.
