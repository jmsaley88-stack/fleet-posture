# Fleet Operational Posture Monitor

**Live Demo:** https://jmsaley88-stack.github.io/fleet-posture/

A prototype demonstrating operational posture assessment using live public environmental signals — built to answer one question:

> *"Which autonomous fleet service regions currently require operational attention, and why?"*

---

## What This Is

This is not a weather dashboard.

It is a lightweight **operational posture reasoning engine** that ingests noisy public environmental signals, evaluates spatial relevance against simulated autonomous fleet service regions, applies evidence-weighted justification logic, suppresses non-actionable conditions, and surfaces only operationally consequential exceptions — with full escalation rationale.

The most important capability is correctly deciding **not** to escalate.

---

## Architecture

```
RAW SIGNALS
    ↓
SPATIAL RELEVANCE CHECK
(does this intersect or approach an operating region?)
    ↓
SUPPRESSION
(duplicates, advisory-only, outside regions, below threshold)
    ↓
CORROBORATION
(gauge above flood stage, secondary source agreement)
    ↓
PERSISTENCE
(condition maintained across refresh cycles)
    ↓
POSTURE JUSTIFICATION SCORING
    ↓
REGION POSTURE ASSIGNMENT
(NORMAL / MONITOR / WATCH / RESTRICT / SUSPEND)
```

Single HTML file. No backend. No database. Cloudflare Worker handles CORS proxying and AI briefing generation. All data is live from public government sources.

---

## Service Regions

Eleven simulated Waymo public service regions:

| Region | State |
|--------|-------|
| San Francisco Bay Area | CA |
| Los Angeles | CA |
| Phoenix | AZ |
| Miami | FL |
| Nashville | TN |
| Orlando | FL |
| Dallas | TX |
| Houston | TX |
| San Antonio | TX |
| Austin | TX |
| Atlanta | GA |

> These are **simulated operational regions** based on publicly announced Waymo markets. No internal Waymo fleet polygons or telemetry are used or implied.

---

## Data Sources

| Source | Role | Endpoint |
|--------|------|----------|
| NWS Active Alerts | Authoritative hazard trigger | `api.weather.gov/alerts/active/area/{state}` |
| NWS Red Flag Warnings | Fire risk trigger | `api.weather.gov/alerts/active?event=Red Flag Warning` |
| NWS Fire Weather Watches | Fire risk trigger | `api.weather.gov/alerts/active?event=Fire Weather Watch` |
| NWS Fire Zone Polygons | Actual zone geometry per fire alert | `api.weather.gov/zones/fire/{zoneCode}` |
| SPC Day 1 Outlook | Probabilistic severe weather context | `spc.noaa.gov/products/outlook/day1otlk_cat.nolyr.geojson` |
| USGS Earthquakes | Seismic context | `earthquake.usgs.gov/earthquakes/feed/...` |
| USGS Water Gauges | Ground-truth flood corroboration | `waterservices.usgs.gov/nwis/iv/` |
| Cal OES Evacuation Zones | Hard operational exclusion trigger | `services.arcgis.com/BLN4oKB0N1YSgvY8/...` |

Each source has a defined operational role. Not all signals are treated equally.

---

## Posture States

| Posture | Meaning |
|---------|---------|
| **NORMAL** | No operationally relevant conditions detected |
| **MONITOR** | Low-confidence or emerging environmental signal |
| **WATCH** | Conditions degrading — increased operational awareness recommended |
| **RESTRICT** | Routing/dispatch modifications recommended |
| **SUSPEND** | Conditions exceed safe operational tolerance |

---

## Scoring Model

Posture Justification score answers: *"How justified is a posture change for this region right now?"*

```
Base signal score      (event type × source confidence weight)
+ Spatial relevance    (intersects or approaches operating region)
+ Corroboration        (second independent source confirms same condition)
+ Persistence          (condition active across refresh cycles)
− Operational gap      (effects not yet independently confirmed)
− Temporal decay       (stale signal, no new activity)
─────────────────────────────────────────────────────────
= Posture Justification score
```

**Posture thresholds:**

| Score | Posture |
|-------|---------|
| ≥ 90 | SUSPEND |
| ≥ 75 | RESTRICT |
| ≥ 45 | WATCH |
| ≥ 22 | MONITOR |
| < 22 | NORMAL |

**Source confidence weights:**

| Source | Weight |
|--------|--------|
| Cal OES Evac Zones | 0.99 |
| NWS Active Alerts | 0.95 |
| USGS Water Gauges | 0.95 |
| NWS Fire Weather | 0.90 |
| USGS Earthquakes | 0.90 |
| SPC Outlooks | 0.80 |

---

## Suppression Logic

Signals are suppressed when they:
- Have no spatial intersection with any service region
- Are duplicates of already-processed alerts
- Are advisory/probabilistic only and below relevance threshold
- Decay after 4+ refresh cycles without resolution or escalation

**Suppression is a feature.** The Validate Data panel shows a full breakdown of suppression categories on every refresh.

> On a typical day: 50–70% of ingested signals are suppressed as non-operationally-relevant.

---

## Assessment Framework

Each degraded region displays:

- **Operational Assessment** — plain-language rationale for current posture
- **Confidence** — HIGH / MODERATE / LOW based on evidence sufficiency
- **Validation** — whether action is justified on public signal alone, or requires telemetry confirmation
- **Evidence Sufficiency** — ✓/✕ checklist of confirmed vs unconfirmed evidence
- **Why Not [Higher Posture]** — explicit statement of what evidence is missing for further escalation
- **Escalation Triggers** — what conditions would justify moving to the next posture
- **Posture Timeline** — timestamped record of posture changes during session
- **Operational Burden Estimate** — expected fleet operational impact (route reliability, remote assistance demand, dispatch continuity)
- **Contributing Evidence** — plain-language evidence statements, with scoring detail available on expansion

---

## Telemetry Layer

This prototype uses public environmental signals only.

In production, fleet telemetry would serve as a major corroboration source alongside roadway closure data, dispatch anomalies, and remote assistance volume. Simulated production corroboration indicators include:

- Reroute frequency in affected corridors
- Remote assistance demand trajectory
- Vehicle speed degradation
- Hard braking cluster frequency
- Perception system health indicators
- Dispatch continuity metrics

> The system explicitly distinguishes between *environmental authority* (NWS signal credibility) and *operational confidence* (confirmed fleet effect). These are not the same thing.

---

## Telemetry Dependency Classification

Not all signals require telemetry validation before action:

| Telemetry Status | Events |
|-----------------|--------|
| **NOT REQUIRED** | Tornado Warning, Flash Flood Emergency, Evacuation Order, Hurricane Warning intersecting region |
| **RECOMMENDED** | Flash Flood Warning + gauge corroboration, persistent severe warning |
| **REQUIRED** | Advisory-level signals, SPC outlooks, proximity-only conditions |

---

## AI Operational Briefing

On each data refresh, Claude Haiku generates a structured operational SITREP from live signal state:

```
STATUS:     Current posture across all regions
CONCERN:    Primary operational risk over next 60 minutes
CONFIDENCE: Evidence sufficiency and validation status
ACTION:     Single recommended executive action for next 30 minutes
```

The AI does not determine posture. The deterministic engine determines posture. The AI summarizes what the engine decided and produces operator-readable prose.

---

## Live Validation — May 25, 2026

On May 25 at 4:03 PM CDT, the system independently detected:

- NWS Flash Flood Warning covering Fort Worth, Arlington, Grand Prairie (Tarrant County)
- USGS Trinity River gauge at **22.8ft** — 2.8ft above the 20ft flood threshold
- Multiple corroborating Flood Advisories

The system escalated Dallas to **RESTRICT**.

Waymo had suspended Dallas robotaxi service days earlier for the same conditions.

> Source: [Fox4 News — Waymo suspends robotaxi service in Dallas](https://www.fox4news.com/news/waymo-suspends-robotaxi-service-dallas)

This was not a designed outcome. The detection logic reached the same operational conclusion from live public data independently.

---

## Limitations

- **No persistence across sessions** — posture history resets on page reload. Production would use a database.
- **No real fleet telemetry** — corroboration relies on public gauges and secondary NWS signals only.
- **Bounding box regions** — service areas are rectangular approximations. Production would use actual service polygon data.
- **Fire zone geometry** — fetched per-alert from NWS zone endpoints. Adds latency on load.
- **Gauge thresholds** — flood stage values are estimated from NWS published data, not dynamically pulled.
- **Refresh polling** — 5-minute interval. Production would use event-driven updates.

---

## What This Demonstrates

1. **Operational filtering** — reducing hundreds of noisy public signals to a small number of actionable exceptions
2. **Spatial relevance** — only signals that materially intersect or approach operating regions affect posture
3. **Corroboration logic** — single signals rarely justify escalation; independent confirmation increases justification weight
4. **Suppression discipline** — explicit, categorized suppression with full audit trail
5. **Escalation governance** — posture changes are justified, timestamped, and traceable
6. **Evidence sufficiency reasoning** — explicit separation of what is confirmed vs unconfirmed
7. **Restraint as a feature** — the system's ability to correctly NOT escalate is as important as its ability to escalate

---

## Stack

- Vanilla HTML/CSS/JS — no framework, no build step
- Leaflet.js — map rendering
- Cloudflare Worker — CORS proxy + AI briefing endpoint
- Anthropic Claude Haiku — operational SITREP generation
- All data sources: free, public, government APIs

---

*This prototype is not affiliated with Waymo. Service regions are based on publicly announced markets. No internal Waymo systems, data, or infrastructure are used or implied.*
