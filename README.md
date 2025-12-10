# Anticipatory Trigger Catalog & Schema

This repository is an experiment in describing **anticipatory action triggers**
in a simple, human-readable and machine-readable way, and gradually building a
catalogue of trigger models from real plans.

## Why

Today, trigger logic for Early Action Protocols (EAPs), Anticipatory Action Frameworks (AAFs), Anticipatory Action Protocols (AAPs), Start Ready contingency plans and NGO-led plans is usually buried in PDFs and Word files. That makes it hard to compare, test and improve triggers across countries and organisations.

## What

This repo proposes a small **core schema** for encoding triggers and a set of **optional extensions**, plus a growing **catalogue of encoded trigger sets** for different hazards and countries.

It is deliberately:

- **Human-friendly:** programme staff and government colleagues should be able to read the YAML and recognise their trigger logic
- **Technology-neutral:** it does not require a specific platform, model or data source
- **Inclusive:** it is designed to work for Red Cross / Red Crescent EAPs, UN-led CERF AA frameworks, FAO/WFP protocols, Start Network risk pools and NGO-led and locally led AAPs

This is not an official standard of any organisation. It is a working proposal to stimulate discussion and make existing practice easier to see and compare.

## How this relates to the Anticipation Hub's Trigger Database

The **Trigger Database** hosted by the Anticipation Hub is a curated overview of existing trigger models, with high-level information such as hazards, countries, lead time categories, sectors and short narrative descriptions.

This repository is **different and complementary**:

- The database provides **plan-level summaries**;  
- This catalogue encodes the **detailed trigger logic** from the underlying EAPs/AAPs/AAFs in a consistent YAML schema, phase by phase.

Where possible, entries in this catalogue can re-use and reference information from the Anticipatory Trigger Database (hazard names, countries, links, etc.), but this repo does **not** replace that database and does **not** claim to be a complete list of anticipatory action plans. Instead, it focuses on making the trigger mechanisms themselves easier to read, compare and discuss.

## Licence

MIT
