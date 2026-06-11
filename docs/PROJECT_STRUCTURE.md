# Complete Project Structure - BK Pergola et Alu

**Document Purpose:** Complete file and folder structure for the entire project (backend + frontend). Use this as a checklist for project setup.

---

## 📁 Root Level Structure

```
bk-pergola-alu/
├── .git/                           # Git repository (auto-generated)
├── .gitignore                      # Git ignore rules
├── docker-compose.yml              # Local development environment (optional)
├── README.md                        # Main project overview
├── LICENSE                         # Project license
│
├── docs/                           # Documentation (all guides)
│   ├── README.md                   # Docs index
│   ├── ARCHITECTURE.md             # System design & flow ✅
│   ├── DATABASE.md                 # Schema & relationships ✅
│   ├── API_SPEC.md                 # All REST endpoints ✅
│   ├── PROJECT_STRUCTURE.md        # This file (file tree) ✅
│   ├── DEVELOPMENT_PLAN.md         # Week-by-week timeline
│   ├── SETUP_GUIDE.md              # Local dev setup
│   ├── DEPLOYMENT.md               # OVHcloud deployment
│   ├── SECURITY.md                 # Security checklist
│   ├── ADMIN_PANEL.md              # Admin features & mockups
│   ├── I18N_SETUP.md               # French/Arabic setup
│   └── TAILWIND_GUIDE.md           # TailwindCSS guide
│
├── backend/                        # Symfony 6 API
│   ├── src/
│   │   ├── Controller/
│   │   │   ├── AuthController.php
│   │   │   ├── GalleryController.php
│   │   │   ├── ProjectController.php
│   │   │   ├── ContactController.php
│   │   │   ├── TeamController.php
│   │   │   └── HealthController.php
│   │   │
│   │   ├── Entity/
│   │   │   ├── User.php            # Admin user
│   │   │   ├── GalleryItem.php     # Gallery photo
│   │   │   ├── Project.php         # Portfolio project
│   │   │   ├── Contact.php         # Lead submission
│   │   │   ├── Team.php            # Team member
│   │   │   └── RateLimit.php       # Rate limiting (optional)
│   │   │
│   │   ├── Repository/
│   │   │   ├── UserRepository.php
│   │   │   ├── GalleryItemRepository.php
│   │   │   ├── ProjectRepository.php
│   │   │   ├── ContactRepository.php
│   │   │   └── TeamRepository.php
│   │   │
│   │   ├── Service/
│   │   │   ├── ImageService.php    # Image upload & optimization
│   │   │   ├── GalleryService.php  # Gallery logic
│   │   │   ├── ContactService.php  # Lead handling
│   │   │   ├── EmailService.php    # Email notifications (Zimbra)
│   │   │   └── RateLimitService.php
│   │   │
│   │   ├── Security/
│   │   │   ├── PasswordHasher.php
│   │   │   └── SessionManager.php
│   │   │
│   │   ├── Validator/
│   │   │   ├── ImageValidator.php
│   │   │   ├── FormValidator.php
│   │   │   └── CustomConstraints/ (if using Symfony validators)
│   │   │
│   │   ├── Exception/
│   │   │   ├── ValidationException.php
│   │   │   ├── UnauthorizedException.php
│   │   │   └── RateLimitException.php
│   │   │
│   │   └── Kernel.php              # Symfony kernel
│   │
│   ├── config/
│   │   ├── bundles.php
│   │   ├── services.yaml           # Dependency injection
│   │   ├── routes.yaml             # API routes
│   │   └── packages/
│   │       ├── doctrine.yaml       # Database config
│   │       ├── mailer.yaml         # Email config (Zimbra SMTP)
│   │       └── security.yaml       # Security config
│   │
│   ├── migrations/
│   │   ├── Version20260611000000CreateUsers.php
│   │   ├── Version20260611000001CreateGalleryItems.php
│   │   ├── Version20260611000002CreateProjects.php
│   │   ├── Version20260611000003CreateContacts.php
│   │   └── Version20260611000004CreateTeam.php
│   │
│   ├── public/
│   │   ├── index.php               # Entry point
│   │   └── .htaccess               # Apache rewrite rules
│   │
│   ├── storage/                    # File uploads (created at runtime)
│   │   ├── gallery/                # Gallery images
│   │   │   ├── 2025/               # Year directory
│   │   │   │   └── [image files]
│   │   │   └── 2026/
│   │   ├── thumbs/                 # Generated thumbnails
│   │   │   └── 2025/
│   │   ├── team/                   # Team photos
│   │   └── temp/                   # Temporary uploads
│   │
│   ├── .env                        # Environment variables (GIT IGNORED)
│   ├── .env.example                # Template for .env
│   ├── .gitignore                  # Git ignore for backend
│   ├── composer.json               # PHP dependencies
│   ├── composer.lock               # Locked dependencies
│   ├── symfony.lock                # Symfony specific locks
│   └── bin/
│       └── console                 # Symfony CLI
│
├── frontend/                       # Angular 17 Application
│   ├── src/
│   │   ├── app/
│   │   │   │
│   │   │   ├── app.config.ts       # Root app config
│   │   │   ├── app.routes.ts       # Root routing
│   │   │   └── app.component.ts    # Root component
│   │   │
│   │   ├── pages/
│   │   │   ├── home/
│   │   │   │   ├── home.component.ts
│   │   │   │   ├── home.component.html
│   │   │   │   └── home.component.scss
│   │   │   │
│   │   ├── gallery/
│   │   │   ├── gallery-feed/
│   │   │   │   ├── gallery-feed.component.ts
│   │   │   │   ├── gallery-feed.component.html (Pinterest grid)
│   │   │   │   └── gallery-feed.component.scss
│   │   │   ├── gallery-search/
│   │   │   │   ├── gallery-search.component.ts
│   │   │   │   ├── gallery-search.component.html
│   │   │   │   └── gallery-search.component.scss
│   │   │   ├── gallery-lightbox/
│   │   │   │   ├── gallery-lightbox.component.ts (modal)
│   │   │   │   ├── gallery-lightbox.component.html
│   │   │   │   └── gallery-lightbox.component.scss
│   │   │   └── gallery.routes.ts   # Gallery sub-routes
│   │   │
│   │   ├── portfolio/
│   │   │   ├── projects-list/
│   │   │   │   ├── projects-list.component.ts
│   │   │   │   ├── projects-list.component.html
│   │   │   │   └── projects-list.component.scss
│   │   │   ├── project-detail/
│   │   │   │   ├── project-detail.component.ts
│   │   │   │   ├── project-detail.component.html
│   │   │   │   └── project-detail.component.scss
│   │   │   └── portfolio.routes.ts
│   │   │
│   │   ├── services/
│   │   │   ├── gallery.service.ts  # Gallery API calls
│   │   │   ├── project.service.ts  # Project API calls
│   │   │   ├── contact.service.ts  # Contact form submission
│   │   │   ├── team.service.ts     # Team API calls
│   │   │   ├── auth.service.ts     # Authentication (admin)
│   │   │   └── api.service.ts      # Base HTTP client
│   │   │
│   │   ├── shared/
│   │   │   ├── components/
│   │   │   │   ├── header/
│   │   │   │   │   ├── header.component.ts
│   │   │   │   │   ├── header.component.html
│   │   │   │   │   └── header.component.scss
│   │   │   │   ├── footer/
│   │   │   │   │   ├── footer.component.ts
│   │   │   │   │   ├── footer.component.html
│   │   │   │   │   └── footer.component.scss
│   │   │   │   ├── hero/
│   │   │   │   │   ├── hero.component.ts
│   │   │   │   │   ├── hero.component.html
│   │   │   │   │   └── hero.component.scss
│   │   │   │   ├── loading-spinner/
│   │   │   │   ├── error-message/
│   │   │   │   └── modal/
│   │   │   │
│   │   │   ├── pipes/
│   │   │   │   └── safe.pipe.ts    # For sanitizing HTML
│   │   │   │
│   │   │   ├── interceptors/
│   │   │   │   ├── http-error.interceptor.ts
│   │   │   │   ├── http-loader.interceptor.ts
│   │   │   │   └── http-auth.interceptor.ts
│   │   │   │
│   │   │   └── guards/
│   │   │       └── admin.guard.ts  # Route protection for admin
│   │   │
│   │   ├── admin/
│   │   │   ├── dashboard/
│   │   │   │   ├── admin-dashboard.component.ts
│   │   │   │   ├── admin-dashboard.component.html
│   │   │   │   └── admin-dashboard.component.scss
│   │   │   │
│   │   │   ├── gallery-manager/
│   │   │   │   ├── gallery-list/
│   │   │   │   │   ├── gallery-list.component.ts
│   │   │   │   │   ├── gallery-list.component.html
│   │   │   │   │   └── gallery-list.component.scss
│   │   │   │   ├── gallery-upload/
│   │   │   │   │   ├── gallery-upload.component.ts
│   │   │   │   │   ├── gallery-upload.component.html
│   │   │   │   │   └── gallery-upload.component.scss
│   │   │   │   └── gallery-edit/
│   │   │   │       ├── gallery-edit.component.ts
│   │   │   │       ├── gallery-edit.component.html
│   │   │   │       └── gallery-edit.component.scss
│   │   │   │
│   │   │   ├── project-manager/
│   │   │   │   ├── project-list/
│   │   │   │   ├── project-form/
│   │   │   │   └── project-edit/
│   │   │   │
│   │   │   ├── contact-manager/
│   │   │   │   ├── contact-dashboard.component.ts (inbox)
│   │   │   │   ├── contact-dashboard.component.html
│   │   │   │   ├── contact-detail/
│   │   │   │   └── contact-export/ (CSV)
│   │   │   │
│   │   │   ├── team-manager/
│   │   │   │   ├── team-list/
│   │   │   │   ├── team-form/
│   │   │   │   └── team-edit/
│   │   │   │
│   │   │   └── admin.routes.ts    # Admin sub-routes (protected)
│   │   │
│   │   ├── i18n/                  # Internationalization
│   │   │   ├── en.json            # English
│   │   │   ├── fr.json            # French
│   │   │   ├── ar.json            # Arabic (RTL)
│   │   │   └── i18n.service.ts    # Language switcher
│   │   │
│   │   ├── models/
│   │   │   ├── gallery.model.ts
│   │   │   ├── project.model.ts
│   │   │   ├── contact.model.ts
│   │   │   ├── team.model.ts
│   │   │   └── api-response.model.ts
│   │   │
│   │   ├── assets/
│   │   │   ├── images/
│   │   │   │   ├── logo.png
│   │   │   │   ├── logo.svg
│   │   │   │   ├── hero-bg.jpg
│   │   │   │   └── icons/
│   │   │   ├── videos/
│   │   │   │   └── [hero video or demos]
│   │   │   ├── fonts/
│   │   │   ├── styles/
│   │   │   │   ├── global.scss     # Global styles
│   │   │   │   ├── variables.scss  # Colors, sizes, fonts
│   │   │   │   ├── mixins.scss     # SCSS helpers
│   │   │   │   └── tailwind.scss   # TailwindCSS config
│   │   │   └── favicon.ico
│   │   │
│   │   ├── environments/
│   │   │   ├── environment.ts      # Development
│   │   │   └── environment.prod.ts # Production
│   │   │
│   │   └── main.ts               # App bootstrap
│   │
│   ├── angular.json               # Angular CLI config
│   ├── tsconfig.json              # TypeScript config
│   ├── tsconfig.app.json          # App TypeScript config
│   ├── tsconfig.spec.json         # Spec TypeScript config
│   ├── tailwind.config.js          # TailwindCSS config
│   ├── postcss.config.js           # PostCSS config
│   ├── package.json               # NPM dependencies
│   ├── package-lock.json          # Locked dependencies
│   ├── .gitignore                 # Frontend gitignore
│   ├── .env                       # Environment variables (GIT IGNORED)
│   ├── .env.example               # Template for .env
│   ├── karma.conf.js              # Testing config (optional)
│   └── jest.config.js             # Jest testing (optional)
│
└── .github/                       # GitHub specific files
    ├── workflows/                 # CI/CD workflows (optional)
    │   └── deploy.yml             # Auto-deploy on push
    └── ISSUE_TEMPLATE/
        └── bug_report.md
```

---

## 🔄 File Creation Checklist

### Phase 1: Backend Setup (Week 1)

**Core Entities:**
- [ ] `backend/src/Entity/User.php` - Admin user (email, password_hash, created_at)
- [ ] `backend/src/Entity/GalleryItem.php` - Gallery photo (image, keywords, featured, project_id)
- [ ] `backend/src/Entity/Project.php` - Portfolio project (title, description, category, location)
- [ ] `backend/src/Entity/Contact.php` - Lead (name, email, phone, message, status)
- [ ] `backend/src/Entity/Team.php` - Team member (name, bio, image, position)

**Repositories:**
- [ ] `backend/src/Repository/UserRepository.php`
- [ ] `backend/src/Repository/GalleryItemRepository.php`
- [ ] `backend/src/Repository/ProjectRepository.php`
- [ ] `backend/src/Repository/ContactRepository.php`
- [ ] `backend/src/Repository/TeamRepository.php`

**Services:**
- [ ] `backend/src/Service/ImageService.php` - Upload, resize, optimize
- [ ] `backend/src/Service/GalleryService.php` - Gallery business logic
- [ ] `backend/src/Service/ContactService.php` - Lead handling, email
- [ ] `backend/src/Service/EmailService.php` - SMTP via Zimbra
- [ ] `backend/src/Service/RateLimitService.php` - IP-based limiting

**Security:**
- [ ] `backend/src/Security/PasswordHasher.php` - bcrypt hashing
- [ ] `backend/src/Security/SessionManager.php` - Session handling

**Controllers:**
- [ ] `backend/src/Controller/AuthController.php` - Login/logout
- [ ] `backend/src/Controller/GalleryController.php` - Gallery CRUD
- [ ] `backend/src/Controller/ProjectController.php` - Project CRUD
- [ ] `backend/src/Controller/ContactController.php` - Lead management
- [ ] `backend/src/Controller/TeamController.php` - Team CRUD
- [ ] `backend/src/Controller/HealthController.php` - Health check

**Configuration & Migrations:**
- [ ] `backend/.env.example` - Template
- [ ] `backend/config/services.yaml` - Dependency injection
- [ ] `backend/config/routes.yaml` - API routes
- [ ] `backend/config/packages/doctrine.yaml` - Database
- [ ] `backend/config/packages/mailer.yaml` - Zimbra SMTP
- [ ] `backend/migrations/` - All 5 migrations

**Root Files:**
- [ ] `backend/.gitignore`
- [ ] `backend/composer.json` (updated with dependencies)

---

### Phase 2: Frontend Setup (Week 2)

**Routing:**
- [ ] `frontend/src/app/app.routes.ts` - Root routes
- [ ] `frontend/src/pages/gallery/gallery.routes.ts`
- [ ] `frontend/src/pages/portfolio/portfolio.routes.ts`
- [ ] `frontend/src/admin/admin.routes.ts`

**Pages (Lazy Loaded):**
- [ ] `frontend/src/pages/home/home.component.ts/html/scss`
- [ ] `frontend/src/pages/gallery/gallery-feed.component.ts/html/scss`
- [ ] `frontend/src/pages/gallery/gallery-search.component.ts/html/scss`
- [ ] `frontend/src/pages/gallery/gallery-lightbox.component.ts/html/scss`
- [ ] `frontend/src/pages/portfolio/projects-list.component.ts/html/scss`
- [ ] `frontend/src/pages/portfolio/project-detail.component.ts/html/scss`
- [ ] `frontend/src/pages/contact/contact-form.component.ts/html/scss`
- [ ] `frontend/src/pages/team/team.component.ts/html/scss`
- [ ] `frontend/src/pages/about/about.component.ts/html/scss`

**Shared Components:**
- [ ] `frontend/src/shared/components/header/header.component.ts/html/scss`
- [ ] `frontend/src/shared/components/footer/footer.component.ts/html/scss`
- [ ] `frontend/src/shared/components/hero/hero.component.ts/html/scss`
- [ ] `frontend/src/shared/components/loading-spinner/loading-spinner.component.ts/html/scss`
- [ ] `frontend/src/shared/components/error-message/error-message.component.ts/html/scss`

**Services:**
- [ ] `frontend/src/app/services/api.service.ts` - HTTP client base
- [ ] `frontend/src/app/services/gallery.service.ts`
- [ ] `frontend/src/app/services/project.service.ts`
- [ ] `frontend/src/app/services/contact.service.ts`
- [ ] `frontend/src/app/services/team.service.ts`
- [ ] `frontend/src/app/services/auth.service.ts`
- [ ] `frontend/src/app/services/i18n.service.ts`

**Security:**
- [ ] `frontend/src/shared/guards/admin.guard.ts`
- [ ] `frontend/src/shared/interceptors/http-error.interceptor.ts`
- [ ] `frontend/src/shared/interceptors/http-auth.interceptor.ts`
- [ ] `frontend/src/shared/pipes/safe.pipe.ts`

**Admin Panel:**
- [ ] `frontend/src/admin/dashboard/admin-dashboard.component.ts/html/scss`
- [ ] `frontend/src/admin/gallery-manager/gallery-list.component.ts/html/scss`
- [ ] `frontend/src/admin/gallery-manager/gallery-upload.component.ts/html/scss`
- [ ] `frontend/src/admin/gallery-manager/gallery-edit.component.ts/html/scss`
- [ ] `frontend/src/admin/project-manager/project-list.component.ts/html/scss`
- [ ] `frontend/src/admin/project-manager/project-form.component.ts/html/scss`
- [ ] `frontend/src/admin/contact-manager/contact-dashboard.component.ts/html/scss`
- [ ] `frontend/src/admin/team-manager/team-list.component.ts/html/scss`

**Models & Types:**
- [ ] `frontend/src/app/models/gallery.model.ts`
- [ ] `frontend/src/app/models/project.model.ts`
- [ ] `frontend/src/app/models/contact.model.ts`
- [ ] `frontend/src/app/models/team.model.ts`
- [ ] `frontend/src/app/models/api-response.model.ts`

**Internationalization:**
- [ ] `frontend/src/i18n/fr.json` - French translations
- [ ] `frontend/src/i18n/ar.json` - Arabic translations (RTL)
- [ ] `frontend/src/app/services/i18n.service.ts`

**Styling & Assets:**
- [ ] `frontend/src/assets/styles/global.scss`
- [ ] `frontend/src/assets/styles/variables.scss`
- [ ] `frontend/src/assets/styles/mixins.scss`
- [ ] `frontend/src/assets/images/logo.png`
- [ ] `frontend/src/assets/images/logo.svg`
- [ ] `frontend/src/tailwind.config.js`

**Configuration:**
- [ ] `frontend/.env.example`
- [ ] `frontend/angular.json` (build config)
- [ ] `frontend/tailwind.config.js` (TailwindCSS)
- [ ] `frontend/postcss.config.js`
- [ ] `frontend/tsconfig.json`

---

### Phase 3: Configuration & Root

**Backend:**
- [ ] `backend/.gitignore` - Ignore `/vendor`, `/storage`, `.env`, `*.log`
- [ ] `backend/composer.json` - Dependencies specified
- [ ] `backend/public/.htaccess` - Apache rewrite rules
- [ ] `backend/config/packages/mailer.yaml` - Zimbra SMTP

**Frontend:**
- [ ] `frontend/.gitignore` - Ignore `/node_modules`, `/dist`, `.env`
- [ ] `frontend/package.json` - Dependencies installed
- [ ] `frontend/angular.json` - Build & serve config

**Root:**
- [ ] `.gitignore` - Global rules
- [ ] `docker-compose.yml` (optional for local dev)
- [ ] `README.md` - Project overview ✅
- [ ] `docs/` - All documentation files ✅

---

## 📊 File Count Summary

| Section | Files | Status |
|---------|-------|--------|
| Backend Entities | 5 | Phase 1 |
| Backend Repositories | 5 | Phase 1 |
| Backend Services | 5 | Phase 1 |
| Backend Controllers | 6 | Phase 1 |
| Backend Config | 6 | Phase 1 |
| Backend Total | ~30 files | Phase 1 |
| Frontend Pages | 9 | Phase 2 |
| Frontend Components | 5 | Phase 2 |
| Frontend Services | 7 | Phase 2 |
| Frontend Admin | 8 | Phase 2 |
| Frontend Config | 7 | Phase 2 |
| Frontend Total | ~50 files | Phase 2 |
| Documentation | 12 | ✅ |
| **TOTAL** | **~100 files** | - |

---

## 🗂️ Storage Directories (Created at Runtime)

These folders don't exist in Git but are created when the app runs:

```
backend/storage/
├── gallery/
│   ├── 2025/
│   │   ├── pergola-01.jpg
│   │   ├── pergola-02.jpg
│   │   └── ... (all user uploads)
│   └── 2026/
│
├── thumbs/
│   ├── 2025/
│   │   ├── pergola-01.jpg (200x200)
│   │   └── ... (thumbnails)
│   └── 2026/
│
└── team/
    ├── member-01.jpg
    ├── member-02.jpg
    └── ... (team photos)
```

**.gitignore entries:**
```
backend/storage/gallery/*
backend/storage/thumbs/*
backend/storage/team/*
!backend/storage/.gitkeep
```

Use `.gitkeep` files to preserve folder structure in Git.

---

## 🔑 Key Files by Purpose

### API Routes
- `backend/config/routes.yaml` - All endpoint definitions

### Database Schema
- `backend/config/packages/doctrine.yaml` - Connection settings
- `backend/migrations/` - All schema changes

### Email Configuration
- `backend/config/packages/mailer.yaml` - Zimbra SMTP settings

### Frontend Routing
- `frontend/src/app/app.routes.ts` - Main routes
- `frontend/src/pages/gallery/gallery.routes.ts` - Gallery routes
- `frontend/src/admin/admin.routes.ts` - Protected admin routes

### Styling
- `frontend/src/assets/styles/global.scss` - All global styles
- `frontend/src/assets/styles/variables.scss` - Colors, sizes, fonts
- `frontend/tailwind.config.js` - TailwindCSS configuration

### Internationalization
- `frontend/src/i18n/fr.json` - French (primary)
- `frontend/src/i18n/ar.json` - Arabic (RTL)
- `frontend/src/app/services/i18n.service.ts` - Language switcher

### Environment
- `backend/.env` - Database, mail, app secrets (NOT in Git)
- `backend/.env.example` - Template (in Git)
- `frontend/.env` - API base URL (NOT in Git)
- `frontend/.env.example` - Template (in Git)

---

## 📝 Directory Naming Conventions

| Convention | Example | Purpose |
|-----------|---------|---------|
| **Kebab-case** | `gallery-feed`, `admin-dashboard` | Component folders & files |
| **snake_case** | `password_hash`, `created_at` | Database columns |
| **camelCase** | `galleryService`, `projectDetail` | TypeScript variables & functions |
| **PascalCase** | `GalleryService`, `ProjectDetail` | Classes & components |
| **UPPER_SNAKE** | `DB_HOST`, `API_URL` | Constants & env vars |

---

## 🚀 Creation Order (Recommended)

**Week 1 (Backend):**
1. Create all Entity files
2. Run migrations (creates tables)
3. Create Repository files
4. Create Service files
5. Create Controller files

**Week 2 (Frontend):**
1. Set up routing
2. Create page components
3. Create shared components
4. Create services
5. Create admin panel

**Week 3 (Polish):**
1. Connect frontend to backend
2. Test all flows
3. Optimize images
4. Deploy to OVHcloud

---

## ✅ File Structure Verification Checklist

Before deployment, verify:

- [ ] All 5 entities exist and have migrations
- [ ] All 5 repositories exist and extend base repository
- [ ] All 5 services are properly dependency-injected
- [ ] All 6 controllers have correct route mappings
- [ ] All 9 pages are lazy-loaded in routing
- [ ] All shared components are imported correctly
- [ ] `.env` files are `.gitignored`
- [ ] `.env.example` files exist with dummy values
- [ ] `storage/` directories have `.gitkeep` files
- [ ] `node_modules/` and `/vendor` are `.gitignored`
- [ ] `dist/` and `build/` are `.gitignored`

---

**Document Version:** 1.0  
**Last Updated:** June 2026  
**Status:** Complete
