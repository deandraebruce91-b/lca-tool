# LCA — Raw Materials Landed Cost Allocator

A browser-based calculator that takes the total costs of shipping a raw materials shipment (trims, fabrics, components) and distributes them to each item, giving you the true per-unit landed cost.

Your supplier's FOB price is not your real cost. Once you add freight, duties, brokerage, surcharges, and handling, the actual cost per piece/yard/meter can be 15–40% higher. This tool makes that gap visible.

## What It Does

- Allocates shipment-level costs (freight, brokerage, insurance, drayage, handling, customs fees) across items using configurable methods: by weight, volume (CBM), unit count, or FOB value
- Handles mixed cartons — when multiple items share a package, weight and volume are split proportionally with no double-counting
- Calculates duties per item using ad valorem, specific, or compound rates, plus ADD/CVD surcharges
- Captures item-level surcharges (MOQ, dyeing, finish upcharges) and supplier-level fees (bank fees, declaration fees)
- Supports multiple units of measure (pieces, yards, meters, pairs, sets, rolls) and weight units (grams, kg, oz, lbs)
- Tags items with internal style numbers and production lot numbers for traceability
- Warns when incoterms suggest potential double-counting (e.g., freight entered on a CIF shipment)
- Persists an Item Master and shipment history in browser storage
- Exports results to CSV
- Full reconciliation check confirming every entered dollar is allocated

## Who It's For

Buyers and sourcing teams who need to see the true landed cost of raw materials — not just the FOB price on a purchase order. Built for the workflow of someone sitting with a packing list, commercial invoice, and freight invoice open at the same time.

## Quick Start

### Option 1: Static HTML (no build tools)

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/lca.git
cd lca

# Open directly (requires a React-compatible environment)
# Or use the pre-built index.html if available
```

### Option 2: Vite + React

```bash
# Clone and install
git clone https://github.com/YOUR_USERNAME/lca.git
cd lca
npm install

# Dev server
npm run dev

# Production build
npm run build
```

### Option 3: Claude.ai Artifact

Copy the contents of `src/App.jsx` into a Claude.ai conversation as a React artifact. It runs directly in the browser with no build step.

## Project Structure

```
lca/
├── .github/
│   └── workflows/
│       └── ci.yml              # GitHub Actions CI pipeline
├── docs/
│   ├── CALCULATIONS.md         # Complete calculation logic and formulas
│   ├── DATA_MODEL.md           # Full data model specification
│   ├── HOW_TO_USE.md           # User guide organized by source document
│   └── AUDIT.md                # Supply chain strategist audit findings
├── src/
│   └── App.jsx                 # Complete application source
├── .gitignore
├── LICENSE
├── README.md
├── package.json
└── vite.config.js
```

## Documentation

| Document | Description |
|----------|-------------|
| [Calculations](docs/CALCULATIONS.md) | Every formula, allocation method, remainder distribution, and edge case |
| [Data Model](docs/DATA_MODEL.md) | Complete entity specifications — shipment, packages, items, costs |
| [How to Use](docs/HOW_TO_USE.md) | Field-by-field guide organized by source document |
| [Audit](docs/AUDIT.md) | Supply chain strategist review with findings and resolutions |

## Key Design Decisions

**Carton-first data model.** The packing list is the source of truth for physical structure. Packages (cartons/wraps) are the parent entity; items live inside packages. This matches how raw materials are actually packed — unlike finished goods where one SKU per carton is standard.

**Per-component allocation methods.** Freight allocates by weight. Insurance by FOB value. Brokerage by unit count. Each cost component has a smart default but the user can override per shipment. Fixed costs (flat fees) default to unit-count allocation; scalable costs default to weight or value.

**Duties are direct, not allocated.** Duty rates vary by item (metal vs textile vs plastic). They're calculated per item based on HTS rate, not pooled at the shipment level and spread.

**Three allocation scopes.** Shipment-level costs spread across all items. Supplier-level costs spread across that supplier's items only. Item-level surcharges apply directly to the item they belong to.

## Tech Stack

- React (functional components, hooks)
- Zero dependencies beyond React
- Browser localStorage for persistence
- Pure CSS-in-JS styling (no external stylesheets)
- Single-file architecture for portability

## License

MIT — see [LICENSE](LICENSE) for details.
