# Agicap — READY: Prod S3 buckets with anonymous LIST enabled (LOW/MED)

## Summary
Three PROD buckets under *.agicap.com allow anonymous bucket LIST (inventory) in
addition to serving content: attacker can enumerate every object name and download
any artifact without credentials.

## Evidence (PoC, read-only)
1. cm-dashboard-front-prod (hosted at dashboard-front-front.agicap.com)
   GET /?list-type=2 -> ListBucketResult, 365 keys under dashboards-app-prod/
   (prod dashboard web build, latest bundle 2023-07-12; multiple hashed builds retained)
2. common-maintenance-prod (prod-maintenance-bucket.agicap.com)
   GET /?list-type=2 -> 5 keys: v1/products.json, v1/product/{accounts-payable,
   cash-collection,cashflow,data-integration}.json (prod maintenance flags)
3. agc-translations (translations.agicap.com) — 1000 i18n keys (likely public by design)
4. core-flipper-assets-prod (flipper-assets.agicap.com) — 916 keys incl. internal
   deployment script assets_list_generator.sh

Objects are GET-able anonymously (e.g. dashboard JS bundles, maintenance JSONs).
Content scan of the largest bundle (3.1MB) found no secrets/credentials.

## Impact
- Attacker gains full object inventory + read access to prod frontend build artifacts
  and maintenance configuration, across multiple hash-named builds (deployment history).
- No credentials or customer data observed; impact limited to information disclosure
  of internal structure (bucket names, build history, internal tooling artifacts).

## Recommendation
Disable anonymous LIST (BlockPublicAccess / bucket policy deny s3:ListBucket), keep
only website/object serving if required.

## Note (NOT reported per policy reject list)
openapi.agicap.com returns 500 "key build strategy" rate-limiter error for
unauthenticated /api/* calls instead of 401 — rate-limit class is on their reject list.
