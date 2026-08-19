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
