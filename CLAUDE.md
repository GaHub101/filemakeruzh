# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

This repository holds documentation for a Claris FileMaker Pro multi-file
solution (a dental/orthodontic practice management system), not application
source code. There is no build, lint, or test tooling — nothing here is
compiled or executed outside of FileMaker Pro / FileMaker Server.

- `docs/FILEMAKER_SERVER_SETUP.md` — installation/hosting guide for Claris
  FileMaker Server.
- `docs/FILEMAKER_ARCHITEKTUR.md` — architecture analysis extracted directly
  from the binary `.fmp12` files (German). Read this before reasoning about
  how the files relate; it supersedes the 2004-log guesswork below.
- The actual `.fmp12` database files are **not committed** — they are
  extracted locally under `_Filemaker_Interim/` and excluded via
  `.gitignore`, because they contain real patient data. Do not add them to
  git without explicit confirmation from the user.

## FileMaker solution architecture

The solution is a classic FileMaker "multi-file" architecture: each subject
area is a separate `.fmp12` file, linked to the others via external data
source references, rather than a single file with separated tables. Full
details are in `docs/FILEMAKER_ARCHITEKTUR.md`; the essentials:

- **`Stammdaten.fmp12`** is the hub (patient master data + case workflow). It
  references the entire rest of the solution, including every treatment-phase
  variant.
- **`KG Nr.`** (Krankengeschichte-Nummer) is the shared patient key — it is
  present in every file examined.
- Subject-area files (`Befundblatt`, `Konzeptblatt`, `Platzanalyse`,
  `paAnalyse`, `toothsizeanalysis`, `kvBlatt`) reference Stammdaten and each
  other. `Konzeptblatt` acts as the navigation hub between treatment phases.
- **`Bolton.fmp12` is a duplicate of `Platzanalyse.fmp12`** (it still reports
  its own internal path as `Files/Platzanalyse/`), not a separate module.
- **`Interim Finale.fmp12` is genuinely standalone** — no external data
  sources.
- Many subject-area files exist in three variants — base, `Konzept 2`,
  `Reevaluation` — which are treatment-phase snapshots of the same schema
  (1st concept, 2nd concept, re-evaluation), not structurally different files.

### Reading `.fmp12` files without FileMaker

Identifiers inside `.fmp12` are obfuscated with **XOR 0x5A**, which is why a
plain text scan finds nothing. After un-XORing, names are stored as
length-prefixed Latin-1 strings (the length byte is XORed too). Marker bytes
before each string identify the category — see the method section of
`docs/FILEMAKER_ARCHITEKTUR.md`. Calculation formulas are *not* recoverable
this way (stored tokenized with field IDs).
