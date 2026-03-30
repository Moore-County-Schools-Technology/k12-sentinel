# K-12 Sentinel

> **Initial NCET Beta Release (Free just for NC Districts)**

> **BETA DISCLAIMER:** K-12 Sentinel is in active beta development. While we strive for accuracy, all alerts, risk scores, AI classifications, DLP detections, and safety assessments should be independently verified before taking action. This software is provided as-is with no warranty. Do not rely solely on K-12 Sentinel for security or compliance decisions — always confirm findings through your Google Admin Console and established district procedures.

**Self-hosted security monitoring for Google Workspace in K-12 school districts.**

K-12 Sentinel monitors Google Workspace login events in real time, detects compromised accounts through geographic anomaly detection, and automatically investigates suspicious activity across Gmail, Drive, and account settings. Built for K-12 IT administrators who need visibility into security threats across their organization.

---

## Features

- **Real-time Login Monitoring** -- Ingests login events from Google Workspace via Admin SDK Reports API on a 5-minute sync cycle
- **Geographic Risk Scoring** -- Scores each login 0-100 based on geo-fence violations, impossible travel, VPN/proxy usage, datacenter ISP detection, and behavioral patterns
- **Interactive Dashboard** -- Dark-themed SOC-style UI with a Leaflet map, scrollable risk feed, and hourly timeline chart
- **User Detail Drilldown** -- Per-user travel pattern map, login history, risk summary, and quick remediation actions
- **Correlated Threat Engine** -- International logins no longer spam your alerts. High-risk events trigger silent investigations first — Google Chat only fires when post-login evidence (forwarding rules, mass email, recovery changes) corroborates the threat
- **Country Whitelist & Geo Learning** -- District-level country whitelist (admin-configurable) plus per-user learning that auto-whitelists countries after 3+ logins in 90 days
- **Brute Force Detection** -- Immediate alerts for 3+ failed logins from different IPs followed by a successful login from a new IP
- **High-Confidence Signal Alerts** -- Instant alerts for Google-confirmed compromises and Google's `is_suspicious` flag — no investigation delay
- **Automated Alerts** -- Notifications via Slack, Microsoft Teams, Google Chat, or email. Alerts include device type, OS, authentication method, and subject lines for mass-send events
- **Automated Investigations** -- Auto-triggered forensic analysis of Gmail activity, Drive shares, account changes, and bouncebacks on high-risk events
- **Investigation Timeline** -- Unified chronological view stitching login events, findings, email activity, admin audit events, and remediation actions into a single narrative with category filtering
- **Inline Email Analysis** -- See top subject lines, recipient breakdown (internal vs external), hourly volume chart, and sample recipients directly in investigations — no need to visit Google Admin Console
- **30-Day Sign-In History** -- Expanded login history with device, OS, auth method, and concurrent session detection (logins from different locations within 5 minutes)
- **Admin Audit Integration** -- Track who changed recovery emails, added delegates, enrolled/unenrolled 2FA, suspended accounts, and granted admin privileges — with actor identification
- **One-Click Containment** -- Single button executes the full containment sequence: revoke sessions, reset password, remove forwarding, delete suspicious filters, disable POP/IMAP, revoke third-party app access. Step-by-step progress shown in real time.
- **Individual Remediation Actions** -- 9 granular actions available independently: revoke sessions, reset password, suspend/unsuspend account, remove forwarding, delete filters, disable POP/IMAP, revoke OAuth, revoke app passwords
- **Case Management** -- Mark investigations as Open, Contained, False Positive, Escalated, or Resolved with notes. False positive feedback reinforces per-user geo learning. Full audit trail. Investigation Center shows resolution status badges, filter by status, and dashboard counts.
- **Alert Follow-Ups** -- Containment actions and status changes send follow-up notifications to all configured alert channels
- **Mass Email Content Scanning** -- When a mass send is detected, samples message bodies and runs hybrid phishing classification: heuristic analysis (URL reputation, domain spoofing, urgency patterns, credential harvesting) with optional local LLM escalation. Alerts arrive pre-classified as phishing, spam, legitimate bulk, or inconclusive. **All AI processing happens on your server** — no email content ever leaves your network.
- **Data Loss Prevention (DLP)** -- Scans outbound email AND Google Drive documents for SSNs, credit card numbers, dates of birth, grade data, and custom patterns you define. Alerts fire before sensitive data leaves your district. Live test field in Settings to verify pattern matching.
- **Drive Content Scanning** -- When a Google Doc, Sheet, or Slide is shared externally, K-12 Sentinel exports the document text and runs DLP scanning automatically. Catches sensitive data inside documents, not just email.
- **Email Attachment Scanning** -- Scans attachment contents (PDFs, Word docs, spreadsheets, images with OCR) for DLP violations and phishing links. All processing happens on your server.
- **Phishing Email Quarantine** -- When a phishing mass send is detected, admins can click one button to trash those messages from every recipient's inbox across the district. Admin-approved, not automatic — you stay in control.
- **AI Investigation Assistant** -- Click "Summarize with AI" on any investigation and get a plain-English explanation of what happened. Chat with the AI to ask follow-up questions. All processing happens on your server using the local LLM — no data leaves your network.
- **Compromised Credential Monitoring** -- Automatically checks if district email addresses appear in known data breaches (via Have I Been Pwned). Alerts when accounts show up in new breaches. Breach badges appear on user profiles during investigations.
- **OAuth App Governance** -- See every third-party app granted access to your district's Google accounts. Risk scoring flags apps with broad permissions (Gmail read, Drive write). New "Apps" page to review, approve, or block apps across the domain. Scans automatically every 6 hours.
- **Daily/Weekly Digest Reports** -- Scheduled summary emails with trend data: high-risk events, investigations, mass email activity, top risky users, new country logins, remediation actions. Delivered as chat message + full HTML email. Configure frequency and recipients in Settings.
- **Automated Incident Reports (PDF)** -- One-click PDF generation from any investigation. Professional reports with executive summary, timeline, findings, remediation log — auto-generated and emailed when cases are resolved or escalated. Ready for insurance claims, legal, and board reporting.
- **Role-Based Alert Thresholds** -- Staff and student accounts have different mass-send detection thresholds to reduce false positives
- **Configurable Thresholds** -- Admin-only Settings page to tune risk scoring, mass send detection, DLP sensitivity, and investigation finding thresholds through the UI — no restarts needed
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

### 3. Run the Setup Wizard

The setup wizard is a browser-based tool that walks you through the entire configuration — no `.env` file editing required. It guides you through Google Workspace setup with direct links to the right pages, validates every input, lets you test alert webhooks, and generates all configuration files automatically.

```bash
mkdir k12-sentinel && cd k12-sentinel
docker run -it --rm -p 3000:3000 -v "$(pwd)":/config ghcr.io/moore-county-schools-technology/sentinel-setup
```

Open **http://your-server-ip:3000** in your browser and follow the 7-step wizard:

1. **Welcome** — overview and checklist of what you'll need
2. **District Information** — your domain names, location, timezone
3. **Google Workspace** — guided setup with direct links to GCP console (the wizard tells you exactly where to click)
4. **Dashboard Access** — who can log in and view the dashboard
5. **Alert Channels** — configure Google Chat, Slack, Teams, or email alerts (with test buttons)
6. **Networking** — choose between automatic HTTPS (Caddy), existing reverse proxy (Traefik), or HTTP-only for testing
7. **Review & Generate** — verify everything and generate your configuration

The wizard creates all the files you need: `.env`, `docker-compose.yml`, and an encrypted copy of your service account credentials.

### 4. Deploy

```bash
docker compose up -d
```

That's it. Visit your configured domain (e.g., `https://sentinel.yourdistrict.org`).

> **Note:** The first sync backfills 180 days of historical login data — this may take several minutes. Login events from Google can take up to 30 minutes to appear in the API. This is normal.

<details>
<summary><strong>Manual Setup (Advanced)</strong> — click to expand if you prefer to configure manually</summary>

### Manual Google Admin Console Setup

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
5. Add OAuth scopes. K-12 Sentinel uses **tiered scopes** — only the Core tier is required. Optional tiers enable additional features and can be added later without downtime.

**Core (required)** — login monitoring, risk scoring, dashboard, alerts, investigations:

```
https://www.googleapis.com/auth/admin.reports.audit.readonly,
https://www.googleapis.com/auth/admin.directory.user.readonly,
https://www.googleapis.com/auth/admin.directory.orgunit.readonly
```

**Gmail (optional)** — content scanning, phishing quarantine, forwarding/delegate detection, containment of mail settings:

```
https://www.googleapis.com/auth/gmail.modify,
https://www.googleapis.com/auth/gmail.settings.basic,
https://www.googleapis.com/auth/gmail.settings.sharing
```

**Security (optional)** — session revocation, OAuth token revocation, app password revocation, account suspend/unsuspend, password reset, OAuth app governance:

```
https://www.googleapis.com/auth/admin.directory.user.security
```

**Drive (optional)** — DLP scanning on externally shared Google Docs, Sheets, and Slides:

```
https://www.googleapis.com/auth/drive.readonly
```

To enable all features, add all scopes comma-separated. To start minimal, add only the Core scopes — the system detects which tiers are authorized and disables unavailable features gracefully.

| Scope | Tier | Why it's needed |
|-------|------|-----------------|
| `admin.reports.audit.readonly` | Core | Reads login events, Gmail delivery logs, and admin audit events from the Reports API. Core data source for all monitoring. |
| `admin.directory.user.readonly` | Core | Looks up user profiles, org units, and 2FA enrollment status for risk context and user detail pages. |
| `admin.directory.orgunit.readonly` | Core | Resolves organizational unit paths so staff/student classification and org-based filtering work correctly. |
| `gmail.modify` | Gmail | Powers phishing quarantine — moves confirmed phishing emails to trash across recipient inboxes. Also reads message samples for content scanning. |
| `gmail.settings.basic` | Gmail | Reads and removes mail forwarding addresses and POP/IMAP settings during account containment. Detects suspicious auto-forwarding rules. |
| `gmail.settings.sharing` | Gmail | Reads and removes mail delegates (send-as permissions) during containment. Detects unauthorized delegate additions. |
| `admin.directory.user.security` | Security | Revokes user sessions, app passwords, and third-party OAuth tokens during containment. Powers OAuth app governance. Required for account suspend/unsuspend and password reset. |
| `drive.readonly` | Drive | Exports Google Docs, Sheets, and Slides text for DLP scanning when documents are shared externally. |

> **Scope status:** After startup, check `GET /api/health/scopes` to see which tiers are active. The backend logs which scope groups are available on every restart.

6. Click **Authorize**

> **Important:** The `GOOGLE_ADMIN_EMAIL` in your `.env` must be a **super admin** account.

#### Configure .env

```bash
cp .env.example .env
```

Edit `.env` with your district's values. At minimum, set:

```bash
GOOGLE_ADMIN_EMAIL=admin@yourdistrict.org
SESSION_SECRET=generate-a-random-string-here
ALLOWED_EMAILS=youradmin@yourdistrict.org
STAFF_DOMAINS=yourdistrict.org
STUDENT_DOMAINS=yourdistrict.net
# Your district office location (use Google Maps to find coordinates)
DISTRICT_LAT=35.24
DISTRICT_LNG=-79.77
ALERT_CHANNELS=gchat
ALERT_GCHAT_WEBHOOK_URL=https://chat.googleapis.com/v1/spaces/...
```

#### Deploy

```bash
docker compose up -d
```

</details>

Visit your configured domain or `http://your-server-ip` to access the dashboard.

---

## What the Wizard Configures

The setup wizard handles all configuration for you. Here's what each step sets up — you don't need to edit any of this manually.

| Wizard Step | What it configures |
|---|---|
| **District Info** | Your email domains (staff vs student), district location for the geo-fence, timezone |
| **Google Workspace** | Service account credentials (encrypted at rest), domain-wide delegation, super admin email |
| **Dashboard Access** | Who can log in (email allowlist or Google OAuth), who gets admin access to Settings |
| **Alert Channels** | Where security alerts go — Google Chat, Slack, Teams, and/or email (SMTP). Optional AI phishing classification with bundled Ollama. |
| **Networking** | How users reach the dashboard — automatic HTTPS (Caddy), existing reverse proxy (Traefik), or HTTP for testing |

**After deployment, configure everything else through the Settings page** in the dashboard — no config files or restarts needed. This includes: risk scoring thresholds, mass send limits, AI phishing classification (LLM endpoint), DLP patterns and custom rules, OAuth app governance, digest report scheduling, and student safety signals.

<details>
<summary><strong>Environment Variable Reference</strong> — for advanced users who need to edit .env manually</summary>

All of these are auto-generated by the setup wizard. You only need this reference if you're doing manual setup or need to change a specific value after deployment.

| Variable | Description |
|----------|-------------|
| **Google Workspace** | |
| `GOOGLE_ADMIN_EMAIL` | Super admin email for API access |
| `GOOGLE_CUSTOMER_ID` | Workspace customer ID (default: `my_customer`) |
| `SERVICE_ACCOUNT_PASSPHRASE` | Passphrase to decrypt the encrypted service account key |
| **Dashboard Access** | |
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth client ID (if using Google sign-in) |
| `GOOGLE_OAUTH_CLIENT_SECRET` | OAuth client secret |
| `SESSION_SECRET` | Random string for session encryption |
| `ALLOWED_EMAILS` | Comma-separated emails allowed to log in |
| `ADMIN_EMAILS` | Comma-separated emails with Settings page access |
| `BASE_URL` | Public URL (e.g., `https://sentinel.yourdistrict.org`) |
| **District** | |
| `STAFF_DOMAINS` | Staff email domains (comma-separated) |
| `STUDENT_DOMAINS` | Student email domains (comma-separated) |
| `DISTRICT_LAT` / `DISTRICT_LNG` | District center coordinates (auto-set by wizard from address) |
| `DISTRICT_RADIUS_MILES` | Geo-fence radius in miles (default: `100`) |
| `DISTRICT_TIMEZONE` | Local timezone (default: `America/New_York`) |
| **Alerts** | |
| `ALERT_CHANNELS` | Which channels: `gchat`, `slack`, `teams`, `email` |
| `ALERT_GCHAT_WEBHOOK_URL` | Google Chat webhook URL |
| `ALERT_SLACK_WEBHOOK_URL` | Slack webhook URL |
| `ALERT_TEAMS_WEBHOOK_URL` | Teams webhook URL |
| `ALERT_EMAIL_TO` / `ALERT_EMAIL_FROM` | Email alert addresses |
| `ALERT_SMTP_HOST` / `ALERT_SMTP_PORT` | SMTP server (default: `smtp.gmail.com:587`) |
| `ALERT_SMTP_USER` / `ALERT_SMTP_PASS` | SMTP credentials |
| **Optional** | |
| `GEO_PROVIDER` | `ip-api` (free) or `maxmind` (local DB) |
| `LLM_ENDPOINT` | OpenAI-compatible URL for phishing classification |
| `LLM_MODEL` | Model name (default: `llama3.1:8b`) |

</details>

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
| **Country Whitelist** | District-level whitelisted countries (default: US). Logins from whitelisted countries skip the 80-point international penalty. Per-user geo learning auto-whitelists after 3+ logins in 90 days. |
| **Mass Send Detection** | Staff recipients/hr (100), staff messages/hr (200), student recipients/hr (25), student messages/hr (50) |
| **Investigation Findings** | External/internal email thresholds per role, bounceback thresholds per role |
| **AI Phishing Classification** | LLM endpoint, model, API key — configure local Ollama or any OpenAI-compatible endpoint |
| **Data Loss Prevention** | Toggle built-in patterns (SSN, credit card, DOB, GPA), add custom regex rules, set sensitivity thresholds, test patterns with sample text |
| **OAuth App Governance** | Enable/disable, scan frequency (1-24 hours), manage known educational apps list |
| **Digest Reports** | Frequency (daily/weekly/both/off), send time, day of week, additional email recipients |

A "Reset to Defaults" button restores all values to the built-in defaults. If no settings have been configured, the system uses defaults out of the box.

## Alerts

Alerts use a **correlated threat engine** — most high-risk logins trigger a silent investigation first, and notifications only fire when post-login evidence confirms the threat.

**Immediate alerts** (no delay):
- Brute force: 3+ failed logins from different IPs then success from new IP
- Google-confirmed compromise (`account_disabled_password_leak`)
- Google suspicious flag (`is_suspicious`)
- Watchlisted user login

**Corroborated alerts** (investigate first, alert if evidence found):
- International login from unknown country + post-login evidence (forwarding, mass email, recovery changes, etc.)

**Dashboard-only** (no push notification):
- International login with no post-login evidence
- Logins from whitelisted or per-user learned countries

### Mass Send Detection

Uses **AND logic** with role-based thresholds:

| Role | Trigger (both must be met) |
|------|---------------------------|
| **Staff** | 100+ unique recipients AND 200+ messages per hour |
| **Student** | 25+ unique recipients AND 50+ messages per hour |

When triggered, the system **samples message bodies** and runs hybrid content classification:
1. **Heuristic analysis** (always) -- phishing domains, URL mismatches, urgency language, credential harvesting, risky attachments
2. **LLM escalation** (optional) -- if heuristics are inconclusive and `LLM_ENDPOINT` is configured, sends samples to a local model for classification

Alerts arrive pre-classified: `PHISHING`, `Spam`, `likely legitimate`, or `content unclear`. Emails classified as legitimate bulk are logged for audit but do not trigger push notifications — only phishing, spam, and inconclusive classifications notify. No LLM required -- heuristics alone are useful.

**AI classification runs 100% on your server.** Email content is analyzed by a local AI model — no student data, email bodies, or alert details are ever sent to OpenAI, Google, Anthropic, or any external service. The model runs on your hardware, in your network, behind your firewall. There is zero cloud AI dependency.

**Enabling it:** The setup wizard has an optional "AI Phishing Classification" section on the Alert Channels page. If you enable it:
- The wizard checks if Ollama is already running on your server
- If not, it offers to **bundle Ollama as a Docker container** in your deployment — no extra installation needed
- After `docker compose up -d`, just run one command to download the model:
  ```bash
  docker compose exec sentinel-ollama ollama pull llama3.1:8b
  ```
- You can also point to any existing OpenAI-compatible endpoint instead

---

## Google Workspace Edition Compatibility

K-12 Sentinel works with **all Google Workspace for Education editions**, including the free Fundamentals tier. All features are available on every edition -- no upgrade required.

| Feature | Fundamentals (Free) | Standard | Plus |
|---------|:-------------------:|:--------:|:----:|
| Login event monitoring & risk scoring | Yes | Yes | Yes |
| Geo-fence & impossible travel detection | Yes | Yes | Yes |
| Automated alerts (Slack, Teams, Chat, email) | Yes | Yes | Yes |
| Authentication method in alerts (password, security key, etc.) | Yes | Yes | Yes |
| Google suspicious login flag in alerts | Yes | Yes | Yes |
| Device type & OS in alerts (when available) | Yes | Yes | Yes |
| Gmail investigation (sent emails, filters, forwarding) | Yes | Yes | Yes |
| Drive sharing investigation | Yes | Yes | Yes |
| Mass send detection with subject lines & attachment flags | Yes | Yes | Yes |
| Remediation (password reset, suspend, sign out) | Yes | Yes | Yes |
| Correlated threat engine (investigate-then-alert) | Yes | Yes | Yes |
| Country whitelist & per-user geo learning | Yes | Yes | Yes |
| Investigation timeline (unified chronological view) | Yes | Yes | Yes |
| Inline email analysis (subjects, recipients, hourly chart) | Yes | Yes | Yes |
| 30-day sign-in history with concurrent session detection | Yes | Yes | Yes |
| Admin audit with actor identification | Yes | Yes | Yes |
| One-click account containment (9 actions) | Yes | Yes | Yes |
| Case management (5 resolution statuses + notes) | Yes | Yes | Yes |
| Alert follow-up notifications | Yes | Yes | Yes |
| AI phishing classification (local LLM, no cloud dependency) | Yes | Yes | Yes |
| Phishing email quarantine (admin-triggered) | Yes | Yes | Yes |
| Data Loss Prevention (SSN, credit card, FERPA, custom rules) | Yes | Yes | Yes |
| Drive content scanning (DLP on shared documents) | Yes | Yes | Yes |
| Email attachment scanning (PDF, Office, images with OCR) | Yes | Yes | Yes |
| OAuth app governance (risk scoring, approve/block) | Yes | Yes | Yes |
| Daily/weekly digest reports (chat + HTML email) | Yes | Yes | Yes |
| Automated PDF incident reports | Yes | Yes | Yes |
| Admin Settings page | Yes | Yes | Yes |

> **Note on device info:** Device type and OS version are populated by Google primarily for mobile and tablet logins. Desktop browser logins may not include this information regardless of edition. When unavailable, these fields are simply blank -- all other alert details still appear.

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

- **Contact:** Jonathan Vargas — [jvargas@ncmcs.org](mailto:jvargas@ncmcs.org) (Moore County Schools)
- **Request Access:** [Open an access request](https://github.com/Moore-County-Schools-Technology/k12-sentinel/issues/new?template=access-request.yml&title=%5BAccess+Request%5D+Your+District+Name)
- **Bug Reports & Features:** [Open an issue](https://github.com/Moore-County-Schools-Technology/k12-sentinel/issues/new)

---

## License

Proprietary. Contact Jonathan Vargas ([jvargas@ncmcs.org](mailto:jvargas@ncmcs.org)) at Moore County Schools for licensing information.
