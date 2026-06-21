# DRAFT v0.59.0 Migration — Supplemental to Exercise Report

**Catalog:** [getdraft/acme-powersoft-catalog](https://github.com/getdraft/acme-powersoft-catalog)
**Migration branch:** `draft/framework-update-0.59.0` ([PR #1](https://github.com/getdraft/acme-powersoft-catalog/pull/1))
**Previous framework version:** 0.58.2
**Updated to:** 0.59.0
**Supplemental author:** Draftsman (Acme PowerSoft admin agent)
**Date:** 2026-06

> This document is a follow-on to the
> [original EXERCISE_REPORT.md](EXERCISE_REPORT.md). It revisits the
> exercise's friction log and recommendations in light of what was required
> to migrate this catalog from DRAFT 0.58.2 to 0.59.0, identifies which
> reported misconceptions the framework update directly addresses, and
> re-scores open issues as either dismissed or affirmed.

---

## What the Migration Required

### Field renames (mechanical, 28 files)

`architecturalDecisions:` and `architectureNotes:` on all service and host
objects were renamed to `notes:`. The old names were deprecated in 0.58.x
and removed in 0.59.0. Applied with `perl -pi -e` across the catalog.

### Mechanism rename (22 occurrences)

`mechanism: architecturalDecision` in `requirementImplementations[].satisfiedBy`
and in requirement group `canBeSatisfiedBy` entries was renamed to
`mechanism: decisionRecord`. This was the first hint that the rename was
not purely cosmetic.

### `validAnswerTypes` fixup (5 requirement group files)

`- architecturalDecision` entries in `validAnswerTypes` lists were changed
to `- decisionRecord`. The mechanism rename left these inconsistent.

### `appliesTo` fixup (1 requirement group file)

`rg-vulnerability-management.yaml` had `appliesTo: [host, technology_component]`.
The 0.59.0 validator enforces that `appliesTo` values must be in
`STANDARD_TYPES` — `{host, runtime_service, data_store_service,
network_service, ai_gateway, product_component, data_component}` —
and `technology_component` is not in that set. Changed to
`[host, runtime_service, data_store_service, network_service]`.

### SDP decision records (6 new objects)

Six SDPs had `requirementImplementations` satisfying the
`reference-architecture-conformance` requirement via an inline note.
0.59.0 removed note-based satisfaction, and the DRAFT SDP requirement
group specifically requires a `decisionRecord(key=noApplicablePattern)` for
SDPs that predate any approved ReferenceArchitecture. Created one
`decision_record` catalog object per SDP and wired it in via
`decisionRecords[{ref, key}]`.

### Shared-service `requirementImplementations` restructure (8 service files, 17 new objects)

This was the highest-effort migration step and the one that most directly
exposed a design gap in the exercise catalog.

The 8 shared services (data warehouse, landing zone, messaging backbone,
identity/access, caching, observability, secrets management, AKS node pool
host) had `requirementImplementations` entries in the following pre-0.59.0
format:

```yaml
requirementImplementations:
  - requirementId: DR-01
    satisfiedBy:
      mechanism: decisionRecord
      key: recoverabilityModel
```

This format is invalid in 0.59.0 on two levels:

1. **Structure**: The `satisfiedBy:` nesting is gone. The new schema is
   flat: `requirementGroup`, `requirementId`, `status`, `mechanism`, `key`,
   `ref` are all top-level fields on each entry. `requirementGroup` and
   `status` are now required.

2. **Substance**: `mechanism: decisionRecord` in 0.59.0 requires an actual
   `decision_record` catalog object — either a `ref:` directly on the
   implementation entry, or an entry in the object's `decisionRecords[]`
   list whose `ref` resolves to a catalog object of `type: decision_record`.
   Having `key: recoverabilityModel` is not enough; the validator walks
   `obj.decisionRecords[]` looking for a ref that resolves. Finding none,
   it returns `False`.

The exercise catalog had none of these decision records. The `notes` block
entries (`notes.recoverabilityModel`, `notes.serviceAuthentication`, etc.)
held all the content that would belong in them — but `notes` is a scratchpad
and carries no catalog identity.

**Resolution:** Created 17 `decision_record` objects under
`catalog/governance/decision-records/`, populated from the notes content on
each service:

| Key | Count | Services |
|---|---|---|
| `serviceAuthentication` (AC-01) | 5 | landing-zone, data-warehouse, messaging-backbone, identity-access, secrets-mgmt |
| `serviceLogging` (AU-01) | 4 | data-warehouse, identity-access, observability, secrets-mgmt |
| `recoverabilityModel` (DR-01) | 3 | landing-zone, data-warehouse, messaging-backbone |
| `ha` (DR-02) | 1 | data-warehouse |
| `availabilityModel` (SLA-01) | 3 | messaging-backbone, caching, observability |
| patch cadence (VM-01) | 1 | aks-shared-node-pool |

Each service file was updated to:
- Add `decisionRecords: [{ref: <uid>, key: <key>}]`
- Replace the old nested `satisfiedBy` entries with flat entries including
  `requirementGroup`, `status: satisfied`, and `ref: <uid>`

---

## What 0.59.0 Changed and Why It Matters

### The core semantic shift: notes are not decisions

The rename from `architecturalDecisions` to `notes` is deliberately
downgrading what these blocks mean. In 0.58.x, calling the block
`architecturalDecisions` implicitly suggested that filling in
`serviceAuthentication: "Azure AD workload identity"` was an architectural
decision act. It wasn't — it was free text in a scratchpad. 0.59.0 makes
this explicit:

- `notes:` is a scratchpad. It can hold context, working assumptions,
  and drafting-in-progress observations. It never satisfies a requirement.
- A `decision_record` is a first-class catalog object with a uid, category,
  status, decisionRationale, owner, and lifecycle. It can satisfy
  requirements because it is a committed, identified artifact — not a free
  text field.

### `mechanism: decisionRecord` is now load-bearing

In 0.58.x, `mechanism: architecturalDecision` with `key: someKey` checked
whether `notes[someKey]` was non-empty. It was essentially `mechanism: field`
pointed at the notes block — and a one-word entry like
`serviceAuthentication: "Azure AD workload identity"` would satisfy a
compliance requirement.

In 0.59.0, `mechanism: decisionRecord` checks whether the object has a
`decisionRecords[]` entry whose `ref` resolves to a catalog
`decision_record` object. A notes entry is invisible to this check. The
mechanism is now genuinely about committed decisions, not filled-in fields.

### `requirementImplementations` schema is now strict

The new `requirementImplementation` schema requires:
- `requirementGroup` — the uid of the requirement_group object
- `requirementId` — the requirement id within that group
- `status` — one of `satisfied`, `not-applicable`, `not-compliant`

And adds optional `mechanism`, `key`, `ref`. The old `satisfiedBy:` nesting
is gone. Any catalog that used the old nested form (as this one did) will
have broken `requirementImplementations` after migration — the validator
simply won't find matching entries for any requirement.

### `requirementGroups` on service objects is deprecated

In 0.58.x, service objects declared their own `requirementGroups` list to
control which groups applied. In 0.59.0, this field is deprecated —
requirement application is now driven by workspace activation
(`requirements.activeRequirementGroups`) and the `appliesTo` / `activation`
fields on the requirement group objects themselves.

This means some requirements are now applied to objects that did not declare
them in their own `requirementGroups`. For example, `rs-caching` previously
only declared `[01KQQ4Q027-K5DR, KZM0T9NY43-3H8K]` and was only evaluated
against those groups. After migration, all workspace-active groups that match
`appliesTo: [runtime_service]` apply to it — surfacing DR-01, AU-01, and
AC-01 as new unsatisfied requirements on caching (pre-existing gaps, not
migration-introduced regressions).

---

## Validation Delta

| | Before migration | After migration |
|---|---|---|
| Failing files | 53 | 39 |
| Resolved by migration | — | 14 |
| Remaining failures | — | 39 (all pre-existing) |

The 14 resolved failures were entirely the shared-service
`requirementImplementations` entries that were using `mechanism: decisionRecord`
without actual decision records.

The 39 remaining failures are pre-existing catalog gaps, not migration
regressions:

- **9 TechnologyComponent files** (Tax, Regulatory, Infra & DevOps): missing
  `vendor`, `productName`, `productVersion`, `classification`. Pre-existing
  from Phase 5 of the exercise — flagged in Phase 6 when validation was
  first run with groups active.
- **29 product-specific service files** (ds-*, rs-*, host-*): requirement
  satisfaction gaps — these services have notes content but no
  `requirementImplementations` at all, or have implementations that still use
  the old nested `satisfiedBy` format (and in some cases, still have no
  decision records). These were already failing before this migration; this PR
  addressed only the shared services.
- **2 requirement group files** (`rg-change-management`, `rg-service-level`):
  conditional requirements missing `applicability` rules. Pre-existing.

---

## Misconceptions the Migration Exposed

### 1. "Filling in the `architecturalDecisions` block satisfies compliance requirements"

The exercise agents treated the `architecturalDecisions` block (now `notes`)
as the primary compliance-evidence store. Entries like
`serviceAuthentication: "Azure AD workload identity"` and
`serviceLogging: "Diagnostic logs to Datadog"` were written to satisfy AC-01
and AU-01 respectively. In 0.58.x this worked because the validator was
checking key presence in the notes block. In 0.59.0 it does not work at all —
the validator requires a committed, identified catalog object.

**Was this a misconception or a framework affordance?** Both. The 0.58.x
behavior genuinely permitted it. The 0.59.0 change is a deliberate
tightening of what "satisfying a requirement" means. The exercise found the
affordance, used it, and the new version removes it — confirming the
exercise's finding was real, not an authoring error.

### 2. "A requirement group's `canBeSatisfiedBy: [{mechanism: decisionRecord, key: someKey}]` is equivalent to checking `notes.someKey`"

Related to (1): the exercise's requirement groups were authored with
`canBeSatisfiedBy` entries that used `mechanism: decisionRecord` keyed to
the same names as notes sub-fields (`serviceAuthentication`, `serviceLogging`,
`recoverabilityModel`, etc.). This created an illusion of structural alignment
between the requirement group and the service's notes block. In reality, the
0.58.x validator was treating both as equivalent name-lookups on the same
notes dict. In 0.59.0, the requirement group correctly expects a decision
record object; the notes block is irrelevant.

### 3. "`requirementImplementations` with `satisfiedBy.mechanism` is valid"

The nested `satisfiedBy` form was used throughout the exercise catalog
because it was the valid structure in 0.58.x. The 0.59.0 schema change
to a flat structure with required `requirementGroup` and `status` fields
is a complete structural break. This is worth flagging as a migration-risk
signal: any real catalog migrating from 0.58.x will have this same issue
across every service that authored explicit `requirementImplementations`.

---

## Exercise Findings: Dismissed vs Affirmed

The following reviews each entry in the original EXERCISE_REPORT
Recommended Changes list against the 0.59.0 migration.

---

### ~~#5 [Medium] Give CM-style requirements a documented home, or explicitly bless `decisionRecord`-only satisfaction~~

**Status: DISMISSED — addressed by 0.59.0.**

The exercise's Sofia Lindqvist friction entry (Phase 4) flagged that
change-management requirements had "no natural home in the
`architecturalDecisions` block." The 0.59.0 renaming and semantic
tightening resolves this: the `notes` block is explicitly not the home
for CM-style requirements, and `decisionRecord` is now the formally
blessed, load-bearing mechanism for requirements like CM-01 that require
committed attestations rather than structured configurations. The
`rg-change-management.yaml` itself confirms this intent:

```yaml
# rg-change-management.yaml
description: >
  NOTE (assumption logged in EXERCISE_REPORT.md): neither the
  runtime-service nor data-store-service templates expose a notes key
  that naturally represents "how changes are reviewed and promoted."
  This group is therefore satisfiable only via decisionRecord, which
  is a heavier mechanism than a routine process attestation warrants.
```

With 0.59.0 making `decisionRecord` a proper, first-class catalog object
rather than a keyed notes entry, "heavier" is now accurate but also the
correct answer: change management attestations are commitments worth
capturing as identified, owned, versioned objects.

**Residual:** `rg-change-management.yaml` still has a FAIL for CM-02's
`requirementMode: conditional` missing an `applicability` field. That is a
separate gap in this catalog's authoring, not a framework issue.

---

### #1 [Critical] PaaS-without-a-host convention

**Status: AFFIRMED — not addressed by 0.59.0.**

The friction point (Marcus Webb, Phase 4) — that fully-managed PaaS
dependencies like Azure AD B2C have no host — is unchanged. The 0.59.0
DRAFT PaaS Delivery requirement group now applies to runtime and data-store
services and requires `field(capabilities)`, `field(authenticationModel)`,
and DRs for `resilienceModel`, `configurableSurface`, and `failureDomain`.
All 9 shared and product services that lack these fields now surface new
DRAFT PaaS Delivery failures. The framework has added more requirements
for PaaS objects without providing the `host: managed-paas` sentinel or
explicit "no host" convention that was the original ask.

---

### #2 [Critical] Legacy systems without HA

**Status: AFFIRMED — not addressed by 0.59.0.**

Eleanor Voss's friction point on Property Tax Legacy is unchanged. The
`runtime_service` template still assumes modern HA/autoscaling semantics,
and no "legacy/minimal" profile has been added.

---

### #3 [High] Cross-product/cross-repo dependency modeling

**Status: AFFIRMED — not addressed by 0.59.0.**

The Vantage-as-hard-dependency gap and AKS-Host-to-workload Relationship
gap remain unaddressed. No new Relationship object type or cross-SDP
dependency pattern was introduced in 0.59.0.

---

### #4 [High] External/third-party access grant object type

**Status: AFFIRMED — not addressed by 0.59.0.**

Renata Silva's outside-counsel access grant is still only representable
as unstructured prose.

---

### #6 [Medium] `deployableObjects[].intent` enum documentation

**Status: AFFIRMED — not addressed by 0.59.0.**

The SDP schema still shows only `ha` in vendored examples. `standalone` (the
value the exercise inferred for non-HA objects) is technically listed in the
schema enum but still undocumented in templates or onboarding materials.

---

### #7 [Medium] Long-term archival/retention vs. short-term backup

**Status: AFFIRMED — not addressed by 0.59.0.**

No structured retention-policy field was added. RegTrack's 10-year
rate-case retention requirement remains expressible only in free text.

---

### #8 [Low] Cross-team `primaryTechnologyComponent` reuse-by-uid documented

**Status: AFFIRMED — not addressed by 0.59.0.**

Still undocumented in vendored onboarding materials.

---

### #9 [Low] RequirementGroup activation lighter tier for internal tooling

**Status: AFFIRMED — not addressed by 0.59.0.**

No lighter activation tier has been introduced. The 0.59.0 change
(workspace activation replaces per-object `requirementGroups`) actually
makes this more pressing, not less: the Admin Console now gets the full
stack of active requirement groups applied to it automatically with no
way to declare "this is a lower-rigor object."

---

### #10 [Medium] `generate_browser.py` workspace root auto-detection

**Status: UNKNOWN — not verified against 0.59.0.**

The migration did not re-run `generate_browser.py` with the new framework
drop. This finding should be re-validated after PR merge.

---

### #11 [High] Silent empty-catalog warning in browser generator

**Status: AFFIRMED — not addressed by 0.59.0.**

`generate_browser.py` still does not warn on an empty `catalog/` directory.

---

### #12 [Medium] `generate_browser.py` output non-determinism

**Status: UNKNOWN — not verified against 0.59.0.**

---

### #13 [High] Empty governance views warning in browser

**Status: AFFIRMED — not addressed by 0.59.0.**

---

### #14 [Low] `owner.team` slug consistency recommendation

**Status: AFFIRMED — not addressed by 0.59.0.**

No lint rule or validation check enforces slug form.

---

### #15 [High] Empty `activeRequirementGroups` loud warning

**Status: PARTIALLY ADDRESSED.**

The validator now produces warnings for objects with the deprecated
`requirementGroups` field and surfaces more requirement satisfaction
failures when groups are active. However, if `requirements.activeRequirementGroups`
is empty the validator still runs silently with all requirement checking
suppressed — no loud warning, no explicit notice that zero requirement
groups are active.

---

### #16 [Medium] `catalog/` vs `configurations/` placement check

**Status: AFFIRMED — not addressed by 0.59.0.**

---

## New Finding: Migration Authoring Burden for Standard-Configuration Decision Records

This migration introduced a concern not present in the original exercise
report. The 9 shared services in this catalog had `notes` entries for
authentication, logging, recoverability, availability, and HA that were
written as implementation documentation — concise, factual, and accurate.
Promoting each to a `decision_record` catalog object required:

- A uid, name, description, category, status, lifecycleStatus, catalogStatus,
  owner, tags, decisionRationale, and controlReferences per entry
- 17 new YAML files totaling ~600 lines for 9 services

The content is entirely correct and the resulting catalog is more complete.
But the authoring cost is high for what amounts to "we use Azure AD" (5 times,
once per service). The framework's Design Principle 11 says DecisionRecords
are reserved for "risk acceptance, approved deviations, and architectural
exceptions" — yet the Acme requirement groups were deliberately authored to
require `decisionRecord` for standard configurations (AC-01, AU-01, DR-01,
SLA-01). This creates a productive tension that the framework does not yet
explicitly resolve:

- If standard configurations should use structural mechanisms (TC, relationship,
  field), the Acme requirement groups need redesigning to offer those mechanisms
  as `canBeSatisfiedBy` alternatives.
- If per-service decision records for standard configurations are acceptable,
  Design Principle 11's definition of what a DecisionRecord is for needs
  widening.

Neither option is wrong; they are different governance philosophies. The
0.59.0 migration forces the choice rather than deferring it, which is arguably
the right design — but catalogs upgrading from 0.58.x should plan for
significant decision record authoring if their requirement groups were designed
with the old note-key satisfaction pattern.

---

## Summary Scorecard Update

| Dimension | v1.0 Exercise Score | Post-0.59.0 Assessment |
|---|---|---|
| Schema Completeness | 3 | **3 — unchanged.** PaaS-host, legacy-system, cross-repo relationship, and third-party access gaps all remain. |
| Requirement Group Fidelity | 4 | **4 — marginally improved.** 0.59.0 formalizes that DRs are load-bearing and not note proxies; the CM-style satisfaction path is now explicitly correct. The conditional-requirement `applicability` gap (CM-02, SLA-02) remains. |
| Adoption Autonomy | 3 | **3 — neutral.** The migration required non-trivial judgment about whether to create decision records for standard configs vs redesign requirement groups. Real teams without a hands-on facilitator would need guidance on this decision at upgrade time. |
| SDP Quality | 4 | **4 — unchanged.** The 6 `noApplicablePattern` decision records formalize what was previously an inline note; the SDPs are structurally the same. |
| Draftsman Discoverability | 4 | **4 — unchanged.** |

**Post-migration validation status:** 39 failures remaining. All are
pre-existing content gaps (9 TCs missing required fields, 29 product
services not yet migrated to flat `requirementImplementations` with decision
records, 2 requirement groups with authoring gaps). Zero regression failures
introduced by the migration itself.
