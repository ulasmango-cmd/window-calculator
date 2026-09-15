# WinCalc

> A configure–price–quote application for PVC and aluminium windows and doors — configure a unit, see a live preview, get an itemized price, and issue a quotation, bill of materials, or PDF.

## Live Demo

**https://wincalc-api-tx2h.onrender.com**

The application is login-gated. A read/write demonstration account is provided:

| | |
|---|---|
| Username | `demo` |
| Password | `WinCalc-Demo-2026!` |

> This is a shared demonstration account on a seeded dataset — not production data. The service runs on a free tier and sleeps after a period of inactivity, so the first request may take up to a minute to wake.

## Overview

WinCalc is a quoting tool for window and door fabricators and installers. The workflow it replaces is the spreadsheet-and-calculator routine that most small fabricators still use: measuring a unit, looking up profile and glass prices, applying labour and margin, and writing the result into a quote by hand.

The application turns that into a structured pipeline. A unit is configured by type, dimensions, profile system, glazing, colour, hardware, and accessories. The result is priced from a configurable catalog with explicit pricing rules, previewed visually, and persisted as a calculation that can be reused, exported, or rolled into a customer quotation.

The interface follows the **Windows 11 Fluent** design language — Segoe UI typography, system accent colour, and light/dark themes — with the interaction model of a desktop calculator: a large result display, instant recalculation as parameters change, and keyboard shortcuts for the common actions.

## Features

- **Parametric configurator** — configure window and door units by type, dimensions, opening direction, and construction details
- **Live visual preview** — a 2D technical preview that updates as the unit is configured, plus a 3D representation of the assembled unit
- **Itemized pricing** — every unit resolves to a transparent cost breakdown: profile, glazing, hardware, accessories, labour, and margin
- **Configurable catalog** — window types, profile systems, glass types, colours, hardware, handles, and accessories are all data-driven rather than hard-coded
- **Pricing rules engine** — pricing behaviour is expressed as rules, so commercial changes do not require a code change
- **Quotations and calculations** — save, revisit, duplicate, and revise calculations, then generate a customer-facing quotation
- **Bill of materials** — derive a per-order BOM from configured units for procurement and cutting
- **PDF export** — produce branded quotation and BOM documents
- **Customer records** — keep customers alongside their calculations and quotations
- **Role-based access** — separate Administrator and Employee surfaces
- **Multi-language** — Albanian, English, and Romanian, with the interface built for additional locales
- **Backup and restore** — administrative backup tooling for the catalog and business data

## Technology

**Frontend**
- React 19 with Vite
- Tailwind CSS 4
- Zustand for application state
- React Router for navigation, with route-level code splitting
- React Three Fiber for 3D unit preview
- Recharts for administrative reporting
- i18next for localization

**Backend**
- Node.js with Express
- PostgreSQL as the system of record
- JWT-based authentication with bcrypt password hashing
- PDF generation for quotations and bills of materials

**Infrastructure**
- Frontend served from Vercel's edge network
- API and background services hosted on Render
- Managed PostgreSQL for production data

## Architecture

```mermaid
graph TB
  subgraph Client
    UI[React SPA<br/>Configurator · Quotes · BOM · Admin]
  end

  subgraph Edge
    CDN[Vercel CDN<br/>static assets + SPA routing]
  end

  subgraph Application
    API[Express API<br/>auth · catalog · pricing · documents]
  end

  subgraph Data
    DB[(PostgreSQL)]
    FS[Object / file storage<br/>images and generated documents]
  end

  UI -->|HTTPS| CDN
  CDN -->|/api/*| API
  UI -->|HTTPS| API
  API --> DB
  API --> FS
```

The pricing engine is deliberately isolated from the data layer: pricing behaviour is expressed as rules evaluated against a configured unit and a catalog snapshot, which keeps the calculation reproducible and testable independently of the database.

For a fuller breakdown, see [`ARCHITECTURE.md`](ARCHITECTURE.md).

## Screenshots

Screenshots are pending capture from the live instance. The application is a login-gated business tool, so preview imagery is produced from the seeded demonstration dataset rather than production data. In the meantime, the live demo above can be opened directly with the demonstration account.

## Technical Highlights

- **Rule-driven pricing** — pricing is resolved from composable rules rather than imperative branching, so a new commercial policy is a data change, not a release
- **Reproducible quotes** — a saved calculation captures the inputs and resolved pricing context, so a quote issued months ago can be explained and reproduced
- **Parameterized preview** — the 2D technical preview and 3D model are both derived from the same unit model, so geometry, dimensions, and the priced configuration cannot drift apart
- **Route-level code splitting** — secondary surfaces are lazily loaded, keeping the initial bundle small and the configurator responsive
- **Optimistic, recalculation-first UI** — the configurator recalculates immediately on change, which is the interaction model the target users expect from desktop tools
- **Multi-tenant catalog design** — catalog entities are versioned and referenced by id, so historical quotes remain valid when the catalog changes

## Deployment

The current production topology is a single service: the Express application serves the built client and the API from one origin, backed by managed PostgreSQL.

| Layer | Host | Responsibility |
|-------|------|----------------|
| Application | Render | Express API, authentication, pricing, document generation, SPA delivery |
| Database | Managed PostgreSQL | Catalog, customers, calculations, quotations |

Serving both tiers from one origin removes an entire class of cross-origin and cookie-scope problems, and keeps the deployment cheap to operate. The application also supports a split topology — a static client on an edge network proxying `/api/*` to the API service — which is the preferred arrangement when the client should be served from a CDN.

Configuration is environment-driven. Database credentials, JWT signing secrets, and service URLs are supplied as environment variables at deploy time and are never committed to source control.

## Project Status

**Live.** The application is deployed and serving. Verified end to end:

- API health check returns `{"status":"ok","database":"postgresql"}`
- Database schema applied and catalog seeded (22 tables; 41 window types, 9 glass types, 13 colours)
- Authentication working — the demonstration account signs in successfully
- Single-page application served and client-side routing resolving

The production stack runs as a single service: the Express application serves both the API and the built client, backed by managed PostgreSQL.

Known operational constraints of the current deployment:

- The service runs on a free tier and idles when unused; the first request after a period of inactivity is slow
- The managed database is a free-tier instance with a limited lifetime and will need migrating to a permanent plan
- Screenshots are pending capture from the live instance

## Ownership

Built by **[ulasmango-cmd](https://github.com/ulasmango-cmd)**.

- GitHub profile: https://github.com/ulasmango-cmd
- Production source: private

## License

The implementation is proprietary. This repository contains portfolio documentation only; it does not grant rights to copy, modify, or redistribute the production source code, pricing logic, or brand assets.
