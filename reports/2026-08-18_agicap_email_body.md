Subject: Vulnerability Report — Anonymous LIST on production GCS buckets

Hi Agicap security team,

Read-only finding on your production GCS storage (per your bug-bounty policy).

**Summary:** Production GCS buckets expose anonymous bucket LIST (allUsers can
enumerate/download every object, no credentials).

**Affected buckets:**
- cm-dashboard-front-prod (365 objects — prod dashboard builds)
- core-flipper-assets-prod (916 objects — incl. internal tooling scripts)
- common-maintenance-prod (5 objects — prod maintenance config)
- agc-translations (1000+ objects)

**Proof (GET only, no auth):**
curl https://storage.googleapis.com/cm-dashboard-front-prod/?list-type=2  -> 200
curl https://storage.googleapis.com/core-flipper-assets-prod/?list-type=2 -> 200
Objects are anonymously downloadable (verified on JS bundles + maintenance JSONs).

**Escalation checked (negative):** anonymous PUT is denied (403 AccessDenied,
verified with a self-cleaning probe — nothing was left behind). No defacement/
poisoning possible.

**Impact:** object inventory + artifact disclosure (deployment history, internal
tooling); content is non-sensitive. Suggest removing allUsers list permission while
keeping public GET where web-serving is intended.

Happy to provide full logs. Best,
