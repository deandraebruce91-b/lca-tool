# Contributing

## Getting Started

1. Fork the repo
2. Clone your fork
3. Install dependencies: `npm install`
4. Start dev server: `npm run dev`
5. Make your changes
6. Test the build: `npm run build`
7. Open a PR

## Architecture

The entire application lives in a single file: `src/App.jsx`. This is intentional — it keeps the tool portable (can be dropped into a Claude artifact or any React environment) and avoids premature abstraction.

### Code Structure Within App.jsx

```
Constants          → Enums, lookup tables, configuration
Utilities          → uid(), R() rounding, fmt(), pct(), storage helpers
Factories          → mkShip(), mkPkg(), mkItem(), etc. — create empty records
Calculation Engine → calc() — pure function, shipment in, results out
CSV Export         → expCsv() — generates and downloads CSV
Styles             → Color constants, base input styles
Primitives         → In, Sl, Tg, Bt, Sc, Fl — reusable UI components
Guide              → How-To Guide page component
Main App           → State management, handlers, page routing, all views
```

### Key Principles

**Calculation engine is a pure function.** `calc(shipment)` takes the shipment object and returns `{ errors, warnings, results, reconciliation, summary }`. No side effects, no state mutation.

**Three allocation scopes.** Shipment-level costs → all items. Supplier-level costs → that supplier's items. Item-level surcharges → that item only.

**Package-first hierarchy.** Physical structure (packages) is the parent. Items live inside packages. This matches how raw materials packing lists are structured.

## Conventions

- All monetary calculations use round half up to 2 decimal places
- Variable names in the engine are terse but consistent: `tU` = total units, `tF` = total FOB, etc.
- UI primitives use short names: `In` = input, `Sl` = select, `Bt` = button, `Sc` = section, `Fl` = field label
- Colors are in the `K` object — don't hardcode hex values

## What Needs Work

See the GitHub Issues for current priorities. Common areas:

- ERP integration layer (Momentis, Lazr)
- PDF/OCR packing list parsing
- Database backend (Supabase) to replace localStorage
- Historical comparison across shipments
- Automated HTS code lookup
