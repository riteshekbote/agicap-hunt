# Agicap — READY: Production GCS buckets anonymous LIST (LOW, verified root cause)

## Finding
Production Google Cloud Storage buckets under *.agicap.com expose anonymous bucket
LIST (storage.objects.list granted to allUsers): anyone can enumerate the full object
inventory and download every artifact without credentials.

## Affected
- gs://cm-dashboard-front-prod        (365 objects — prod dashboard builds, hash-named
                                       chunks, multiple build generations)
- gs://core-flipper-assets-prod       (916 objects — incl. internal tooling scripts)
- gs://common-maintenance-prod        (5 objects — prod maintenance config)
- gs://agc-translations               (1000+ objects — i18n)
- gs://cm-dashboard-front-preprod     (536 objects — dashboards-app-dev/ dev builds,
                                       RE-VERIFIED listable 2026-08-19)
(Served publicly at dashboard-front-front / flipper-assets / prod-maintenance-bucket /
translations .agicap.com — GCS UploadServer, x-guploader-uploadid header)

## PoC (no credentials, GET only)
    curl "https://storage.googleapis.com/cm-dashboard-front-prod/?list-type=2"
    -> 200, full <ListBucketResult> (all 365 keys)
    curl "https://storage.googleapis.com/core-flipper-assets-prod/?list-type=2"
    -> 200, all 916 keys
    curl "https://storage.googleapis.com/cm-dashboard-front-preprod/?list-type=2"
    -> 200, all 536 keys (dashboards-app-dev/)
Objects anonymously GET-able (verified: dashboard JS bundles, v1/products.json,
assets_list_generator.sh).

## Re-verification 2026-08-19 (all four original buckets still live, HTTP 200):
    cm-dashboard-front-prod, core-flipper-assets-prod, common-maintenance-prod,
    agc-translations -> each returned <ListBucketResult> with KeyCount=1 today.
    Staging variants (cm-dashboard-front-staging etc.) -> NoSuchBucket (404), so the
    env-suffix pattern does NOT widen arbitrarily; preprod is the only extra hit.

## Escalation checked — NEGATIVE (important for triage)
Anonymous PUT probe (unique self-cleaning key, text/plain):
    -> 403 AccessDenied (S3-compat XML error). No anonymous write. No object left
       behind. Cannot deface/poison assets.

## Impact
- Full object inventory + read of prod build artifacts (deployment history via
  multi-generation hashed chunks), internal tooling scripts, prod maintenance config.
- Content is non-sensitive frontend assets; no customer data, no secrets, no write
  access -> LOW severity. Configuration hygiene + minor information disclosure.

## Fix
Remove allUsers/objects.list (keep objects.get only on buckets that must serve
public assets), or serve via GCS backend with signed/auth paths.

## Session trail (read-only, 1 rps, no data modification; write probe self-cleaning)
Full probe log in knowledge/session-2026-08-18-opencode.md

## 2026-08-19 object-inventory grep result (triager pass)
- No sensitive objects in any of the 5 buckets: config.json (prod+preprod) = public
  runtime config, USERFLOW_API_KEY EMPTY; common-maintenance-prod = 5 maintenance JSONs;
  rest = icons/translations. GCS finding = Low/Info framing (no secrets found), N/A risk.
