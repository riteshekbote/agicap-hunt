# agicap-hunt

24/7 multi-model bug-hunting automation for the **Agicap Bug Bounty** program.

- **Scope**: `agicap.com` + all subdomains (fintech SaaS: cash-flow management, bank aggregation, invoicing, forecasting)
- **Disclosure**: email to `bugbounty@agicap.com` (policy: https://agicap.com/en/bug-bounty/)
- 5 opencode models (Big Pickle, Nemotron 3 Ultra, Longcat, Ling 3.0, Laguna) hunt in parallel every 10 minutes
- Subdomain recon pipeline (subfinder + crt.sh + wayback + dnsx + httpx) daily at 02:20 UTC
- JS recon pipeline (endpoint/sourcemap/secret extraction from live app bundles) every 5 minutes
- All testing **read-only / non-destructive** — Agicap policy forbids data modification, DoS, and pre-disclosure without consent

## Agicap policy — what is NOT reportable (per their bug-bounty page)
CORS config, missing security headers, TLS weakness, DNS config, rate limit, self-XSS, low-value CSRF, MiTM, non-exploitable, automated-output-only reports. See `scope.yml` for the full rejected list — the analyst models are instructed to never spend cycles on these.

## What pays
Exploitable findings with real PoC: IDOR / broken access control on accounts, cash flow, invoices, banking; auth bypass; business logic; SSRF/XXE/SQLi/RCE; stored XSS with user impact.

| Artifact | Purpose |
|---|---|
| `recon/scope.txt` | Seed subdomain list |
| `inventory/` | Recon + JS inventory results |
| `leads/` | Candidate findings (UNVALIDATED) |
| `reports/` | Ranked hypotheses + valid findings |
| `knowledge/index.md` | Verified learnings + rejected classes (RAG) |
| `findings.md` | JS recon output |
| `scope.yml` | Program scope + rules (edit to adjust) |

## Reporting
Submit via email to `bugbounty@agicap.com` with: affected URL, vulnerability type, impact, reproduction steps, working PoC, and a fix recommendation. Payments are discretionary, bank transfer only, invoice required.
