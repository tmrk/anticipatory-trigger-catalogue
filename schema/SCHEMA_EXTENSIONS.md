# Schema extensions for anticipatory action triggers (v0.1.2)

This repository defines a small **core schema** for encoding anticipatory action triggers in a simple, human-readable format (see `core-template.yaml`).

Many plans (EAPs, sEAPs, AAPs, AAFs, Start Ready contingency plans, NGO AAPs) need a bit more detail. This document describes **optional extensions** you can add to the core where needed.

You do **not** have to use all of these. Start with the core and only add fields when they help to reflect what is already in the plan.

---

## 1. Extra source fields

These extend the existing `source` block.

### 1.1 Original document and languages

```yaml
source:
  original_document_type: "Anticipatory Action Framework"
  # e.g. "Anticipatory Action Protocol", "Crisis Response Programme", etc.

  # If the plan exists in multiple languages, you can note that here.
  other_document_languages:
    - "es"
    - "fr"
```

### 1.2 Scheme / network detail (optional refinement)

The core uses a single string `aa_scheme` (e.g. `"RCRC_DREF"`, `"CERF_AA"`, `"START_READY"`, `"WAHAFA"`, `"GOV_AA"`, `"OTHER"`). If you need more nuance without adding much bulk, you can use a small dictionary:

```yaml
source:
  aa_scheme: "RCRC_DREF"
  aa_scheme_detail:
    owner: "IFRC"
    instrument: "DREF"
    code: "sEAP2024NP01"
```

All fields here are optional. If you want the shortest possible files, just keep the single `aa_scheme` string.

## 2. Data and indicator extensions

The core schema keeps indicators inline within each phase. If you want to re-use indicators or define data sources explicitly, you can add the following.

### 2.1 Data sources

The core template already shows the minimal structure:

```yaml
data_sources:
  - id: "GLOFAS"
    name: "Global Flood Awareness System"
    provider: "ECMWF"
    type: "model"      # observation | forecast | index | model | manual | other
    url: "https://www.globalfloods.eu/"
```

If you want, you can also record default units or conventions associated with this source:

```yaml
data_sources:
  - id: "DHM_GAUGES"
    name: "DHM real-time river gauge network"
    provider: "Department of Hydrology and Meteorology, Nepal"
    type: "observation"
    url: "https://example.org/dhm/gauges"
    default_units:
      river_stage: "m"
      discharge: "m3/s"
```

These extra fields are optional; they do not affect trigger firing logic, but can help when generating or checking human-readable text.

### 2.2 Indicator catalogue

For some plans (e.g. multi-basin flood protocols, multi-hazard frameworks), the same indicator is used in several phases. You can define it once in an `indicator_catalog`, then reuse it with `use_indicator` inside phases.

```yaml
indicator_catalog:
  - id: "CDI"
    description: "Combined Drought Index (0–100%)"
    data_source_id: "CDI_PAK"
    metric_type: "index"     # value, probability, index, category, boolean, manual
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
```

When using `use_indicator`, the indicator inherits fields from the catalog (data_source_id, variable, metric_type, unit, etc.), and only the threshold `condition` is specified inline.

This can reduce duplication and file size when the same indicators appear in many phases.


## 3 Spatial conditions

Spatial conditions are the main way to describe where indicators are measured and where a phase applies.

There are two complementary mechanisms:

1. `spatial_condition_id` on an indicator, linking it to a concrete object in the top-level `spatial_conditions` catalogue (station, basin, area, grid cell).
2. An inline `spatial_condition` inside a `condition`, for designs like "CDI ≥ 70 in at least 3 districts".

### 3.1 Top-level spatial_conditions catalogue

Minimal example for a river gauge:

```yaml
spatial_conditions:
  - id: "station_karnali_chisapani"
    label: "Karnali River at Chisapani gauge"
    feature_type: "station"         # station | basin | area | grid_cell | other
    station:
      provider: "DHM"
      station_name: "Chisapani"
      station_type: "river_gauge"
      river_name: "Karnali"
      country: "NP"
```

You can optionally add geometry and basin information when available:

```yaml
spatial_conditions:
  - id: "station_karnali_chisapani"
    label: "Karnali River at Chisapani gauge"
    feature_type: "station"
    station:
      provider: "DHM"
      station_name: "Chisapani"
      station_type: "river_gauge"
      river_name: "Karnali"
      country: "NP"
    geometry:
      type: "Point"
      coordinates: [80.967, 28.683]   # [lon, lat]
    basin:
      name: "Karnali"
      code: "KARNALI_MAIN"
```

For area-based triggers, you can use feature_type `"area"` or `"basin"` and describe admin areas or polygons:

```yaml
spatial_conditions:
  - id: "area_karnali_basin_districts"
    label: "Target municipalities in Karnali basin"
    feature_type: "area"
    admin_areas:
      - admin_level: "district"
        name: "Kailali"
      - admin_level: "district"
        name: "Bardiya"
```

### 3.2 Inline spatial_condition for "number of admin units meeting threshold"

Use this when the trigger text refers to a minimum number of districts, provinces, etc. meeting a threshold, for example: "CDI ≥ 70 in at least 3 districts".

```yaml
phases:
  - phase_id: "T2"
    indicators:
      - use_indicator: "CDI"
        condition:
          operator: ">="
          value: 70
          spatial_condition:
            metric: "admin_units_meeting_threshold"
            admin_level: "district"   # e.g. district | province | municipality | basin
            operator: ">="
            value: 3                  # at least 3 districts meet the CDI threshold
```

Semantics: evaluate the indicator per admin unit at `admin_level`, count how many units meet the threshold, then apply `spatial_condition.operator` and `spatial_condition.value` to that count.

---

## 4. Logic extensions

The core already expects phase-level `trigger_logic.boolean_expression` and optional `stop_logic boolean_expression`. If you want, you can also add an informal explanation without changing the logic:

```yaml
phases:
  - phase_id: "T2"
    trigger_logic:
      boolean_expression: "i_cdi70_3districts AND i_rain_extreme_3d"
      explanation: "Activation when CDI ≥ 70 in at least 3 districts and extreme rainfall forecast in next 3 days."
```


## 5. Using extensions in practice

A realistic encoding workflow could be:

1. Start with the core only (top-level fields, data_sources, spatial_conditions, phases, indicators, trigger_logic).
2. Add **only** those extensions that reduce duplication or clarify essential semantics (e.g. `indicator_catalog`, inline `spatial_condition`).
3. Avoid recording financing, governance or organisational details here unless they directly affect the trigger logic.

The aim is that each YAML file remains compact, but still contains all the information needed to reproduce the trigger thresholds, locations, lead times and logical combinations from the original plan.

