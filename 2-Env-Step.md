Here's your updated `.env` file. Now the important part — **GCP Firewall Ports to enable:**

---

## GCP Firewall Rules to Create

Go to **GCP Console → VPC Network → Firewall Rules → Create Firewall Rule**

| Rule Name | Direction | Ports | Source | Purpose |
|---|---|---|---|---|
| `allow-openalgo-web` | Ingress | **TCP 5000** | `0.0.0.0/0` | Flask web UI |
| `allow-openalgo-ws` | Ingress | **TCP 8765** | `0.0.0.0/0` | WebSocket live data |
| `allow-postgres` | Ingress | **TCP 5532** | `10.160.0.0/20` (internal only) | PostgreSQL (VM to VM) |
| `allow-ssh` | Ingress | **TCP 22** | Your IP | SSH access |

**Quick gcloud commands:**
```bash
# Allow Flask web app (port 5000)
gcloud compute firewall-rules create allow-openalgo-web \
  --allow tcp:5000 --source-ranges 0.0.0.0/0 --description "OpenAlgo Web UI"

# Allow WebSocket (port 8765)
gcloud compute firewall-rules create allow-openalgo-ws \
  --allow tcp:8765 --source-ranges 0.0.0.0/0 --description "OpenAlgo WebSocket"

# Allow PostgreSQL internal only (VM to VM)
gcloud compute firewall-rules create allow-postgres-internal \
  --allow tcp:5532 --source-ranges 10.160.0.0/20 --description "PostgreSQL Internal"
```

---

## What Changed in Your `.env`

| Setting | Old | New |
|---|---|---|
| `BROKER_API_KEY` | placeholder | `0b4b4f57-11f3-4c35-852d-4ca50276edcd` |
| `BROKER_API_SECRET` | placeholder | `lxq6l976rj` |
| `REDIRECT_URL` | `127.0.0.1/<broker>` | `http://34.14.172.95:5000/upstox/callback` |
| `HOST_SERVER` | `127.0.0.1` | `http://34.14.172.95:5000` |
| `FLASK_HOST_IP` | `127.0.0.1` | `0.0.0.0` (accept external) |
| `WEBSOCKET_HOST` | `127.0.0.1` | `0.0.0.0` |
| `WEBSOCKET_URL` | `ws://127.0.0.1` | `ws://34.14.172.95:8765` |
| `ZMQ_HOST` | `127.0.0.1` | `0.0.0.0` |
| `CORS_ALLOWED_ORIGINS` | `127.0.0.1` | `http://34.14.172.95:5000` |
| `FLASK_ENV` | `development` | `production` |
| `APP_KEY` | default | Fresh generated |
| `API_KEY_PEPPER` | default | Fresh generated |

---

## Upstox Developer Portal

Set this as your **Redirect URL** in Upstox Developer Portal:

```
http://34.14.172.95:5000/upstox/callback
```

**If your GCP public IP changes later**, update only these 4 places in `.env`:
- `REDIRECT_URL`
- `HOST_SERVER`
- `WEBSOCKET_URL`
- `CORS_ALLOWED_ORIGINS`

Replace `34.14.172.95` with the new IP. Then restart the app.
