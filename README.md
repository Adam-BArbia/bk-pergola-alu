# BK Pergola et Alu - Professional Website

Professional catalog and lead generation website for **BK pergola et alu** — specializing in aluminum pergola and metal structure construction for residential and commercial clients.

## 🎯 Project Overview

**Tech Stack:**
- **Frontend:** Angular 17.x (standalone components) + TailwindCSS
- **Backend:** Symfony 6.x REST API
- **Database:** MySQL 8.0
- **Hosting:** OVHcloud Startup (Tunisia)
- **Domain:** bk-pergola.tn (SSL via Let's Encrypt)
- **Email:** Zimbra SMTP (included with hosting)
- **Video:** OVHcloud Video Center (free)

**Core Features:**
- 🎨 Responsive portfolio gallery with image optimization
- 📞 Multi-channel lead capture (Email, WhatsApp, Phone)
- 👥 Team member profiles
- 🛠️ Admin panel (project CRUD, gallery management, lead tracking)
- 🌍 i18n infrastructure (French + Arabic ready)
- 📊 SEO optimized (meta tags, schema.org, sitemap, robots.txt)
- ⚡ Performance optimized (lazy loading, image optimization, caching)
- 📱 Mobile-first responsive design

## 📊 Project Stats

- **Launch Timeline:** 2-3 weeks
- **Expected Monthly Visitors:** ~3,000
- **Pages:** 6 main (Home, Portfolio, Services, Team, Contact, About)
- **Admin Features:** Project/Gallery CRUD, multi-channel lead dashboard, contact management
- **Authentication:** Multi-admin support (password-protected, bcrypt-hashed, session-based)
- **Priority Device:** Mobile phone → desktop

## 📚 Documentation

| Document | Purpose |
|----------|---------|
| [ARCHITECTURE.md](./docs/ARCHITECTURE.md) | System design, deployment model, tech decisions |
| [DATABASE.md](./docs/DATABASE.md) | Complete schema, relationships, migrations |
| [API_SPEC.md](./docs/API_SPEC.md) | All REST endpoints, request/response examples |
| [PROJECT_STRUCTURE.md](./docs/PROJECT_STRUCTURE.md) | Directory tree and file manifest |
| [SETUP.md](./docs/SETUP.md) | Local development environment |
| [DEPLOYMENT.md](./docs/DEPLOYMENT.md) | OVHcloud deployment guide |
| [SECURITY.md](./docs/SECURITY.md) | Authentication, CORS, validation, secrets |
| [DEVELOPMENT_PLAN.md](./docs/DEVELOPMENT_PLAN.md) | Week-by-week sprint schedule |
| [ADMIN_PANEL.md](./docs/ADMIN_PANEL.md) | Admin features and mockup |
| [I18N_SETUP.md](./docs/I18N_SETUP.md) | Internationalization (FR/AR) infrastructure |

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- PHP 8.1+
- MySQL 8.0
- Git
- Composer

### Local Development

See [SETUP.md](./docs/SETUP.md) for detailed instructions.

### Deploy to OVHcloud

See [DEPLOYMENT.md](./docs/DEPLOYMENT.md) for step-by-step instructions.

## 🎨 Key Design Decisions

### Frontend: Angular 17 Standalone + TailwindCSS
- **Why Standalone?** Simplified, modern Angular (no NgModule boilerplate)
- **Why TailwindCSS?** Utility-first, custom look (avoids "Bootstrap template" feel)
- **Why not NgRx?** Services + RxJS Observables scale fine for this volume

### Backend: Symfony REST API
- **Why REST, not GraphQL?** Simpler, stateless, easier to test and secure
- **Why Symfony?** Familiar (you've used it), built-in auth, validation, error handling

### Database: MySQL with Doctrine ORM
- **Why ORM?** Type-safe, migrations, relationship management
- **Why MySQL?** Included with OVHcloud Startup, reliable, proven

### Email & Lead Capture
- **SMTP:** Direct from OVHcloud Zimbra (free, already paid for)
- **WhatsApp:** Simple `wa.me/` button (free, immediate)
- **Multi-channel Admin Dashboard:** All leads in one inbox, tagged by source

### Authentication
- **Sessions, not JWT:** Simple, secure for multi-admin, no token complexity
- **Passwords:** bcrypt-hashed, stored in database
- **Timeout:** 8 hours of inactivity
- **Future-ready:** Easy to add more admins through DB

### i18n (Internationalization)
- **Day 1:** French only, full infrastructure ready
- **Later:** Add Arabic with RTL layout (dictionary files added when needed)
- **Storage:** Static JSON translation files (no DB overhead)

## 🔐 Security Highlights

- ✅ Passwords hashed with bcrypt
- ✅ CSRF protection on forms
- ✅ Input validation & sanitization
- ✅ CORS configured (same domain)
- ✅ Rate limiting on contact form
- ✅ Admin session timeout (8 hours)
- ✅ Secrets in .env (not committed)
- ✅ SQL injection prevented (Doctrine ORM)

## 📊 Admin Panel Features

- **Project Management:** Create, edit, delete projects with full CRUD
- **Gallery Management:** Upload/manage project images with optimized storage
- **Lead Dashboard:** View all contacts (email + WhatsApp) with status tracking
- **Contact Status:** Mark leads as "New", "Contacted", "Converted"
- **CSV Export:** Download leads for CRM integration
- **Team Management:** Manage team member profiles
- **Settings:** Email templates, contact preferences

## 📈 Performance Targets

- **Page Load Time:** <2s ideal, <3s acceptable
- **Lighthouse Score:** 90+
- **Mobile-first:** Optimized for phones, scalable to desktop
- **Image Optimization:** Lazy loading, responsive sizing
- **Caching:** Browser cache + HTTP headers

## 🎓 Development Approach

This is your first production project. Documentation includes:
- ✅ Detailed explanations (not just code)
- ✅ Week-by-week sprint plan (realistic timeline)
- ✅ Learning resources for new concepts (RxJS, Tailwind, etc.)
- ✅ Common pitfalls and solutions
- ✅ Testing & debugging strategies

## 📞 Support

All documentation is self-contained. See specific doc for help with:
- Setup issues → [SETUP.md](./docs/SETUP.md)
- Deployment problems → [DEPLOYMENT.md](./docs/DEPLOYMENT.md)
- API questions → [API_SPEC.md](./docs/API_SPEC.md)
- Admin panel → [ADMIN_PANEL.md](./docs/ADMIN_PANEL.md)

---

**Project Owner:** BK Pergola et Alu  
**Repository:** https://github.com/Adam-BArbia/bk-pergola-alu  
**Domain:** bk-pergola.tn  
**Launch Target:** 2-3 weeks
