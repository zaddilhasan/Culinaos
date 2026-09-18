# Culinaos — Web Platform Final Handover Build

Industry-oriented web platform foundation for multi-branch restaurant groups. This package is the **enterprise SaaS owner deployment** for one restaurant group: 20 branches, one owner, one branch manager per branch, and up to 5 employees per branch. IoT hardware and ERP integrations are intentionally excluded. A companion Flutter employee app source package is provided separately.

## Run

Install Docker Desktop on Windows. From this folder in PowerShell:

```powershell
docker compose up --build
```

Open:

- Web app: http://localhost:8000/
- Swagger: http://localhost:8000/api/docs/
- OpenAPI: http://localhost:8000/api/schema/
- Admin: http://localhost:8000/admin/
- Health: http://localhost:8000/api/v1/health/

For a clean demo reset:

```powershell
docker compose down -v --remove-orphans
docker compose up --build
```

## Default SaaS accounts

Password for all seeded accounts: `DemoPass123!`

### Owner
`owner@culinaos.local`

### Branch managers
`manager.branch01@culinaos.local` through `manager.branch20@culinaos.local`

### Employees
`employee.branch01.01@culinaos.local` through 5 employees per branch, for branches 01–20.

**Default password:** `ChangeMe123!` — change it immediately after deployment.

## SaaS-specific controls
- Owner dashboard can see all 20 branches.
- Branch manager access is restricted to the manager's branch.
- Employee app/API access is password + token based.
- Owner/manager/employee password changes are supported; the owner can reset a manager/employee password when staff changes.
- Branch names are editable by the owner or that branch's manager.
- Inventory records include entry date, quantity in stock, expiry date and configurable alert lead time from 3–10 days.
- Employees can set their branch expiry alert lead time; expiry notifications appear for the relevant employee and also in manager/owner notifications.
- A day-before-expiry notification is always sent to the relevant employee plus manager/owner.
- The background inventory worker runs the expiry sweep automatically.

## Steps 1–7 scope

1. **SaaS tenancy:** organizations, branches, users, memberships, roles and branch/organization data scope.
2. **Food safety core:** tasks, temperature records, server-side limits, PASS/FAIL, issue and corrective-action workflow, audit events.
3. **Operations:** suppliers, inventory/batches, expiry status, waste records, equipment and maintenance, cleaning and pest control.
4. **Compliance:** HACCP, staff training, allergen controls, evidence metadata and traceability.
5. **Automation:** recurring task schedules, escalation rules and in-app notifications.
6. **Management intelligence:** command center dashboard, branch-aware KPIs, analytics endpoint and CSV temperature report.
7. **API/security/verification:** token authentication for future mobile use, scoped API access, pagination/filtering, OpenAPI/Swagger, automated tests and Docker verification scripts.

## Live client workflow

1. Sign in as a branch employee or manager.
2. Open Command Center.
3. Run the failure scenario or record a temperature.
4. Submit 8.5 °C for the walk-in refrigerator.
5. Server evaluates the configured 0–5 °C range.
6. A failed reading creates a high-severity issue.
7. A corrective action is created.
8. Audit activity records the transition.
9. Owner/manager notifications are created for the branch.
10. Show inventory, maintenance, training, allergens, cleaning, pest, HACCP and branch views.

## Verification

Windows:

```powershell
.\scripts\verify.ps1
```

The script runs Django checks, migration checks, automated tests, health/API/schema checks and the login page check.

## Architecture

- Django 5.2
- Django REST Framework
- PostgreSQL 16
- DRF Spectacular / OpenAPI
- Docker Compose
- Server-rendered web operations console
- REST API contract designed for the future Flutter employee application

## Deployment boundary

This is a client handover/demo build and development foundation, not a production deployment. Before live restaurant operations: use managed secrets, HTTPS, production WSGI/ASGI, object storage, background workers, monitoring, backups, CI/CD, rate limiting and security review.

## Deployment: Render + Supabase

This package is prepared for deployment of the Django web/API service on Render with PostgreSQL hosted by Supabase.

### Render
- Web Service runtime: Python
- Build command: `./build.sh`
- Start command: `python -m gunicorn config.wsgi:application --bind 0.0.0.0:$PORT`
- `render.yaml` is included as a starting point.
- Secrets such as `DATABASE_URL` and `DJANGO_SECRET_KEY` must be configured in Render and are not committed to GitHub.

### Supabase
Use the Supabase PostgreSQL connection string as the Render `DATABASE_URL`.

### Expiry automation
The command `python manage.py run_inventory_alerts` performs one expiry sweep. A Render Cron Job can be added after the web service is working; do not run the continuous Docker inventory worker in the Render Web Service.

The Docker Compose stack remains for local development.
