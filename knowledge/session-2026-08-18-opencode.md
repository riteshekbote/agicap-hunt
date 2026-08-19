# Agicap — session draft 2026-08-18 (opencode live hunt, NOT yet submitted)

## Finding A (candidate, LOW-MED) — Open S3 bucket LISTING on prod buckets
- cm-dashboard-front-prod (dashboard-front-front.agicap.com): 365 objects = PROD dashboard
  builds (dashboards-app-prod/, latest 2023-07-12, stale but real prod artifacts)
- common-maintenance-prod (prod-maintenance-bucket.agicap.com): 5 maintenance-config JSONs
  (v1/products.json, v1/product/*.json) — app maintenance flags
- agc-translations (translations.agicap.com): 1000 i18n JSONs (public by design)
- core-flipper-assets-prod (flipper-assets.agicap.com): 916 assets incl. internal tool
  assets_list_generator.sh
- All: ListBucketResult XML on GET / = anonymous LIST + object GET (no keys needed)
- Content scan: no secrets in main chunk (521.f70e76b9a519cff6.js, 3.1MB) or index.html
- PoC: curl https://dashboard-front-front.agicap.com/ -> ListBucketResult with 365 keys

## Finding B (candidate, LOW/INFO) — openapi.agicap.com unauth -> 500 rate-limiter crash
- GET /api/companies (and all /api/*) without auth -> 500 problem+json
  detail: "The current request and the key build strategy can't be able to build a valid
  key to control rate limit" + traceId — auth bypass to controller, error-handling leak
- /swagger/v1/swagger.json = 918KB public spec (18 paths, 23 ops, global Bearer) — intended
  documentation gateway (their own config points OPENAPI_HOST here)
- Internal swaggers (di-business-core, di-business-file-import, di-banking-export):
  /internal/swagger -> 403 gated (good)

## Recon intel (from app.agicap.com/assets/config.json — public runtime config)
- identity.agicap.com (OIDC authority; currently 522 origin down), permissions.agicap.com
  (522), aggregator.agicap.com, di-business-core.agicap.com, transformationmatrix.agicap.com,
  business-definition.agicap.com, invoices-management.agicap.com, suppliers.agicap.com,
  payments.agicap.com (CF 429), pnl.agicap.com, openapi.agicap.com, di-* family
- SEGMENT_KEY=z2x1pAsy6PyJ9OmBBAnRTZWb0gxNDhk9 (Segment write key — public by design)
- NEVERBOUNCE_PUBLIC_KEY=public_9d65e0325565b6e3dd5b2db79437de6d (public by design)
- n8n-webhooks.agicap.com: / = {"status":"healthy"}, all else 401 {"detail":"Unauthorized"}
  (properly gated, no bypass with dummy headers)

## Closures
- app.agicap.com + api.agicap.com = SPA shells (Angular/React), all paths 200/302 HTML
- api.agicap.com assets bundle: API base from env (VITE_PORTAL_SERVER_URL) — no hardcode
- dashboard bucket main bundle: no secrets (only vendor libs: jsPDF/moment/quill)

## Next (HUMAN)
- Decide on Finding A (bucket listing) submission — subject "Vulnerability Report" to
  bugbounty@agicap.com (per scope.yml), or via their disclosed policy channel
- identity.agicap.com retry when origin recovers (522 now)

## JS deep-dive (all files checked)
- cm-dashboard-front-prod: all 141 JS chunks downloaded (34MB gz -> 164MB) + decompressed
  -> VERDICT: clean. API resolution 100% env-driven (VITE_*), only hardcoded host =
  flipper-assets.agicap.com; no secrets (1 false-positive AKIA), no endpoints; OIDC PKCE
  (oidc-client, oidc-silent-renew); 2023 build. 3rdpartylicenses.txt = standard Angular
  manifest; internal @agicap/* packages = private npm, org-scoped, NOT squattable.
- app.agicap.com live shell (12 chunks, 252KB): clean; only external = HubSpot forms.
- mfe.agicap.com: placeholder shell; all referenced assets 404 -> MFE hosting gated.
- config.json = 237 keys, complete topology: ~50 MFE routes (mfe.agicap.com/*), API
  hosts incl. NEW: client.agicap.com (client portal, SPA /en/), ai-assistant.agicap.com
  (FastAPI alive, docs disabled), account-manager-service, risk-management,
  debt-management (all 404-alive), prod-umami.agicap.cloud (Next.js page), DATEV OIDC
  login.datev.de, DEVELOPER_PORTAL_URL=api.agicap.com.
- Reportable findings UNCHANGED: only GCS anonymous LIST (Low).

## Session 2 (2026-08-19) — re-verification + new digs (1 rps, read-only)
- RE-VERIFIED Finding A: cm-dashboard-front-prod, core-flipper-assets-prod,
  common-maintenance-prod, agc-translations ALL still return HTTP 200 ListBucketResult
  (list-type=2, max-keys=1) on storage.googleapis.com TODAY. Finding still valid.
- NEW BUCKET: cm-dashboard-front-preprod -> HTTP 200 ListBucketResult, 536 objects,
  all under dashboards-app-dev/ (IsTruncated=false). Same anonymous-LIST pattern.
  Staging/preprod variants of other buckets = NoSuchBucket (404).
- delorean.agicap.com/v2/feature-flags = 439 flags, HTTP 200, unauthenticated, NO
  emails/secrets found; by-design public runtime config (FEATURE_FLAGS_ENDPOINT in
  config.json; browser fetches it w/o creds). Info-only, NOT separately reportable.
- app.agicap.com shell bundles (main-TV6GAO7K.js 114KB, polyfills-56NDLJYW.js) re-pulled:
  still env-driven (AGICAP_API_HOST/REBACCA_API_HOST/ASSISTANT_API_URL/AGGREGATOR_API_HOST),
  no hardcoded hosts/secrets. Relative paths seen: /api/bff-categories/v2,
  /api/core/v1/companies/{e}/connexion-info, /connect/token (OIDC). No new surface.
- openapi.agicap.com swagger.json unchanged: 18 paths / 23 ops (same list as before).
- New-host probes (1 GET each): internal-financing, smart-assistant (/docs,/openapi.json
  404), settings, my-swan-account, spend-back, collect-api (Express NotFoundException),
  forecasting, short-term-cash-management /api/bff (404), contract-management (000/EOF)
  -> no exposed docs, no unauth APIs. internal-financing /swagger -> 301 -> 403 (gated).
- graphql: api.agicap.com/graphql = SPA fallback HTML, openapi 404, app 302 -> no GraphQL.
- VERDICT: Finding A (GCS anonymous LIST) confirmed live; extend to cm-dashboard-front-preprod.
