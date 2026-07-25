# FileMaker-Lösung — Architekturanalyse

Grundlage: **Datenbank-Design-Bericht (DDR)** aus FileMaker Pro 19, XML-Export
vom 25.07.2026, 29 Dateien. Damit sind Beziehungen, Match-Felder und
Berechnungsformeln belegt statt erschlossen.

Ablageort laut DDR: `M:\_Filemaker_UL\` — alle Dateien liegen im selben
Verzeichnis auf einem gemappten Laufwerk.

## Kernbefund: Der Verbund entstand durch Dateiduplikation

Die 29 Dateien gehen auf rund **9 Schema-Linien** zurück. Der Beleg steht in
den Basistabellennamen — beim Duplizieren einer Datei wandert der alte
Tabellenname mit:

| Datei | Basistabelle | Felder |
|---|---|---|
| `Behandlung.fmp12` | **Konzeptblatt** | 112 |
| `Reevaluation.fmp12` | **Konzeptblatt** | 108 |
| `Konzeptblatt 2.fmp12` | **Konzeptblatt** | 108 |
| `Modelle.fmp12` | **Platzanalyse** | 7 |
| `Photo.fmp12` | **Platzanalyse** | 61 |
| `Röntgenbild.fmp12` | **Platzanalyse** | 10 |
| `Platzanalyse Konzept 2/ZWUL 1/ZWUL 2` | **Platzanalyse** | 111 / 131 / 131 |

`Photo`, `Modelle` und `Röntgenbild` sind also keine eigens entworfenen Module,
sondern ausgeräumte Platzanalyse-Klone. `Bolton` stammt ebenfalls von
Platzanalyse ab, hat die Basistabelle aber umbenannt und ist inzwischen
divergiert (175 Felder / 56 Berechnungen gegenüber 131 / 61).

### Vollständig redundante Dateien (identischer Feldsatz)

| Gruppe | Felder |
|---|---|
| `Platzanalyse` = `Platzanalyse ZWUL 1` = `Platzanalyse ZWUL 2` | 131 |
| `toothsizeanalysis moyers` = `… Konzept 2` = `… Reevaluation` = `… seipel` | 146 |
| `Etiketten` = `Etiketten_sto` | 128 |
| `Konzeptblatt 2` = `Reevaluation` | 108 |
| `kvBlatt Konzept 2` = `kvBlatt Reevaluation` | 108 |
| `paAnalyse Konzept 2` = `paAnalyse Reevaluation` | 19 |
| `Behandlung` = `Termine` | 112 |

Bemerkenswert: `toothsizeanalysis seipel` hat denselben Feldsatz wie die
Moyers-Variante — die beiden Analyseverfahren unterscheiden sich also nicht im
Schema, sondern nur in den hinterlegten Referenzwerten.

## Verknüpfungsschlüssel

**`KG Nr` verbindet praktisch alles.** Von 190 Join-Prädikaten im gesamten
Verbund nutzen 184 dieses Feld. Sämtliche Beziehungen sind vom Typ `Equal`.

Zwei Fallstricke:

- **Schreibweise uneinheitlich:** `Konzeptblatt`, `Reevaluation`, `Termine` und
  `Behandlung` nennen das Feld `KG nr` (kleines n), alle übrigen `KG Nr`. Für
  jede Migration oder API-Anbindung relevant, da Feldnamen unterschieden werden.
- **Einzige inhaltlich andere Beziehung:**
  `kvBlatt.KV zu anderem Konzept = Konzeptblatt.Nummer Beh. Konzept` — verknüpft
  einen Kostenvoranschlag mit einem bestimmten Behandlungskonzept statt mit dem
  Patienten. Existiert in allen drei kvBlatt-Varianten.

In `Stammdaten` ist `KG Nr` als eindeutig und nicht leer validiert.

## Beziehungsstruktur

`Stammdaten` ist der Hub mit 19 Tabellenvorkommen und 16 Beziehungen — mehr als
doppelt so viele wie jede andere Datei. Von dort geht je eine `KG Nr`-Beziehung
zu jeder Fachdatei und zu jeder Phasenvariante.

Die Fachdateien sind zusätzlich **untereinander** vernetzt: Jede referenziert
typischerweise Stammdaten, Befundblatt, Konzeptblatt, kvBlatt, paAnalyse und
Platzanalyse. Es ist ein weitgehend vollvermaschtes Netz, keine Sternstruktur.

`Interim Finale.fmp12` ist die einzige Datei **ohne jede Beziehung** — 1
Tabellenvorkommen, 0 Relationen, keine externen Datenquellen.

## Defekte

### Drei kaputte Beziehungen

Das rechte Match-Feld fehlt, die Beziehung greift also nicht:

| Datei | Beziehung |
|---|---|
| `Stammdaten` | `Stammdaten.KG Nr` → `Befundblatt Reevaluation.???` |
| `Stammdaten` | `Stammdaten.KG Nr` → `Platzanalyse Reevaluation.???` |
| `Behandlung` | `Konzeptblatt.KG nr` → `Konzeptblatt 2.???` |

### Zehn Verweise auf nicht existierende Dateien

`Befundblatt Reevaluation` · `Befundblatt Reevaluation 2` · `Befundblatt ZWUL 1` ·
`Befundblatt ZWUL 2` · `Platzanalyse Reevaluation` · `Platzanalyse Konzept 3` ·
`toothsizeanalysis moyers Konzept 3` · `Eticketten` · `Etticketten II` ·
`Eticketten IV`

Teils Altlasten, teils Tippfehler (`Etticketten`), teils Dateien, die es nie gab
(`Konzept 3`). Vor dem Hosting bereinigen — sonst erzeugen sie beim Öffnen
„Datei suchen"-Dialoge.

### `Interim Finale.fmp12` ist ein Ausreisser

**794 Felder, 408 Berechnungen** — mit Abstand die komplexeste Datei des
Verbunds, bei nur 3 Scripts, 2 Layouts und null Beziehungen. Eine vollständig
denormalisierte Einzeltabelle: alles, was auf dem Blatt erscheint, ist ein
eigenes Feld. Es ist zugleich die Datei, in der beim Leeren zwei
Patientendatensätze übersehen wurden.

## Berechnungsformeln

Vollständig im DDR enthalten, in deutscher Funktionssyntax (`Wenn`, `Falls`,
`LiesAlsZahl`, `Runden`, `Hole`, `MusterAnzahl`). Beispiele:

```
// Platzanalyse — Grösse des bleibenden Fünfers aus Milchfünfer-Messung
grösse des fünfers = (milchfünfer im modell * fünfer im rö) / milchfünfer im rö

// Platzanalyse — Zahnbreitensumme, nur wenn alle Einzelwerte vorliegen
Summe OK 6 = Wenn( LiesAlsZahl(z13)="" ODER … ; ""; z13+z12+z11+z21+z22+z23 )

// toothsizeanalysis — geschlechtsabhängige Referenzwerte
Pat mean OK1 = Wenn( Stammdaten::Geschlecht = "m" ; mean OK1 m ; mean OK1 w )
```

**Wichtig für eine spätere API-Anbindung:** Berechnungen greifen teilweise über
Dateigrenzen zu (`Stammdaten::Geschlecht`, `Konzeptblatt::off`). Die
Zahnnomenklatur folgt dem FDI-Schema (`z13`, `z21`, `z46` …), was für eine
strukturierte Neumodellierung entgegenkommt.

## Konsequenzen für die Migration

1. **Redundanz zuerst abbauen.** Von 29 Dateien sind mindestens 12 exakte
   Duplikate. `ZWUL 1`/`ZWUL 2` und `Etiketten_sto` sind vermutlich vergessene
   Arbeitskopien — vor dem Hosting klären, ob sie noch gebraucht werden.
2. **Verweise bereinigen** — die 10 toten Referenzen und 3 kaputten Beziehungen.
3. **Feldnamen vereinheitlichen** (`KG Nr` vs `KG nr`), sonst schleppt sich der
   Fallstrick in jede Folgelösung.
4. **Die Phasenvarianten sind keine Schemaunterschiede.** `Konzept 2` und
   `Reevaluation` haben identische Feldsätze. In einer Neumodellierung ist das
   ein Datensatzattribut „Phase", keine eigene Datei — das allein reduziert den
   Verbund um rund die Hälfte.
5. **`Interim Finale` gesondert behandeln** — andere Bauart, keine Anbindung,
   und der bekannte Restbestand an Patientendaten.

## Erhebungsstand

29 Dateien mit vollständigem DDR. Nicht erfasst, weil beim Export nicht
vorhanden: `Befundblatt Reevaluation`, `Platzanalyse Reevaluation` — beide
werden von `Stammdaten` referenziert, existieren aber nicht (siehe Defekte).
