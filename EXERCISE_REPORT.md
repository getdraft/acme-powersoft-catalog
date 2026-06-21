# DRAFT v1.0 Adoption Exercise — Final Report

**Company:** Acme PowerSoft (fictional, modeled on the shape of a mid-size
regulated-utility software vendor — Powerplan/Roper Technologies served
as the research basis for the products, org structure, and technology mix)
**Framework under test:** [getdraft/draftsman](https://github.com/getdraft/draftsman)
**Exercise window:** 2026-06 (single continuous session)
**Author:** Priya Natarajan (Admin agent), compiling on behalf of all
participating agents

---

## Executive Summary

DRAFT's object model held up well end-to-end: six product teams with
genuinely different shapes — a clean cloud-native API, a legacy on-premise
PowerBuilder/Oracle system, a scheduled batch/ELT pipeline, an internal
admin tool, and a managed-PaaS reporting capacity — were all represented
without inventing new object types, and the resulting catalog (7
repositories, 1 RequirementGroup baseline, 9 shared-service objects, 19
product-specific objects, 6 SoftwareDeploymentPatterns) tells a coherent,
traceable adoption story end-to-end through commit history. It is not yet
ready to call v1.0. The single most important thing to fix first is the
**lack of any convention for objects that don't cleanly fit "infrastructure
I own" (TechnologyComponent/Host) or "code I deploy with HA/scaling
semantics" (RuntimeService/DataStoreService/NetworkService)** — fully
managed PaaS components with no host, legacy single-instance systems with
no scaling story, and external-party access grants all had to be
shoehorned into the nearest available type with an honesty-over-conformance
workaround, and a v1.0 framework being rolled out company-wide will hit
this on day one, every time, across nearly every org.

---

## Assumption Log

| Domain | Assumption | Confidence | Source Signal |
|---|---|---|---|
| Company identity | Acme PowerSoft is a fictionalized stand-in for a Powerplan/Roper-style vertical-market software vendor serving regulated utilities (tax, asset management, regulatory compliance) | High | Explicit exercise instruction (Phase 2) |
| Cloud provider | All modern infrastructure runs on Microsoft Azure (AKS, Azure SQL, Azure AD B2C, Key Vault, Service Bus, APIM, ADLS Gen2) | Medium | Companies in this vertical (utility/regulated enterprise software) skew Microsoft-stack; chosen for internal consistency across 6 products rather than confirmed externally |
| Primary application runtime | .NET 8 / ASP.NET Core is the default stack for new services; legacy systems are PowerBuilder | Medium | Roper-family vertical-market software has a well-documented history of .NET and PowerBuilder/4GL legacy systems; used as the basis for Acme Tax Suite's split |
| Data platform | Snowflake is the analytics warehouse; ADLS Gen2 is the landing zone | Medium | Common modern pairing; chosen to give the Data & Analytics SDP a distinct shape from the transactional products rather than confirmed against a real Powerplan stack |
| Org structure | 6 engineering teams (Core Platform, Tax & Compliance, Asset & Lease, Regulatory, Data & Analytics, Infra & DevOps) map 1:1 to 6 product lines plus one platform team | High | Explicit exercise design (Phase 2) |
| Shared-service ownership | Core Platform, Infra & DevOps, and Data & Analytics ended up owning all 7 shared services; Tax, Asset & Lease, and Regulatory own none | Medium | Emerged organically from which team's product repo most plausibly originated each shared dependency (identity from the platform/entitlement team, observability/secrets from the platform-engineering team, the warehouse from the data team) — not dictated upfront |
| Compliance baseline | Acme's internal controls are SOC2-aligned, expressed as a 10-control set (AC, DR, CM, VM, SLA, AU) | Medium | SOC2 is the de facto baseline for B2B SaaS vendors selling into enterprise/utility customers; specific control IDs (AC-01, DR-01, etc.) are invented for this exercise |
| Regulatory retention requirement | Rate-case evidentiary records require 10-year retention | Medium | Typical state utility-commission rate-case recordkeeping requirements; not sourced from a specific jurisdiction |
| Ticket numbers (VANT-340, TAX-1142, DATA-212, etc.) | All fictional, invented inline as each friction point was discovered, to make gaps feel like real backlog items rather than abstract notes | Low (by design) | Exercise convention — these do not reference any real issue tracker |
| Legacy system scope | Only one product (Acme Tax Suite / Property Tax) carries a true legacy on-premise component; all others are fully modern | Medium | Chosen to create exactly one hard schema-fit test case rather than diluting the signal across multiple legacy systems |
| Internal tool bar | The Admin Console is treated as lower-availability/lower-rigor than customer-facing products (99% vs 99.5–99.9%) | Medium | Reasonable real-world practice for internal tooling; not dictated by DRAFT itself |

---

## Agent Disposition Log

| Agent | Role | Disposition | Effect on exercise outcome |
|---|---|---|---|
| Priya Natarajan | Admin | — (facilitator) | Authored the RequirementGroup baseline and answered every cross-team question raised by other agents (TechnologyComponent reuse pattern, legacy-system object-type fit, Service Bus scoping, internal-tool rigor) within the same session — no question went unanswered, keeping the exercise unblocked throughout. |
| Marcus Webb | Core Platform lead / domain owner (Identity, API Gateway, Redis) | Enthusiastic | Validated Priya's shared-services draft with zero blocking questions and unilaterally added the Redis entitlement cache that was missing from it — the clearest example of "enthusiastic adoption surfaces gaps the Admin didn't know existed," but also produced zero friction-log entries of his own, which somewhat limited stress-testing from this persona. |
| Sofia Lindqvist | Infra & DevOps lead / domain owner (Messaging, Observability, Secrets, AKS Host) | Informed | Reviewed every draft carefully, explicitly declined to unilaterally satisfy requirements that needed a DecisionRecord (deferring to Phase 5/governance rather than overstepping), and asked a real, recorded question about RequirementGroup rigor for internal tooling before proceeding — the disposition that produced the most measured, audit-trail-friendly commit history. |
| Jamal Ibrahim | Data & Analytics lead / domain owner (Snowflake, ADLS landing zone) | Enthusiastic | Promoted the ADLS landing zone out of his own product repo into the shared catalog unprompted, and was the agent who hit the most structurally different workload (scheduled batch via Airflow) — pushed through the runtime_service template's request-driven assumptions without raising it as a blocker, but did flag the single-scheduler HA gap (DATA-212) as a direct byproduct of answering honestly. |
| Eleanor Voss | Tax & Compliance lead | Skeptical | Explicitly registered a concern *before* contributing (whether runtime_service is even the right type for a legacy on-prem system), waited for the Admin's ruling, then proceeded — produced the exercise's highest-value friction-log entries (legacy-system type fit, the "n/a is a valid answer" precedent, two new real-feeling tickets) precisely because she refused to silently round up a compliant-looking but inaccurate record. |
| Renata Silva | Regulatory lead | Skeptical | Did not wait for a ruling before proceeding (unlike Eleanor) but flagged two concrete schema gaps in the same commit she used to work around them (10-year retention with no structured field, external-party access grants with no object type) — a slightly different flavor of skepticism: "I'll keep moving, but I'm writing this down." |
| Derek Owusu | Asset & Lease lead | Informed | Asked two scoped, practical questions (is cross-team TechnologyComponent reuse intended? should this Service Bus topic be dedicated or shared?), got Priya's answers, then executed cleanly — the most "by-the-book" onboarding of any team, useful as the control case against which the others' friction stands out. |

**Methodology note:** Phase 4 domain ownership of the 7 shared services
ended up concentrated in the three Enthusiastic/Informed personas (Marcus,
Sofia, Jamal) purely as a byproduct of which product each shared
dependency most plausibly originated from — neither Skeptical-disposition
lead (Eleanor, Renata) happened to own a shared service. As a result, no
Skeptical pushback occurred during Phase 4's shared-service drafting;
both Skeptical agents' pushback is concentrated in Phase 5, against their
own product's catalog entries instead. This is a reasonable real-world
outcome (skepticism doesn't appear on a schedule) but means the exercise
did not specifically test "a skeptical domain owner disputes a shared
service they don't own" — worth noting if this exercise is re-run with
different ownership assignments.

---

## Friction Log

| Phase | Agent | Friction Point | Resolution | Schema Gap? |
|---|---|---|---|---|
| 3 | Admin (Priya) | GitHub fine-grained PAT lacked the `workflow` scope, rejecting any push touching `.github/workflows/*.yml` | Standardized on `ci/` directory for pipeline definitions across all 7 repos, documented in-repo and in commit messages (tracked as DEVOPS-204) | N |
| 4 | Marcus Webb | `runtime-service`/`data-store-service` templates have no documented convention for `host` on a fully-managed PaaS dependency (e.g. Azure AD B2C has no host at all) | Set `host: null` with an inline comment flagging the gap rather than inventing a fake Host object | **Y** |
| 4 | Sofia Lindqvist | Change-management requirements (CM-01/CM-02) have no natural home in the `architecturalDecisions` block on `runtime_service`/`data_store_service` — that block has named keys for auth/secrets/logging/monitoring/availability/scalability/recoverability/failure-domain, but nothing for "how are changes reviewed and promoted" | Declined to invent an unstructured workaround herself; deferred to the RequirementGroup's own `canBeSatisfiedBy` options (decisionRecord / narrative) and logged it for the Admin/report rather than overstep as a domain owner | **Y** |
| 4 | Sofia Lindqvist | No Relationship object yet links each product's AKS workloads back to the shared baseline Host, despite every containerized RuntimeService in the catalog implicitly depending on it | Left unmodeled rather than invent 6+ Relationship objects unilaterally; flagged for the report | **Y** |
| 4 | Jamal Ibrahim | Honestly describing Snowflake's `encryption.atRest` as "not yet enabled" (Tri-Secret Secure pending, DATA-088) rather than rounding up to look compliant | No workaround needed — the schema accepted an honest "not yet" answer without complaint, which is itself a positive finding | N |
| 4 | (process) | A `git add -A` staged unrelated pending files, bundling Jamal's ADLS/landing-zone contribution into Marcus's Redis commit under the wrong author | Caught via `git log --stat`, fixed with `git reset --soft` and a clean re-split into two correctly-attributed commits, force-pushed | N (process error, not a DRAFT issue) |
| 4 | (process) | An unquoted heredoc's backtick-quoted word inside a YAML comment was interpreted by the shell as command substitution, silently deleting the word from a committed file | Caught by re-grepping the committed content, fixed with a transparent `fix:` commit explaining the cause | N (tooling error, not a DRAFT issue) |
| 5 | Eleanor Voss | The `runtime_service` template's implicit assumptions (HA, autoscaling, managed-identity auth) actively misrepresent a 20+ year old single-instance on-prem legacy system if answered uncritically | Used `runtime_service` as the closest available type per Admin guidance, with explicit `n/a`/`manual` answers throughout rather than fabricated maturity; produced two new real tickets (TAX-1142, TAX-1143) as a direct byproduct of answering honestly | **Y** |
| 5 | Eleanor Voss / Jamal Ibrahim | The SoftwareDeploymentPattern `deployableObjects[].intent` field has no documented enum in the vendored templates/examples — every example uses `ha`; no value exists for "deliberately not HA" | Inferred `standalone` as a value and flagged the inference rather than assuming `ha` would be silently accepted as accurate | **Y** |
| 5 | Renata Silva | Regulatory evidentiary records need 10-year retention, which exceeds Azure SQL's native PITR window and is handled by a manual annual export-to-archive process with no representation anywhere in the object model (not a backup policy field, not a separate object) | Described in free text on the DataStoreService's `architecturalDecisions.recoverabilityModel`/`backup` answers; no structured field existed to hold it properly | **Y** |
| 5 | Renata Silva | Outside counsel receives a real, audited, named read-access grant to rate-case data; DRAFT has no object type for "external/third-party access grant" — it isn't a Capability, RequirementGroup answer, or Relationship | Described in free text on the DataStoreService's `accessControl` answer | **Y** |
| 5 | Renata Silva | Power BI Embedded is a fully-managed PaaS reporting capacity that fits neither the "infrastructure I own" nor "code I deploy with HA semantics" buckets cleanly | Modeled as a TechnologyComponent (closer fit than RuntimeService, since there's no code deployed into it), but its availability/recoverability now live implicitly in Microsoft's SLA with no representation in this catalog | **Y** |
| 5 | Derek Owusu | Unclear whether cross-team `primaryTechnologyComponent` reuse by uid (rather than each team cataloging a duplicate record) was the intended pattern | Confirmed by the Admin as intended and correct; no schema gap, just missing documentation/onboarding guidance | N (docs gap) |
| 5 | Sofia Lindqvist | Unclear whether RequirementGroups should have a lighter activation tier for low-stakes internal tooling, versus answering the same mandatory questions with a deliberately lower bar | Confirmed by the Admin: "mandatory-but-the-bar-can-vary" is the intended v1.0 pattern; flagged as worth reconsidering for v1.1 since it produces identical catalog content to a hypothetical lighter tier but different audit semantics | N (design question, not a hard gap) |
| 5 | Marcus Webb | No object models the fact that every other product's RuntimeService calls the Vantage Entitlement API at request time, making Core Platform a hard dependency for the entire company | Described narratively in the SDP's `interServiceConnections` field rather than as a structured cross-repo Relationship | **Y** (same root cause as the AKS-Host Relationship gap above) |

---

## Scorecard

| Dimension | Score (1–5) | Narrative |
|---|---|---|
| Schema Completeness | **3** | The core five object types (TechnologyComponent, Host, RuntimeService, DataStoreService, NetworkService) and RequirementGroup/Capability vocabulary covered the large majority of real content cleanly — every product's primary application and database modeled without difficulty. But the exercise surfaced a consistent pattern of real gaps clustered around three edges: fully-managed PaaS with no host concept (Identity provider, Power BI Embedded), systems with no HA/scaling story at all (legacy on-prem), and relationships/access patterns that cross object or organizational boundaries (cross-product service dependency, external-party access grants, archival-vs-backup distinction). None of these were exotic — they're the kind of thing any mid-size enterprise catalog will hit immediately. |
| Requirement Group Fidelity | **4** | Acme's 10-control SOC2-aligned internal baseline translated into 6 native RequirementGroup objects with real `relatedCapability` references and minimal friction — the one consistent rough edge (CM-01/CM-02 change-management requirements having no natural `architecturalDecisions` key) is narrow and well-understood, not a structural problem with the RequirementGroup model itself. |
| Adoption Autonomy | **3** | No single agent ever got fully blocked — every ambiguity was resolved by the Admin within the same session and work continued — but that required the Admin to make six distinct judgment calls (PaaS host convention, legacy-system object-type fit, cross-team component reuse, Service Bus scoping, internal-tool rigor, `intent` enum) that a real v1.0 rollout would need answered in documentation *before* teams hit them, not invented ad hoc by whoever happens to be facilitating. A real company without a hands-on facilitator agent would likely stall at the legacy-system and PaaS-host questions specifically. |
| SDP Quality | **4** | All six SoftwareDeploymentPatterns are grounded in real source material (each product repo's actual `infra/main.tf`, `stack.yaml`, and source layout), correctly reuse shared-service objects rather than duplicating them, and honestly represent non-uniform reality (Tax's split modern/legacy networkZones, Data & Analytics' single-scheduler caveat, the Admin Console's deliberately lower bar) rather than flattening everything into a uniform-looking pattern. The one real weakness is structural, not contentful: cross-SDP dependencies (everyone calling Vantage) live only in prose, not in queryable relationships. |
| Draftsman Discoverability | **4** | For five of six products, a repo's `stack.yaml` + `infra/main.tf` + source layout was sufficient to infer an accurate RuntimeService/DataStoreService/SDP with no additional input. The sixth (Property Tax Legacy) could only be modeled accurately because its README and inline SQL comments carried context that no IaC or stack manifest captured — a real Draftsman run against a legacy repo with sparser documentation would likely under-infer rather than mis-infer, which is a safer failure mode but still a real limitation worth flagging. |

**Overall v1.0 Readiness Score: 3.6 / 5** — DRAFT's core object model and
RequirementGroup mechanism are sound and adoption-ready for a typical
modern, cloud-native product; the framework is not yet ready to call v1.0
for a *heterogeneous* real-world company until it has an explicit answer
for PaaS-without-a-host, legacy-systems-without-HA, and cross-object
relationships, because every one of those showed up in a six-product
sample, not in a hypothetical edge case.

---

## Recommended Changes Before v1.0

1. **[Critical]** Define and document an explicit convention for `host`
   on fully-managed PaaS RuntimeService/DataStoreService/NetworkService
   objects (e.g. an allowed `host: managed-paas` sentinel, or an explicit
   "no host" flag distinct from an unfilled-in field). Addresses the
   Friction Log entry on Identity & Access / Power BI Embedded host
   modeling (Phase 4, Marcus Webb; Phase 5, Renata Silva).
2. **[Critical]** Provide explicit guidance — and ideally a lighter-weight
   object type or a documented "minimal/legacy" profile of
   `runtime_service` — for systems with no HA, autoscaling, or managed-
   identity story at all, so teams aren't left inferring whether `n/a` is
   an acceptable answer. Addresses the Friction Log entry on Property Tax
   Legacy (Phase 5, Eleanor Voss).
3. **[High]** Add a documented pattern (or a lightweight Relationship
   object usage example) for expressing cross-product/cross-repo runtime
   dependencies, distinct from the existing within-SDP `deployableObjects`
   relationships. Addresses the Vantage-as-hard-dependency gap (Phase 5,
   Marcus Webb) and the AKS-Host-to-workload gap (Phase 4, Sofia
   Lindqvist).
4. **[High]** Add an object type, or an extension to an existing one, for
   external/third-party access grants (e.g. outside counsel, regulators,
   auditors) — currently representable only as unstructured prose.
   Addresses the Friction Log entry on RegTrack's outside-counsel access
   pattern (Phase 5, Renata Silva).
5. **[Medium]** Give change-management-style requirements (CM-01/CM-02
   pattern) a documented home in the `architecturalDecisions` block, or
   explicitly bless `decisionRecord`-only satisfaction as the intended
   path. Addresses the Friction Log entry from Phase 4 (Sofia Lindqvist).
6. **[Medium]** Document the valid enum (or confirm it's intentionally
   open) for `deployableObjects[].intent` in the SoftwareDeploymentPattern
   schema — every vendored example uses `ha`, leaving no precedent for
   "deliberately not highly available." Addresses Phase 5 friction
   (Eleanor Voss, Jamal Ibrahim).
7. **[Medium]** Add a structured field (or a documented pattern) for
   long-term archival/retention policy, distinct from short-term
   backup/PITR — `architecturalDecisions.recoverabilityModel` and `backup`
   currently conflate "can we restore last week's data" with "must we keep
   this for a decade for regulatory reasons." Addresses the Friction Log
   entry on RegTrack's 10-year retention requirement (Phase 5, Renata
   Silva).
8. **[Low]** Clarify in onboarding documentation that cross-team
   `primaryTechnologyComponent` reuse-by-uid is the intended pattern (it
   is, per this exercise's Admin ruling, but nothing in the vendored docs
   says so explicitly). Addresses Phase 5 friction (Derek Owusu).
9. **[Low]** Decide and document whether RequirementGroup activation
   should support a lighter tier for low-stakes internal tooling, or
   explicitly confirm that "same mandatory question, lower documented
   bar" is the permanent intended pattern rather than an interim one.
   Addresses Phase 5 friction (Sofia Lindqvist).

---

## Repository Index

| Repository | Contents |
|---|---|
| [getdraft/acme-powersoft-catalog](https://github.com/getdraft/acme-powersoft-catalog) | Vendored DRAFT framework, 6 company RequirementGroups, 9 shared-service objects, 6 teams' product-specific catalog objects + ONBOARDING.md + sdp.yaml, this report |
| [getdraft/acme-powersoft-core-platform-product](https://github.com/getdraft/acme-powersoft-core-platform-product) | Acme Vantage — entitlement/tenancy API (.NET, AKS, Postgres) |
| [getdraft/acme-powersoft-tax-compliance-product](https://github.com/getdraft/acme-powersoft-tax-compliance-product) | Acme Tax Suite — Fixed Assets engine (.NET/SQL) + Property Tax Legacy (PowerBuilder/Oracle, on-prem) |
| [getdraft/acme-powersoft-asset-lease-product](https://github.com/getdraft/acme-powersoft-asset-lease-product) | Acme Asset & Lease Manager — ARO accretion engine (.NET, SQL, Service Bus) |
| [getdraft/acme-powersoft-regulatory-product](https://github.com/getdraft/acme-powersoft-regulatory-product) | Acme RegTrack — cost-of-service engine (.NET, SQL) + Power BI Embedded rate-case dashboard |
| [getdraft/acme-powersoft-data-analytics-product](https://github.com/getdraft/acme-powersoft-data-analytics-product) | Acme Insight Hub — Airflow ELT pipeline + churn-risk model (Python, ADLS, Snowflake) |
| [getdraft/acme-powersoft-infra-devops-product](https://github.com/getdraft/acme-powersoft-infra-devops-product) | Reusable Terraform modules, shared CI template, Datadog dashboards, Next.js admin console |
