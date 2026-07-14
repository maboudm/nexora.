# Nexora

Multi-tenant cloud security posture management (CSPM) platform. Runs misconfiguration, IaC, and compliance scans across AWS (with Azure/GCP groundwork), scores findings by exploitability and business impact, and maps multi-hop attack paths to MITRE ATT&CK.

Built solo as a hands-on project to go deep on backend engineering and cloud security at the same time.

[![CI](https://github.com/maboudm/nexora/actions/workflows/ci.yml/badge.svg)](https://github.com/maboudm/nexora/actions/workflows/ci.yml)
[![CodeQL](https://github.com/maboudm/nexora/actions/workflows/codeql.yml/badge.svg)](https://github.com/maboudm/nexora/actions/workflows/codeql.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## What it does

- **Scans** cloud accounts and IaC repositories using [Prowler](https://github.com/prowler-cloud/prowler) and [Checkov](https://github.com/bridgecrewio/checkov), plus a small set of custom CIS AWS Benchmark checks, for roughly 2,000 combined checks.
- **Prioritizes findings** with a risk score built from CVSS, EPSS, network exposure, and asset criticality, then packs a "what to fix this week" list using a knapsack-style optimizer under an engineer-hours budget.
- **Maps attack paths** with a graph model (NetworkX) that traces multi-hop routes from internet-facing assets to sensitive resources, tagged with MITRE ATT&CK technique IDs.
- **Stores evidence** for compliance frameworks (CIS, PCI-DSS, ISO 27001, SOC 2, HIPAA, NIST, GDPR) as a SHA-256/HMAC hash-chained ledger, so records can be checked for tampering.
- **Handles multi-tenant auth and billing**: JWT (HS256) auth, RBAC, Google OAuth 2.0 and SAML 2.0 SSO, Stripe billing.
- **Ships itself**: Dockerized, deployed behind Caddy with automated HTTPS, managed with systemd, with GitHub Actions for CI, CodeQL scanning, and image publishing.

## Stack

| Layer | Tech |
|---|---|
| Backend | Python 3.11/3.12, FastAPI, Pydantic v2 |
| Frontend | Next.js 16 (App Router), React, TypeScript, Tailwind, shadcn/ui |
| Data | SQLite (via Prisma for the frontend layer) |
| Cloud SDKs | boto3 (AWS); Azure/GCP clients scaffolded |
| Graph / scoring | NetworkX, custom Bayesian-style scoring |
| Auth | PyJWT, bcrypt, OAuth 2.0, SAML 2.0 |
| Infra | Docker (multi-stage), Docker Compose, systemd, Caddy, GitHub Actions |

Repo size: ~39K lines of Python across 40+ backend modules, ~17K lines of TypeScript/TSX on the frontend, 176 FastAPI routes.

## Architecture

```
nexora/
├── backend/
│   ├── gateway/            # FastAPI app entrypoint, persistence, websocket
│   ├── scanner_engine/      # Orchestrates scan runs
│   ├── prowler_integration/  } wraps the open-source
│   ├── checkov_integration/  } scanning engines
│   ├── rule_engine/          # Custom CIS AWS Benchmark checks
│   ├── risk_engine/, blast_radius/, finding_prioritization/
│   │                         # CVSS x EPSS x exposure scoring, knapsack remediation planner
│   ├── attack_graph/, kill_chain/
│   │                         # NetworkX attack-path modeling, MITRE ATT&CK mapping
│   ├── evidence_vault/, audit/, compliance_engine/
│   │                         # Hash-chained evidence store, framework mapping
│   ├── credential_manager/, sso/, ciem/
│   │                         # Auth, SSO, cloud identity/entitlement analysis
│   ├── billing/             # Stripe integration
│   ├── multi_account/        # AWS Organizations discovery/rollup
│   ├── drift_detection/, realtime_detection/
│   └── tests/
├── src/                     # Next.js frontend (dashboards, onboarding, admin)
├── deploy/                  # systemd unit, env template
├── .github/workflows/       # CI, CodeQL, Docker publish
└── docker-compose.yml
```

## Running it locally

```bash
git clone https://github.com/maboudm/nexora.git
cd nexora

# backend
pip install -r requirements.txt
cp deploy/env.example .env   # fill in AWS/OAuth/Stripe test credentials
uvicorn backend.gateway.app:app --reload

# frontend
bun install    # or npm install
bun run dev    # or npm run dev
```

Full production setup (Docker, systemd, Caddy/HTTPS) is documented in `deploy/`.

## Testing

Test coverage is currently limited to a pipeline-level integration test (`backend/tests/test_pipeline.py`) that exercises the scan → score → attack-path flow end-to-end using [`moto`](https://github.com/getmoto/moto) to mock AWS. Unit tests for individual engines are the main gap and the next priority — flagged here rather than glossed over.

## Status & honest limitations

This is a solo project, not a production SaaS with paying customers. Specifically:
- Azure and GCP support is scaffolded in the code but AWS is the only cloud provider that's been exercised end-to-end.
- Test coverage is thin relative to the size of the codebase (see above).
- It hasn't been through a third-party security review, which matters for a tool that requests read-only IAM access to real cloud accounts.

The goal of building it was to get real experience with the backend and cloud-security concepts listed above, not to launch a company. Happy to walk through any part of the design in an interview.

## Security

See [SECURITY.md](SECURITY.md) for the threat model and how to report a vulnerability.

## License

Apache 2.0 — see [LICENSE](LICENSE).
