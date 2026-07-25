# Neubau Datenmodell — KO-Behandlungsworkflow

Entwurfsdokument für den Neubau der FileMaker-Lösung. Grundlage ist der
DDR-Export vom 25.07.2026 (29 Dateien) und die modulweise Bedarfsklärung mit
dem Auftraggeber.

Analyse des Altbestands: `FILEMAKER_ARCHITEKTUR.md`.

## Warum neu gebaut wird

Die Altlösung modelliert **Wiederholung als Spalten oder als eigene Dateien,
nie als Zeilen**:

- Die Behandlungsphase steckt in der Nachkommastelle der `KG Nr` (`362588.1`),
  belegt durch `Befundblatt::Reeval nummer`. Parallel dazu existieren separate
  Phasendateien *und* die Felder `Befundnr` / `Nummer Beh. Konzept` — drei
  konkurrierende Mechanismen für dasselbe.
- Weil `Stammdaten.KG Nr` = `362588` nie auf `362588.1` matcht, hängen
  Reevaluationen relational teilweise in der Luft.
- Die Zahl der Reevaluationen ist dadurch auf 9 gedeckelt.

Im Neubau ist die Befundrunde ein **Datensatz**, und beim Anlegen entstehen die
zugehörigen Befundkomponenten automatisch leer und verknüpft.

## Umfang

Von 29 Dateien bleiben 6 Fachmodule plus ein separates Forschungsmodul.

**Bleibt:** Stammdaten · Befundblatt · Konzeptblatt · Platzanalyse (inkl. Bolton) ·
toothsizeanalysis moyers · paAnalyse (nur auf Anforderung) · Interim Finale
(Forschungsdatenerfassung, separat)

**Entfällt ersatzlos:** Platzanalyse ZWUL 1/2 · Etiketten · Etiketten_sto ·
Behandlung · Termine · Modelle · Photo · Röntgenbild · kvBlatt ·
toothsizeanalysis seipel · Bolton als eigene Datei · sämtliche Phasenvarianten

**Altdaten werden nicht migriert.** Der Altbestand bleibt reines Archiv.

## Datenmodell

Eine FileMaker-Datei, mehrere Tabellen. Surrogatschlüssel (UUID, `Hole(UUID)`
als Automatikwert) für alle Beziehungen. `KG Nr` bleibt fachlicher Schlüssel
ausschliesslich auf `Patient` — ohne Dezimalstelle.

```
Patient ──1:n── Untersuchung ──1:1── Befundblatt     (automatisch)
   │                  │        ──1:1── Platzanalyse   (automatisch)
   │                  │        ──1:1── Toothsize      (automatisch)
   │                  │        ──0:1── paAnalyse      (nur auf Anforderung)
   │                  │
   │                  └──1:n── Konzept                (auf Anforderung)
```

### Tabelle `Untersuchung` (neu)

Diese Tabelle existiert im Altbestand nicht — sie ist der Kern des Umbaus.

| Feld | Typ | Bemerkung |
|---|---|---|
| `_pk_Untersuchung` | Text | UUID, Automatikwert `Hole(UUID)` |
| `_fk_Patient` | Text | UUID des Patienten |
| `Sequenz` | Zahl | 0 = Anfangsbefund, 1..n = Reevaluation n |
| `Datum` | Datum | Erhebungsdatum |
| `Status` | Text | in Arbeit / abgeschlossen |
| `Phase_Anzeige` | Berechnung | `Wenn( Sequenz = 0 ; "Anfangsbefund" ; "Reevaluation " & Sequenz )` |

Auf allen Komponententabellen kommt `_fk_Untersuchung` hinzu, auf `Konzept`
zusätzlich `_pk_Konzept` und `Konzept_Nr`; auf `Patient` `_pk_Patient`.

## Feldübernahme je Modul

Trennstrich-Attrappen und `Copy`/`Kopie`-Duplikate werden gestrichen (46
Felder). Trennlinien gehören als Layout-Objekt auf das Formular, nicht als Feld
in die Tabelle. Berechnungsfelder werden mit ihren Formeln unverändert aus dem
DDR übernommen — sie liegen dort im Klartext vor (deutsche Syntax: `Wenn`,
`Falls`, `LiesAlsZahl`, `Runden`, `Hole`).

### `Patient`  ← `Stammdaten.fmp12`

110 Felder im Original → **94 übernehmen**, 16 streichen (11 davon Berechnungsfelder).

**Normale Felder (83):**

`KG Nr` · `Start KG Nr` · `Name` · `Geburtstag` · `Geschlecht` · `Unterlagen Datum in Stammdaten` · `Status` · `Abrechnung` · `Abgerechnet` · `IV` · `Hintergrund` · `Stud.kurs` · `Stud Kurs früher` · `Stud Kurs jetzt` · `Stud Kurs Zuteilung` · `Stud Kurs Termin` · `Stud Kurs Semester` · `Stud Kurs Thema` · `Bahandler` · `Heute` · `Bemerkungen` · `gewünschtes Staff Datum` · `Instrukter Staff Datum` · `UL Kontrolle` · `Modelle Raphe` · `Modelle getrimmt` · `Fotos i.O.` · `Fotos Dia` · `Fotos Papier` · `Rx OPT` · `Rx HR` · `Rx PA` · `Rx FR IK` · `Rx FR RK` · `Rx BW` · `Rx Front` · `Rx V` · `UL Auswertung` · `Platzanalyse i.O.` · `Tooth size i.O.` · `Befundblatt i. O.` · `FR i.O.` · `PA i.O.` · `HR i.O.` · `bevorzugter Tag` · `welcher Tag` · `Abrechnungsart` · `Sortierung nach` · `‹Instruktor-Name›` · `‹Instruktor-Name›` · `‹Instruktor-Name›` · `‹Instruktor-Name›` · `Spez fall` · `Zeigen B ass` · `Zeigen C ass` · `Fall gezeigt am` · `Konzept unterschrieben am` · `Fall zeigen ok` · `Konzept unterschrieben ok` · `Dringend` · `übernommen` · `Apparatur` · `digital photos` · `digital model` · `Behandlung abgebrochen` · `Abrechnung 2` · `Diagnostik abgeschlossen` · `Behandlungsstand` · `Bemerkungen Behandlung` · `Bemerkungen Unterlagen` · `Konzeptblatt i. O.` · `KG inkl. Etikette` · `Kopien in Ordner` · `Brief i.O.` · `Ordner i.O.` · `KG ok` · `Befund auswerten ok` · `UL vollständig` · `Kategorie` · `Next Termin` · `Next` · `UL vollständig io` · `‹Instruktor-Name›`

**Berechnungsfelder (11)** — Formeln 1:1 aus dem DDR übernehmen:

`Konzept` · `Anzahl Fälle` · `Anzahl Fälle 2` · `Datum heute` · `offpriv` · `gewünschter Instruktor` · `Modelle i. O.` · `Rx i. O.` · `Fall zeigen ok copy` · `dringend status` · `übernommen Status Liste`

**Streichen (16):**

`---------------` · `-------------` · `--------------` · `----------------` · `------------------` · `Platzanalyse i.O. Kopie` · `Tooth size i.O. Kopie` · `Befundblatt i. O. Kopie` · `FR i.O. Kopie` · `PA i.O. Kopie` · `HR i.O. Kopie` · `Zeigen B ass Copy` · `Zeigen C ass Copy` · `konzept unterschrieben ok Copy` · `Konzeptblatt i. O. Kopie` · `‹Instruktor-Name›`


### `Befundblatt`  ← `Befundblatt.fmp12`

136 Felder im Original → **123 übernehmen**, 13 streichen (21 davon Berechnungsfelder).

**Normale Felder (102):**

`Unterlagendatum` · `Anamnese` · `gleich ungleich` · `li re` · `Name` · `KG Nr` · `Alter` · `Enface` · `sag` · `vert` · `transv` · `ok` · `uk` · `Anlagen` · `Lage und Zustand` · `skelettal` · `dental` · `wt richtung` · `Profil` · `Typ` · `Besonderheiten` · `Grösse Pat` · `Ruhelage` · `IOR` · `sag Aglbew` · `vert Aglbew` · `trans Aglbew` · `Sprache` · `Gewohnheiten` · `Weichteile` · `CMD` · `Status ORL` · `Tons pal` · `Adenoide` · `Atmg` · `spezifcation` · `spez OK re` · `spez OK li` · `spez UK re` · `spez UK li` · `Entwicklungsstand` · `Abgleit OK re` · `Abgleit UK li` · `Abgleit UK re` · `Abgleit OK li` · `Zahngrösse Bem` · `Overjet` · `Overbite` · `einviertel II` · `einhalb II` · `dreiviertel II` · `Kl II` · `plusminus` · `Befundnr` · `Oj wo gemessen` · `Ob wo gemessen` · `IV` · `MLOzuG` · `Mili OK zu G nach li od re` · `Nase` · `Kinn` · `MLUzuO` · `Mili OK zu UK nach li od re` · `Zahn 16 26 in Kl. I` · `skel Klasse` · `skel. Divergenz` · `skel. Gnathie` · `Witswert` · `OK Front zu FH` · `OK Front zu Basis` · `UK Front zu Basis` · `UK Front zu FH` · `Nasenlippenprofil` · `Lippenstufe` · `genereller Profilverlauf` · `Schädelbasis steil flach` · `Schädelbasis kurz lang` · `Wt` · `MH` · `Interesse` · `überwiesen` · `Name der überwiesen` · `wegen` · `handalter` · `sex` · `average` · `Disklusion` · `voraussichtliche grösse leer` · `Etikette sag` · `Etikette vert` · `Etikette transv` · `Etikette OK` · `Etikette UK` · `Etikette i.O.` · `Befundblatt i.O.` · `HR i.O.` · `skip1` · `skip2` · `skip3` · `FR i.O.` · `Fotos i.O.` · `UL vollstaendig`

**Berechnungsfelder (21)** — Formeln 1:1 aus dem DDR übernehmen:

`Zahngrösse` · `Voraussichtliche Grösse` · `Reeval nummer` · `diff Tag` · `diff Monat` · `diff Jahr` · `Alter Jahr` · `Alter Monat` · `tag1` · `monat2` · `monat3` · `jahr2` · `monat4` · `jahr3` · `Alter Jahr Monat` · `Abgleitbewegung sag` · `Abgleitbewegung vert` · `Abgleitbewegung transv` · `Zahngrösse2` · `Abgleitbewegung mm` · `percentage`

**Streichen (13):**

`------------ Alter` · `-------- Steuerung` · `--------- Modelle` · `---------- Röntgen` · `--------- Funktion` · `--------------- ORL` · `---------- Befund` · `---------------------` · `Anamnese Kopie` · `Anamnese Kopie2` · `Anamnese Kopie3` · `Anamnese Kopie4` · `Anamnese Kopie5`


### `Konzept`  ← `Konzeptblatt.fmp12`

123 Felder im Original → **122 übernehmen**, 1 streichen (2 davon Berechnungsfelder).

**Normale Felder (120):**

`KG nr` · `Var Name` · `Staff date` · `Staff mit` · `Konzept` · `Phase I` · `Phase II` · `Phase III` · `Phase IV` · `Verankerungsansprüche OK` · `Verankerungsansprüche UK` · `Mittel` · `Ex 1` · `Ex 2` · `Ex 3` · `Ex 4` · `P O 1` · `P O 2` · `P O 3` · `P O 4` · `P  U 1` · `P  U 2` · `P  U 3` · `P  U 4` · `HG 1` · `HG 2` · `HG 3` · `GZ 1` · `GZ 2` · `GZ 3` · `GZ 4` · `Kat` · `KV` · `off` · `Besonderes` · `zusätzliches 2` · `Ex/NonEx` · `konzept sag` · `konzept vert` · `konzept trans` · `konzept OK` · `konzept UK` · `konzept Overjet` · `konzept Overbite` · `konzept PB OK` · `konzept PB UK` · `SO1` · `SO2` · `SO3` · `SO4` · `SO5` · `SO6` · `SO7` · `SO8` · `SO9` · `SU1` · `SU2` · `SU3` · `SU4` · `SU5` · `SU6` · `SU7` · `SU8` · `SU9` · `Nummer Beh. Konzept` · `skelettal sagittal` · `skelettal vertikal` · `Zeitbedarf 1` · `Zeitbedarf 2` · `Zeitbedarf 3` · `Zeitbedarf 4` · `Zeitbedarf 5` · `HG 4` · `Zeit zur nächsten Sitzung 1` · `Zeit zur nächsten Sitzung 2` · `Zeit zur nächsten Sitzung` · `Zeit zur nächsten Sitzung 3` · `Zeit zur nächsten Sitzung 4` · `Zeit zur nächsten Sitzung 5` · `Zeit zur nächsten Sitzung 6` · `Zeit zur nächsten Sitzung 7` · `Zeit zur nächsten Sitzung 8` · `Zeit zur nächsten Sitzung 9` · `Instr 1` · `Instr 2` · `Instr 3` · `Instr 4` · `Instr 5` · `Instr 6` · `Instr 7` · `Instr 8` · `Instr 9` · `Zeitbedarf 6` · `Zeitbedarf 7` · `Zeitbedarf 8` · `Zeitbedarf 9` · `Laborauftrag 1` · `Laborauftrag 2` · `Laborauftrag 3` · `Laborauftrag 4` · `Laborauftrag 5` · `Laborauftrag 6` · `Laborauftrag 7` · `Laborauftrag 8` · `Laborauftrag 9` · `zusätzliches` · `Konzept von NA` · `sagittal` · `::sag` · `Betrag KV` · `Konzeptblatt sag` · `Konzeptblatt vert` · `Konzeptblatt transv` · `Konzeptblatt OK` · `Konzeptblatt UK` · `Konzepttyp` · `Konzeptblatt i.O.` · `Behandlungskonzept` · `Besonderes_` · `Konzept unterschrieben`

**Berechnungsfelder (2)** — Formeln 1:1 aus dem DDR übernehmen:

`Bolton` · `Info detailliert`

**Streichen (1):**

`HG 3 Kopie`


### `Platzanalyse`  ← `Platzanalyse.fmp12`

131 Felder im Original → **121 übernehmen**, 10 streichen (61 davon Berechnungsfelder).

**Normale Felder (60):**

`KG Nr` · `fünfer im rö` · `milchfünfer im rö` · `milchfünfer im modell` · `fünfer` · `z11` · `z12` · `z13` · `z14` · `z15` · `z16` · `z21` · `z22` · `z23` · `z24` · `z25` · `z26` · `z31` · `z32` · `z33` · `z34` · `z35` · `z36` · `z41` · `z42` · `z43` · `z44` · `z45` · `z46` · `Bolton6 Max Mand X` · `Bolton12 Max Mand X` · `Moyers Prozent` · `Direkt1` · `Direkt2` · `Direkt3` · `Direkt4Q` · `Auswahl MMId` · `Q1Vorh Platz` · `Q2Vorh Platz` · `Q3Vorh Platz` · `Q4Vorh Platz` · `Front Bilanz OK` · `Front Bilanz UK` · `Bemerkungen` · `Zahn der im Rx gemessen` · `Referenzzahn` · `Front Bilanz` · `NB` · `Bem` · `k12` · `k21` · `k11` · `k22` · `k42` · `k41` · `k31` · `k32` · `Platzanalyse i.O.` · `Toothsize i.O.` · `MZ im Rx gemessen`

**Berechnungsfelder (61)** — Formeln 1:1 aus dem DDR übernehmen:

`grösse des fünfers` · `grösse des fünfers Text` · `fünfer Faktor` · `mes dist diff` · `Summe OK 6` · `Summe UK 6` · `Summe OK 12` · `Summe UK 12` · `B6 OK diff` · `B6 UK diff` · `B12 OK diff` · `B12 UK diff` · `B6 OK exess def` · `B6 UK exess def` · `B12 OK ex def` · `B12 UK ex def` · `Bolton 6` · `Bolton 12` · `BS OK 543` · `BS OK 345` · `BS UK 345` · `BS UK 543` · `BS Text` · `OK 3er` · `Ingervall UK Text` · `Ingervall OK Text` · `Ingervall UK` · `Ingervall OK` · `Moyers OK` · `Moyers OK Text` · `Moyers UK` · `Moyers UK Text` · `Muller add OK` · `Muller add UK` · `Muller OK erforderl. Platz` · `Muller UK erforderl. Platz` · `Mueller OK BS` · `Mueller UK BS` · `Muller OK Text` · `Muller UK Text` · `Bilanz Direkt1` · `Bilanz Direkt2` · `Bilanz Direkt3` · `Bilanz Direkt4Q` · `Anschrift Muller Moyers Ingervall direkt` · `OK Muller Moyers Ingervall direkt` · `UK Muller Moyers Ingervall direkt` · `Platz1Quadrant` · `Platz2Quadrant` · `Platz3Quadrant` · `Platz4Quadrand` · `Total UK` · `Total OK` · `UK Front Summe` · `OK Front Summe` · `FS OK 12` · `FS UK 12` · `FS OK Kurvatur gemessen` · `FS UK Kurvatur gemessen` · `FS UK Platzbilanz` · `FS OK Platzbilanz`

**Streichen (10):**

`----Grösse 5er................` · `----Zahngrössen...........` · `.......Bolton..................` · `---  BS ---  3er.............` · `---Platzanalyse ------` · `------- Ingervall` · `------- Moyers ---` · `------- Müller ---` · `------- Direkt ---` · `---- Platzbilanz....................`


### `Toothsize`  ← `toothsizeanalysis moyers.fmp12`

146 Felder im Original → **140 übernehmen**, 6 streichen (24 davon Berechnungsfelder).

**Normale Felder (116):**

`KG Nr` · `Var Step` · `Var Anzahl SD` · `Var Balken` · `mean OK1 m` · `mean OK1 w` · `sd OK1 m` · `sd OK1 w` · `mean OK2 m` · `mean OK2 w` · `sd OK2 m` · `sd OK2 w` · `mean OK3 m` · `mean OK3 w` · `sd OK3 m` · `sd OK3 w` · `mean OK4 m` · `mean OK4 w` · `sd OK4 m` · `sd OK4 w` · `mean OK5 m` · `mean OK5 w` · `sd OK5 m` · `sd OK5 w` · `mean OK6 m` · `mean OK6 w` · `sd OK6 m` · `sd OK6 w` · `mean OK7 m` · `mean OK7 w` · `sd OK7 m` · `sd OK7 w` · `mean OK8 m` · `mean OK8 w` · `sd OK8 m` · `sd OK8 w` · `mean UK1 m` · `mean UK1 w` · `sd UK1 m` · `sd UK1 w` · `mean UK2 m` · `mean UK2 w` · `sd UK2 m` · `sd UK2 w` · `mean UK3 m` · `mean UK3 w` · `sd UK3 m` · `sd UK3 w` · `mean UK4 m` · `mean UK4 w` · `sd UK4 m` · `sd UK4 w` · `mean UK5 m` · `mean UK5 w` · `sd UK5 m` · `sd UK5 w` · `mean UK6 m` · `mean UK6 w` · `sd UK6 m` · `sd UK6 w` · `mean UK7 m` · `mean UK7 w` · `sd UK7 m` · `sd UK7 w` · `mean UK8 m` · `mean UK8 w` · `sd UK8 m` · `sd UK8 w` · `z11p Abweichung` · `z11n Abweichung` · `z12p Abweichung` · `z12n Abweichung` · `z13p Abweichung` · `z13n Abweichung` · `z14p Abweichung` · `z14n Abweichung` · `z15p Abweichung` · `z15n Abweichung` · `z16p Abweichung` · `z16n Abweichung` · `z21p Abweichung` · `z21n Abweichung` · `z22p Abweichung` · `z22n Abweichung` · `z23p Abweichung` · `z23n Abweichung` · `z24p Abweichung` · `z24n Abweichung` · `z25p Abweichung` · `z25n Abweichung` · `z26p Abweichung` · `z26n Abweichung` · `z31p Abweichung` · `z31n Abweichung` · `z32p Abweichung` · `z32n Abweichung` · `z33p Abweichung` · `z33n Abweichung` · `z34p Abweichung` · `z34n Abweichung` · `z35p Abweichung` · `z35n Abweichung` · `z36p Abweichung` · `z36n Abweichung` · `z41p Abweichung` · `z41n Abweichung` · `z42p Abweichung` · `z42n Abweichung` · `z43p Abweichung` · `z43n Abweichung` · `z44p Abweichung` · `z44n Abweichung` · `z45p Abweichung` · `z45n Abweichung` · `z46p Abweichung` · `z46n Abweichung`

**Berechnungsfelder (24)** — Formeln 1:1 aus dem DDR übernehmen:

`Pat mean OK1` · `Pat mean OK2` · `Pat mean OK3` · `Pat mean OK4` · `Pat mean OK5` · `Pat mean OK6` · `Pat mean UK1` · `Pat mean UK2` · `Pat mean UK3` · `Pat mean UK4` · `Pat mean UK5` · `Pat mean UK6` · `Pat sd OK1` · `Pat sd OK2` · `Pat sd OK3` · `Pat sd OK4` · `Pat sd OK5` · `Pat sd OK6` · `Pat sd UK1` · `Pat sd UK2` · `Pat sd UK3` · `Pat sd UK4` · `Pat sd UK5` · `Pat sd UK6`

**Streichen (6):**

`-----------------------` · `-------- OK ---------` · `-------- UK ---------` · `------------------------` · `--------------` · `--------`


### `paAnalyse`  ← `paAnalyse.fmp12`

23 Felder im Original → **23 übernehmen**, 0 streichen (2 davon Berechnungsfelder).

**Normale Felder (21):**

`Name` · `Geb` · `KG Nr` · `Alter` · `Datum PA` · `FW` · `NW` · `MWi` · `MW` · `MMW re` · `MMW li` · `M t J re` · `M t J li` · `IMW` · `ICW` · `tilt li` · `tilt re` · `PA-Bemerkungen` · `PA_Bemerkungen` · `PA Analyse i.O.` · `PA Analyse`

**Berechnungsfelder (2)** — Formeln 1:1 aus dem DDR übernehmen:

`MWi zu MW` · `OPTilt`


## Zu bereinigende Altlasten beim Bau

**Instruktoren als Feldnamen.** In `Stammdaten` sind fünf Instruktoren-
Nachnamen als eigene Felder angelegt, und das Berechnungsfeld
`gewünschter Instruktor` mappt Wochentage fest auf diese Namen. Bei drei
Personalwechseln pro Jahr erzwingt das eine Schemaänderung bei jedem Wechsel.

→ Im Neubau als eigene Tabelle `Instruktor` (oder mindestens Werteliste)
modellieren, mit `Aktiv`-Kennzeichen und Zuordnung über einen Fremdschlüssel.
Die Wochentag-Zuordnung wird dann Daten statt Formel.

**Doppelte Felder.** `paAnalyse` führt `PA-Bemerkungen` *und* `PA_Bemerkungen`,
`Konzeptblatt` führt `Besonderes` und `Besonderes_`. Vor dem Bau fachlich
klären, welches geführt wird, und nur dieses übernehmen.

**`Start KG Nr`** in `Stammdaten` ist auf Eindeutigkeit validiert wie `KG Nr`.
Zweck vor der Übernahme klären — vermutlich Altlast der Nummernvergabe.

## Automatik

### `Neuer Patient`
1. `Patient` anlegen, `KG_Nr` vergeben und auf Eindeutigkeit prüfen
2. `Neue Untersuchung anlegen` aufrufen → erzeugt Sequenz 0 = Anfangsbefund

### `Neue Untersuchung anlegen ( _fk_Patient )`
1. `Sequenz` = Max(vorhandene Sequenz des Patienten) + 1
2. `Untersuchung` anlegen; `_fk_Patient`, `Sequenz`, `Datum` setzen
3. Je einen leeren Datensatz in `Befundblatt`, `Platzanalyse` und `Toothsize`
   anlegen und `_fk_Untersuchung` setzen — **nicht** `paAnalyse`
4. Ab Sequenz ≥ 1: Wechsel von `Patient.Behandlungsstand` auf „Reevaluation"
   **vorschlagen**; abweichenden Stand nicht still überschreiben
5. Auf das Befundblatt-Layout der neuen Untersuchung springen

### `paAnalyse anlegen ( _fk_Untersuchung )`
Manuell über Button auf dem Untersuchungs-Layout. Legt den optionalen
Datensatz an, wenn eine PA-Analyse tatsächlich erhoben wird.

### `Neues Konzept ( _fk_Untersuchung )`
Manuell über Button. `Konzept_Nr` = Max je Patient + 1. Mehrere Konzepte pro
Befundrunde sind zulässig — das bildet das heutige „Konzeptblatt 2" ab.

**Umsetzungshinweis:** Datensätze über dedizierte Utility-Layouts je Tabelle
anlegen (Standard-Pattern für scriptgesteuertes Erzeugen), nicht über
„Erstellung von Datensätzen in dieser Beziehung zulassen". Damit bleibt die
Erzeugung explizit und im Script nachvollziehbar.

## Forschungsmodul (Interim Finale)

794 Felder, 408 Berechnungen, null Beziehungen — die untypischste Datei des
Verbunds und laut Auftraggeber die Forschungsdatenerfassung. Bewusst **nicht**
als weitere Untersuchungs-Komponente modelliert.

Eigener Arbeitsschritt. Vorab zu entscheiden:

- Ob die Erfassung in FileMaker bleibt. Für Studiendaten ist **REDCap** der
  akademische Standard, meist über die Universität kostenlos verfügbar, mit
  Audit-Trail und Pseudonymisierung — und sauber getrennt von der klinischen
  Dokumentation, was rechtlich der deutlichere Schnitt ist.
- Falls FileMaker: die 794 Felder fachlich sichten. Ein grosser Teil dürfte
  redundant zu den Befundmodulen sein und liesse sich als Ableitung statt als
  Erfassung modellieren.
- Diese Datei enthielt die beim Leeren übersehenen Patientendatensätze — vor
  jeder Weitergabe bereinigen.

## Offene Punkte

- **Stichtag für den Umstieg.** Ohne Datenmigration existieren laufende
  Behandlungen in der neuen Lösung nicht. Entweder im Altbestand zu Ende führen
  (Parallelbetrieb) oder beim nächsten Termin neu anlegen — die Entscheidung
  muss vor dem Start stehen.
- **Zukunft des Forschungsmoduls** (siehe oben).
- **Konsolidierung der verteilten Kopien** bleibt zwingend und ist dringender
  als der Neubau. Sie dient jetzt nicht der Migration, sondern der Sicherung
  von Forschungsdaten und Altfällen.
- Die neue Datei sollte **von Beginn an gehostet** laufen, sonst entsteht das
  Kopienproblem sofort wieder.

## Verifikation

- `Neuer Patient` → genau 1 `Patient`, 1 `Untersuchung` mit Sequenz 0, je 1
  leerer verknüpfter Datensatz in `Befundblatt`, `Platzanalyse`, `Toothsize`;
  **kein** `paAnalyse`-Datensatz.
- `Neue Untersuchung anlegen` → Sequenz 1 mit derselben Struktur; die
  Datensätze der Sequenz 0 bleiben unverändert und zugeordnet. Dreimal
  ausführen → Sequenz 1, 2, 3 ohne Lücke oder Kollision.
- `paAnalyse anlegen` und `Neues Konzept` erzeugen je genau einen verknüpften
  Datensatz; mehrfaches `Neues Konzept` zählt `Konzept_Nr` sauber hoch.
- Anlegen einer Reevaluation schlägt den Statuswechsel vor und überschreibt
  einen abweichenden `Behandlungsstand` nicht ohne Bestätigung.
- Für 5 konstruierte Testfälle `Summe OK 6`, `grösse des fünfers` und die
  geschlechtsabhängigen Moyers-Referenzwerte gegen die Altdatei mit identischen
  Eingaben vergleichen.
- Zweiter Patient mit bereits vergebener `KG_Nr` wird abgewiesen.
