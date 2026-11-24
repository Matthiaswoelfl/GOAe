# Servicehinweis-Monitor (KLTB0570, RE-ambulant)

Lokales, browserbasiertes Dashboard für den Rückstand heller ambulanter Arztrechnungen
(Belege mit Servicehinweis in der KLTB0570), je Stichtag. Eine einzige Datei
(`servicehinweis_monitor.html`), kein Server, keine Installation — einfach im Browser öffnen.

## Bedienung in 3 Schritten (täglicher Abzug)

1. **Excel hochladen** (Button oben rechts oder Drag & Drop) — die KLTB0570-Exportdatei
   (`KLTB0570_am_JJJJMMTT_DOK….xlsx`) auswählen.
2. Im Bestätigungsdialog **Stichtag, Zeilen- und Belegzahl prüfen** (der Stichtag wird aus dem
   Dateinamen erkannt, lässt sich aber bei Bedarf überschreiben) und auf **Speichern** klicken.
3. Fertig — der neue Stichtag ist sofort in der Zeitscheibe, in „Servicehinweise" und im
   „Verlauf" verfügbar. Für denselben Stichtag erneut hochladen ersetzt den vorhandenen Abzug
   (nach Rückfrage).

Alle Abzüge bleiben lokal im Browser gespeichert (IndexedDB) — auch nach einem Neuladen der
Seite. Über „Abzüge verwalten…" in der Sidebar lassen sich alle gespeicherten Stichtage
einsehen, löschen sowie als JSON exportieren/importieren (z. B. um den Verlauf mit
Kolleg:innen zu teilen, ohne die Excel-Dateien weiterzugeben).

## Datenschutz

- Die Excel-Datei selbst wird **nicht gespeichert** — nur die für die Auswertung nötigen,
  aggregierten Felder (siehe Abschnitt „Was wird gespeichert" unten).
- **Versicherungsnummern (VNR), Beträge, `ZUST`, `TA_KOMB_EB1` und rohe Detailtexte werden
  nie persistiert.** Detailtexte werden nur in bereits ausgewerteter Form (erkannter
  Feldcode bzw. Sammel-Flag „sonstige") gespeichert.
- Die Rohdatei verlässt beim Einlesen (Web Worker) nie den Browser; es gibt keinen Server,
  keine Netzwerk-Übertragung.
- Reale Testdateien gehören lokal in einen Ordner `daten/` — dieser ist über `.gitignore`
  von Commits ausgeschlossen und darf nie in das Repository gelangen.

### Was wird gespeichert (je Snapshot)

Stichtag, Dateiname/Importzeitpunkt/Zeilenzahl, sowie je Hinweis-Zeile: Beleg-Index
(auf `DOK_NR` verweisend, die Nummer selbst nur als anonyme Zähleinheit), Dok.-Art,
Servicehinweis (Kategorie × Code), ob ein konkreter Hellwerdegrund vorliegt und welche
Feldcodes dafür erkannt wurden. Siehe `ShMonitorCore.buildSnapshot()` im Quelltext für das
genaue Schema.

## Konfiguration

Alles Fachliche steht zentral im eingebetteten `core.js`-Block der HTML-Datei
(Suche nach `KATEGORIE_LABEL`, `FELD_LABEL`, `FELD_DETAIL_RE`):

- **`FELD_LABEL`** — sprechende Bezeichnungen für die Feldcodes aus den Detailtexten
  (`Feld X in Y fehlt`). Unbekannte Codes werden roh angezeigt; neue Codes einfach ergänzen.
  Hinweis: Das Label „Behandler-Nr." für `BEH-PE-NR` ist ein Vorschlag und sollte vom
  Fachbereich noch bestätigt werden (siehe Build Plan, Abschnitt 12).
- **`KATEGORIE_LABEL`** — Anzeige-Label je Servicehinweis-Kategorie (`Z31`, `FRAGL`, `TEILW`).
- Design-Tokens (DKV-Farben, Radien, Schatten) stehen im `:root{}`-Block im `<style>`-Teil
  ganz oben in der HTML-Datei — dort gegen offizielle Werte aus dem DKV-Brandportal
  tauschbar, sobald vorhanden.

## Umfang dieser Version

Enthalten: Ansicht „Übersicht" (Kennzahlen + Tabelle + Diagramme), Ansicht
„Servicehinweise" (vollständige Tabelle mit Such-/Sortier-/Gruppierfunktion, CSV-Export),
Ansicht „Verlauf" (Zeitreihe über mehrere Stichtage), vollständige Sidebar-Filterung
(Zeitscheibe, Dok.-Art, Kategorie, Servicehinweis, Drilldown auf fehlende Felder),
mehrtägige Snapshot-Verwaltung mit Export/Import.

Bewusst **nicht** enthalten (siehe Build Plan): Tarif-Auswertung (`TA_KOMB_EB1`) und
Bewertung der Hellwerdegründe nach Automatisierbarkeit — beides auf Wunsch für eine
spätere Ausbaustufe zurückgestellt. Ebenfalls optional und noch offen: Altersstruktur des
Rückstands (Phase 5, aus `EING_DAT`).

## Tests

`scripts/` (im Build-Verzeichnis, nicht Teil der ausgelieferten HTML-Datei) enthält die
Playwright- und Node-Testskripte, die alle Sollwerte aus dem Build Plan gegen die reale
Beispieldatei prüfen.
