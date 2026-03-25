# K-12 Sentinel

**Self-hosted security monitoring for Google Workspace in K-12 school districts.**

K-12 Sentinel monitors Google Workspace login events in real time, detects compromised accounts through geographic anomaly detection, and automatically investigates suspicious activity across Gmail, Drive, and account settings. Built for K-12 IT administrators who need visibility into security threats across their organization.

---

## Features

- **Real-time Login Monitoring** -- Ingests login events from Google Workspace via Admin SDK Reports API on a 5-minute sync cycle
- **Geographic Risk Scoring** -- Scores each login 0-100 based on geo-fence violations, impossible travel, VPN/proxy usage, datacenter ISP detection, and behavioral patterns
- **Interactive Dashboard** -- Dark-themed SOC-style UI with a Leaflet map, scrollable risk feed, and hourly timeline chart
- **User Detail Drilldown** -- Per-user travel pattern map, login history, risk summary, and quick remediation actions
- **Automated Alerts** -- Notifications via Slack, Microsoft Teams, Google Chat, or email when high-risk events are detected
- **Org Unit Filtering** -- Filter all dashboard data by Google Workspace organizational unit hierarchy
- **Automated Investigations** -- Auto-triggered forensic analysis of Gmail activity, Drive shares, account changes, and bouncebacks on high-risk events
- **Role-Based Alert Thresholds** -- Staff and student accounts have different mass-send detection thresholds to reduce false positives
- **Configurable Thresholds** -- Admin-only Settings page to tune risk scoring, mass send detection, and investigation finding thresholds through the UI — no restarts needed
- **Remediation Actions** -- Password reset, account suspension, 2FA enforcement, and session termination from the dashboard
- **Watchlist** -- Pin users for enhanced monitoring regardless of risk score
- **Trend Analytics** -- Daily risk trends, top risky countries/users, and risk distribution charts
- **Bulk Actions** -- CSV export, report generation, and email list copy for incident response

---

## Architecture

```
+-----------------+     +------------------+     +----------------------+
|    Frontend     |---->|     Backend      |---->|   Google Admin SDK   |
|  React 19/Vite  |     |   Express/TS     |     |  Reports & Directory |
|  Leaflet Maps   |     |   SQLite (WAL)   |     |  Gmail & Drive APIs  |
|  Recharts       |     |   node-cron      |     +----------------------+
|  TanStack Query |     |   Passport OAuth |---->+----------------------+
+-----------------+     +------------------+     |  IP Geolocation      |
   Nginx (Docker)           Port 3001            |  ip-api / MaxMind    |
   Port 80/443                                   +----------------------+
```

---

## Prerequisites

- **Google Workspace** for Education (any edition, including Fundamentals/free) with Admin SDK access
- **Docker** and **Docker Compose** installed on your server
- A domain name with DNS pointing to your server
- *(Optional)* MaxMind GeoLite2 database for local IP geolocation

---

## Quick Start

### 1. Get Access

K-12 Sentinel is distributed as private Docker images. To request access:

1. [Open an access request issue](https://github.com/Moore-County-Schools-Technology/k12-sentinel/issues/new?template=access-request.yml&title=%5BAccess+Request%5D+Your+District+Name) on this repository
2. Include your district name, contact email, and GitHub username
3. Once approved, you'll be granted read access to the container images

### 2. Login to the Container Registry

```bash
echo "YOUR_ACCESS_TOKEN" | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

### 3. Download Deployment Files

```bash
git clone https://github.com/Moore-County-Schools-Technology/k12-sentinel.git
cd k12-sentinel
```

### 4. Google Admin Console Setup

#### Create a GCP Project

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g., "k12-sentinel")

#### Enable APIs

Navigate to **APIs & Services > Library** and enable:
- Admin SDK API
- Gmail API
- Google Drive Activity API

#### Create a Service Account

1. Go to **IAM & Admin > Service Accounts**
2. Click **Create Service Account**
   - Name: `k12-sentinel-reader`
   - No IAM roles needed (domain-wide delegation handles access)
3. Click into the created service account
4. Go to **Keys > Add Key > Create New Key > JSON**
5. Download the key file and save as `service-account.json` in the deployment directory
6. Note the **Client ID** (numeric) from the service account details page

#### Configure Domain-Wide Delegation

1. Go to [admin.google.com](https://admin.google.com)
2. Navigate to **Security > Access and data control > API controls > Domain-wide delegation**
3. Click **Add new**
4. Paste the service account's **Client ID**
5. Add the following OAuth scopes (comma-separated):

```
https://www.googleapis.com/auth/admin.reports.audit.readonly,
https://www.googleapis.com/auth/admin.directory.user.readonly,
https://www.googleapis.com/auth/admin.directory.orgunit.readonly,
https://www.googleapis.com/auth/gmail.readonly
```

6. Click **Authorize**

> **Important:** The `GOOGLE_ADMIN_EMAIL` in your `.env` must be a **super admin** account.

### 5. Configure

```bash
cp .env.example .env
```

Edit `.env` with your district's values. At minimum, set:

```bash
# Required
GOOGLE_ADMIN_EMAIL=admin@yourdistrict.org
SESSION_SECRET=generate-a-random-string-here
ALLOWED_EMAILS=youradmin@yourdistrict.org

# Your district's email domains
STAFF_DOMAINS=yourdistrict.org
STUDENT_DOMAINS=yourdistrict.net

# Your district's location (for geo-fence)
DISTRICT_LAT=your-latitude
DISTRICT_LNG=your-longitude

# At least one alert channel
ALERT_CHANNELS=gchat
ALERT_GCHAT_WEBHOOK_URL=https://chat.googleapis.com/v1/spaces/...
```

### 6. Deploy

```bash
docker compose up -d
```

Visit your configured domain or `http://your-server-ip` to access the dashboard.

---

## Configuration Reference

### Google Workspace

| Variable | Required | Description |
|----------|----------|-------------|
| `GOOGLE_SERVICE_ACCOUNT_KEY_PATH` | Yes | Path to GCP service account key (default: `./service-account.json`) |
| `GOOGLE_ADMIN_EMAIL` | Yes | Super admin email for API impersonation |
| `GOOGLE_CUSTOMER_ID` | No | Google Workspace customer ID (default: `my_customer`) |
| `GOOGLE_OAUTH_CLIENT_ID` | No | OAuth client ID for admin login |
| `GOOGLE_OAUTH_CLIENT_SECRET` | No | OAuth client secret |
| `SESSION_SECRET` | Yes | Random string for session encryption |
| `ALLOWED_EMAILS` | No | Comma-separated emails allowed to log in |
| `BASE_URL` | No | Public URL (e.g., `https://sentinel.yourdistrict.org`) |

### Admin Authorization

| Variable | Required | Description |
|----------|----------|-------------|
| `ADMIN_EMAILS` | No | Comma-separated emails with admin access (can change Settings). If empty, all authenticated users are regular users. |
| `ADMIN_GROUP` | No | Google Group email (e.g., `sentinel-admins@yourdistrict.org`). Members of this group get admin access. Checked via Directory API. |

Admin users see a "Settings" gear icon in the navigation bar and can configure all detection thresholds through the UI. Non-admin users can view the dashboard and investigations but cannot change settings.

### District Configuration

| Variable | Required | Description |
|----------|----------|-------------|
| `STAFF_DOMAINS` | Yes | Comma-separated staff email domains (e.g., `district.org,district.support`) |
| `STUDENT_DOMAINS` | Yes | Comma-separated student email domains (e.g., `students.district.org`) |
| `DISTRICT_LAT` | No | District center latitude (default: `35.24`) |
| `DISTRICT_LNG` | No | District center longitude (default: `-79.77`) |
| `DISTRICT_RADIUS_MILES` | No | Geo-fence radius in miles (default: `100`) |
| `DISTRICT_TIMEZONE` | No | Local timezone (default: `America/New_York`) |

### Geolocation

| Variable | Required | Description |
|----------|----------|-------------|
| `GEO_PROVIDER` | No | `ip-api` (free, 45 req/min) or `maxmind` (local DB). ip-api falls back to MaxMind for IPv6 |
| `MAXMIND_DB_PATH` | No | Path to MaxMind GeoLite2 database |

### Alerting

| Variable | Required | Description |
|----------|----------|-------------|
| `ALERT_CHANNELS` | No | Comma-separated: `slack`, `teams`, `gchat`, `email` |
| `ALERT_SLACK_WEBHOOK_URL` | No | Slack incoming webhook URL |
| `ALERT_TEAMS_WEBHOOK_URL` | No | Microsoft Teams webhook URL |
| `ALERT_GCHAT_WEBHOOK_URL` | No | Google Chat webhook URL |
| `ALERT_EMAIL_TO` | No | Recipient email address |
| `ALERT_EMAIL_FROM` | No | Sender email address |
| `ALERT_SMTP_HOST` | No | SMTP server hostname (default: `smtp.gmail.com`) |
| `ALERT_SMTP_PORT` | No | SMTP server port (default: `587`) |
| `ALERT_SMTP_USER` | No | SMTP username |
| `ALERT_SMTP_PASS` | No | SMTP password |

### Investigation

| Variable | Required | Description |
|----------|----------|-------------|
| `INVESTIGATION_AUTO_THRESHOLD` | No | Risk score to auto-trigger investigation (default: `60`) |
| `INVESTIGATION_DEFAULT_WINDOW_HOURS` | No | Hours of history to analyze (default: `48`) |
| `INVESTIGATION_MAX_CONCURRENT` | No | Max parallel investigations (default: `3`) |

### Server

| Variable | Required | Description |
|----------|----------|-------------|
| `PORT` | No | Backend server port (default: `3001`) |
| `DB_PATH` | No | SQLite database path (default: `./data/sentinel.db`) |

---

## Risk Scoring

Each login event receives a risk score from 0-100 based on three checks:

- **Geo-Fence** (0-80 pts) -- Distance from district, international logins
- **Impossible Travel** (0-70 pts) -- Physically impossible movement between logins
- **Suspicious Patterns** (0-55 pts) -- VPN/proxy, datacenter ISPs, off-hours, new locations, compromised passwords

**Risk Levels:** High (>=60) | Medium (30-59) | Low (<30)

All thresholds above are defaults and can be changed by admins through the **Settings** page in the UI.

## Settings Page

Admin users can tune all detection thresholds through the Settings page (gear icon in the nav bar). Changes take effect within 5 minutes — no container restart needed.

**Configurable values:**

| Section | Settings |
|---------|----------|
| **Risk Scoring** | Alert threshold (70), auto-investigation threshold (60), high risk cutoff (60), medium risk cutoff (30) |
| **Mass Send Detection** | Staff recipients/hr (100), staff messages/hr (200), student recipients/hr (25), student messages/hr (50) |
| **Investigation Findings** | External/internal email thresholds per role, bounceback thresholds per role |

A "Reset to Defaults" button restores all values to the built-in defaults. If no settings have been configured, the system uses defaults out of the box.

## Alerts

Alerts fire when:
- Risk score >= 70
- Login from a new country
- Watchlisted user logs in
- Mass email detected (role-based thresholds)
- Critical investigation findings (forwarding, suspicious filters, Drive exfiltration)

### Mass Send Detection

Uses **AND logic** with role-based thresholds:

| Role | Trigger (both must be met) |
|------|---------------------------|
| **Staff** | 100+ unique recipients AND 200+ messages per hour |
| **Student** | 25+ unique recipients AND 50+ messages per hour |

---

## Google Workspace Edition Compatibility

K-12 Sentinel works with **all Google Workspace for Education editions**, including the free tier. All features are available regardless of which edition your district uses.

| Feature | Fundamentals (Free) | Standard | Plus |
|---------|:-------------------:|:--------:|:----:|
| Login event monitoring & risk scoring | Yes | Yes | Yes |
| Geo-fence & impossible travel detection | Yes | Yes | Yes |
| Automated alerts (Slack, Teams, Chat, email) | Yes | Yes | Yes |
| Gmail investigation (sent emails, filters, forwarding) | Yes | Yes | Yes |
| Drive sharing investigation | Yes | Yes | Yes |
| Mass send detection | Yes | Yes | Yes |
| Remediation (password reset, suspend, sign out) | Yes | Yes | Yes |
| Admin Settings page | Yes | Yes | Yes |
| Domain-wide delegation | Yes | Yes | Yes |

**How it works:** K-12 Sentinel uses the Admin SDK Reports API, Directory API, and Gmail API — all available on every edition via domain-wide delegation with a service account. The paid editions (Standard/Plus) add Google's own security investigation UI and BigQuery log exports, but the underlying APIs that K-12 Sentinel depends on are not edition-gated.

**What the paid editions add (not related to K-12 Sentinel):**
- Security Investigation Tool in Admin Console (Standard, Plus)
- BigQuery audit log exports (Standard, Plus)
- Log export to Google Security Operations (Standard, Plus)
- Advanced endpoint management (Plus)

---

## Updating

```bash
docker compose pull
docker compose up -d
```

---

## Troubleshooting

### "googleConnected: false" in health check

- Verify `GOOGLE_ADMIN_EMAIL` is a super admin
- Confirm the service account JSON file exists at the configured path
- Check that domain-wide delegation is configured with the correct Client ID and scopes

### No events after first start

- Login events can take up to 30 minutes to appear in the Google Admin Reports API
- The first sync backfills 180 days of historical data — this may take several minutes
- Check backend logs: `docker compose logs backend`

### Rate limiting (ip-api)

- The free tier allows 45 requests per minute
- For higher volume, use MaxMind: set `GEO_PROVIDER=maxmind` and provide `MAXMIND_DB_PATH`

---

## Support

- **Request Access:** [Open an access request](https://github.com/Moore-County-Schools-Technology/k12-sentinel/issues/new?template=access-request.yml&title=%5BAccess+Request%5D+Your+District+Name)
- **Bug Reports & Features:** [Open an issue](https://github.com/Moore-County-Schools-Technology/k12-sentinel/issues/new)

---

## License

Proprietary. Contact Moore County Schools Technology for licensing information.
