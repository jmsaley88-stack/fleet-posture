# 🛰️ Fleet Operational Posture Monitor

> **Operational posture reasoning engine for autonomous fleet operations — built on live public environmental signals**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-jmsaley88--stack.github.io-blue?style=for-the-badge)](https://jmsaley88-stack.github.io/fleet-posture/)
![Status](https://img.shields.io/badge/Status-Live-brightgreen?style=for-the-badge)
![Data](https://img.shields.io/badge/Data-Live%20Government%20APIs-orange?style=for-the-badge)

---

## 🎯 What It Does

This is **not a weather dashboard.**

It answers one operational question:

> *"Which autonomous fleet service regions currently require attention — and is escalation actually justified?"*

The system ingests live public environmental signals, filters them spatially against simulated Waymo service regions, applies evidence-weighted justification logic, and surfaces only operationally consequential conditions — with full escalation rationale, restraint justification, and governance framing.

**The most important capability is correctly deciding NOT to escalate.**

---

## ✅ Live Validation

The system has independently matched real Waymo operational decisions twice — from public data alone.

| Date | Event | System Decision | Real Decision |
|------|-------|----------------|---------------|
| May 25, 2026 | NWS Flash Flood Warning, Tarrant County + Trinity River at 22.8ft (2.8ft above flood stage) | **RESTRICT** | Waymo suspended Dallas service |
| May 27, 2026 | NWS Flash Flood Warning, Dallas/Kaufman counties at 9:24AM CDT + Trinity River at 22.9ft | **RESTRICT** | Active flooding confirmed |

> These were not designed outcomes. The detection logic reached the same conclusions as real operational decisions independently.
>
> 📰 [Fox4: Waymo suspends robotaxi service in Dallas](https://www.fox4news.com/news/waymo-suspends-robotaxi-service-dallas)

---

## 🔍 Detection Pipeline

```
50 raw signals ingested
    ↓
Spatial relevance check  →  41 suppressed (outside all service regions)
    ↓
Deduplication            →   1 suppressed (duplicates)
    ↓
Threshold filter         →   4 suppressed (below relevance threshold)
    ↓
Corroboration scoring    →  gauge trend weighting (rising / stable / falling)
    ↓
Posture ceiling check    →  SUSPEND gated to tornado/emergency events only
    ↓
4 region-relevant signals → 2 posture changes
```

---

## 🗺️ Service Regions

11 simulated Waymo public service regions:

`San Francisco Bay Area` · `Los Angeles` · `Phoenix` · `Miami` · `Nashville` · `Orlando` · `Dallas` · `Houston` · `San Antonio` · `Austin` · `Atlanta`

---

## 📡 Data Sources

| Source | Type | Role |
|--------|------|------|
| NWS Active Alerts | Authoritative | Primary hazard trigger |
| NWS Red Flag / Fire Weather | Authoritative | Fire risk |
| NWS Fire Zone Polygons | Authoritative | Precise zone geometry |
| USGS Water Gauges | Physical sensor | Flood corroboration + trend |
| USGS Earthquakes | Physical sensor | Seismic context |
| Cal OES Evacuation Zones | Authoritative | Hard operational exclusion |
| SPC Day 1 Outlook | Probabilistic | Awareness layer only |

---

## 📊 Posture States

| State | Meaning | Authority Required |
|-------|---------|-------------------|
| 🟢 **NORMAL** | No relevant conditions | — |
| 🔵 **MONITOR** | Emerging signal, low confidence | Auto |
| 🟡 **WATCH** | Conditions developing | Auto |
| 🟠 **RESTRICT** | Route/dispatch modifications warranted | Regional ops review |
| 🔴 **SUSPEND** | Conditions exceed safe tolerance | Command approval |

---

## 🧠 Scoring Model

```
Base signal score      (event type × source confidence)
+ Spatial relevance    (intersects or approaches operating region)
+ Corroboration        (independent source confirms)
+ Persistence          (active across refresh cycles)
− Operational gap      (effects not yet confirmed)
− Temporal decay       (stale signal)
= Posture Justification score
```

**Gauge trend weighting:**
- 📈 Rising → full corroboration (+18) — active threat
- ➡️ Stable → partial corroboration (+10) — persistent, not worsening
- 📉 Falling → no corroboration (0) — recession phase, improving

**Posture ceiling:** SUSPEND requires Tornado Warning, Flash Flood Emergency, Hurricane Warning, or Evacuation Order. Prevents cumulative scoring drift from producing unjustified escalation.

> Thresholds are **configurable policy parameters** — not mathematically validated boundaries. They require operational calibration against historical incidents.

---

## 🔍 What Each Region Shows

When a region is degraded, the system displays:

- **Operational Assessment** — plain-language rationale
- **Confidence Dimensions** — Signal / Operational / Escalation / Recovery
- **Escalation Rationale Delta** — exactly what NEW evidence caused the posture change
- **Evidence Sufficiency** — ✓/✕ checklist (confirmed vs unconfirmed)
- **Why Not [Higher Posture]** — explicit missing evidence
- **Escalation Triggers** — what would justify next posture level
- **Posture Timeline** — timestamped transitions with delta rationale
- **Operational Burden Estimate** — route reliability, remote assistance, dispatch continuity
- **Governance Notation** — authority and review requirement
- **Recovery Status** — stabilization cycles before normalization

---

## 🤖 AI Operational Briefing

Auto-generates on every refresh using Claude Haiku:

```
STATUS:     Dallas RESTRICT, Austin MONITOR. Nine regions NORMAL.
CONCERN:    Flash flood warning intersects Dallas corridors.
            Route reliability risk increasing in next 60 min.
CONFIDENCE: NWS warning authoritative. Fleet impact unconfirmed.
ACTION:     Begin Dallas corridor review. Hold Austin steady.
```

The AI summarizes what the **deterministic engine** decided — it does not determine posture.

---

## ⚖️ Suppression as a Feature

> Most dashboards show you everything. This one shows you only what matters.

The system actively tracks and categorizes suppressed signals:

- Signals outside all service regions
- Duplicate alerts
- Advisory-only, below threshold
- Below operational relevance threshold

**"Escalations Avoided"** is a first-class metric in the header — alongside signals ingested, suppressed, and relevant.

---

## 🏗️ Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML/CSS/JS — no framework |
| Map | Leaflet.js |
| Proxy | Cloudflare Worker |
| AI Briefing | Anthropic Claude Haiku |
| Data | NWS, USGS, SPC, Cal OES — all free public APIs |

---

## ⚠️ Limitations

- No session persistence — posture history resets on reload
- No real fleet telemetry — corroboration uses public gauges only
- Bounding box regions — not actual Waymo service polygons
- Thresholds uncalibrated — require tuning against historical incidents
- No capacity modeling — remote assistance saturation, dispatch backlog not modeled

---

## 📋 What This Demonstrates

1. Reducing hundreds of noisy signals to operationally relevant exceptions
2. Suppression discipline with full categorized audit trail
3. Evidence sufficiency reasoning — confirmed vs unconfirmed explicitly separated
4. Escalation governance — every posture change justified and traceable
5. Escalation rationale delta — what NEW evidence caused each transition
6. Posture ceiling logic — signal authority gates reachable posture states
7. Restraint as a feature — NOT escalating is as important as escalating
8. Gauge trend intelligence — rising vs receding rivers treated differently
9. Recovery discipline — normalization requires sustained improvement
10. Live validation — matched real operational decisions twice in two days

---

*Not affiliated with Waymo. Service regions based on publicly announced markets. No internal Waymo data, systems, or infrastructure used or implied.*
