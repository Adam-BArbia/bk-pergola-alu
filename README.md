# BK Pergola et Alu - Professional Catalog Website

A production-ready website for **BK pergola et alu**, showcasing aluminum pergolas and metal structures with a focus on lead generation and portfolio showcase.

## 🎯 Project Overview

- **Company**: BK pergola et alu (Tunisia-based)
- **Domain**: bk-pergola.tn
- **Hosting**: OVHcloud Startup
- **Timeline**: 2-3 weeks
- **Tech Stack**: Angular 17 (Frontend) + Symfony 6 (Backend) + MySQL

## 📋 Key Features

### Lead Generation
- Contact form (email, phone, message)
- WhatsApp integration (simple link)
- Admin dashboard for lead management
- Email notifications via Zimbra SMTP

### Portfolio Showcase
- Dynamic project gallery
- Project categories and filtering
- Responsive image display
- Admin panel for project management

### Company Info
- Homepage with hero section
- Services overview
- Team profiles
- SEO optimization

### Internationalization
- French (primary)
- Arabic infrastructure (translations ready)
- RTL support for Arabic

## 📁 Project Structure

```
bk-pergola-alu/
├── docs/                          # Complete documentation
│   ├── ARCHITECTURE.md            # System design & flow
│   ├── DATABASE.md                # Schema & relationships
│   ├── API_SPEC.md                # All endpoints
│   ├── PROJECT_STRUCTURE.md       # File structure
│   ├── SETUP_GUIDE.md             # Local development
│   ├── DEPLOYMENT.md              # OVHcloud deployment
│   ├── SECURITY.md                # Security checklist
│   ├── DEVELOPMENT_PLAN.md        # Week-by-week timeline
│   ├── ADMIN_PANEL.md             # Admin features
│   ├── I18N_SETUP.md              # Internationalization
│   └── TAILWIND_GUIDE.md          # CSS framework guide
├── backend/                       # Symfony API
│   ├── src/
│   ├── config/
│   ├── migrations/
│   ├── public/
│   ├── .env.example
│   ├── composer.json
│   └── symfony.lock
├── frontend/                      # Angular App
│   ├── src/
│   ├── angular.json
│   ├── package.json
│   └── tsconfig.json
├── docker-compose.yml             # Local dev environment (optional)
└── README.md                       # This file
```

## 🚀 Quick Start

### Prerequisites
- PHP 8.1+
- Node.js 18+
- MySQL 8.0+
- Composer
- Angular CLI

### Local Development Setup

1. **Clone repository**
```bash
git clone https://github.com/Adam-BArbia/bk-pergola-alu.git
cd bk-pergola-alu
```

2. **Backend setup** (see `docs/SETUP_GUIDE.md`)
```bash
cd backend
composer install
cp .env.example .env
# Configure database in .env
php bin/console doctrine:migrations:migrate
php bin/console server:run
```

3. **Frontend setup** (see `docs/SETUP_GUIDE.md`)
```bash
cd ../frontend
npm install
ng serve
```

4. Open `http://localhost:4200`

## 📚 Documentation

All detailed documentation is in the `docs/` folder:

| Document | Purpose |
|----------|---------|
| **ARCHITECTURE.md** | System design, component flow, tech decisions |
| **DATABASE.md** | Complete MySQL schema with relationships |
| **API_SPEC.md** | All backend endpoints with examples |
| **PROJECT_STRUCTURE.md** | Every file and folder to create |
| **SETUP_GUIDE.md** | Local dev environment setup |
| **DEPLOYMENT.md** | Step-by-step OVHcloud deployment |
| **SECURITY.md** | Security checklist and best practices |
| **DEVELOPMENT_PLAN.md** | Week-by-week sprint plan |
| **ADMIN_PANEL.md** | Admin features and design |
| **I18N_SETUP.md** | French/Arabic setup |
| **TAILWIND_GUIDE.md** | CSS framework tutorial |

## 🔄 Development Workflow

1. **Read `ARCHITECTURE.md`** - Understand the system
2. **Read `DEVELOPMENT_PLAN.md`** - Know what to build each week
3. **Follow `SETUP_GUIDE.md`** - Set up local environment
4. **Reference `API_SPEC.md`** - Build backend endpoints
5. **Reference `PROJECT_STRUCTURE.md`** - Create files
6. **Follow `DEPLOYMENT.md`** - Deploy to OVHcloud

## 🛠️ Tech Stack Details

### Frontend
- **Angular 17** with standalone components
- **Tailwind CSS** for styling
- **RxJS** for reactive programming
- **Angular i18n** for French/Arabic

### Backend
- **Symfony 6** with API Platform
- **Doctrine ORM** for database
- **MySQL** for data storage
- **Symfony Mailer** for email notifications

### Hosting
- **OVHcloud Startup** PHP hosting
- **Zimbra SMTP** for email (built-in)
- **Video Center Free** for videos
- **Let's Encrypt SSL** (free)

## 📊 Key Metrics

- **Target users**: ~3,000/month
- **Page load**: <2-3 seconds
- **Mobile-first design**: Phone → Tablet → Desktop
- **SEO**: Optimized with schema.org

## 🔐 Security Features

- ✅ Password hashing (bcrypt)
- ✅ Session-based authentication
- ✅ CSRF protection
- ✅ Input validation & sanitization
- ✅ SQL injection prevention (Doctrine ORM)
- ✅ CORS configuration
- ✅ Rate limiting on forms

## 📞 Contact & Support

For questions about this project structure, refer to the documentation files. For company information:
- **Email**: contact@bk-pergola.tn
- **WhatsApp**: [Link on website]
- **Website**: bk-pergola.tn

## 📄 License

This project is for BK pergola et alu. All rights reserved.

---

**Ready to build?** Start with `docs/ARCHITECTURE.md` → `docs/DEVELOPMENT_PLAN.md` → `docs/SETUP_GUIDE.md`
