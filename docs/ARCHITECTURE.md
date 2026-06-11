# System Architecture - BK Pergola et Alu

**Document Purpose:** Complete system design, deployment model, data flow, and technical decision rationale.

---

## 1. System Overview

This document defines the production architecture for a lead-generation website for BK Pergola et Alu. The system is designed for:

- **~3,000 monthly visitors** (scale-ready, no premature optimization)
- **Student-friendly complexity** (simple, readable, documented)
- **OVHcloud Startup hosting** (single server, included MySQL)
- **Real client business** (production-grade security & reliability)

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        End Users (Browser)                      │
└──────────────────────────┬───────────────────────────────────────┘
                           │ HTTPS (SSL/TLS)
                           ↓
┌─────────────────────────────────────────────────────────────────┐
│                    OVHcloud Server (Startup)                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Frontend (Angular)                                         │ │
│  │ ├─ Static HTML/CSS/JS (built, served by nginx/Apache)    │ │
│  │ └─ http://localhost:4200 (dev) → dist/ (prod)           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↕                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Backend (Symfony REST API)                                 │ │
│  │ ├─ /api/gallery (GET, POST, PUT, DELETE)                 │ │
│  │ ├─ /api/contacts (POST, GET dashboard)                   │ │
│  │ ├─ /api/auth (login, logout)                             │ │
│  │ ├─ /api/uploads (image handling)                         │ │
│  │ └─ public/index.php (entry point)                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           ↕                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Database (MySQL)                                           │ │
│  │ ├─ projects                                               │ │
│  │ ├─ gallery_items                                          │ │
│  │ ├─ contacts                                               │ │
│  │ ├─ admins                                                 │ │
│  │ └─ team_members                                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ File Storage                                               │ │
│  │ ├─ /storage/gallery/ (gallery images)                    │ │
│  │ ├─ /storage/thumbs/ (optimized thumbnails)              │ │
│  │ └─ /public/uploads/ (web-accessible)                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Email Service (Zimbra SMTP)                               │ │
│  │ └─ contact@bk-pergola.tn (outbound notifications)        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Technology Stack Rationale

### Frontend: Angular 17.x Standalone + TailwindCSS

**Why Angular?**
- You have experience with Angular
- Modern, production-ready framework
- Built-in routing, guards, dependency injection
- Strong community, excellent documentation

**Why Standalone Components?**
- Simplified approach (no NgModules to manage)
- Modern Angular 14+ feature
- Easier for students to understand
- Less boilerplate code

**Why TailwindCSS (not Bootstrap)?**
- You wanted to avoid "Bootstrap template look"
- Utility-first approach = full design control
- Learning curve moderate, very flexible
- No CSS class naming headaches
- Better performance (unused CSS tree-shaking)

**State Management: Services + RxJS (not NgRx)**
- For 3K users + single admin dashboard, simple services are sufficient
- You can upgrade to NgRx later if needed
- Less complexity, easier to learn and test
- Pattern: Centralized services (GalleryService, ContactService) emit data via Observables

### Backend: Symfony 6.x REST API

**Why Symfony?**
- You have Twig experience, can ramp up quickly
- Built-in security, validation, error handling
- Doctrine ORM = type-safe, migrations, relationships
- RESTful by default, no extra framework needed

**Why REST (not GraphQL)?**
- Simpler for first project
- Stateless, cacheable, scalable
- Easier to test and debug
- Sufficient for this feature set (not complex queries)

**Why not API Platform?**
- Yes, we could use Symfony's API Platform bundle
- But for learning purposes, raw controllers are clearer
- Can add API Platform later for auto-documentation

### Database: MySQL 8.0 + Doctrine ORM

**Why MySQL?**
- Included with OVHcloud Startup hosting
- Mature, proven, reliable
- Good performance for expected volume
- Simple to back up

**Why Doctrine ORM (not raw SQL)?**
- Prevents SQL injection automatically
- Type-safe relationships
- Automatic migrations
- Code-first database design

---

## 3. Data Flow & Component Interaction

### User Access Flow (Example: Viewing Pinterest Gallery)

```
1. User visits https://bk-pergola.tn/gallery
   │
2. Angular router loads GalleryComponent
   │
3. Component calls GalleryService.getAllItems()
   │
4. GalleryService makes HTTP GET /api/gallery
   │
5. Symfony GalleryController receives request
   │
6. GalleryController queries Database via GalleryItemRepository
   │
7. MySQL returns gallery items with image references
   │
8. GalleryController returns JSON response (thumbnails, descriptions, keywords)
   │
9. GalleryService parses JSON, emits via Observable
   │
10. GalleryComponent subscribes, receives data
   │
11. Component displays Pinterest grid (lazy-loaded images)
```

### User Clicks Photo → Lightbox Opens

```
1. User clicks photo in grid
   │
2. Angular calls GalleryService.getItem(id)
   │
3. GalleryService makes HTTP GET /api/gallery/{id}
   │
4. Symfony GalleryController:
   ├─ Gets the item details
   ├─ If item has project_id → fetches related photos
   └─ Returns { item, description, keywords, relatedPhotos }
   │
5. Lightbox opens with:
   ├─ Main photo (full size)
   ├─ Description (FR)
   ├─ Keywords
   └─ (If has project) Related photos below
```

### Search Gallery by Keywords

```
1. User types "pergola" in search box
   │
2. Angular calls GalleryService.search('pergola')
   │
3. GalleryService makes HTTP GET /api/gallery/search?q=pergola
   │
4. Symfony searches database:
   ├─ FULLTEXT search on keywords column
   ├─ OR description contains "pergola"
   └─ Returns matching items
   │
5. GalleryService emits results
   │
6. Component displays filtered gallery
```

### Admin Login Flow

```
1. Admin visits https://bk-pergola.tn/admin/login
   │
2. Admin submits username + password
   │
3. AuthService calls POST /api/auth/login
   │
4. Symfony AdminAuthenticator verifies credentials against admins table
   │
5. On success:
   ├─ Creates session (PHP $_SESSION)
   ├─ Returns session token
   └─ Stores in Angular AuthService
   │
6. Angular Router Guard checks authenticated state
   │
7. If valid, allows access to admin pages
   │
8. On logout, destroys session
```

### Admin Uploads Gallery Photo

```
1. Admin goes to /admin/gallery
   │
2. Admin chooses:
   ├─ Option A: New standalone photo (no project)
   ├─ Option B: Add to existing project
   └─ Option C: Create new project + add photos
   │
3. Admin:
   ├─ Selects image file
   ├─ Types description (FR)
   ├─ Enters keywords (comma-separated)
   ├─ Optionally selects project
   └─ Clicks "Upload"
   │
4. Angular UploadService validates:
   ├─ File type (jpg, png, webp)
   ├─ File size (<5MB)
   └─ Dimensions (min 800px)
   │
5. Uploads to POST /api/gallery/upload
   │
6. Symfony ImageService:
   ├─ Moves file to /storage/gallery/
   ├─ Generates thumbnail (200x200)
   ├─ Optimizes for web (compression)
   └─ Returns image_path + thumbnail_path
   │
7. Symfony saves to gallery_items table:
   ├─ project_id (null if standalone)
   ├─ image_path
   ├─ thumbnail_path
   ├─ description
   ├─ keywords
   └─ featured flag
   │
8. Admin sees confirmation: "Photo uploaded!"
```

### Lead Capture Flow

```
User fills contact form:
   │
2. Angular ContactForm validates (email format, required fields)
   │
3. User clicks "Send"
   │
4. ContactService calls POST /api/contacts
   │
5. Symfony ContactController:
   ├─ Validates input
   ├─ Saves to contacts table
   ├─ Sends email to contact@bk-pergola.tn
   └─ Returns success response
   │
6. Angular shows "Message sent" confirmation
   │
7. Admin sees new lead in dashboard (tab: "Contacts")
```

---

## 4. Security Architecture

### Authentication & Authorization

**Admin Login:**
- Username + Password stored in `admins` table
- Password hashed with bcrypt (PHP's `password_hash()`)
- Session stored server-side (PHP `$_SESSION`)
- Session token sent to client in secure HTTP-only cookie
- Session timeout: 8 hours of inactivity

**Session Guard (Frontend):**
- Every admin route protected by `AdminAuthGuard`
- Guard checks if session is valid
- If expired/invalid, redirects to login

**API Endpoints:**
- Public endpoints: `/api/gallery` (GET), `/api/contacts` (POST), `/api/projects` (GET)
- Protected endpoints: `/api/gallery` (POST, PUT, DELETE), all admin endpoints
- Each request checked for valid session before processing

### Input Validation & Sanitization

**Frontend:**
- Angular FormValidation for email, phone, required fields
- Type checking on TypeScript models
- HTML sanitization (Angular sanitizes by default)

**Backend:**
- Symfony validation constraints on entities
- Doctrine query builder prevents SQL injection
- Input filtering (trim, strip tags where needed)

### CORS (Cross-Origin Resource Sharing)

**Configuration:**
- Same domain (no CORS issues initially)
- If future mobile app: Configure CORS headers to allow specific origins
- Default: Allow same-origin only

### CSRF Protection

**Forms:**
- Symfony CSRF tokens on state-changing requests (POST, PUT, DELETE)
- Angular includes token from backend in X-CSRF-TOKEN header
- Server verifies on each request

---

## 5. Deployment Model

### Single-Server Architecture (OVHcloud Startup)

**Current:**
- One physical server in Tunisia
- Frontend + Backend + Database on same machine
- Shared resources (CPU, RAM, disk)

**Why this works for now:**
- 3,000 visitors/month = ~4 concurrent users peak
- Startup plan CPU/RAM sufficient
- No complex scaling needed yet
- Simpler to manage and monitor

**Future Scaling:**
- **Phase 2:** Separate database server (if traffic grows)
- **Phase 3:** Multiple app servers with load balancer
- **Phase 4:** Caching layer (Redis) for frequently accessed data
- Architecture already supports this (stateless API design)

### Deployment Process

```
Development (Local)
   │
   ├─ Frontend: npm run build → dist/ folder
   │
   ├─ Backend: composer install + php artisan migrate
   │
Test & Verify (Local)
   │
Deploy to OVHcloud
   │
   ├─ Upload dist/ folder to web root
   │
   ├─ Upload backend/ folder to app directory
   │
   ├─ SSH: Run migrations (php bin/console doctrine:migrations:migrate)
   │
   ├─ SSH: Clear caches (php bin/console cache:clear --env=prod)
   │
   └─ Update DNS if needed
```

**See [DEPLOYMENT.md](./DEPLOYMENT.md) for step-by-step instructions.**

---

## 6. Database Architecture (Updated for Pinterest Gallery)

### Entity Relationships

```
admins (1) ──→ (many) projects
           └─→ (many) created galleries

projects (1) ──→ (many) gallery_items
          └─→ (0..1) per item [optional]

gallery_items 
  ├─ Can be standalone (project_id = NULL)
  ├─ Or linked to project (project_id = project.id)
  └─ Searchable by keywords

contacts (0..1) ──→ (many) admins
               [admin assigned - optional]

team_members [standalone - no foreign keys]
```

### Table Overview (Updated)

| Table | Purpose | Key Fields |
|-------|---------|-----------|
| `admins` | Website administrators | id, username, password_hash, email, created_at |
| `projects` | Portfolio project groupings | id, title, description, category, featured, created_at |
| `gallery_items` | Individual photos (NEW - replaces project_images) | id, project_id (nullable), image_path, description, keywords, featured, uploaded_at |
| `contacts` | Lead submissions | id, name, email, phone, message, status, source, created_at |
| `team_members` | Team profiles | id, name, title, bio, image_path, position |

**Key Change:** `gallery_items` replaces `project_images`
- Old: Photos mandatory tied to projects
- New: Photos can be standalone OR linked to projects
- Enables Pinterest-like flexible gallery

**See [DATABASE.md](./DATABASE.md) for complete schema with SQL.**

---

## 7. API Design Principles

### RESTful Endpoints

**Naming Convention:**
- Resources as nouns: `/api/gallery`, `/api/contacts`, `/api/projects`
- HTTP verbs for actions: GET (read), POST (create), PUT (update), DELETE (remove)
- IDs in path: `/api/gallery/5`

**Example Endpoints (Gallery-focused):**

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/gallery` | List all gallery items (Pinterest feed) |
| GET | `/api/gallery/{id}` | Get single item with related project photos |
| GET | `/api/gallery/search?q=pergola` | Search by keywords |
| POST | `/api/gallery` | Upload new photo (admin) |
| PUT | `/api/gallery/{id}` | Update photo details (admin) |
| DELETE | `/api/gallery/{id}` | Delete photo (admin) |
| GET | `/api/projects` | List all projects |
| POST | `/api/projects` | Create project (admin) |
| POST | `/api/contacts` | Submit contact form (public) |
| GET | `/api/contacts` | List leads (admin) |
| POST | `/api/auth/login` | Admin login |
| POST | `/api/auth/logout` | Admin logout |

**Response Format:**
- Success: `{ "data": {...}, "message": "OK" }` with HTTP 200
- Error: `{ "error": "...", "details": "..." }` with HTTP 4xx/5xx

**See [API_SPEC.md](./API_SPEC.md) for complete specification.**

---

## 8. Frontend Architecture Layers

### Component Hierarchy

```
AppComponent (root)
├─ HeaderComponent (nav, language switch)
├─ RouterOutlet (page routing)
└─ FooterComponent (company info, socials)

Public Pages:
├─ HomePage
│  ├─ HeroSection
│  ├─ FeaturedGallery (uses GalleryGrid)
│  └─ CTASection
├─ GalleryPage (Pinterest feed)
│  ├─ SearchBar
│  ├─ GalleryGrid
│  │  └─ GalleryCard (loop) - shows thumbnail
│  └─ GalleryLightbox (opens on photo click)
│     ├─ Main photo (full size)
│     ├─ Description + keywords
│     └─ RelatedPhotos (if part of project)
├─ ServicesPage
├─ TeamPage
├─ ContactPage
│  └─ ContactForm
├─ AboutPage
├─ AdminLoginPage
│  └─ LoginForm
└─ AdminDashboard (guard-protected)
   ├─ DashboardHome
   ├─ GalleryManagement (CRUD + upload)
   │  ├─ UploadForm (standalone or to project)
   │  ├─ GalleryList (with edit/delete)
   │  └─ ProjectManager
   ├─ LeadDashboard (contacts)
   │  └─ LeadList (status, source, filters)
   └─ TeamManagement
```

### Service Layer

```
Injectable Services:
├─ GalleryService (GET/POST/PUT/DELETE gallery items, search)
├─ ProjectService (GET/POST/PUT/DELETE projects)
├─ ContactService (POST contact, GET leads)
├─ AuthService (login, logout, session check)
├─ UploadService (file upload handling)
├─ I18nService (language switching)
└─ NotificationService (toast messages)

Observable Pattern:
- Each service exposes Observable<Data>
- Components subscribe with | async pipe
- Automatic unsubscribe on component destroy
```

### Routing & Guards

```
Routes:
├─ / (home)
├─ /gallery (Pinterest gallery)
├─ /gallery/:id (lightbox view)
├─ /services (services page)
├─ /team (team page)
├─ /contact (contact form)
├─ /about (about page)
├─ /admin/login (public)
└─ /admin/** (protected by AdminAuthGuard)
   ├─ /admin/dashboard
   ├─ /admin/gallery (CRUD)
   ├─ /admin/projects (CRUD)
   ├─ /admin/leads (contact dashboard)
   └─ /admin/team (team profiles)

Guards:
└─ AdminAuthGuard
   ├─ Check if session is valid
   ├─ If not, redirect to /admin/login
   └─ If expired, log out and redirect
```

---

## 9. Backend Architecture Layers

### Controller Layer

- Receives HTTP requests
- Delegates to Services
- Returns JSON responses
- Handles error responses

### Service Layer

- Contains business logic
- Validates input
- Calls repositories
- Sends emails, handles files
- No direct HTTP knowledge

### Repository Layer

- Doctrine ORM queries
- Type-safe, auto-generated
- Example: `GalleryItemRepository::searchByKeywords()`
- Example: `ContactRepository::findRecent(10)`

### Entity Layer

- Database models (Doctrine entities)
- Validation constraints (annotations)
- Relationships defined here
- Migrations generated from entities

---

## 10. Gallery Image Handling & File Storage (Updated)

### Upload Flow

```
1. Admin selects image in gallery uploader
2. Angular UploadService validates:
   ├─ File type (jpg, png, webp)
   ├─ File size (<5MB)
   └─ Dimensions (min 800px wide)
3. Uploads to POST /api/gallery/upload
4. Symfony ImageService:
   ├─ Moves file to /storage/gallery/
   ├─ Generates thumbnail (200x200)
   ├─ Optimizes for web (compression)
   └─ Returns image_path + thumbnail_path
5. Symfony saves to gallery_items:
   ├─ image_path, thumbnail_path
   ├─ description (FR)
   ├─ keywords (searchable)
   ├─ project_id (null if standalone)
   └─ featured flag
6. Frontend displays in gallery grid
```

### Image Optimization

- **Storage:** Original files in `/storage/gallery/`
- **Thumbnails:** Auto-generated 200x200px in `/storage/thumbs/`
- **Lazy Loading:** Images load on scroll (Angular lazy-load directive)
- **Responsive Images:** Multiple sizes for mobile/desktop (CSS)
- **Format:** Accept JPG, PNG; convert to WebP where supported

---

## 11. Email System

### Configuration

- **SMTP Server:** OVHcloud Zimbra (included with hosting)
- **From Address:** contact@bk-pergola.tn
- **Credentials:** Zimbra SMTP username/password in `.env`

### Email Triggers

| Event | Recipient | Template |
|-------|-----------|----------|
| Contact form submitted | contact@bk-pergola.tn | New lead notification |
| Admin uploads gallery item | (optional) | Confirmation |
| (Optional) Contact reply | user email | Custom response |

### Implementation

- Symfony Mailer component
- Twig email templates in `templates/emails/`
- Queue-based (optional for later scaling)

---

## 12. Internationalization (i18n) Architecture

### Day 1: French Only

- **Frontend:** All UI text in French (hardcoded + translation service)
- **Backend:** API responses in English (standard for APIs)
- **Database:** No language column (single language initially)

### Future: Add Arabic

**Infrastructure Ready:**
- Angular `localize` module installed
- Translation JSON files: `src/assets/i18n/fr.json`, `ar.json`
- i18n service handles switching
- TailwindCSS supports RTL (class `dir="rtl"`)

**HTML Layout:**
- French: `<html lang="fr" dir="ltr">`
- Arabic: `<html lang="ar" dir="rtl">`
- CSS adapts automatically (Tailwind RTL utilities)

**Dictionary Files:**
- Add `ar.json` when translating (future task)
- No code changes needed, just JSON key-value pairs

**See [I18N_SETUP.md](./I18N_SETUP.md) for implementation details.**

---

## 13. Performance & Caching Strategy

### Browser Caching

- Static assets (CSS, JS) cached for 1 year
- HTML cached for 5 minutes (content changes)
- Images cached for 30 days

### HTTP Caching Headers

- `Cache-Control: max-age=...` set by backend
- ETag for conditional requests
- 304 Not Modified responses reduce bandwidth

### Database Caching (Future)

- Currently: Direct queries
- If scaling: Add Redis for frequently accessed data
- Example: Featured gallery items cache (5-minute TTL)

### Image Optimization

- Lazy loading (images load on scroll)
- Responsive sizing (mobile 500px, desktop 1200px)
- WebP format with PNG fallback
- Compression before storage
- FULLTEXT search on keywords for fast filtering

---

## 14. Monitoring & Logging

### Error Logging

- **Backend:** Symfony logs in `var/log/`
- **Frontend:** Console logs + optional Sentry integration
- **Database:** MySQL slow query log

### Health Checks

- Status endpoint: GET `/api/health` (returns 200 if DB connected)
- Used by monitoring services to detect downtime

### Uptime Monitoring (Optional)

- UptimeRobot (free) pings `/api/health` every 5 minutes
- Alerts via email if down

---

## 15. Security Checklist

✅ **Authentication:**
- Passwords hashed with bcrypt
- Session-based (PHP $_SESSION)
- 8-hour timeout
- Logout clears session

✅ **Authorization:**
- Admin routes protected by guards
- Database checks on backend
- No business logic in frontend

✅ **Data Protection:**
- HTTPS/SSL enforced (Let's Encrypt free)
- SQL injection prevented (Doctrine ORM)
- XSS prevented (Angular sanitization)
- CSRF tokens on forms

✅ **Input Validation:**
- Frontend: Type checking + Angular validators
- Backend: Symfony constraints
- Trim, escape, filter where needed

✅ **Secrets:**
- `.env` file (not committed to Git)
- Database password, SMTP credentials, API keys in `.env`
- `.gitignore` prevents accidental commits

✅ **Rate Limiting:**
- Contact form: 5 submissions per IP per hour
- Login: 5 attempts per IP per hour

---

## 16. Deployment Architecture Summary

### Pre-Production (Development)

- Local MySQL on your laptop
- Angular dev server (`ng serve`)
- Symfony dev server (`php bin/console server:run`)
- Hot reload enabled
- Full error messages

### Production (OVHcloud)

- Managed MySQL (OVHcloud)
- Compiled Angular build (`ng build --prod`)
- Symfony in production mode (`APP_ENV=prod`)
- Error emails instead of stack traces
- Caching enabled
- Optimized database queries

---

## 17. Scaling Considerations (Future)

The current architecture is **scale-ready** (no refactoring needed if traffic grows):

### Phase 1 (Current): Single Server
- Everything on one server
- Works for 3K-10K users/month
- Current state

### Phase 2: Database Separation
- Move MySQL to dedicated server
- Keep app on web server
- Better resource management
- Easier backups

### Phase 3: Multiple App Servers
- Load balancer distributes requests
- Session data in Redis (shared across servers)
- Stateless app design ready

### Phase 4: CDN & Caching
- Static assets on CDN
- Image optimization at edge
- Redis cache layer
- Full enterprise setup

**Key:** Current REST API design supports all phases without code changes.

---

## 18. Technology Decision Summary

| Decision | Choice | Why |
|----------|--------|-----|
| Frontend | Angular 17 + Tailwind | Modern, familiar, customizable |
| Backend | Symfony 6 REST API | Proven, secure, scalable |
| Database | MySQL + Doctrine ORM | Included, type-safe, migrations |
| Gallery | Flexible gallery_items table | Supports Pinterest-style layout |
| Authentication | Sessions (bcrypt) | Simple, secure, multi-admin ready |
| State Management | Services + RxJS | No overkill, scales fine |
| Email | Zimbra SMTP | Free, included with hosting |
| Hosting | OVHcloud Startup | Tunisia-based, affordable, SSL included |
| i18n | Static JSON | Low overhead, flexible |
| Image Storage | Local disk + optimization | Free, fast for low volume |
| Styling | TailwindCSS | Utility-first, no template feel |

---

## 19. Key Files & Entry Points

### Backend Entry Point
- `backend/public/index.php` → Symfony kernel → routes.yaml → controllers

### Frontend Entry Point
- `frontend/src/main.ts` → bootstrapApplication() ��� AppComponent → routing

### Configuration Files
- `backend/.env` → Database, SMTP, app settings
- `backend/config/routes.yaml` → API routes
- `backend/src/Kernel.php` → Symfony configuration
- `frontend/src/app/app.routes.ts` → Frontend routing
- `frontend/angular.json` → Angular build config
- `frontend/tailwind.config.js` → TailwindCSS customization

---

## 20. Next Steps

1. **Read [DEVELOPMENT_PLAN.md](./DEVELOPMENT_PLAN.md)** — Week-by-week sprint schedule
2. **Read [DATABASE.md](./DATABASE.md)** — Complete schema with gallery_items table
3. **Read [API_SPEC.md](./API_SPEC.md)** — All endpoints with examples
4. **Follow [SETUP.md](./SETUP.md)** — Set up local development
5. **Follow [DEPLOYMENT.md](./DEPLOYMENT.md)** — Deploy to OVHcloud

---

**Architecture Version:** 2.0 (Updated with gallery_items table)  
**Last Updated:** June 2026  
**Status:** Ready for development
