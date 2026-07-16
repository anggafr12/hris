# HRIS — Human Resource Information System

A full-featured, multi-company **Human Resource Information System (HRIS)** built with **Laravel 10**. The platform centralizes the entire employee lifecycle — from recruitment and onboarding through attendance, payroll, performance, and offboarding — behind a role-based, multi-tenant web application.

<p align="left">
  <img alt="PHP" src="https://img.shields.io/badge/PHP-8.1+-777BB4?logo=php&logoColor=white">
  <img alt="Laravel" src="https://img.shields.io/badge/Laravel-10.x-FF2D20?logo=laravel&logoColor=white">
  <img alt="MySQL" src="https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white">
  <img alt="Bootstrap" src="https://img.shields.io/badge/Bootstrap-4-7952B3?logo=bootstrap&logoColor=white">
  <img alt="Alpine.js" src="https://img.shields.io/badge/Alpine.js-3-8BC0D0?logo=alpinedotjs&logoColor=white">
</p>

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [User Roles](#user-roles)

---

## Overview

HRIS is a **server-rendered, monolithic web application** organized around a clean separation between a Laravel backend (business logic, persistence, authorization, integrations) and a Blade-based frontend (a responsive admin dashboard). It is designed as a **multi-tenant SaaS**: a Super Admin operates the platform and sells subscription plans to Companies, and each Company manages its own employees, data, and HR workflows in isolation.

The codebase spans **100+ Eloquent models** and **110+ controllers**, covering more than 30 distinct HR domains.

---

## Key Features

### Core HR
- **Employee management** — profiles, documents, departments, designations, and branches
- **Attendance** — manual and **biometric** clock-in/out, with bulk import
- **Timesheets & overtime** tracking
- **Leave management** — configurable leave types, approval workflow, and leave calendar
- **Holidays** and company-wide event scheduling

### Payroll & Finance
- **Salary setup**, payslip generation, and payslip types
- **Allowances, commissions, loans, and deductions**
- **Accounting** — chart of accounts, deposits, expenses, transfers, payees/payers, and income vs. expense reports

### Recruitment (ATS)
- **Job postings** and a public careers board
- **Applicant tracking** — pipeline stages, candidate notes, and interview scheduling
- **Offer letters** and **joining letters** with templating

### Talent & Performance
- **Appraisals**, goal tracking, indicators, and competencies
- **Training** management with trainers and training types
- **Awards, promotions, transfers, resignations, terminations, warnings, and complaints**

### Platform & Collaboration
- **Role-Based Access Control (RBAC)** with granular permissions
- **Real-time messaging** and notifications
- **Support ticketing**, contracts, company policies, and announcements
- **Zoom meeting** integration and Google Calendar sync
- **AI-assisted content generation** (OpenAI) for templates and descriptions
- **Multi-language** UI (i18n) including English and Indonesian
- **Subscription plans**, coupons, referral program, and a built-in landing-page builder

---

## Tech Stack

### Backend
| Concern | Technology |
|---|---|
| Language / Framework | **PHP 8.1**, **Laravel 10** |
| Database | **MySQL** via Eloquent ORM & schema migrations |
| Authentication | Laravel Breeze (web) + **Laravel Sanctum** (API tokens) |
| Authorization | **spatie/laravel-permission** (roles & permissions) |
| Modular architecture | **nwidart/laravel-modules** |
| Data import/export | **maatwebsite/excel** |
| Real-time chat | **munafio/chatify** + Pusher |
| AI integration | **orhanerday/open-ai** |
| Other | milon/barcode, spatie/laravel-google-calendar, laravelcollective/html, doctrine/dbal |

### Frontend
| Concern | Technology |
|---|---|
| Templating | **Blade** (server-side rendering) |
| UI framework | **Bootstrap** admin theme + custom SCSS/CSS |
| Interactivity | **Alpine.js**, jQuery ecosystem |
| Build tooling | **Laravel Mix** (webpack), **Tailwind CSS**, PostCSS |
| Charts & viz | ApexCharts |
| UX components | Select2, SweetAlert2, FullCalendar, Flatpickr, Dropzone, Quill/Summernote editors, Swiper |
| Real-time client | Pusher JS, cookie-consent |

### Integrations
- **40+ payment gateways** — Stripe, PayPal, Razorpay, Midtrans, Xendit, Flutterwave, Paystack, Mollie, Mercado Pago, PayTM, and more
- **Email** — SMTP, Mailgun, Postmark, Amazon SES
- **Video/Calendar** — Zoom, Google Calendar

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                          Client (Browser)                      │
│     Blade views · Bootstrap · Alpine.js · ApexCharts           │
│                     Pusher (real-time)                         │
└───────────────────────────────┬──────────────────────────────┘
                                 │ HTTP / WebSocket
┌───────────────────────────────▼──────────────────────────────┐
│                        Laravel 10 Application                  │
│                                                                │
│  Routing (web.php · api.php · auth.php · channels.php)          │
│         │                                                      │
│  Middleware  ──►  Auth (Sanctum) · RBAC · XSS · CSRF · Tenant  │
│         │                                                      │
│  Controllers (110+)  ──►  Services / Utility · Mail · Jobs      │
│         │                                                      │
│  Eloquent Models (100+)  ──►  Policies & Permissions           │
│         │                                                      │
│  Modules (nwidart)  ──►  e.g. LandingPage                      │
└───────────────────────────────┬──────────────────────────────┘
                                 │
┌───────────────────────────────▼──────────────────────────────┐
│        MySQL   ·   File Storage   ·   3rd-party APIs           │
│   (payments · email · Zoom · Google Calendar · OpenAI)        │
└──────────────────────────────────────────────────────────────┘
```

**Design highlights**

- **Multi-tenancy** — a single application instance serves many companies; data is scoped per company, with a Super Admin overseeing plans and billing.
- **RBAC everywhere** — every action is guarded by `spatie/laravel-permission`, so menus, routes, and operations adapt to the signed-in user's role.
- **Modular by design** — feature packages (e.g. `LandingPage`) live under `Modules/` and can be toggled independently via `modules_statuses.json`.
- **Convention-driven MVC** — thin controllers delegate to Eloquent models and a shared `Utility` layer for cross-cutting settings (branding, email templates, currency, etc.).

---

## Project Structure

```
├── app/
│   ├── Http/
│   │   ├── Controllers/     # 110+ controllers (HR domains + payment gateways)
│   │   └── Middleware/      # Auth, RBAC, XSS, tenant/plan guards
│   └── Models/              # 100+ Eloquent models
├── Modules/
│   └── LandingPage/         # Self-contained feature module (nwidart)
├── config/                  # Framework & integration configuration
├── database/
│   ├── migrations/          # Schema definitions
│   └── seeders/             # Default roles, permissions, demo data
├── resources/
│   ├── views/               # Blade templates (dashboard + 60+ feature areas)
│   └── lang/                # i18n JSON dictionaries (en, id, ...)
├── routes/
│   ├── web.php              # Main application routes (~1.7k lines)
│   ├── api.php              # Sanctum-protected API
│   ├── auth.php             # Authentication routes
│   └── channels.php         # Broadcast channels
├── public/                  # Front controller, compiled assets, JS libs
└── tailwind.config.js · webpack.mix.js
```

---

## Getting Started

### Requirements
- PHP **8.1+** with extensions: `bcmath`, `curl`, `dom`, `fileinfo`, `gd`, `imagick`, `mbstring`, `mysqlnd`, `pdo`, `zip`
- **Composer 2**
- **Node.js 16+** and npm
- **MySQL 8** (or MariaDB)

### Installation

```bash
# 1. Clone
git clone https://github.com/anggafr12/hris.git
cd hris

# 2. Install PHP dependencies
composer install

# 3. Install & build front-end assets
npm install
npm run dev          # or: npm run prod  (production build)

# 4. Environment
cp .env.example .env
php artisan key:generate

# 5. Configure the database in .env, then run migrations + seeders
php artisan migrate --seed

# 6. Link storage & serve
php artisan storage:link
php artisan serve
```

Then open **http://localhost:8000**.

> A web-based installer is also available (`rachidlaasri/laravel-installer`) at `/install` for a guided setup on shared hosting.

---

## Configuration

Set the following in your `.env` (see `.env.example` for the full list):

```dotenv
APP_NAME='HRIS'
APP_URL=http://localhost

DB_DATABASE=hris
DB_USERNAME=root
DB_PASSWORD=

# Mail (SMTP)
MAIL_HOST=
MAIL_USERNAME=
MAIL_PASSWORD=

# Real-time (Pusher)
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
```

Payment gateways, Zoom, Google Calendar, and OpenAI credentials are configured from the in-app **Settings** area or via `config/services.php`.

> **Security:** `.env`, database dumps, logs, and user uploads are intentionally excluded from version control via `.gitignore`. Never commit real credentials — rotate any key that has been exposed.

---

## User Roles

| Role | Capabilities |
|---|---|
| **Super Admin** | Operates the platform: subscription plans, coupons, global settings, and tenant companies |
| **Company** | Manages its own employees, payroll, recruitment, and HR configuration |
| **Employee** | Self-service portal: attendance, leave requests, payslips, documents, and profile |

Permissions are fully customizable per role through the RBAC settings.
