# Claris FileMaker Server – Installationsanleitung

Diese Anleitung beschreibt die Einrichtung eines Claris FileMaker Servers zum Hosten
des mehrteiligen FileMaker-Lösungsverbunds (Stammdaten + Fachblatt-Dateien) für ca.
15 gleichzeitige Clients.

## Voraussetzungen

- **Lizenz:** Claris FileMaker Server, Concurrent-Connections-Lizenz für 15 Nutzer.
- **Plattform:** Windows Server, macOS Server oder Linux (Ubuntu LTS, seit FileMaker
  Server 19.6 offiziell unterstützt, auch als Docker-Container). Alternative ohne
  eigenen Server: Claris Cloud (Hosting durch Claris).
- **Hardware:** mind. 4 CPU-Kerne, 8–16 GB RAM (abhängig von Nutzerzahl/Datenmenge),
  SSD-Speicher.
- **Netzwerk:** statische interne IP-Adresse im selben Netz wie die Clients (oder
  VPN-Zugriff für Aussenstandorte).

## Benötigte Firewall-Ports

| Port | Zweck |
|---|---|
| 5003 | FileMaker-Client-Datenbankverbindungen |
| 16000–16001 | Admin Console / interne Server-Kommunikation |
| 80 / 443 | WebDirect (Browser-Zugriff), HTTPS empfohlen |
| 5013 | ODBC/JDBC (falls externe Reports benötigt) |

## Schritt 1 — Claris FileMaker Server installieren

1. Installationspaket von Claris herunterladen (Lizenzschlüssel bereithalten).
2. Installer ausführen, Rolle wählen (Single-Machine-Deployment reicht für 15
   Clients meist aus).
3. Admin-Konto und Passwort für die Admin Console festlegen.
4. Nach der Installation die Admin Console im Browser öffnen:
   `https://<server-ip>:16001`.
5. SSL-Zertifikat einrichten (self-signed für Tests, gültiges Zertifikat für den
   Produktivbetrieb — Pflicht, da Patientendaten verarbeitet werden).

## Schritt 2 — Datenbankdateien vorbereiten

Der bestehende Dateisatz hat eine lange Historie (mehrere Formatwechsel seit
FileMaker 5/6/7, wiederholte "falsch geschlossene Datei"-Ereignisse in den
Wiederherstellungs-Logs). Vor dem Hosten:

1. **Konsolidieren:** Nur den aktuell gültigen Dateisatz auswählen — alte Kopien
   und Duplikate gehören ins Archiv, nicht auf den Server.
2. **Wiederherstellen/Recover:** Jede Datei in einer aktuellen FileMaker-Pro-Version
   öffnen und über *Datei → Wiederherstellen* prüfen/reparieren lassen, bevor sie
   auf den Server kommt.
3. **Format-Kompatibilität sicherstellen:** Aktuelle Server-Versionen benötigen ein
   zeitgemässes Dateiformat — ggf. mit "Speichern unter" in der neuesten
   FileMaker-Pro-Version neu speichern.
4. **Externe Datenquellen prüfen:** Die Dateien referenzieren sich gegenseitig per
   Dateiname (Stammdaten als zentrale Datei, referenziert von den meisten
   Fachblatt-Dateien; Photo ebenfalls von den meisten Fachblatt-Dateien
   referenziert). Sicherstellen, dass alle referenzierten Dateien gemeinsam im
   selben Ordner/Host liegen, sonst brechen die Beziehungen.

## Schritt 3 — Dateien hosten

1. In der Admin Console: *Datenbanken → Datenbank hochladen*.
2. Alle zusammengehörigen Dateien gemeinsam hochladen (wegen der
   Cross-File-Referenzen).
3. Automatisches Öffnen beim Serverstart aktivieren.
4. Zugriffsrechte pro Datei/Konto in der Admin Console kontrollieren — individuelle
   Konten mit Privilegien-Sets statt eines gemeinsamen Admin-Kontos für alle
   Nutzer.

## Schritt 4 — Sicherheit

Da patientenbezogene Daten verarbeitet werden:

- Verschlüsselung im Ruhezustand aktivieren (Server-seitig verfügbar seit
  FileMaker Server 18+).
- SSL/TLS für alle Client-Verbindungen erzwingen.
- Individuelle Benutzerkonten mit rollenbasierten Privilegien statt eines
  Admin-Kontos für alle.
- Serverstandort in der Schweiz/EU (Patientengeheimnis/nDSG-Anforderungen); bei
  Fremd-Hosting Auftragsverarbeitungsvertrag abschliessen.

## Schritt 5 — Backup einrichten

- In der Admin Console: *Zeitplan → Progressive Backups* aktivieren (z. B. alle
  15 Minuten inkrementell plus tägliches Vollbackup).
- Backup-Ziel auf separatem Speicher/Standort (nicht nur lokal auf demselben
  Server).

## Schritt 6 — Clients anbinden

1. FileMaker Pro (Claris Pro) auf allen Arbeitsplätzen installieren.
2. *Datei → Remote öffnen* → Server in der Liste auswählen (oder per
   Favoriten-Host-Adresse).
3. Bei Bedarf WebDirect-URL für Browser-Zugriff kommunizieren, falls kein
   FileMaker-Pro-Client auf jedem Arbeitsplatz installiert werden soll.

## Schritt 7 — Test & Monitoring

- Mit 2–3 Clients gleichzeitig testen (Datensätze anlegen/bearbeiten), dann
  schrittweise auf alle 15 hochskalieren.
- Admin Console → *Clients*-Tab zur Überwachung aktiver Verbindungen nutzen.
