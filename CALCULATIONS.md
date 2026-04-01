# Calculation Logic

Complete reference for every formula and allocation rule in the LCA tool.

## 1. Rounding

All monetary values use **round half up** to 2 decimal places. This matches standard commercial accounting and is applied at every calculation step, not just at final output.

```
R(value) = Math.round((value + Number.EPSILON) * 100) / 100
```

## 2. Weight Conversion

User enters weight in their preferred unit. The engine converts to kilograms internally.

| Input Unit | Multiplier to KG |
|------------|-------------------|
| Grams      | × 0.001           |
| Kilograms  | × 1               |
| Ounces     | × 0.0283495       |
| Pounds     | × 0.453592        |

```
unitWeightKg = rawWeight × conversionFactor
totalItemWeight = quantity × unitWeightKg
```

## 3. Package-Level Calculations

### CBM (Cubic Meters)

```
packageCBM = (lengthCm × widthCm × heightCm) / 1,000,000
```

If any dimension is 0 or missing, packageCBM = 0.

### Shared Carton Distribution

When multiple items share a package, each item's share of the package's weight and volume is calculated proportionally.

**Priority 1 — By item weight (preferred):**
```
itemShareOfWeight = (itemTotalWeight / sumOfAllItemWeightsInPackage) × packageGrossWeight
itemShareOfCBM = (itemTotalWeight / sumOfAllItemWeightsInPackage) × packageCBM
```

**Priority 2 — By FOB value (fallback when weights are 0):**
```
itemShareOfWeight = (itemTotalFOB / sumOfAllItemFOBsInPackage) × packageGrossWeight
itemShareOfCBM = (itemTotalFOB / sumOfAllItemFOBsInPackage) × packageCBM
```

**Priority 3 — Equal split (fallback when both weight and FOB are 0):**
```
itemShareOfWeight = packageGrossWeight / numberOfItemsInPackage
itemShareOfCBM = packageCBM / numberOfItemsInPackage
```

## 4. Shipment Totals (Denominators)

These are summed across all items in the shipment and serve as denominators for allocation:

```
totalUnits    = SUM(item.quantity) for all items
totalFOB      = SUM(item.quantity × item.unitCostFOB × item.fxRate) for all items
totalWeight   = SUM(item.shareOfPackageWeight) for all items
totalCBM      = SUM(item.shareOfPackageCBM) for all items
```

## 5. Shipment-Level Cost Allocation

For each cost component (freight, brokerage, insurance, etc.), the tool allocates the total amount across all items using the selected method.

### Allocation Methods

**By Weight:**
```
itemAllocation = (itemShareOfWeight / totalShipmentWeight) × costAmount
```

**By Volume (CBM):**
```
itemAllocation = (itemShareOfCBM / totalShipmentCBM) × costAmount
```

**By Unit Count:**
```
itemAllocation = (itemQuantity / totalShipmentUnits) × costAmount
```

**By FOB Value:**
```
itemAllocation = (itemTotalFOB / totalShipmentFOB) × costAmount
```

### Remainder Distribution

After rounding all per-item allocations to 2 decimal places, the sum may not exactly equal the entered cost amount. The remainder is applied to the item with the **largest allocated dollar amount** (not largest quantity) to minimize percentage distortion.

```
remainder = costAmount - SUM(roundedAllocations)
if |remainder| > 0.001:
    find item with max(allocatedAmount)
    item.allocation += remainder
```

### Validation

```
SUM(allItemAllocations) must equal costAmount ± $0.01
```

If the difference exceeds $0.01, reconciliation reports FAIL.

## 6. Supplier-Level Cost Allocation

Supplier fees (bank fees, declaration fees) are allocated only across items matching that supplier code, proportional to FOB value.

```
itemAllocation = (itemTotalFOB / supplierTotalFOB) × supplierCostAmount
```

If supplierTotalFOB = 0, costs are split equally across that supplier's items.

If no items match the supplier code, a warning is displayed and the cost is not allocated.

## 7. Duty Calculation

Duties are calculated per item, not allocated from a pool. Three duty types are supported:

### Ad Valorem (percentage of value)

```
dutyPerUnit = unitCostFOB × fxRate × (dutyRatePct / 100)
```

### Specific (flat per-unit amount)

```
dutyPerUnit = dutySpecificAmount
```

### Compound (percentage + flat)

```
dutyPerUnit = (unitCostFOB × fxRate × (dutyRatePct / 100)) + dutySpecificAmount
```

### ADD/CVD Surcharge

Applied on top of regular duty:

```
dutyPerUnit += unitCostFOB × fxRate × (addCvdPct / 100)
```

### Direct Total Override

If the user enters a duty total amount for the line:

```
dutyPerUnit = dutyAmountTotal / quantity
```

If both a rate and a total are entered, the tool warns and uses the total.

## 8. Item-Level Surcharges

MOQ surcharges, dyeing charges, and finish upcharges are summed and divided across the item's units:

```
surchargePerUnit = SUM(allSurchargeAmounts) / quantity
```

## 9. Final Landed Cost

```
allocatedCostPerUnit = SUM(allShipmentAllocations + allSupplierAllocations) / quantity
landedCostPerUnit = unitCostFOB + allocatedCostPerUnit + dutyPerUnit + surchargePerUnit
```

## 10. Output Metrics

### Per-Item

```
costInflation% = ((landedCost - FOB) / FOB) × 100
```

### Shipment Summary

```
totalShipmentCost  = SUM(shipmentCosts) + SUM(supplierCosts) + SUM(allSurcharges) + SUM(allDuties)
totalLandedValue   = SUM(landedPerUnit × quantity) for all items
avgInflation%      = ((totalLandedValue - totalFOB) / totalFOB) × 100
freightFactor%     = (totalShipmentCost / totalFOB) × 100
costPerUnit        = totalShipmentCost / totalUnits
costPerCBM         = totalShipmentCost / totalCBM
costPerKG          = totalShipmentCost / totalWeight
containerUtil%     = (totalCBM / containerCapacityCBM) × 100    [FCL only]
```

### Container Capacities

| Container Type | Capacity (CBM) |
|----------------|-----------------|
| 20' Standard   | 33.2            |
| 40' Standard   | 67.7            |
| 40' High Cube  | 76.3            |
| 45' High Cube  | 86.0            |

## 11. Edge Cases

| Scenario | Behavior |
|----------|----------|
| Allocation denominator is 0 | Error — blocks calculation, identifies method and cause |
| Item quantity is 0 | Error — blocks calculation |
| FOB is $0 | Warning — item absorbs $0 in value-based allocation |
| Weight missing for weight-based allocation | Error — blocks calculation |
| Dimensions missing for volume-based allocation | Allocates 0 CBM to that item |
| Both duty rate and duty total entered | Warning — uses total, ignores rate |
| Supplier code on cost doesn't match any items | Warning — cost is not allocated |
| Incoterms include freight but freight cost entered | Warning — possible double-count |
| FX rate on item overrides shipment FX | Item-level rate used for that item only |
