# Schema extensions for anticipatory action triggers (v0.1)

This repository defines a small **core schema** for encoding anticipatory action triggers in a simple, human-readable format (see `core-template.yaml`).

Many plans (EAPs, sEAPs, AAPs, AAFs, Start Ready contingency plans, NGO AAPs) need a bit more detail. This document describes **optional extensions** you can add to the core where needed.

You do **not** have to use all of these. Start with the core and only add fields when they help to reflect what is already in the plan.

---

## 1. Extra source and metadata fields

These can be added under the existing `source` and `metadata` blocks.

### 1.1 Source

```yaml
source:
  original_document_type: "Anticipatory Action Framework"
  # e.g. "Anticipatory Action Protocol", "Crisis Response Programme", etc.

  # If the plan exists in multiple languages, you can note that here.
  other_document_languages:
    - "es"
    - "fr"
```

### 1.2 Metadata: organisations and systems

```yaml
metadata:
  participating_orgs:
    - "WFP"
    - "FAO"
    - "Welthungerhilfe"

  # Financing or risk financing instruments that this trigger set unlocks.
  funding_instruments:
    - name: "CERF Anticipatory Action Window"
      provider: "OCHA"
      notes: "Funds released when activation trigger is met."

    - name: "Start Ready Risk Pool 3"
      provider: "Start Network"
      notes: "Contingency plan implemented by local NGOs."

    - name: "WAHAFA"
      provider: "Welthungerhilfe"
      notes: "Facility financing locally led AAPs."

  # Systems this trigger set is embedded in.
  linked_systems:
    - "National social protection system"
    - "National DRM law"
    - "Start Ready risk pool"

  # Local custodians responsible for maintaining the plan.
  custodians:
    - "Local NGO X"
    - "District AA Working Group"
```

These fields are useful for plans from CERF, Start Network, FAO/WFP, WHH and other NGOs where the trigger is explicitly tied to a financing mechanism or national system.

## 2. Data and indicator extensions

The core schema keeps indicators inline within each phase. If you want to re-use indicators or define data sources explicitly, you can add the following.

### 2.1 Data sources

```yaml
data_sources:
  - id: "GLOFAS"
    name: "Global Flood Awareness System"
    provider: "ECMWF"
    type: "model"      # observation | forecast | index | model | manual | other
    url: "https://www.globalfloods.eu/"
    notes: "Used for river discharge and return period thresholds."

  - id: "CDI_PAK"
    name: "Combined Drought Index for Pakistan"
    provider: "Pakistan Meteorological Department"
    type: "index"
    url: "https://example.org/pmd/cdi"
    notes: "Composite of SPI, VHI, soil moisture, ENSO, etc."
```

### 2.2 Indicator catalogue

```yaml
indicator_catalog:
  - id: "CDI"
    description: "Combined Drought Index (0–100%)"
    data_source_id: "CDI_PAK"
    metric_type: "index"     # value, probability, index, category, boolean, manual
    unit: "%"
    notes: "See original protocol for weighting."

  - id: "RIVER_Q_10YR_PROB"
    description: "Probability of ≥10-year return-period flood in target reach"
    data_source_id: "GLOFAS"
    metric_type: "probability"
    unit: "%"
```

Within a phase you can then reference catalogued indicators:

```yaml
phases:
  - phase_id: "T2"
    indicators:
      - use_indicator: "CDI"
        condition:
          operator: ">="
          value: 70
          unit: "%"

      - use_indicator: "RIVER_Q_10YR_PROB"
        condition:
          operator: ">="
          value: 50
          unit: "%"
```

### 2.3 Spatial conditions

For triggers that explicitly refer to numbers of districts, provinces, basins, etc., you can add a `spatial_condition` to an indicator:

```yaml
      - use_indicator: "CDI"
        condition:
          operator: ">="
          value: 70
          unit: "%"
        spatial_condition:
          metric: "admin_units_meeting_threshold"
          admin_level: "district"
          operator: ">="
          value: 3
```

This covers designs like: "CDI ≥ 70 in at least three districts".

---

## 3. Logic and governance extensions

The core only requires a human-readable explanation of the trigger logic. If you want more structure, add these.

### 3.1 Machine expressions for trigger and stop logic

```yaml
phases:
  - phase_id: "T2"
    trigger_logic:
      human_readable: >
        Activation occurs when CDI ≥ 70 in at least three districts.
      expression: "CDI AND CDI_admin_units"
      expression_language: "simple_ids"

    stop_logic:
      human_readable: >
        Stand down if CDI falls below 70 in all districts before the lead time ends.
      expression: "NOT CDI"
      expression_language: "simple_ids"
```

`expression_language` is intentionally vague for now; you can treat `simple_ids` as "Boolean expressions using indicator ids with AND/OR/NOT".

### 3.2 Phase-level language tag

If a phase's trigger text is taken from a non-English annex, you can note it:

```yaml
  - phase_id: "fase1"
    source_trigger_language: "es"   # original trigger text in Spanish
```

### 3.3 Governance

For plans where decision roles and validation bodies are clearly specified (e.g. CERF, Start Ready, FAO/WFP, WHH, EAPs aligned with national AA TWGs), you can add:

```yaml
    governance:
      decision_role: "Anticipatory Action TWG Chair"
      validation_body: "National AA Technical Working Group"
      requires_government_concurrence: true
      notes: "Local early warning committees must confirm situation on the ground."
```

---

## 4. Using extensions in practice

A realistic encoding workflow could be:

1. Start from `schema/core-template.yaml`
2. Fill only the required core fields for your first few plans
3. Add optional blocks from this document only when the plan clearly uses them (e.g. CERF or Start Ready financing, FAO CDI, explicit TWG roles)
4. Keep the original trigger text in `source_trigger_text` for each phase so practitioners can always verify that the structured encoding matches the plan

The aim is to keep the core small and readable, while still giving enough flexibility to cover Red Cross, UN, Start Network, NGO and locally led anticipatory action plans.
