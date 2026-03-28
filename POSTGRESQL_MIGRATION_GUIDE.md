# OpenAlgo — PostgreSQL Migration Reference
## Files Changed & What to Update When IP Changes

---

## Your Current PostgreSQL Config

| Field       | Value                    |
|-------------|--------------------------|
| Host        | `10.160.0.2`             |
| Port        | `5532`                   |
| Database    | `algodb`                 |
| Username    | `algo_user`              |
| Password    | `algo@369`               |
| Full URL    | `postgresql://algo_user:algo%40369@10.160.0.2:5532/algodb` |

> **Note:** The `@` in the password is URL-encoded as `%40`

---

## IF YOUR INSTANCE IP CHANGES — Edit Only 1 File

### `.env` (or `.sample.env` before first deploy)

**Location:** `openalgo/.env`

**Find these 5 lines and replace `10.160.0.2` with your NEW IP:**

```env
DATABASE_URL = 'postgresql://algo_user:algo%40369@NEW_IP_HERE:5532/algodb'
LATENCY_DATABASE_URL = 'postgresql://algo_user:algo%40369@NEW_IP_HERE:5532/algodb'
LOGS_DATABASE_URL = 'postgresql://algo_user:algo%40369@NEW_IP_HERE:5532/algodb'
SANDBOX_DATABASE_URL = 'postgresql://algo_user:algo%40369@NEW_IP_HERE:5532/algodb'
HEALTH_DATABASE_URL = 'postgresql://algo_user:algo%40369@NEW_IP_HERE:5532/algodb'
```

**Quick sed command to replace IP in one shot:**
```bash
# Replace old IP with new IP in .env
sed -i 's/10.160.0.2/NEW_IP_HERE/g' /path/to/openalgo/.env

# Restart the service
sudo systemctl restart openalgo-*
```

That's it! Only `.env` needs to change. All database files read from this env.

---

## All Files Modified (for reference)

| # | File | What Changed |
|---|------|-------------|
| 1 | `.sample.env` | 5 DATABASE URLs changed from `sqlite:///` → `postgresql://` |
| 2 | `requirements.txt` | Added `psycopg2-binary==2.9.10` |
| 3 | `requirements-nginx.txt` | Added `psycopg2-binary==2.9.10` |
| 4 | `pyproject.toml` | Added `psycopg2-binary==2.9.10` to dependencies |
| 5 | `database/master_contract_status_db.py` | Fixed top-level `os.makedirs()` crash with PostgreSQL URL |
| 6 | `database/health_db.py` | Guarded `init_health_db()` SQLite-only path creation |
| 7 | `database/latency_db.py` | Guarded `init_latency_db()` SQLite-only path creation |
| 8 | `database/traffic_db.py` | Guarded `init_logs_db()` SQLite-only path creation |

**NOT changed (already PostgreSQL-safe):**
- `database/telegram_db.py` — already checks `startswith("sqlite:///")`
- All other `database/*.py` files — already have `if "sqlite" in DATABASE_URL` guards
- `HISTORIFY_DATABASE_URL` — kept as DuckDB (separate engine, not SQLAlchemy)

---

## Pre-Deployment Checklist

### 1. On your PostgreSQL server (10.160.0.2:5532)
```sql
-- Ensure database and user exist
CREATE DATABASE algodb;
CREATE USER algo_user WITH PASSWORD 'algo@369';
GRANT ALL PRIVILEGES ON DATABASE algodb TO algo_user;

-- PostgreSQL 15+ also needs:
\c algodb
GRANT ALL ON SCHEMA public TO algo_user;
```

### 2. Allow remote connections
```bash
# In postgresql.conf
listen_addresses = '*'

# In pg_hba.conf (allow your app server IP)
host  algodb  algo_user  YOUR_APP_SERVER_IP/32  md5
```

### 3. On your app server (Ubuntu)
```bash
# Copy .sample.env to .env
cp .sample.env .env

# Install dependencies
pip install -r requirements.txt
# or for nginx deployment:
pip install -r requirements-nginx.txt

# Run the app — tables auto-create on first start
python app.py
```

---

## Rollback to SQLite

If you ever need to switch back:
```env
DATABASE_URL = 'sqlite:///db/openalgo.db'
LATENCY_DATABASE_URL = 'sqlite:///db/latency.db'
LOGS_DATABASE_URL = 'sqlite:///db/logs.db'
SANDBOX_DATABASE_URL = 'sqlite:///db/sandbox.db'
# Remove HEALTH_DATABASE_URL (falls back to sqlite automatically)
```

No code changes needed — the `if "sqlite" in DATABASE_URL` guards handle both modes.
