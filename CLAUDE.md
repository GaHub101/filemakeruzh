# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

This repository holds documentation for a Claris FileMaker Pro multi-file
solution (a dental/orthodontic practice management system), not application
source code. There is no build, lint, or test tooling — nothing here is
compiled or executed outside of FileMaker Pro / FileMaker Server.

- `docs/FILEMAKER_SERVER_SETUP.md` — installation/hosting guide for Claris
  FileMaker Server.
- `docs/FILEMAKER_ARCHITEKTUR.md` — architecture analysis (German), based on
  the FileMaker Pro 19 Database Design Report (DDR) covering 29 files. Read
  this before reasoning about how the files relate.
- The actual `.fmp12` database files are **not committed** — they are
  extracted locally under `_Filemaker_Interim/` and excluded via
  `.gitignore`, because they contain real patient data. Do not add them to
  git without explicit confirmation from the user.

## FileMaker solution architecture

The solution is a classic FileMaker "multi-file" architecture: each subject
area is a separate `.fmp12` file, linked to the others via external data
source references, rather than a single file with separated tables. Full
details are in `docs/FILEMAKER_ARCHITEKTUR.md`; the essentials:

- **`KG Nr` is the universal key** — 184 of 190 join predicates across the
  solution use it, all `Equal`. Beware the inconsistent spelling: `Konzeptblatt`,
  `Reevaluation`, `Termine` and `Behandlung` call it `KG nr` (lowercase n).
- **`Stammdaten.fmp12`** is the hub (19 table occurrences, 16 relationships),
  but the subject-area files are also heavily meshed with each other — this is
  not a star topology.
- **The solution was built by duplicating files, not by design.** The 29 files
  trace back to ~9 schema lineages; base table names give away the origin
  (`Photo`, `Modelle`, `Röntgenbild` all still carry base table `Platzanalyse`).
  At least 12 files are exact schema duplicates of another file.
- The `Konzept 2` / `Reevaluation` variants have **identical field sets** to
  their base file — they are treatment phases, not schema variants.
- **`Interim Finale.fmp12`** is the outlier: 794 fields, 408 calculations, zero
  relationships, fully denormalized. Also the file where leftover patient
  records were found.
- Known defects: 3 relationships with a missing match field, 10 references to
  files that do not exist. Details in the architecture doc.

### Regenerating the analysis

The source is a Database Design Report. In FileMaker Pro 19 the advanced tools
are included — enable via *Einstellungen → Allgemein → "Erweiterte Werkzeuge
verwenden"*, restart, then *Werkzeuge → Datenbank-Design-Bericht*, select all
files, format XML. The DDR contains full calculation formulas and relationship
definitions; it is schema-only and safe to work with.

The DDR XML is UTF-16. Relationships live under `RelationshipGraph`, fields and
calculations under `BaseTableCatalog/BaseTable/FieldCatalog`.
