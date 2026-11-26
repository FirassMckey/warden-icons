# Warden Icons

A lightweight, privacy-focused **Cloudflare Worker** that fetches, serves, and caches website favicons for self-hosted **Bitwarden / Vaultwarden** instances (such as [`warden-worker`](https://github.com/aspect-build/warden-worker) or Vaultwarden setups) using **Cloudflare Workers KV**.

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
├── worker.js                         # Cloudflare Worker script
├── wrangler.toml                     # Wrangler configuration (generic placeholders, logs & workers.dev disabled)
├── .github/workflows/deploy-icons.yaml # Automated CI/CD deployment & KV provisioning workflow
├── LICENSE                           # MIT License
└── README.md                         # Project documentation
