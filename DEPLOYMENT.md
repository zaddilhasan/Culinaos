# Culinaos — Render + Supabase deployment

## 1. GitHub
Upload the contents of this package to the private `culinaos` GitHub repository.

## 2. Supabase
Create the Culinaos Supabase project and obtain its PostgreSQL connection string.

## 3. Render Web Service
In Render:
- New -> Web Service
- Connect the private GitHub repository
- Runtime: Python
- Build command: `./build.sh`
- Start command: `python -m gunicorn config.wsgi:application --bind 0.0.0.0:$PORT`

Set:
- `DATABASE_URL` = Supabase PostgreSQL connection string
- `DJANGO_SECRET_KEY` = strong generated secret
- `DJANGO_ENV` = production
- `DJANGO_DEBUG` = 0
- `DJANGO_ALLOWED_HOSTS` = `.onrender.com`
- `CSRF_TRUSTED_ORIGINS` = your HTTPS Render service URL
- `TIME_ZONE` = `Asia/Dhaka`
- `CRON_SECRET` = strong generated secret

The included `render.yaml` can also be used as a starting point for a Blueprint deployment.

## 4. Verify
After deployment, test:
- `/`
- `/admin/`
- `/api/v1/health/`
- `/api/docs/`

Then run the inventory alert command once from an appropriate environment:
```bash
python manage.py run_inventory_alerts
```

## 5. Expiry automation
After the web service is working, create a Render Cron Job that runs:
```bash
python manage.py run_inventory_alerts
```
The command performs one sweep and exits. Do not use the Docker Compose continuous worker on the Render Web Service.

## 6. Production notes
This package is Render-ready, but before real restaurant operations you should still configure backups, monitoring, object storage for uploaded evidence/media, rate limiting, and a security review. Do not commit production secrets to GitHub.
