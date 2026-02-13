# Maps Scraper Integration Plan

Comprehensive plan to integrate Google Maps scraping feature into the main WhatsApp Tools application.

---

## Overview

### Problem Statement

User needs to collect business contact data from Google Maps efficiently, with ability to:

1. Search by query (conventional) - e.g., "restoran jakarta barat"
2. Search by coordinate grid (mobile emulation) - e.g., specific lat/lng with 20 results max
3. Generate coordinate grids for systematic area coverage via OpenStreetMap

### Goals

- Integrate maps scraper into main app (`new-client` + `new-server`)
- Provide user-friendly UI with real-time progress
- Save scraped data to database (Contact model)
- Format phone numbers to consistent format (Indonesian: 628xxx)
- Support CSV, Excel, JSON export + database storage
- **JobId-based navigation** - User can view job details at `/maps-scraper/jobs/[jobId]`
- **Full table features** - Search, filtering, sorting, and pagination
- **OSM Coordinate Visualization** - Toggle to show/hide coordinate points on map

---

## Project Type

| Type             | Value                |
| ---------------- | -------------------- |
| **Project Type** | WEB (Full-stack)     |
| **Backend**      | NestJS (new-server)  |
| **Frontend**     | Next.js (new-client) |
| **Database**     | SQLite (Prisma)      |
| **Approach**     | Backend First (B)    |

---

## Success Criteria

| Criteria                 | Metric                                          |
| ------------------------ | ----------------------------------------------- |
| Conventional mode works  | Can scrape 50+ places from a query              |
| Coordinate mode works    | Can scrape exactly 20 places per coordinate     |
| OSM generator works      | Can generate grid points for kecamatan/kota     |
| Phone formatting         | All phones normalized to 628xxx format          |
| Database integration     | Places saved to Contact model                   |
| Real-time progress       | Socket.io progress updates visible              |
| Export works             | CSV, Excel, JSON download functional            |
| Job detail page          | `/maps-scraper/jobs/[jobId]` shows full details |
| Table features           | Search, filter, sort, paginate all working      |
| OSM coordinate toggle    | User can show/hide coordinate markers           |
| **Automated tests pass** | Backend unit tests for critical services        |

---

## Tech Stack

> **Architecture Decision**: Use **separate Puppeteer instance** (NOT shared with WhatsApp)

| Component          | Technology                       | Rationale                                      |
| ------------------ | -------------------------------- | ---------------------------------------------- |
| Backend Framework  | NestJS                           | Already in use, consistent                     |
| Browser Automation | Puppeteer (separate instance)    | Dedicated scraper browser, no conflict with WA |
| Anti-Detection     | puppeteer-extra + stealth plugin | Lightweight wrapper (~100KB)                   |
| Database           | Prisma + SQLite                  | Already in use                                 |
| Real-time          | Socket.io / NestJS Gateway       | Existing pattern in warmer/blast               |
| Phone Formatting   | Custom util (libphonenumber-js)  | Reliable, supports Indonesian numbers          |
| OSM API            | Nominatim API                    | Free, no API key required                      |
| Grid Generation    | Turf.js                          | Powerful geospatial library                    |
| Map Visualization  | Leaflet / React-Leaflet          | Lightweight, open-source map library           |

---

## Color Palette (Perfect Color Rules)

Following `/perfect-color` workflow for professional UI:

| Element        | Light Mode                          | Dark Mode                   |
| -------------- | ----------------------------------- | --------------------------- |
| Background     | `bg-slate-50`                       | `dark:bg-zinc-900`          |
| Card           | `bg-white`                          | `dark:bg-zinc-800`          |
| Primary Text   | `text-slate-900`                    | `dark:text-slate-100`       |
| Secondary Text | `text-slate-600`                    | `dark:text-slate-400`       |
| Primary Action | `bg-indigo-600 hover:bg-indigo-700` | same + `dark:bg-indigo-500` |
| Success        | `text-emerald-600`                  | `dark:text-emerald-400`     |
| Error          | `text-rose-600`                     | `dark:text-rose-400`        |
| Warning        | `text-amber-600`                    | `dark:text-amber-400`       |
| Border         | `border-slate-200`                  | `dark:border-zinc-700`      |
| Hover State    | `hover:bg-slate-100`                | `dark:hover:bg-zinc-700`    |

---

## File Structure

```
new-server/src/core/
├── maps-scraper/
│   ├── maps-scraper.module.ts        [NEW]
│   ├── maps-scraper.controller.ts    [NEW]
│   ├── maps-scraper.service.ts       [NEW]
│   ├── maps-scraper.gateway.ts       [NEW] (Socket.io)
│   ├── services/
│   │   ├── conventional-scraper.service.ts  [NEW]
│   │   ├── coordinate-scraper.service.ts    [NEW]
│   │   ├── osm-generator.service.ts         [NEW]
│   │   ├── browser-manager.service.ts       [NEW] (Separate instance)
│   │   └── network-parser.ts                [NEW] (Port from Python)
│   ├── utils/
│   │   ├── phone-formatter.util.ts   [NEW]
│   │   └── maps-url-builder.util.ts  [NEW]
│   ├── dto/
│   │   ├── create-job.dto.ts         [NEW]
│   │   ├── generate-coords.dto.ts    [NEW]
│   │   ├── query-results.dto.ts      [NEW] (pagination, filter, sort)
│   │   └── scraped-place.dto.ts      [NEW]
│   └── types/
│       └── maps-scraper.types.ts     [NEW]

new-server/prisma/
└── schema.prisma                     [MODIFY] Add MapsScrapeJob, MapsScrapeResult

new-server/src/core/maps-scraper/
└── __tests__/                        [NEW] Unit tests
    ├── phone-formatter.util.spec.ts
    ├── conventional-scraper.service.spec.ts
    ├── coordinate-scraper.service.spec.ts
    └── osm-generator.service.spec.ts

new-client/src/
├── app/(dashboard)/maps-scraper/
│   ├── page.tsx                      [MODIFY] Main page
│   ├── jobs/
│   │   └── [jobId]/
│   │       └── page.tsx              [NEW] Job detail page
│   └── components/
│       ├── mode-selector.tsx         [NEW]
│       ├── conventional-form.tsx     [MODIFY]
│       ├── coordinate-form.tsx       [MODIFY]
│       ├── osm-generator.tsx         [NEW]
│       ├── results-table.tsx         [NEW] Full features
│       ├── coordinate-map.tsx        [NEW] Leaflet map
│       ├── progress-display.tsx      [NEW]
│       └── save-dialog.tsx           [MODIFY]
├── hooks/
│   └── use-maps-scraper.ts           [MODIFY]
└── services/
    └── maps-scraper.service.ts       [NEW]
```

---

## Task Breakdown

### Phase 1: Database Schema

---

#### Task B1: Schema Update

**Agent:** `backend-specialist`  
**Skills:** `database-design`, `clean-code`  
**Priority:** P0 (Foundation)  
**Dependencies:** None

**Schema Changes:**

```prisma
model MapsScrapeJob {
  id              String   @id @default(uuid())
  name            String?  // User-defined job name
  mode            String   @default("CONVENTIONAL") // CONVENTIONAL, COORDINATE
  status          String   @default("PENDING")      // PENDING, RUNNING, PAUSED, COMPLETED, FAILED

  // Configuration
  queryTemplate   String?  // For conventional: "apotek {location}"
  queries         String?  // JSON array of queries
  coordinates     String?  // JSON array of {lat, lng, query}

  // Progress
  totalQueries    Int      @default(0)
  currentIndex    Int      @default(0)
  totalResults    Int      @default(0)

  // Settings
  isHeadless      Boolean  @default(true)
  saveToContacts  Boolean  @default(false)
  phoneFilter     String   @default("ALL") // ALL, WITH_PHONE, WITHOUT_PHONE

  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  startedAt       DateTime?
  completedAt     DateTime?

  // Relations
  results         MapsScrapeResult[]

  @@index([status])
  @@index([createdAt])
}

model MapsScrapeResult {
  id              String   @id @default(uuid())
  jobId           String
  job             MapsScrapeJob @relation(fields: [jobId], references: [id], onDelete: Cascade)

  // Place Data
  name            String
  phoneRaw        String?  // Original phone from maps
  phoneFormatted  String?  // Normalized to 628xxx
  address         String?
  fullAddress     String?
  category        String?
  rating          Float?
  reviewsCount    Int?
  website         String?
  email           String?
  hours           String?
  mapsLink        String?

  // Coordinates
  latitude        Float?
  longitude       Float?

  // Source
  sourceLat       Float?
  sourceLng       Float?
  query           String?

  // Status
  savedToContact  Boolean  @default(false)
  contactId       String?  // Link to Contact if saved

  createdAt       DateTime @default(now())

  @@unique([jobId, mapsLink]) // Prevent duplicates
  @@index([jobId])
  @@index([phoneFormatted])
  @@index([name])
}
```

**VERIFY:**

```bash
cd new-server
npx prisma migrate dev --name add_maps_scraper
npx prisma generate
```

---

### Phase 2: Backend Services

---

#### Task B2: Browser Manager Service

**Agent:** `backend-specialist`  
**Skills:** `nodejs-best-practices`  
**Priority:** P0 (Foundation)  
**Dependencies:** B1

**Purpose:** Manage dedicated Puppeteer instance for scraping (separate from WhatsApp)

**Key Features:**

- Launch/close browser on demand
- Apply stealth plugin
- Handle browser crashes gracefully
- Configure viewport for mobile/desktop emulation

**VERIFY:**

```bash
npm run test -- browser-manager
```

---

#### Task B3: Module Structure

**Agent:** `backend-specialist`  
**Skills:** `nodejs-best-practices`, `clean-code`  
**Priority:** P1  
**Dependencies:** B1, B2

**OUTPUT:**

- `maps-scraper.module.ts` with all providers
- Controller, Service, Gateway shells
- Register in `app.module.ts`

**VERIFY:**

```bash
cd new-server
npm run build
```

---

#### Task B4: Phone Formatter Utility

**Agent:** `backend-specialist`  
**Skills:** `clean-code`  
**Priority:** P1  
**Dependencies:** None

**INPUT:** Various phone formats: `(022) 123-4567`, `+62 812 3456 7890`, `08123456789`

**OUTPUT:** Utility that normalizes to `628123456789` format

**VERIFY (Unit Test):**

```typescript
// phone-formatter.util.spec.ts
describe("formatPhone", () => {
  it("should format mobile with dash", () => {
    expect(formatPhone("0812-3456-7890")).toBe("6281234567890");
  });
  it("should format with country code", () => {
    expect(formatPhone("+62 812 3456 7890")).toBe("6281234567890");
  });
  it("should format landline", () => {
    expect(formatPhone("(022) 123-4567")).toBe("62221234567");
  });
  it("should return null for invalid", () => {
    expect(formatPhone("invalid")).toBeNull();
  });
});
```

Run: `npm run test -- phone-formatter`

---

#### Task B5: Conventional Scraper Service

**Agent:** `backend-specialist`  
**Skills:** `nodejs-best-practices`  
**Priority:** P1  
**Dependencies:** B2, B3, B4

**INPUT:** Query string like "apotek bandung"

**OUTPUT:** Array of scraped places with phone formatted

**Implementation Notes:**

- Port logic from Python `main.py`
- Use hybrid extraction (DOM + Network interception)
- Aggressive scrolling for full results
- Emit WebSocket events for real-time updates

**VERIFY (Unit Test + Manual):**

```bash
# Unit test with mocked Puppeteer
npm run test -- conventional-scraper

# Manual test (run server, use curl)
curl -X POST http://localhost:3001/api/maps-scraper/jobs \
  -H "Content-Type: application/json" \
  -d '{"mode": "CONVENTIONAL", "queries": ["kafe bandung"]}'
```

Expected: Job created, scrape returns 50+ results

---

#### Task B6: Coordinate Scraper Service

**Agent:** `backend-specialist`  
**Skills:** `nodejs-best-practices`  
**Priority:** P1  
**Dependencies:** B2, B3, B4

**INPUT:** Coordinate `{lat: -6.9175, lng: 107.6191, query: "apotek"}`

**OUTPUT:** Array of **exactly 20 places** near coordinate

**Implementation Notes:**

- iPhone emulation for mobile view
- Max 20 results per coordinate (mobile limit)
- Network interception for complete data

**VERIFY (Unit Test + Manual):**

```bash
# Unit test
npm run test -- coordinate-scraper

# Manual test
curl -X POST http://localhost:3001/api/maps-scraper/jobs \
  -H "Content-Type: application/json" \
  -d '{"mode": "COORDINATE", "coordinates": [{"lat": -6.9175, "lng": 107.6191, "query": "apotek"}]}'
```

Expected: Job created, scrape returns exactly 20 results

---

#### Task B7: OSM Generator Service

**Agent:** `backend-specialist`  
**Skills:** `api-patterns`  
**Priority:** P1 (Required for v1)  
**Dependencies:** B3

**INPUT:**

- Area name: "Kecamatan Nogosari, Boyolali"
- Grid spacing: 500m (optional, default 500m)

**OUTPUT:**

- Array of street names (mode: streets)
- Array of grid coordinates with spacing (mode: grid)
- Bounding box for the area

**Implementation Notes:**

- Use Nominatim API for geocoding + boundary
- Use Turf.js for grid point generation
- Cache results to avoid API abuse

**VERIFY:**

```bash
# Test Nominatim query
curl "https://nominatim.openstreetmap.org/search?q=Kecamatan+Nogosari&format=json&polygon_geojson=1"

# Unit test
npm run test -- osm-generator
```

---

#### Task B8: WebSocket Gateway

**Agent:** `backend-specialist`  
**Skills:** `nodejs-best-practices`  
**Priority:** P1  
**Dependencies:** B5, B6

**Events:**
| Event | Payload |
| ------------------- | -------------------------------------------- |
| `scraper:status` | `{ jobId, status }` |
| `scraper:progress` | `{ jobId, current, total, percentage }` |
| `scraper:data` | `{ jobId, results: MapsScrapeResult[] }` |
| `scraper:error` | `{ jobId, error: string }` |
| `scraper:complete` | `{ jobId, totalResults }` |

**VERIFY:** Browser console shows socket events during scrape

---

#### Task B9: REST Endpoints

**Agent:** `backend-specialist`  
**Skills:** `api-patterns`  
**Priority:** P1  
**Dependencies:** B5, B6, B7

**Endpoints:**

| Method | Path                                      | Description                                  |
| ------ | ----------------------------------------- | -------------------------------------------- |
| POST   | `/maps-scraper/jobs`                      | Create scrape job                            |
| GET    | `/maps-scraper/jobs`                      | List all jobs (with pagination)              |
| GET    | `/maps-scraper/jobs/:id`                  | Get job detail                               |
| GET    | `/maps-scraper/jobs/:id/results`          | Get results (search, filter, sort, paginate) |
| POST   | `/maps-scraper/jobs/:id/start`            | Start/resume job                             |
| POST   | `/maps-scraper/jobs/:id/pause`            | Pause job                                    |
| POST   | `/maps-scraper/jobs/:id/stop`             | Stop job                                     |
| DELETE | `/maps-scraper/jobs/:id`                  | Delete job                                   |
| POST   | `/maps-scraper/generate-coords`           | Generate coordinates from OSM                |
| GET    | `/maps-scraper/jobs/:id/export`           | Export results (CSV/Excel/JSON)              |
| POST   | `/maps-scraper/jobs/:id/save-to-contacts` | Save to Contact model                        |

**Query Params for `/results`:**

- `search`: string (search name, address, phone)
- `category`: string (filter by category)
- `hasPhone`: boolean (filter with/without phone)
- `sortBy`: name | rating | createdAt
- `sortOrder`: asc | desc
- `page`: number (default 1)
- `limit`: number (default 50)

**VERIFY:**

```bash
curl http://localhost:3001/api/maps-scraper/jobs/xxx/results?search=cafe&hasPhone=true&sortBy=rating&sortOrder=desc&page=1&limit=20
```

---

### Phase 3: Frontend UI

---

#### Task F1: Hook & Service Update

**Agent:** `frontend-specialist`  
**Skills:** `nextjs-react-expert`  
**Priority:** P0  
**Dependencies:** B9

**Files:**

- `hooks/use-maps-scraper.ts` - Connect to backend API + WebSocket
- `services/maps-scraper.service.ts` - API methods

**VERIFY:** Console log shows API responses

---

#### Task F2: Job Detail Page

**Agent:** `frontend-specialist`  
**Skills:** `frontend-design`, `/perfect-color`  
**Priority:** P1  
**Dependencies:** F1

**Route:** `/maps-scraper/jobs/[jobId]`

**Components:**

- Job header (name, status, progress bar)
- Tabbed results (Table, Map View)
- Action buttons (Start, Pause, Stop, Export, Save to Contacts)

**VERIFY:** Navigate to `/maps-scraper/jobs/xxx` shows job detail

---

#### Task F3: Results Table (Full Features)

**Agent:** `frontend-specialist`  
**Skills:** `frontend-design`, `clean-code`  
**Priority:** P1  
**Dependencies:** F2

**Features:**
| Feature | Implementation |
| ----------- | ------------------------------------------- |
| Search | Debounced input, search name/address/phone |
| Filter | Category dropdown, hasPhone checkbox |
| Sort | Click column header to sort |
| Pagination | Page controls, items per page selector |
| Selection | Checkbox for bulk actions |
| Actions | View on Maps, Save to Contact |

**Columns:** Name, Phone, Category, Address, Rating, Reviews, Website

**VERIFY:** All table features work in `/maps-scraper/jobs/[jobId]`

---

#### Task F4: Coordinate Map Component

**Agent:** `frontend-specialist`  
**Skills:** `frontend-design`  
**Priority:** P1  
**Dependencies:** F2

**Features:**

- Leaflet map with OpenStreetMap tiles
- Display coordinate points as markers
- Toggle visibility (show/hide button)
- Click marker to see result details
- Cluster markers when zoomed out

**VERIFY:** Map shows coordinates, toggle works

---

#### Task F5: OSM Generator Panel

**Agent:** `frontend-specialist`  
**Skills:** `frontend-design`  
**Priority:** P1 (Required for v1)  
**Dependencies:** F1

**Features:**

- Area name input with autocomplete
- Grid spacing slider (100m - 1000m)
- Generate button
- Preview generated coordinates on mini-map
- Add to coordinate list button

**VERIFY:** Enter "Kecamatan Coblong" → generates grid → shows on map

---

#### Task F6: Polish Forms & Main Page

**Agent:** `frontend-specialist`  
**Skills:** `frontend-design`, `/perfect-color`  
**Priority:** P2  
**Dependencies:** F3, F4, F5

**Updates:**

- Apply perfect-color palette
- Ensure dark mode works
- Mobile responsive
- Loading states
- Error handling UI

**VERIFY:** UI looks professional in light and dark mode

---

### Phase 4: Testing

---

#### Task T1: Backend Unit Tests

**Agent:** `backend-specialist`  
**Skills:** `testing-patterns`  
**Priority:** P2  
**Dependencies:** B4, B5, B6, B7

**Test Files:**

- `phone-formatter.util.spec.ts`
- `conventional-scraper.service.spec.ts`
- `coordinate-scraper.service.spec.ts`
- `osm-generator.service.spec.ts`

**Run All:**

```bash
cd new-server
npm run test -- --grep "maps-scraper"
```

---

#### Task T2: Integration Test

**Agent:** `backend-specialist`  
**Skills:** `testing-patterns`  
**Priority:** P3  
**Dependencies:** B9

**Test:** Create job → Start → Wait for results → Verify data

```bash
npm run test:e2e -- maps-scraper
```

---

#### Task T3: Manual End-to-End Test

**Agent:** User  
**Priority:** P3  
**Dependencies:** All

**Steps:**

1. Navigate to `/maps-scraper`
2. **Test Conventional Mode:**
   - Enter query "kafe hegarmanah bandung"
   - Click Start
   - Verify progress updates
   - Verify 50+ results in table
3. **Test Coordinate Mode:**
   - Generate coordinates for "Kecamatan Coblong, Bandung"
   - Start scrape
   - Verify 20 results per coordinate
4. **Test Table Features:**
   - Search "cafe"
   - Filter by hasPhone
   - Sort by rating
   - Navigate pages
5. **Test Map:**
   - Toggle coordinate visibility
   - Click marker to see details
6. **Test Export:**
   - Export to CSV
   - Verify file downloads
7. **Test Save to Contacts:**
   - Select results
   - Save to contacts
   - Verify in Contacts page

---

## Phase X: Final Verification Checklist

```markdown
## ✅ PHASE X CHECKLIST

### Code Quality

- [ ] `npm run lint` passes
- [ ] `npm run build` succeeds (both client and server)
- [ ] No TypeScript errors

### Security

- [ ] Rate limiting on scrape endpoints
- [ ] Input validation on all DTOs

### Testing

- [ ] Unit tests pass: `npm run test -- maps-scraper`
- [ ] Phone formatter handles all edge cases
- [ ] Coordinate mode returns exactly 20 results

### Manual Verification

- [ ] Conventional mode scrapes 50+ results
- [ ] Coordinate mode scrapes 20 results per point
- [ ] OSM generator creates grid for kecamatan
- [ ] Phone numbers normalized to 628xxx
- [ ] Export to CSV works
- [ ] Save to contacts works
- [ ] Real-time progress visible
- [ ] Table search/filter/sort/paginate works
- [ ] Map toggle shows/hides coordinates

### UI Quality (Perfect Color)

- [ ] No default HTML colors (red, blue, green)
- [ ] Uses slate/zinc for neutrals
- [ ] Uses indigo for primary actions
- [ ] Uses emerald/rose/amber for semantic colors
- [ ] Dark mode fully supported
- [ ] WCAG contrast requirements met
```

---

## Risk Assessment

| Risk                 | Mitigation                               |
| -------------------- | ---------------------------------------- |
| Google rate limiting | Add delays, rotate user agents           |
| Selector changes     | Document selectors, easy to update       |
| OSM API limits       | Cache results, respect rate limits       |
| Large datasets       | Pagination, streaming, virtual scrolling |
| Browser crashes      | Auto-restart, graceful error handling    |

---

## Timeline Estimate

| Phase            | Duration        |
| ---------------- | --------------- |
| Schema + Backend | 6-8 hours       |
| Frontend UI      | 4-6 hours       |
| Testing & Polish | 2-3 hours       |
| **Total**        | **12-17 hours** |

---

## Execution Order

```
B1 → B2 → B3 → B4 → B5 → B6 → B7 → B8 → B9 → F1 → F2 → F3 → F4 → F5 → F6 → T1 → T2 → T3
```

1. **Start with Backend First** - Complete B1-B9 before frontend
2. **Use Existing Patterns** - Follow warmer/blast module structure
3. **Incremental Testing** - Test each service as built
4. **Perfect Color** - Apply color rules during UI development
5. **Phone Formatting is Critical** - Use libphonenumber-js for reliability
