# Warden Icons

A lightweight, privacy-focused **Cloudflare Worker** that fetches, serves, and caches website favicons for self-hosted **Bitwarden / Vaultwarden** instances (such as [`warden-worker`](https://github.com/qaz741wsd856/warden-worker) or Vaultwarden setups) using **Cloudflare Workers KV**.

> **AI Disclosure:** This project—including the Worker implementation, Wrangler configuration, GitHub Actions deployment workflow, and documentation—was created and configured with the assistance of AI.

---

## Features

- **Bitwarden-Compatible Endpoint:** Responds to `GET /icons/<hostname>/icon.png` requests with automatic SVG and PNG content-type detection.
- **Smart Domain Fallback:** Checks exact hostnames first, then falls back to parent domains (e.g., `auth.example.com` -> `example.com`) while respecting multi-label TLDs like `.co.uk` and `.com.au`.
- **Two-Tier Icon Sourcing:** Fetches icons from DuckDuckGo's icon service first, falling back to Google's favicon service on misses.
- **30-Day Edge & Browser Caching:** Stores fetched icons in Cloudflare Workers KV (`warden.icons` binding) for 30 days (`2592000` seconds) and sets matching `Cache-Control` headers.
- **Strict CORS & Hostname Validation:** Restricts `Access-Control-Allow-Origin` to your configured vault origin and blocks invalid or internal (`localhost`) hostnames.
- **Zero-Log & Private-by-Default Configuration:** Disables `*.workers.dev` production subdomains, preview deployment URLs, and Cloudflare Worker observability/invocation logs.
- **Automated GitHub Actions Deployment:** Automatically creates the `warden-icons-cache` KV namespace if missing, injects secrets at build time without exposing your domain in Git history, and deploys via Wrangler.

---

## Repository Structure

```text
├── worker.js                           # Cloudflare Worker script
├── wrangler.toml                       # Wrangler configuration (generic placeholders, logs & workers.dev disabled)
├── .github/workflows/deploy-icons.yaml # Automated CI/CD deployment & KV provisioning workflow
├── LICENSE                             # MIT License
└── README.md                           # Project documentation
```

---

## Setup & Deployment

You can deploy this repository entirely through the **GitHub** and **Cloudflare** web dashboards without installing local CLI tools.

### 1. Fork or Clone This Repository
Create your own copy of this repository on GitHub. Keep `wrangler.toml` with its generic placeholders so your custom domain and Cloudflare credentials never appear in your repository files or commit history.

### 2. Configure GitHub Actions Secrets
In your GitHub repository, navigate to **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**, and add the following three secrets:

| Secret Name | Description | Example |
| :--- | :--- | :--- |
| `CLOUDFLARE_API_TOKEN` | A Cloudflare API Token with **Workers Scripts: Edit**, **Workers KV Storage: Edit**, and **Account Settings: Read** permissions. | `your_cloudflare_api_token` |
| `CLOUDFLARE_ACCOUNT_ID` | Your Cloudflare Account ID (found on the right sidebar of your Cloudflare Dashboard overview). | `0123456789abcdef0123456789abcdef` |
| `ALLOWED_ORIGIN` | The exact origin URL of your Bitwarden / Vaultwarden web vault (no trailing slash). | `[https://vault.example.com](https://vault.example.com)` |

### 3. Trigger Deployment
Push any commit to `main` (or open the **Actions** tab -> **Deploy Warden Icons Worker** -> **Run workflow**). The workflow will automatically:
1. Check your Cloudflare account for a KV namespace named `warden-icons-cache` and create it if it does not exist.
2. Inject `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`, `ALLOWED_ORIGIN`, and the KV Namespace ID into `wrangler.toml` inside the ephemeral runner environment.
3. Deploy the `warden-icons` Worker to Cloudflare.

### 4. Attach Your Custom Route in Cloudflare
Because `workers_dev` and `preview_urls` are disabled for privacy, bind the Worker to your vault's `/icons/*` path in Cloudflare:
1. Open the **Cloudflare Dashboard** -> **Workers & Pages** -> **warden-icons**.
2. Go to **Settings** -> **Domains & Routes** -> **Add** -> **Route**.
3. Set the route to `[vault.example.com/icons/](https://vault.example.com/icons/)*` (matching your `ALLOWED_ORIGIN` domain), select your zone, and click **Add route**.

---

## `wrangler.toml` Configuration Reference

| Parameter | Section | Purpose |
| :--- | :--- | :--- |
| `name` | Top-level | Sets the Worker service name (`warden-icons`) in your Cloudflare account. |
| `main` | Top-level | Entry point file (`worker.js`) executed on incoming requests. |
| `compatibility_date` | Top-level | Pins the Cloudflare Workers runtime API behavior (`2026-01-01`) for stability across platform updates. |
| `workers_dev = false` | Top-level | Disables the default `*.workers.dev` subdomain so the Worker is only reachable via your custom route. |
| `preview_urls = false` | Top-level | Disables automatic preview deployment URLs. |
| `[observability]`<br>`enabled = false` | Observability | Globally disables Cloudflare observability telemetry and metrics collection for the Worker. |
| `[observability.logs]`<br>`enabled = false`<br>`invocation_logs = false` | Observability Logs | Disables persistent logs and per-request invocation logging so requested hostnames and client metadata are not logged. |
| `[vars]` | Variables | Holds generic documentation placeholders (`CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`, `ALLOWED_ORIGIN`) that GitHub Actions populates from repository secrets during deployment. |
| `[[kv_namespaces]]` | KV Storage | Binds the `warden-icons-cache` KV namespace to `env["warden.icons"]` in `worker.js` for 30-day favicon caching. |

---

## Response Headers

- `X-Icon-Cache: HIT` — Served directly from Workers KV for the exact requested hostname.
- `X-Icon-Cache: HIT-PARENT` — Served from Workers KV using a cached parent domain icon.
- `X-Icon-Cache: MISS` — Fetched upstream for the exact hostname and stored in Workers KV.
- `X-Icon-Cache: MISS-PARENT` — Fetched upstream from a parent domain and cached under the requested hostname.

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
