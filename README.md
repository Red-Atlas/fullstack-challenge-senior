# Full Stack Challenge – Senior Level

> **Goal**: Evaluate a **Senior Full Stack** candidate's ability to architect, implement, and reason about a modern real estate search platform — with a strong emphasis on Next.js rendering strategies, SEO, API design, and TypeScript across the full stack.
>
> The challenge mirrors the core technical problems at RED Atlas: property data at scale, search & filtering, and delivering a fast, SEO-optimized user experience.

---

## 📌 Scope & Expectations

- **Time budget**: ~1 week. We are not looking for perfection — we are looking for good judgment under time constraints.
- **Focus split**: ~70% frontend / 30% backend. The frontend is the primary evaluation surface.
- The entire stack must run **locally with Docker Compose** (Next.js app, API, PostgreSQL).
- **Bonus** items are optional and only serve to differentiate excellent submissions.
- Deploy to a free tier is not required, but link it in your README if you do it.

> ⚠️ **On the use of AI tools**: We use and encourage AI-assisted development at RED Atlas. Using Copilot, Cursor, Claude, or similar tools is totally fine. What we evaluate is your **technical judgment** — the decisions you made, why you made them, and whether you can defend them. Your README and ADRs carry as much weight as the code itself.

---

## 🧩 Required Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | Next.js 14+ (App Router), TypeScript, React |
| **Styling** | Tailwind CSS or CSS Modules (your choice — justify it) |
| **Backend** | Node.js + TypeScript — Express or NestJS (justify your choice) |
| **Database** | PostgreSQL 14+ with PostGIS |
| **ORM** | Prisma or TypeORM with migrations |
| **Containerization** | Docker Compose |
| **Testing** | Jest + React Testing Library (frontend), Jest + Supertest (backend) |

> Shared TypeScript types between frontend and backend are expected and evaluated.

---

## 🏗️ Domain

You are building **PropSearch** — a simplified real estate listing search platform.

### Data Model

```
Property
  id              UUID PK
  title           TEXT
  description     TEXT
  address         TEXT
  city            TEXT
  country         TEXT
  price           NUMERIC
  area_m2         NUMERIC
  property_type   ENUM (apartment, house, office, land)
  status          ENUM (available, sold, rented)
  coordinates     GEOMETRY(Point, 4326)   -- PostGIS
  created_at      TIMESTAMPTZ
  updated_at      TIMESTAMPTZ

Listing
  id              UUID PK
  property_id     UUID FK → Property
  listing_type    ENUM (sale, rent)
  price           NUMERIC
  currency        ENUM (USD, ARS, COP)
  published_at    TIMESTAMPTZ
  expires_at      TIMESTAMPTZ
  is_active       BOOLEAN
```

### Seed Dataset

Provide a seed script that generates:
- **5,000 properties** across at least 3 cities (e.g., Buenos Aires, Bogotá, San Juan)
- **8,000 listings** linked to those properties
- Realistic geo-coordinates within each city's bounding box
- The script must be idempotent

---

## ✅ Core Requirements

### 1) Next.js App Router — Rendering Strategies

This is the most important section. You must implement **at least three different rendering strategies** and justify each one.

**Required routes:**

| Route | Suggested Strategy | Notes |
|-------|--------------------|-------|
| `/` — Homepage | SSG or ISR | Hero section, featured listings |
| `/listings` — Search/filter page | SSR | Dynamic filters → SEO requires server-rendered content |
| `/listings/[id]` — Property detail | ISR | Stable content, high traffic |
| `/about` or `/contact` | SSG | Fully static |

> **In your README you must explicitly state** which strategy you used for each route, and **why**. This is non-negotiable. Saying "I used SSR" is not enough — explain the trade-offs you considered.

### 2) Property Search Page (`/listings`)

- Filter by: `city`, `property_type`, `listing_type`, `price_min`, `price_max`, `area_min`, `area_max`
- Filters must be reflected in the URL as query params (shareable/bookmarkable)
- Server-rendered results for SEO (the initial HTML must contain listing data)
- Cursor-based or page-based pagination (justify your choice)
- Debounced client-side filter updates
- Loading and empty states

### 3) Property Detail Page (`/listings/[id]`)

- Full property information display
- Proper `<head>` metadata: `title`, `description`, `og:title`, `og:description`, `og:image`
- Structured data (JSON-LD `RealEstateListing` or equivalent schema.org type)
- A static map or coordinate display (a simple embedded map or coordinate badge is fine — no API key required)

### 4) SEO & Performance

- `sitemap.xml` dynamically generated from the database
- `robots.txt`
- All pages must have unique, meaningful `<title>` and `<meta name="description">` tags
- Core Web Vitals: no egregiously large layout shifts or render-blocking resources
- Images (if any) must use `next/image` with proper `alt` text

### 5) Backend API

Expose a REST API consumed by the Next.js app:

```
GET  /v1/properties          — list with filters + pagination
GET  /v1/properties/:id      — single property with active listings
GET  /v1/properties/nearby   — properties within radius (PostGIS)
POST /v1/properties          — create (admin only)
PUT  /v1/properties/:id      — update (admin only)
DELETE /v1/properties/:id    — soft delete (admin only)
```

- JWT authentication for write endpoints (a simple login endpoint is sufficient)
- Normalized error responses (RFC 7807 or similar)
- Input validation
- OpenAPI / Swagger at `/api/docs`

### 6) Database & Performance

- Appropriate indexes on: `city`, `property_type`, `price`, `status`, `coordinates` (GIST index for PostGIS)
- Document your indexing strategy in the README
- `GET /v1/properties` with filters must return in **p95 ≤ 500 ms** on the seeded dataset (measure locally and include numbers)

### 7) Type Safety

- Shared TypeScript types/interfaces used by both frontend and backend (monorepo, shared package, or copied types with clear structure)
- No `any` unless explicitly justified in a comment
- Strict TypeScript config

### 8) Testing

- **Backend**: integration tests for at least 3 API endpoints (happy path + one error case each)
- **Frontend**: component tests for the search filter UI and at least one page
- Code coverage report must be runnable with a single command

---

## ⭐ Bonus Requirements

- **Geospatial search UI**: "Search near me" button that uses the browser's Geolocation API and calls `/v1/properties/nearby`
- **ISR on-demand revalidation**: webhook or admin action that triggers `revalidatePath` for a property after an update
- **Observability**: structured JSON logs in the API with a `correlation-id` per request
- **Lighthouse score**: include a screenshot of a Lighthouse run on `/listings/[id]` — Performance ≥ 85, SEO = 100
- **ADRs**: brief Architecture Decision Records for: framework choice, rendering strategy per route, pagination strategy, caching approach
- **Deploy**: public URL (Vercel + Railway/Render/Neon or similar)
- **E2E test**: one Playwright or Cypress test covering the search → detail flow

---

## 📂 Deliverables

- Git repository (public GitHub or shared link) with the full codebase
- `docker-compose.yml` that starts: Next.js app, API, PostgreSQL
- A `Makefile` or `scripts/` directory with:
  - `make up` — starts everything
  - `make seed` — runs migrations + seed
  - `make test` — runs all tests
- **README** covering:
  - Local setup (env vars, commands)
  - Rendering strategy per route with justification ← **required**
  - API documentation (or link to Swagger)
  - Index strategy and any performance numbers you measured
  - Your reflection section (see below)

---

## 📝 Reflection Section (Required in your README)

This section is mandatory. It is one of the primary signals we use to evaluate genuine senior judgment versus surface-level implementation.

Answer the following in your own words (2–4 sentences each):

1. **Rendering decisions**: For the `/listings` search page — why did you choose the rendering strategy you used? What would break or degrade if you switched to a different one?

2. **Trade-offs under time pressure**: What corners did you consciously cut to finish in one week, and what would you fix with more time?

3. **A real problem you hit**: Describe one specific technical problem you encountered during this challenge and how you resolved it. (If nothing went wrong, you probably didn't push hard enough.)

4. **Scaling question**: If this platform needed to serve 1 million property listings with 5,000 concurrent users, what are the first three architectural changes you would make?

> There are no right or wrong answers here. We are evaluating whether you think like a senior engineer — someone who understands trade-offs, is honest about limitations, and reasons clearly under constraints.

---

## 📊 Evaluation Rubric

Each criterion is scored 0–3 (0: missing, 1: basic, 2: correct, 3: excellent).
**Senior pass threshold**: ≥ 20/27, with no zero in **Rendering Strategy** or **SEO**.

| Criterion | Weight | Notes |
|-----------|--------|-------|
| **Rendering strategy correctness & justification** | 0–3 | Correct use of SSR/ISR/SSG + written reasoning |
| **SEO implementation** | 0–3 | Metadata, structured data, sitemap, robots |
| **Frontend code quality** | 0–3 | Component design, TypeScript, maintainability |
| **Backend API quality** | 0–3 | Schema, validation, error handling, auth |
| **Database modeling & indexes** | 0–3 | Schema design, migrations, query efficiency |
| **Performance (measured SLOs)** | 0–3 | Actual numbers provided and targets met |
| **Type safety & shared types** | 0–3 | No `any`, strict config, shared contracts |
| **Testing** | 0–3 | Coverage, meaningful assertions |
| **Reflection section** | 0–3 | Depth of reasoning, honesty, senior thinking |

---

## 📌 Final Notes

- We value **explicit reasoning** over cleverness. A simpler solution with a well-argued README scores higher than a complex solution with no explanation.
- **Core** running end-to-end with Docker + the Reflection section answered = evaluable submission.
- Incomplete bonus items are better skipped than half-implemented — a stub that breaks the app hurts your score.
- We will ask follow-up technical questions during the debrief interview based on your submission. Be ready to explain every decision.
