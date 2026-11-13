# REVIEW PACKAGE v2 — HQ country autofill on deals-won

> **This is a REVIEW PACKAGE only.**  
> It is **not** a PR on `Synthesia-Technologies/salesforce`.  
> It is **not** in production.  
> Repo name is deliberate: `REVIEW-PACKAGE-v2-hq-country-autofill`.

**Supersedes:** [v1 review package](https://github.com/NateKocho-synth/hq-country-autofill-deals-won) (enrichment stamp only — no picklist normalize, no child-account fallback).

**Target:** https://github.com/Synthesia-Technologies/salesforce  
**Intended branch:** `hq-country-autofill-deals-won` → `main`  
**Flow:** `[David][Order] Deals Won Slack in #Sales` (`Nikola_Order_Activate_Deals_Won_Slack`)  
**Slack:** https://synthesia.slack.com/archives/C03M9CSNCCS/p1794515430.535159

## What changed vs v1

1. **Country picklist normalize** — before writing parent Billing Country, map ZoomInfo-style labels to Salesforce state/country picklist values (same mappings Antony DW / David inaccurate-country fix use for the high-volume failures Matt hit in UAT):
   - South Korea / Korea, South → `Korea, Republic of`
   - Vietnam → `Viet Nam`
   - Iran → `Iran, Islamic Republic of`
   - Russia → `Russian Federation`
   - Taiwan → `Chinese Taipei`
   - Other values pass through unchanged (US/UK/Germany already match)
2. **Opportunity Account fallback** — if parent ZoomInfo Country and Company Country are both blank, stamp from the Opportunity (contract) Account Billing Country when that is set.
3. Auto-filled Slack note includes **raw** and **stamped** values so normalize is auditable.

## Priority when parent Billing Country is blank

1. Parent ZoomInfo Country (normalized)  
2. Else parent Company Country (normalized)  
3. Else Opportunity Account Billing Country (already a picklist value)  
4. Else Missing HQ Country alert (existing behaviour)

If the Account update still faults (unknown label), Missing HQ Country still fires.

## Files

| File | What |
|------|------|
| `change.diff` | Unified diff vs `origin/main` |
| `hq-country-autofill.patch` | `git format-patch` (all commits) |
| `Nikola_Order_Activate_Deals_Won_Slack.flow-meta.xml` | Full updated flow |

## Land as a real PR (needs Write on salesforce)

```bash
cd /path/to/salesforce
git fetch origin
git checkout -b hq-country-autofill-deals-won origin/main
git am path/to/hq-country-autofill.patch
git push -u origin HEAD
gh pr create --base main --title "fix(flow): auto-fill parent HQ country on deals-won"
```

Until that merges, **production is unchanged**.
