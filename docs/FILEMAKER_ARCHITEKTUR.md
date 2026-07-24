# FileMaker-Lösung — Architekturanalyse

Analyse des Dateiverbunds, rekonstruiert direkt aus den `.fmp12`-Binärdateien.
Ersetzt die frühere, aus dem Konvertierungs-Log von 2004 abgeleitete Vermutung.

## Analysemethode

FileMaker legt Bezeichner in `.fmp12` **XOR-0x5A-verschleiert** ab — deshalb
findet ein normaler Textscan nichts. Nach dem Umkehren dieser Verschleierung
liegen die Bezeichner als **längen-präfixierte Strings** vor: ein Längenbyte
(ebenfalls XOR 0x5A), dann der Name in Latin-1.

Die Kategorie ergibt sich aus den Markerbytes direkt davor:

| Marker (entschlüsselt) | Inhalt |
|---|---|
| `da 5c 5b` | Feldnamen und Layout-Beschriftungen |
| `da 5c 4a` | Externe Dateiverweise, danach Scriptnamen |
| `5a 5c 4a` | Layoutnamen (vermischt mit Schriftarten-/Menü-Katalog) |

Der Block der externen Dateiverweise endet jeweils mit dem Selbstverweis
`Files/<Dateiname>/`; alles danach sind Scriptnamen. Dieses Muster ist über
alle untersuchten Dateien konsistent.

## Verknüpfungsschlüssel

Alle acht untersuchten Dateien enthalten ein Feld **`KG Nr.`**
(Krankengeschichte-Nummer, in Varianten `KG Nr`, `KG Nr:`, `KG.Nr:`). Das ist
der gemeinsame Patientenschlüssel, über den der Verbund zusammenhängt.
`Stammdaten` führt zusätzlich `Befundnummer` und `Konzept Nr.` als eigene
Ordnungsnummern.

## Dateibeziehungen (verifiziert)

### Stammdaten.fmp12 — Hub

Referenziert als einzige Datei praktisch den gesamten Verbund inklusive aller
Behandlungsphasen-Varianten:

Befundblatt · Konzeptblatt · kvBlatt · paAnalyse · Platzanalyse · Photos ·
Interim Finale · Reevaluation · Konzeptblatt 2 · Befundblatt Konzept 2 ·
Befundblatt Reevaluation · Platzanalyse Konzept 2 · Platzanalyse Reevaluation ·
paAnalyse Konzept 2 · paAnalyse Reevaluation · kvBlatt Konzept 2 ·
kvBlatt Reevaluation · eine personalisierte Interim-Finale-Variante

### Fachblatt-Dateien

| Datei | Externe Dateiverweise |
|---|---|
| `Befundblatt` | Stammdaten, Konzeptblatt, paAnalyse, Platzanalyse, kvBlatt, Etiketten |
| `Konzeptblatt` | Stammdaten, Befundblatt, kvBlatt, Platzanalyse, paAnalyse + sämtliche `Konzept 2`-/`Reevaluation`-Varianten |
| `Platzanalyse` | Stammdaten, Befundblatt, Konzeptblatt, kvBlatt, paAnalyse, toothsizeanalysis moyers, toothsizeanalysis seipel, Eticketten (I/II/IV) |
| `paAnalyse` | Stammdaten, Befundblatt, Konzeptblatt, kvBlatt, Platzanalyse |
| `toothsizeanalysis moyers` | Stammdaten, Befundblatt, Konzeptblatt, kvBlatt, paAnalyse, Platzanalyse |
| `Bolton` | identisch mit Platzanalyse — siehe unten |
| `Interim Finale` | **keine** — eigenständig |

### Zwei Korrekturen gegenüber der Log-Rekonstruktion

- **`Bolton.fmp12` ist ein Duplikat von `Platzanalyse.fmp12`.** Die Datei gibt
  ihren eigenen internen Pfad als `Files/Platzanalyse/` an und hat denselben
  Feld- und Scriptbestand. Es ist keine eigenständige Analysedatei, sondern
  eine abgespaltene Kopie.
- **`Interim Finale.fmp12` ist tatsächlich eigenständig** — bestätigt, keine
  externen Datenquellen, nur 5 Scripts.

## Navigationsmuster

Der Verbund wird nicht über Beziehungen allein zusammengehalten, sondern über
Sprung-Scripts. Jede Fachdatei enthält eine gleichförmige Serie
(`zu den Stammdaten`, `zum Befundblatt`, `zum Konzeptblatt`, `zum KV-Blatt`,
`zum PA Analyseblatt`, `zur Platzanalyse`, `Finden in <Datei>`).

`Konzeptblatt` hat mit 61 Scripts den grössten Bestand und fungiert als
Navigationsdrehscheibe zwischen den Behandlungsphasen (`zur Reevaluation`,
`zum befundblatt Konzept 2`, `gehe zu Platz&Bolton Reevaluation` usw.).
`Stammdaten` folgt mit 44 Scripts.

## Layouts je Datei

| Datei | Layouts |
|---|---|
| `Stammdaten` | Patienten (Stammdaten) sh, Liste Befunde, Liste Patient, Status- Liste, Überblick Konzept, Stud. Kurs, Patienten (Abrechnung), Status- Liste sh, Patienten (Stammdaten) Kopie, Liste Patient sh |
| `Befundblatt` | Befundblatt, Liste Patienten, Befundblatt neu, Befundblatt sh, Etiketten |
| `Konzeptblatt` | Konzeptbatt sh, Liste Patienten, Konzeptbatt neu, Liste Patienten sh |
| `Platzanalyse` | Platzanalyse, Liste, Platzanalyse sh |
| `Bolton` | Liste, Bolton sh |
| `paAnalyse` | PA- Analyse, PA- Analyse sh |
| `toothsizeanalysis moyers` | tooth size analysis, Tabelle - Info, + `sh`-Varianten |
| `Interim Finale` | Layout-#1, Interim sh |

Das Suffix `sh` markiert durchgehend eine zweite Layoutvariante je Zweck
(vermutlich für die gehostete/geteilte Nutzung).

## Fachliche Feldstruktur (Auszug)

- **Stammdaten** (94 Bezeichner) — Falladministration und Workflow-Steuerung:
  Behandlungsstand, Nächster Schritt, Nächster Termin, Unterlagen vollständig,
  Staff geplant/gemacht, Instruktor definitiv, Konzept unterschrieben,
  Zeigen des Falles am, Studenten-Kurs/Gruppenassistent, Abrechnung,
  Behandlungskosten, Kategorie, Spez Fall.
- **Befundblatt** (78) — klinische Befundung: MORPHOLOGISCH, FUNKTIONELL,
  skelettal/dental, sag./transv., Profil, Primärkontakte, Abgleitbewegung,
  Weichteile, Tonsillen/Adenoide/Atmung, WITS, H-Diff, Schädelbasis,
  Hand-alter (mit Referenztabelle `accelerated`/`average`/`retarded`).
- **Konzeptblatt** (83) — Behandlungsplanung: Phase, Verankerungsansprüche,
  Hauptprobleme, Überlegungen, Therapie, Bilanz, Kostenvoranschlag,
  Apparaturen-Textbausteine (Mono-HG, Lipbumper, StaBo, FDA, Ablösesect.),
  MKG-Workflow.
- **Platzanalyse / Bolton** (47/38) — Modellanalyse: Zahngrössen,
  Vorhandener Platz, Bolton-Analyse, Referenzzahn, mesio-distale Differenz,
  Kurvatur und Platzbilanz je Kiefer, Verfahren nach Ingervall/Müller/Moyers.
- **paAnalyse** (28) — posteroanteriore Fernröntgenanalyse: facial/nasal/
  maxillary/mandibular width, intermolar/intercanine width, occlusal plane tilt.
- **toothsizeanalysis moyers** (26) — Zahnbreiten-Prognose nach
  R. E. Moyers et al. 1976, geschlechtsgetrennt, mit Balkendarstellung.

## Grenzen dieser Analyse

- **Berechnungsformeln sind nicht auslesbar.** FileMaker speichert sie
  tokenisiert mit Feld-IDs statt als Text; sie werden erst beim Anzeigen
  zusammengesetzt. Für die Formeln führt kein Weg an FileMaker Pro vorbei.
- **Beziehungsdefinitionen** (welches Tabellenvorkommen über welches Feldpaar
  verknüpft ist) konnten nicht strukturiert extrahiert werden — belegt ist der
  gemeinsame Schlüssel `KG Nr.` und welche Dateien einander referenzieren,
  nicht die exakte Verknüpfungsbedingung je Beziehung.
- Untersucht wurden die 8 Dateien des Satzes `Filemaker_Relevant`. Die übrigen
  Dateien des Verbunds (kvBlatt, Photos, Termine, Behandlung, Etiketten,
  Modelle, Röntgenbild, Reevaluation, die `Konzept 2`-/`Reevaluation`-Varianten,
  `toothsizeanalysis seipel`) sind hier nur als Verweisziele belegt, nicht
  selbst analysiert.
