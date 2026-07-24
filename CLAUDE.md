# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

This repository holds documentation for a Claris FileMaker Pro multi-file
solution (a dental/orthodontic practice management system), not application
source code. There is no build, lint, or test tooling — nothing here is
compiled or executed outside of FileMaker Pro / FileMaker Server.

- `docs/FILEMAKER_SERVER_SETUP.md` — installation/hosting guide for Claris
  FileMaker Server.
- The actual `.fmp12` database files are **not committed** — they are
  extracted locally under `_Filemaker_Interim/` and excluded via
  `.gitignore`, because they contain real patient data. Do not add them to
  git without explicit confirmation from the user.

## FileMaker solution architecture

The solution is a classic FileMaker "multi-file" architecture: each subject
area is a separate `.fmp12` file, linked to the others via external data
source references (matched on a shared patient ID), rather than a single
file with separated tables.

- **`Stammdaten.fmp12`** is the hub file (patient master data). Nearly every
  other file references it; it has no unneeded external references itself.
- **`Photo.fmp12`** is a second shared file (patient photos), referenced by
  almost all of the subject-area files.
- Subject-area / analysis files reference Stammdaten, Photo, and each other:
  `Befundblatt`, `Konzeptblatt`, `Platzanalyse` (also links `Bolton` and
  `toothsizeanalysis`), `kvBlatt`, `paAnalyse`, `toothsizeanalysis
  moyers`/`seipel`.
- Files with no cross-references found (likely standalone modules or added
  after the 2004-era conversion logs this was reconstructed from):
  `Behandlung`, `Termine`, `Reevaluation`, `Modelle`, `Etiketten`,
  `Röntgenbild`, `Bolton`, `Interim Finale`.
- Many subject-area files exist in three variants — base, `Konzept 2`,
  `Reevaluation` — which are treatment-phase snapshots of the same schema
  (1st concept, 2nd concept, re-evaluation), not structurally different
  files.
- This architecture was reconstructed from `Konvertierung.log` (a 2004
  format-conversion log listing each file's external references at the
  time); it has not been verified by opening the files in FileMaker Pro, so
  treat it as a starting point, not a guarantee — especially for match
  fields and any relationships added after 2004.
