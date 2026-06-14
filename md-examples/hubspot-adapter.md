# HubSpot Adapter — the front-office connector + CRM-as-REA-events

**Date**: 2026-06-12
**Status**: **Spec (unbuilt).** Demand-pulled — the founder's own company (RFS, selling the RoboSystems solution) is adopting HubSpot's free CRM as the live sales system, and wants that data synced *down* and linked to REA agents the way QuickBooks accounting data is. That makes the founder tenant the **first customer** of this surface. Scoped to **Phases 0–2** (connector → metrics → deals-as-commitments). The full native **RoboFO** product extension is **Phase 3 — named, not built** (operating principle #3: don't spec speculative products ahead of demand).
**Provenance**: User request 2026-06-12 — "I need a CRM for tracking opportunities for selling RoboSystems … sync data from HubSpot and link it to the agent concepts in RoboLedger's REA model, such that activities in RoboFO emit events (even ones that don't create a transaction) that can be aggregated as metrics (how many leads this month → a fact in an Information Block). Eventually, like RoboLedger, switch over to using RoboFO as the CRM." Internal deep-dive (REA model + extensions surface + QB adapter) and HubSpot developer-docs research conducted same day.
**Depends On**: [`adapters.md`](../ref/adapters.md) (the adapter pattern, `Connection`/`ConnectionCredentials`, OAuth provider system, CDC/`write_policy` — QB is the canonical template); [`event-driven-ledger.md`](../ref/event-driven-ledger.md) §2–§3 (base `Agent`/`Event`, `event_class='support'`, capture-only path, duality/commitment primitive); the base schema (`schemas/base.py` — `Agent`/`Event` are base nodes).
**Related**: [`specs/metrics.md`](metrics.md) (Phase 1 *is* the Metrics IB's first non-financial consumer); [`specs/revenue-recognition.md`](revenue-recognition.md) (Phase 2 closed-won deal → `Contract` → rev rec is the value-chain payoff); [`roadmap.md`](../roadmap.md) §5.2; [[project_robox_product_portfolio]] (RoboFO is a planned RoboX product); [[architecture_oltp_olap_rea_gap]] (the REA Tier-1 substrate this rides); [[feedback_qb_customer_surface]] (the arc this mirrors — HubSpot is to RoboFO what QB is to RoboLedger).

---

## 1. The vision and the reframe

The end-state the user described is **RoboFO**: a free-HubSpot-shaped front office — sales-process tracking from marketing → lead → opportunity → sale — that emits REA events (including non-transactional ones countable as metrics) and that you could eventually run *instead of* HubSpot, exactly as RoboLedger is meant to replace QuickBooks.

**The reframe that scopes this spec:** the two things actually being asked for near-term —

1. *link HubSpot CRM data to RoboLedger's REA agents*, and
2. *count lead/activity events as metric facts in an Information Block* —

both ride on infrastructure that **already exists**. `Agent` and `Event` are **base nodes** (not roboledger-scoped — deliberately promoted because every RoboX product needs them; `schemas/base.py`, README Invariant 1). The Metrics IB (`block_type='metric'`) is the **next rung on the IB ladder**, and its stated scope already names *"non-financial observed and aggregated metrics … as time-series facts."* So the near-term build is **"HubSpot as the next connector"** — a sibling of the QuickBooks adapter — *not* a new product extension. Building the full RoboFO OLTP/GraphQL/frontend extension now would be the speculative-product trap (operating principle #3); it waits until you've outgrown HubSpot (Phase 3).

This mirrors the QB→RoboLedger arc precisely: **HubSpot is to RoboFO what QuickBooks is to RoboLedger** — the external system of record now, the native product later, bridged by `Connection.write_policy`.

## 2. The unification payoff — one `Agent`, both sides of the house

Because `Agent` is a base node keyed on `(source, connection_id, external_id)` (the same dedup key QB uses), a HubSpot sync and a QB sync **upsert into the same `agents` table**. The customer who owes you money in the ledger (AR) is the same `Agent` node in your sales pipeline. Cross-source identity resolution (HubSpot company ↔ QB customer ↔ the AR sub-ledger) becomes a graph join on `Agent`, not an integration project.

This is the front-office/back-office bridge the whole RoboX thesis rests on, and it falls out of the existing schema **only if CRM data lands in the same graph as the ledger** — which is the chosen placement (§4).

## 3. The REA mapping (HubSpot → RoboSystems)

| HubSpot object / event | RoboSystems mapping | Fit |
|---|---|---|
| Contact (`0-1`), Company (`0-2`) | base `Agent` (`agent_type='customer'`); `source='hubspot'`, `external_id=hs_object_id` | ✅ Natural — exact QB-agent pattern |
| Owner / sales rep | base `Agent` (`employee`/`self`) | ✅ Natural |
| Activity — Call/Email/Meeting/Note/Task | base `Event`, `event_class='support'`, `event_type='hs_call'…`, capture-only (`apply_handlers=False`) | ✅ Natural — support events are non-posting *by design* |
| Lifecycle-stage change (subscriber→lead→MQL→SQL→opportunity→customer) | base `Event` (support), `event_type='lifecycle_changed'`, counted per period | ✅ This **is** the marketing→sale funnel |
| Lead (`0-136`) / "lead created" | base `Event` (support), `event_type='lead_created'` + an `Agent` for the prospect | ✅ The headline metric source (§ Phase 1) |
| **Deal / Opportunity (`0-3`)** | **REA Commitment** — `Event` linked via `EVENT_OBLIGATED_BY_EVENT` (promise of a future economic event); deal value + stage in `metadata` | ◐ Supported, not yet specialized (Phase 2) |
| Deal closed-won | Commitment → fulfillment trigger → **`Contract`** (ASC 606) → rev-rec schedule → GL | ◐ The convergence point (§7) |
| Pipeline / Deal Stage | `metadata.pipeline` / `metadata.dealstage`; stage transitions are events | ✅ Metadata + event stream |
| Line Item / Product / Quote | deferred — only relevant once deals drive billing (Phase 2+/3) | — out of MVP scope |
| Ticket (`0-5`) | base `Event` (support, `event_category='inquiry'`) — service, not sales | — out of scope (sales-funnel focus) |

**Where it's clean:** Agents, support events, lifecycle counting — the REA literature confirms CRM activities are *business events*, not *economic events* (no resource moves), and the model encodes exactly that via `event_class`. **Where the model already answers the awkward bits:** a Deal isn't a `resource_type`, but a Deal *is* a Commitment, and the schema has the commitment primitive (`EVENT_OBLIGATED_BY_EVENT`/`EVENT_DISCHARGES_EVENT`); the roadmap long-tail already parks "Commitment / open-PO tracking" pending first demand — a HubSpot deal pipeline is that demand. CRM events never fire GL handlers (capture-only), so they never touch the double-entry ledger.

### 3.1 Turning CRM events into a metric — observed vs aggregated

"New leads this month" is **not** a `derived` metric in the [`metrics.md`](metrics.md) §2.1 sense (derived = arithmetic over rs-gaap anchors). It's a count, and there are two provenance shapes for it:

| Path | "New leads this month" = | Trade-off |
|---|---|---|
| **Observed** | a scalar pulled from HubSpot's Search API (`total` of a `hs_lastmodifieddate`-filtered query) | simplest; trusts HubSpot's count; the number isn't in your graph |
| **Aggregated** | a Cypher count of synced `lead_created` `Event` rows per period | more work; the data lives **on the platform** — auditable, queryable, joins with AR/GL/pipeline |

**Recommended: aggregated.** The dogfood thesis ([[project_ai_native_company]]) wants the events *in the graph*, not a number borrowed from HubSpot — and only the aggregated path makes "lead → deal → close → revenue" traceable as one `Event` chain (§7). The Metrics spec reserved *aggregated* for its Phase 5 ("gated on sibling RoboX products existing"); the CRM adapter is the **first aggregated-metric source, arriving via an adapter ahead of those products**, so it pulls that phase forward (metrics §9). Caveat: the shipped `FactProvenance` union (`pivot/schedule/derived/asserted`) has no first-class `observed`/`aggregated` origin yet — making those first-class is an open question for the Metrics build, not this one (§6).

## 4. Placement — same graph as the ledger (decided)

CRM Agents/Events land in the **company's existing (roboledger-enabled) graph**, via a HubSpot `Connection` on that graph, `write_policy='hubspot_authoritative'`. Rationale: `Agent`/`Event` are base, so the sales pipeline and the GL naturally share customer `Agent` nodes (§2). A dedicated CRM graph would re-fragment customer identity and force cross-graph traversal (the roboinvestor pattern) where none is needed. *Rejected alternative:* dedicated CRM graph (cleaner isolation, but loses the unification that is the whole point).

## 5. Adapter architecture — HubSpot as the next connector (QB template)

~80% of the QuickBooks adapter is already provider-agnostic and reused unchanged:

**Reused as-is:** `Connection` + `ConnectionCredentials` (Fernet-encrypted tokens, `write_policy`, `last_cdc_watermark`); the generic `OAuthHandler`; the OAuth router endpoints (`POST /v1/graphs/{g}/oauth/init` + `/callback/{provider}`); the Valkey per-connection sync lock; the Dagster extract→transform→load job shape; the OLTP→OLAP materialization; the base `agents`/`events` tables; the staleness trigger.

**New (the HubSpot-specific ~20%):**

```
adapters/hubspot/
  __init__.py                         # lazy-export HSClient (PEP-562, per adapters/__init__.py table)
  client/api.py                       # HSClient: OAuth + paged objects / search / associations / webhooks
  pipeline/{configs,extract,transform,load,jobs}.py
  pipeline/event_action_mapping.py    # lifecycle stage / deal stage → event_type + event vocabulary
  dbt/models/staging/                 # stg_hs_contacts, stg_hs_companies, stg_hs_deals, stg_hs_engagements
  dbt/models/ledger/                  # agents.sql, events.sql (support)  [NO transactions/entries/line_items]
operations/providers/hubspot_provider.py   # HubSpotOAuthProvider (authorize / token / refresh / validate)
models/api/graphs/connections.py           # add HubSpotConnectionConfig + "hubspot" to ProviderType
# + register "hubspot" in the provider registry; add provider branch in oauth.py + sync.py
```

The structural difference from QB: HubSpot's `dbt/ledger/` produces **`agents` + support `events`**, *not* the `transactions/entries/line_items` GL chain. The base schema already accommodates this — that's exactly what "front-office adapter" means.

### 5.1 Auth — OAuth public app (not private app)

- Authorize `https://app.hubspot.com/oauth/authorize` → token `https://api.hubapi.com/oauth/v3/token`.
- **Access token = 30 min; refresh token = never expires.** Tenant identity = `hub_id` (portal ID) in the token response — store on the `Connection` like QB's `realm_id`.
- Scopes (read-only sync): `crm.objects.contacts.read`, `…companies.read`, `…deals.read`, plus activity scopes. Custom objects are Enterprise-tier only — *don't depend on them*.
- Private apps are single-account and marketplace-ineligible → **wrong for multi-tenant**; use the public-app OAuth flow that slots into the existing `OAuthHandler`.

### 5.2 Sync / CDC — better than QB's

1. **Backfill:** CRM Exports API (`POST /crm/v3/exports/export/async`, 30/day) or paged Search; record max `hs_lastmodifieddate` as the watermark.
2. **Incremental (the everyday path):** Search by `hs_lastmodifieddate ≥ watermark` ascending (note: contacts use `lastmodifieddate`). Cursor pagination; **5 req/s, 10k-result cap, 200/page** — page within the cap and advance the watermark across windows. Advance `Connection.last_cdc_watermark` exactly as QB does.
3. **Real-time (Phase 1+ enhancement):** app-level **Webhooks v3** (`object.creation/propertyChange/deletion/restore/associationChange`), HMAC-SHA256 `X-HubSpot-Signature-v3`, ≤100 events/POST, 10 retries/24h. Webhooks alone are insufficient (retry/drift) → the Search reconciliation pass is the backstop. MVP can be poll-only; webhooks are an opportunistic add.

Discover each object's properties at runtime via `GET /crm/v3/properties/{objectType}` — don't hardcode the property catalog.

## 6. What exists vs. what's new

| Piece | State | Where |
|---|---|---|
| Base `Agent`/`Event` nodes + `(source, connection_id, external_id)` dedup | **exists** | `schemas/base.py`; `models/extensions/roboledger/{agent,event}.py` |
| `event_class='support'` + capture-only (`apply_handlers=False`) non-posting path | **exists** | event model + event-block commands |
| Commitment primitive (`EVENT_OBLIGATED_BY_EVENT`/`EVENT_DISCHARGES_EVENT`) | **exists** | base REA edges |
| `Connection`/`ConnectionCredentials`, generic `OAuthHandler`, OAuth endpoints, Valkey lock, ELT job shape, materialization | **exists** | QB adapter + `operations/providers/`, `routers/graphs/connections/`, `operations/extensions/` |
| `FactProvenance` union + `create_fact_set()` (the stamping spine for metric facts) | **exists** (2026-05-30) — but shipped origins are `pivot/schedule/derived/asserted`; the metrics `observed`/`aggregated` provenances are **not yet first-class** (§3.1 — an open Q for the Metrics build) | `models/api/fact_provenance.py`; `operations/roboledger/fact_set.py` |
| **`HSClient` + HubSpot OAuth provider** | **new** | `adapters/hubspot/client/api.py`; `operations/providers/hubspot_provider.py` |
| **HubSpot ELT pipeline (extract/transform/load) + dbt agents/events models** | **new** | `adapters/hubspot/pipeline/`, `adapters/hubspot/dbt/` |
| **Provider wiring** ("hubspot" in registry, oauth.py/sync.py branches, `HubSpotConnectionConfig`) | **new** | `routers/graphs/connections/`, `models/api/graphs/connections.py` |
| **Metric evaluator** (counts events → derived facts) | **new** (= the Metrics IB un-park) | `specs/metrics.md` |
| **Deal→commitment event mapping** | **new** | `adapters/hubspot/pipeline/event_action_mapping.py` |

The connector is mostly *wiring* over a proven substrate; the genuinely new conceptual work is small (provider client + dbt mapping), with the metric evaluator borrowed from the Metrics spec.

## 7. The convergence — front office → back office in one graph

The strategic payoff (and the demo that proves the RoboX thesis) is the full revenue value chain as one `Event` chain in one graph:

```
HubSpot lead → Deal (REA Commitment) → Closed-Won → Contract (ASC 606) → rev-rec schedule → GL Entry
 support Event      commitment Event       fulfillment trigger    specs/revenue-recognition.md      roboledger
```

A closed-won Deal is exactly the `contract_signed` trigger the **revenue-recognition spec** is waiting for (its Open Question §10.2 names a billing/CRM adapter feeding `contract_signed` as the real-subscription-business follow-on). HubSpot → Deal → `Contract` → rev rec → GL is marketing-to-cash, every hop an `Event`. Phase 2 lays the deal→commitment half; rev-rec closes it when that spec is pulled.

## 8. Phases

| Phase | Work | New product extension? | Effort |
|---|---|---|---|
| **0 — Connector (MVP)** | HubSpot OAuth public-app flow; ELT sync of Contacts/Companies → `Agent`, Activities + lifecycle-stage changes → support `Event` (capture-only), into the company graph; `write_policy='hubspot_authoritative'`; poll-based CDC via Search + watermark | ❌ No — rides base REA + adapter pattern | ~4–6 days |
| **1 — Metrics** | Un-park the Metrics IB; define `robofo:*`/`rs-metric:*` elements via a TaxonomyBlock; record CRM counts per period ("new leads / month", "activities / rep", "MQL→SQL conversion") as **observed/aggregated** metric facts — **not** `derived` (that's arithmetic over rs-gaap anchors; see §3.1 + [`metrics.md`](metrics.md) §2.1). HubSpot is the Metrics IB's **first non-financial source**, arriving via an adapter ahead of the sibling RoboX products the Metrics spec assumed would drive it | ❌ No — un-parks the already-next IB | Metrics-spec effort + ~1–2 days mapping |
| **2 — Deals as Commitments** | Map Deals → commitment events (`EVENT_OBLIGATED_BY_EVENT`); pipeline-value + weighted-forecast reporting; closed-won emits the rev-rec `contract_signed` trigger seam | ❌ No — uses existing commitment primitives | ~3–5 days |
| **3 — RoboFO native (named, not built)** | Full extension via the 18-step add-a-domain checklist (Lead/Opportunity/Campaign/Quote OLTP, GraphQL reads, command writes, frontend app); flip `write_policy` toward native; HubSpot becomes migration source. **Trigger:** you actually want to stop using HubSpot | ✅ Yes | Large — defer |

Webhook real-time delivery is an opportunistic enhancement layered on Phase 0 once poll-based sync is solid, not a separate phase.

## 9. Open questions

1. **Launch timing.** The immediate roadmap axis is the RoboLedger soft launch; Phases 0–2 are **post-launch / launch-parallel** (they don't gate tenant #1). Confirm this doesn't compete with launch work before starting (the [[feedback_launch_week_guardrail]] holds).
2. **One graph or per-company-being-sold graph?** Decided: same graph as the ledger (§4). But the founder's *own* company graph (RFS selling RoboSystems) is the host — confirm that graph exists / is the intended home, vs. a dedicated "RFS sales" graph.
3. **Deal → `Contract` coupling (Phase 2/rev-rec seam).** Whether closed-won auto-creates a draft `Contract` or just emits the trigger event for human/Operator review. Follow the capture-then-approve discipline (don't auto-post).
4. **Metric element namespace.** `robofo:*` vs `rs-metrics:*` for non-financial metric concepts — decide with the Metrics spec so the TaxonomyBlock namespace is consistent across financial and CRM metrics.
5. **Two-way sync (Phase 3 prerequisite).** MVP is read-only (HubSpot-authoritative). Native RoboFO needs the write-back half (HubSpot equivalent of QB's `execute-event-block` round-trip via `metadata.hs_external_id`) — out of scope until Phase 3.
6. **Property catalog drift.** Pull properties at runtime (`/crm/v3/properties/{type}`); don't bake HubSpot's default property list into the dbt models beyond the stable identifiers.

## 10. Why this matters

It converts the user's live operational need (run HubSpot as the CRM *now*, see the data in RoboSystems) into platform capability **on a substrate that's already built** — base `Agent`/`Event`, the adapter pattern, the Metrics IB, the commitment primitive — rather than inventing a product. It delivers the **founder-tenant dogfood** of the front-office/back-office unification, gives the **Metrics IB its first real non-financial consumer** (the un-park trigger), and lays the deal→commitment half of the **marketing-to-cash value chain** that, joined with revenue-recognition, is the most complete demonstration of the REA thesis the platform can make. And it does it the proven way — as another connector, HubSpot-authoritative now, native RoboFO later — exactly mirroring the QuickBooks → RoboLedger arc the product is already validating.
