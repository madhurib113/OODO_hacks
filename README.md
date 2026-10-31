# Brightloop Subscription Management System

A web app for running subscription billing: products and plans, quotations, subscriptions, automatic recurring invoices, discounts, taxes, payments and a finance dashboard. Admins, staff and customers each get their own view.

## Features

- Sign-up with unique email and password rules, plus password reset by email
- Three roles: Admin, Staff and Customer (customers only see their own data, only admin can add staff)
- Products, variants, plans and add-ons
- Quotation templates and quotations that convert into subscriptions
- Subscription lifecycle: Draft, Quotation, Confirmed, Active, Closed (with pause, resume and renew)
- Bulk invoice generation with automatic tax, and a log of every billing run
- Discounts (fixed or percent, with limits and dates). Only the single largest saving is applied to an invoice
- Payments, including partial ones, with paid / outstanding / overdue views
- Finance dashboard with date and customer filters

Billing is safe to re-run: a database constraint on (subscription, period start) means an invoice is never created twice, and missed periods are caught up on the next run.

## Tech stack

- **Backend:** Django, Django REST Framework, JWT auth
- **Frontend:** React (Vite), Tailwind CSS, daisyUI, TanStack Query, Recharts
- **Database:** SQLite for local dev, PostgreSQL with Docker
- **API docs:** Swagger (drf-spectacular)
- **Email:** Resend, falls back to console output without a keys
- **Other:** Docker Compose

## Project structure

```
brightloop/
├── backend/
│   ├── config/        settings and root URLs
│   └── core/          models, views, serializers, permissions,
│                      billing logic (services.py), tests, management commands
├── frontend/
│   └── src/
│       ├── pages/       one file per screen
│       ├── components/  shared UI pieces
│       ├── api.js       API client
│       └── auth.jsx     login state
├── docs/              R&D notes and demo script
└── docker-compose.yml
```

## Running it

**With Docker**

```bash
docker compose up --build
```

App at http://localhost:5173, API docs at http://localhost:8000/api/docs/. The first start seeds 5,000 demo subscriptions, so it takes a minute.

**Without Docker**

```bash
# backend
cd backend
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
python manage.py seed_demo --count 5000 --bill
python manage.py runserver

# frontend (new terminal)
cd frontend
cp .env.example .env
npm install && npm run dev
```

**Tests:** `cd backend && python manage.py test`

**Scheduled billing:** `python manage.py run_billing`

## Demo logins

| Role | Email | Password |
|---|---|---|
| Admin | admin@brightloop.test | Admin@12345 |
| Staff | staff@brightloop.test | Staff@12345 |
| Customer | customer@brightloop.test | Customer@123 |

## Notes

- Money uses `Decimal` and is rounded to 2 places per line. Discounts come off before tax.
- The free-tier deployment setup (Neon + Render + Vercel) and the reasoning behind our choices are in `docs/RND.md`.
