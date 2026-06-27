# TenantStack — Multi-Tenant B2B SaaS Monorepo (Django + Next.js)

[![CI](https://github.com/wataee/tenant-stack/actions/workflows/ci.yml/badge.svg)](https://github.com/wataee/tenant-stack/actions/workflows/ci.yml)
[![Backend](https://img.shields.io/badge/Backend-Django%205.1%20%2B%20DRF-0C4B33.svg)](https://www.djangoproject.com/)
[![Frontend](https://img.shields.io/badge/Frontend-Next.js%2015%20%2B%20React%2019-000000.svg)](https://nextjs.org/)
[![Database](https://img.shields.io/badge/Database-PostgreSQL%2016-336791.svg)](https://www.postgresql.org/)
[![Queue](https://img.shields.io/badge/Queue-Celery%20%2B%20Redis-DC382D.svg)](https://docs.celeryq.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A production-grade, multi-tenant B2B SaaS monorepo built with **Django 5.1 (DRF)** and **Next.js 15 (App Router)**. Features defense-in-depth tenant isolation via PostgreSQL Row-Level Security (RLS), granular Role-Based Access Control (RBAC), audit logging, and asynchronous background worker pipelines.

---

## Architectural Highlights

* **Tenant Isolation with PostgreSQL RLS**: Every SQL query is constrained by tenant context injected into PostgreSQL session variables (`SET LOCAL app.current_tenant_id = :id`), preventing cross-tenant data leaks at the database engine level.
* **Granular RBAC**: Role-based permissions matrix (Owner, Admin, Member, Viewer) with custom organization policy enforcement.
* **Audit Trail**: Immutable logging of administrative actions, permission mutations, and data access.
* **Celery + Redis Background Workers**: Asynchronous task queues for email delivery, report generation, and third-party webhook handling.
* **Modern Next.js 15 App Router**: Server components, optimistic state updates, and clean modular layout.

---

## Monorepo Layout

```
tenant-stack/
├── baseline_backend/        # Django 5.1 / Django REST Framework
│   ├── apps/
│   │   ├── accounts/        # Custom user model & authentication
│   │   ├── organizations/   # Multi-tenancy & membership management
│   │   ├── permissions/     # RBAC policies & permission matrices
│   │   ├── audit/           # Immutable audit logging service
│   │   ├── crm/             # Customer relationship & deal pipelines
│   │   ├── invoicing/       # Invoicing & billing tracking
│   │   └── tasks/           # Celery background workers
│   ├── core/                # Tenant middleware & RLS connection decorators
│   └── manage.py
├── baseline/                # Next.js 15 Frontend
│   ├── src/
│   │   ├── app/             # Next.js App Router (dashboard, auth, tenants)
│   │   ├── components/      # UI components (Tailwind CSS)
│   │   └── lib/             # API clients & session handling
├── docker-compose.yml       # Full-stack local development environment
└── docs/                    # Architecture and operational runbooks
```

---

## Quickstart

### 1. Start full stack with Docker Compose
```bash
git clone https://github.com/wataee/tenant-stack.git
cd tenant-stack

docker compose up -d
```
* **Frontend**: `http://localhost:3000`
* **API Backend**: `http://localhost:8000`
* **Django Admin**: `http://localhost:8000/admin`

### 2. Run Database Migrations & Initial Tenant Seed
```bash
docker compose exec backend python manage.py migrate
docker compose exec backend python manage.py seed_tenant_demo
```

---

## Tenant Isolation Mechanism

The backend implements defense-in-depth tenant security:
1. **Middleware Layer**: Parses request hostname / headers, resolves tenant organization, and validates caller membership.
2. **ORM Layer**: Global querysets filtered by tenant ID automatically.
3. **Database Layer (RLS)**: Enforces row-level security policies directly in PostgreSQL:

```sql
ALTER TABLE organization_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON organization_data
    AS RESTRICTIVE
    USING (organization_id = NULLIF(current_setting('app.current_tenant_id', true), '')::uuid);
```

---

## Testing

```bash
# Backend test suite
cd baseline_backend
pytest apps/ -v

# Frontend linting
cd ../baseline
npm run lint
```

---

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for details.
