# Agicap hunt KB — verified learnings (RAG for all models)
> Rules: never propose a class on the AGICAP REJECT LIST (see scope.yml); never
> duplicate KNOWN-DUP; use ALIVE surface facts. Agicap policy explicitly rejects
> CORS/headers/TLS/DNS/rate-limit/self-XSS/low-value-CSRF/MiTM/non-exploitable.
> What pays: exploitable IDOR/auth-bypass/business-logic/SSRF/XXE/SQLi/RCE/XSS.

## REJECTED CLASSES (Agicap policy — do not propose)
- REJECTED CORS configuration / missing security headers @ *: explicitly listed as
  "non-compliance with best practices" in Agicap policy. Do NOT spend cycles.
- REJECTED rate limit @ *: "Vulnerabilities related to rate limit" explicitly excluded.
- REJECTED login/logout/unauthenticated/low-value CSRF @ *: explicitly excluded.
- REJECTED Self-XSS @ *: explicitly excluded.
- REJECTED TLS config, DNS config, MiTM, network DoS, non-exploitable: explicitly excluded.

## ALIVE SURFACE FACTS (verified)
- 2026-08-16 agicap.com marketing site = Gatsby 4.25.6 SPA (generator tag in HTML),
  served via Contentful (images/assets.ctfassets.net preconnect). GTM GTM-MVH3XQ5.
  robots.txt: sitemap_index.xml + per-locale sitemaps (FR/EN/ES/DE/IT). security.txt:
  Contact: mailto:bugbounty@agicap.com, Expires 2026-12-31, Policy /en/bug-bounty/.
- 2026-08-16 (setup seed, UNVERIFIED) candidate subdomain list seeded in recon/scope.txt
  (app/api/staging/auth/graphql/banking/etc) — to be confirmed by recon pipeline.

## OPEN QUESTIONS
- Where is the real product surface? (app.agicap.com is a fintech SaaS: cash-flow mgmt,
  bank aggregation, invoicing, forecasting). Need to enumerate actual subdomains.
- Which auth provider / API style (REST/GraphQL)? (to be determined by probe phase)
- Mobile app endpoints? (Agicap ships iOS/Android apps)

## FINDING INBOX (validated = move to reports/)
- (empty)


---
## TRIAGE CLOSE-OUT 2026-08-22: GCS LIST finding KILLED — do not submit
Policy check (agicap.com/en/bug-bounty) excludes 'non exploitable vulnerability' +
'best-practices non-compliance'. Decisive evidence: assets_list_generator.sh = trivial
ls-to-JSON script; all objects are public frontend assets already served to every visitor.
Attacker gains: filename list only. N/A probability high; submission effort unjustified.
Buckets re-verified live 2026-08-22 (core-flipper-assets-prod, common-maintenance-prod 200).
