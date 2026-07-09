
📋 Nexora — Project Overview (For Employers)
What is Nexora?
Nexora is a Cloud Security Platform (CNAPP) that helps companies find and fix security vulnerabilities in their cloud infrastructure (AWS, Azure, GCP). Think of it as a "security guard" that continuously monitors your cloud accounts and tells you exactly what's wrong and what to fix first.

It competes with enterprise tools like Wiz (valued at $32 billion), Prisma Cloud by Palo Alto Networks, and Orca Security — but at 1% of their cost.

The Problem It Solves
Companies using AWS/Azure/GCP face:

500+ security misconfigurations on average per cloud account
No idea which ones actually matter (not all vulnerabilities are equal)
Compliance requirements (CIS, PCI-DSS, HIPAA, SOC2, ISO 27001, GDPR)
Manual tracking in spreadsheets (error-prone, time-consuming)
Enterprise tools cost $30,000–$100,000/year (too expensive for most companies)
Nexora solves this by:

Scanning 2,064 security checks across AWS, Azure, and GCP
Telling you which finding to fix first (using Bayesian risk scoring)
Showing how attackers could chain vulnerabilities to reach your critical data
Providing auditor-ready evidence (hash-chained, tamper-proof)
Auto-creating tickets in Jira/Slack when critical issues are found
Costing only $5/month instead of $30,000/year
How It Works (Technical)
1. User Connects Their Cloud Account
text

User logs in → Adds AWS account (IAM role ARN) → Nexora scans via API
Nexora uses read-only IAM roles (SecurityAudit + ReadOnlyAccess) — it never makes changes to the user's cloud.

2. Scan Execution (2,064 checks)
Nexora runs three scanning engines in parallel:

Engine
What it checks
Count
Prowler (open-source)	Cloud misconfigurations (IAM, S3, EC2, RDS, etc.)	915
Checkov (open-source)	Infrastructure-as-Code (Terraform, K8s, Dockerfile)	1,149
Native CIS Rules	CIS AWS Benchmark v3.0 (custom DSL parser)	58
Total		2,064

3. Premium Analysis (6 engines)
After scanning, Nexora runs 6 analysis engines on the findings:

Engine 1: Drift Detection
Compares current scan vs previous scan
Detects what changed (field-level diff)
Risk-weights each change (e.g., "S3 bucket made public" = critical, "tag changed" = low)
Uses RFC 8785 JSON Canonicalization for deterministic hashing
Engine 2: Risk Prioritization (Bayesian)
Not all findings matter equally — some are critical, some are noise
Formula: Risk = CVSS × EPSS × Exposure × Asset_Criticality × Threat_Intel
CVSS: How severe is the vulnerability (0-10)
EPSS: How likely is it to be exploited (0-1, from FIRST.org)
Exposure: Is the asset internet-facing? (1.0) Internal? (0.6) Isolated? (0.2)
Asset Criticality: Is this a production database? (1.0) Or a dev sandbox? (0.15)
Threat Intel: Is there active exploitation in the wild? (1.5x boost)
Then it builds a fix plan using a greedy knapsack algorithm:

"You have 15 engineer-hours this week. Here are the 9 findings to fix that will reduce your risk by 376 points."
Engine 3: Attack Path Analysis (MITRE ATT&CK)
Builds a directed graph of all cloud assets
Finds multi-hop attack paths from internet → crown jewels
Each hop mapped to MITRE ATT&CK tactic:
T1190 (Initial Access) — "Attacker exploits open port"
T1078.004 (Lateral Movement) — "Uses stolen credentials to assume role"
T1552.001 (Credential Access) — "Finds exposed secret"
Computes path probability: P(reach target) = product of hop probabilities
Path risk = probability × target value
Example output: "Attacker can reach your production database (P=61.4%) via: Internet → Web Server → App Server → Database"

Engine 4: Evidence Vault (Hash-Chained)
Every scan finding is stored as cryptographic evidence
Uses Bitcoin-style hash chain: record_hash = SHA256(prev_hash + payload)
Each record signed with HMAC-SHA256 (per-tenant key)
Tamper-evident: if anyone modifies a record, the chain breaks
Auditors can verify: "Yes, control X was passing on date Y"
12-month retention (aligned with SOC2 observation period)
Engine 5: Workflow Integration
When critical findings are detected, auto-creates tickets:
Jira issue (with severity, description, remediation steps)
Slack alert (with rich formatting)
ServiceNow incident
PagerDuty alert
GitHub issue
Bidirectional sync: when ticket is closed, finding is marked resolved
HMAC-SHA256 webhook signatures (prevents forged tickets)
Engine 6: Multi-Account Discovery
Connect AWS Organizations management account
Auto-discovers all member accounts (via ListAccounts API)
Scans each account independently
Weighted posture rollup: org_score = Σ(account_score × resources × criticality) / Σ(weights)
Identifies weakest accounts: "production-app account has 47 findings (12 critical)"
Architecture (30 Backend Engines)
text

Nexora/
├── Frontend (Next.js 16, TypeScript)
│   ├── 20 dashboard tabs
│   ├── AWS onboarding wizard
│   ├── Billing/pricing page
│   └── Landing page (SEO-optimized)
│
├── Gateway (FastAPI, 178 routes)
│   ├── JWT authentication (HS256, 24h)
│   ├── Multi-tenant isolation
│   ├── RBAC (4 roles)
│   ├── Rate limiting (300 req/min)
│   └── Hash-chained audit log
│
├── Scanning Layer
│   ├── Prowler integration (915 checks)
│   ├── Checkov integration (1,149 policies)
│   ├── Native CIS rules (58 rules + DSL parser)
│   ├── Real AWS scanner (boto3: IAM, S3, EC2, RDS, EKS, Lambda, CloudTrail, KMS, EBS, VPC)
│   ├── Azure scanner (azure-mgmt SDK)
│   └── GCP scanner (google-cloud SDK)
│
├── Premium Engines (6)
│   ├── Drift detection (RFC 8785)
│   ├── Risk prioritization (Bayesian + knapsack)
│   ├── Attack paths (MITRE ATT&CK + NetworkX)
│   ├── Evidence vault (SHA256 + HMAC-SHA256)
│   ├── Workflow integration (8 providers)
│   └── Multi-account discovery (AWS Organizations)
│
├── Business Layer
│   ├── Billing (3 tiers, Stripe Checkout)
│   ├── SSO (Google OAuth + SAML 2.0)
│   ├── Scheduler (daily/weekly scans + email alerts)
│   ├── Compliance (7 frameworks)
│   ├── CIEM (IAM privilege escalation detection)
│   ├── Vulnerability management (CVSS v3.1 + NVD CVE)
│   ├── Threat intelligence (IOC matching + APT profiles)
│   ├── Container security (CIS Docker Benchmark)
│   ├── KSPM (CIS Kubernetes Benchmark)
│   ├── IaC security (Terraform HCL scanner)
│   ├── CWPP (runtime threat detection)
│   ├── Auto-remediation (3-tier safety model)
│   ├── Policy engine (Rego-like DSL)
│   ├── Posture scoring (6-factor KPI)
│   ├── Reporting (PDF + Excel + JSON)
│   └── AI copilot (RAG-based)
│
└── Infrastructure
    ├── SQLite (WAL mode, 25+ tables)
    ├── Docker (multi-stage build)
    ├── systemd service
    ├── Caddy reverse proxy (auto-HTTPS)
    └── GitHub Actions CI/CD
Technical Highlights (For Engineering Interviewers)
Cryptography
SHA256 hash chains (Bitcoin-style evidence vault)
HMAC-SHA256 webhook signatures (Stripe + workflow integrations)
JWT HS256 with 24h expiry
bcrypt password hashing
Per-tenant signing keys
Algorithms
Bayesian Belief Networks (attack probability)
Greedy 0/1 knapsack (fix-plan optimizer)
DFS with cycle detection (attack path finding)
Betweenness centrality (choke point identification)
Shannon entropy (secret detection)
System Design
Multi-tenant SaaS (tenant_id always from JWT, never from client)
Event-driven architecture (pub/sub with outbox pattern)
Saga orchestrator (distributed transactions)
Lazy loading (per-tenant data loaded on first access)
Hybrid storage (SQLite for persistence + in-memory cache for speed)
Integration Patterns
Absorbed Prowler as Python dependency (not subprocess)
Absorbed Checkov as Python dependency
Stripe Checkout (redirect-based, no card data on our servers)
Google OAuth 2.0 (authorization code flow)
SAML 2.0 (SP-initiated, with ACS endpoint)
HMAC webhook verification (constant-time comparison)
Verified With Real Tests
Test
What was tested
Result
Login	JWT token generation + verification	✅ 375-char token, 24h expiry
Checkov scan	Real IaC scan on vulnerable Terraform	✅ 21 findings (5 critical, 2 high)
CIS scan	Real CIS rules on cloud assets	✅ 20 findings (4 critical, 5 high)
Drift detection	Two scans compared	✅ 5 new drift events detected
Evidence vault	Hash chain integrity	✅ 64 records, chain verified = True
Risk prioritization	Bayesian scoring + fix plan	✅ 20 findings ranked, 376 risk reduced
Stripe webhook	HMAC-SHA256 signature	✅ Valid accepted, invalid rejected
Load test	10 concurrent users	✅ 210/210 requests, 100% success
Prowler catalog	915 checks loaded	✅ AWS 617, Azure 190, GCP 109
Checkov catalog	1,149 policies loaded	✅ Terraform 941, K8s 113, etc.

What Makes This Project Special
1. Engineering Decisions
Didn't reinvent the wheel: Integrated Prowler (915 checks) and Checkov (1,149) instead of writing 2,064 rules from scratch. This saved 6+ months of development.
Layered architecture: Scanning layer (Prowler/Checkov) is decoupled from analysis layer (premium engines). Can swap scanners without touching analysis.
Production-ready: Docker, systemd, CI/CD, health checks, audit logging, rate limiting — not just a prototype.
2. Security Mindset
Multi-tenant isolation (tenant_id from JWT, never from client)
Hash-chained audit log (tamper-evident)
HMAC webhook signatures (prevent forged requests)
Read-only cloud scanning (never modifies user's infrastructure)
Rate limiting (prevent abuse)
3. Business Understanding
3-tier pricing (Free → $5 Premium → Enterprise)
Stripe integration (real revenue potential)
SSO (Google OAuth + SAML) — what mid-market companies require
Scheduled scans + email alerts (automation saves time)
Compliance evidence (what auditors need)
4. Scale of Work
57,000+ lines of code (backend Python + frontend TypeScript)
30 backend engines (each a separate module with clear responsibility)
178 API routes (RESTful, documented via OpenAPI/Swagger)
20 frontend tabs (full dashboard UI)
7 compliance frameworks implemented
Tech Stack Summary
Category
Technologies
Backend	Python 3.12, FastAPI, Pydantic, JWT, bcrypt, SQLite
Frontend	Next.js 16, TypeScript, Tailwind CSS, Recharts
Cloud Security	Prowler 5.33, Checkov 3.3, boto3, azure-mgmt, google-cloud
Graph Algorithms	NetworkX (attack path analysis)
Cryptography	SHA256, HMAC-SHA256, JWT HS256
Payments	Stripe Checkout + webhooks
Auth	JWT, Google OAuth 2.0, SAML 2.0
Infrastructure	Docker, systemd, Caddy, GitHub Actions
Database	SQLite (WAL mode, 25+ tables)
Reporting	ReportLab (PDF), openpyxl (Excel)
License	Apache 2.0

Creator
Maboud Bakhshi Nezhad

GitHub: @maboudm
Role: Full-stack developer (backend + frontend + DevOps + security)
Project: Nexora CNAPP — designed, built, tested, and documented from scratch
