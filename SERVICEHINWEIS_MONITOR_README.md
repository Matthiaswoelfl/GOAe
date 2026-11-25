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

## Alternative: Ordner-Synchronisierung statt manuellem Hochladen

Statt jede Excel-Datei einzeln hochzuladen, kann der Button **„Ordner verbinden…"** (oben
rechts, sowie im Leerzustand und in der Kompakt-Leiste) einen lokalen Ordner verbinden und
darin automatisch nach neuen `.xlsx`-Dateien suchen. Das ist gedacht für den Fall, dass die
SharePoint-/Teams-Ablage (z. B. „Servicehinweise" unter „DTA KLE/KLS") über den
OneDrive-Sync-Client lokal gespiegelt wird.

**Voraussetzungen:**
- **Chrome oder Edge** (nutzt die File System Access API; in Firefox/Safari ist der Button
  ausgeblendet, das manuelle Hochladen funktioniert dort unverändert).
- Der SharePoint-Ordner muss **über den OneDrive-Sync-Client lokal synchronisiert** sein
  (im SharePoint-/Teams-„Dateien"-Bereich auf „Synchronisieren" klicken), sodass er als
  normaler Ordner im Windows-Explorer erscheint (typischerweise unter
  `C:\Users\<Name>\OneDrive - ERGO Group AG\…`).

**Ablauf:** Beim ersten Klick auf „Ordner verbinden…" öffnet sich die native
Ordnerauswahl des Betriebssystems — dort den synchronisierten Ordner auswählen. Danach liest
das Dashboard automatisch **alle** `.xlsx`-Dateien im Ordner ein: neue Stichtage werden
importiert, bereits vorhandene Stichtage automatisch übersprungen (kein versehentliches
Überschreiben). Eine Zusammenfassung erscheint als Hinweis unten rechts. Ab dem zweiten Mal
genügt ein Klick auf „Ordner synchronisieren" — die Verbindung wird gemerkt (die Datei selbst
verlässt dabei weiterhin nie den Browser).

**Wichtig — pro Person einzurichten:** Diese Verbindung ist nicht an eine bestimmte Person
gebunden, funktioniert aber auch nicht automatisch für alle. Jede Person, die das Dashboard
nutzt, braucht Zugriff auf die SharePoint-Bibliothek, muss den Ordner selbst einmalig über
OneDrive synchronisieren und beim ersten Öffnen des Dashboards einmal selbst auf „Ordner
verbinden…" klicken (Browser-Berechtigungen für Ordnerzugriff sind pro Person/Browser-Profil
und lassen sich nicht zentral vorab erteilen). Jede Person sieht dabei ihren eigenen lokalen
Sync-Pfad — das Dashboard braucht dafür keinen festen Pfad.

## Datenschutz

- Die Excel-Datei selbst wird **nicht gespeichert** — nur die für die Auswertung nötigen,
  aggregierten Felder (siehe Abschnitt „Was wird gespeichert" unten).
- **Versicherungsnummern (VNR), `ZUST`, `TA_KOMB_EB1` und rohe Detailtexte werden nie
  persistiert.** Detailtexte werden nur in bereits ausgewerteter Form (erkannter
  Grund-Schlüssel je Spalte U–X, siehe `GRUND_REGELN`) gespeichert.
- **Rechnungsbeträge (`RECHN_BETR`) werden seit v2 bewusst in aggregierter Form
  persistiert** — je Beleg nur der höchste Betrag über seine Servicehinweis-Zeilen, in
  Cent gerundet (`belegBetragCt`), für den Betragsfilter/die Betragsklassen-Auswertung.
  Einzelne Zeilenbeträge, Nachkommastellen jenseits der Rundung und jede Zuordnung zu
  Name/VNR werden nicht gespeichert.
- Die Rohdatei verlässt beim Einlesen (Web Worker) nie den Browser; es gibt keinen Server,
  keine Netzwerk-Übertragung.
- Reale Testdateien gehören lokal in einen Ordner `daten/` — dieser ist über `.gitignore`
  von Commits ausgeschlossen und darf nie in das Repository gelangen.

### Was wird gespeichert (je Snapshot)

Stichtag, Dateiname/Importzeitpunkt/Zeilenzahl, je Beleg der höchste Rechnungsbetrag in
Cent (`belegBetragCt`), sowie je Hinweis-Zeile: Beleg-Index (auf `DOK_NR` verweisend, die
Nummer selbst nur als anonyme Zähleinheit), Dok.-Art, Servicehinweis (Kategorie × Code),
die geordnete Folge erkannter Grund-Schlüssel je Detailtextspalte (U–X) und deren Anzahl.
Siehe `ShMonitorCore.buildSnapshot()` im Quelltext für das
genaue Schema.

## Konfiguration

Alles Fachliche steht zentral im eingebetteten `core.js`-Block der HTML-Datei:

- **`FELD_LABEL`** — sprechende Bezeichnungen für die Feldcodes aus den Detailtexten
  (`Feld X in Y fehlt`). Unbekannte Codes werden roh angezeigt; neue Codes einfach ergänzen.
  `BEH-PE-NR` ist als „Acxiom-ID" beschriftet.
- **`GRUND_REGELN`** (v2) — priorisierte Regex-Liste, die jeden nicht leeren Detailtext
  (Spalten U–X) auf genau einen „Grund-Schlüssel" abbildet: fehlende Pflichtfelder (wie
  `FELD_LABEL`) UND feiner klassifizierte Freitext-Gründe (Vorgang, Tarif, GO-Faktor,
  GOÄ-Position, Diagnose, Rezept, Sonstiges). Reihenfolge der Liste = Priorität; vom
  Fachbereich ohne Code-Umbau erweiterbar.
- **`BETRAG_KLASSEN`** (v2) — die 6 Rechnungsbetrags-Schwellen für den Betragsfilter
  (< 50 € … > 1.000 €, Grenzen inklusive der unteren Schwelle).
- **`KATEGORIE_LABEL`** — Anzeige-Label je Servicehinweis-Kategorie (`Z31`, `FRAGL`, `TEILW`).
- Design-Tokens (DKV-Farben, Radien, Schatten, inkl. der 10 `--grund-N`-Farben für die
  Grund-Kombinationen) stehen im `:root{}`-Block im `<style>`-Teil ganz oben in der
  HTML-Datei — dort gegen offizielle Werte aus dem DKV-Brandportal tauschbar.

## Umfang dieser Version (v2)

Enthalten: Ansicht „Übersicht" (Kennzahlen + Tabelle + Diagramme, inkl. Betragsklassen-
Diagramm), Ansicht „Servicehinweise" (vollständige Tabelle mit Such-/Sortier-/Gruppier-
funktion, aufklappbarem Detailpanel je Servicehinweis mit Verteilung nach Anzahl
Hellwerdegründe, CSV-Export), Ansicht „Konkrete Hellwerdegründe" (Grund-Kombinationen je
Servicehinweis, gruppiert nach Anzahl der Gründe 1–4, mit Top-N-Auswahl, Farblegende und
CSV-Export), Ansicht „Verlauf" (Zeitreihe über mehrere Stichtage, inkl. gestapeltem
Balkendiagramm der Grund-Anzahl je Stichtag), vollständige Sidebar-Filterung (Zeitscheibe,
Dok.-Art, Kategorie, Servicehinweis, Drilldown auf fehlende Felder, Rechnungsbetrag mit
optional separat ausgewiesenen 0-€-Belegen), mehrtägige Snapshot-Verwaltung mit
Export/Import, optionale Ordner-Synchronisierung für automatischen Mehrfach-Import aus einem
lokal gespiegelten SharePoint-/OneDrive-Ordner (Chrome/Edge, siehe eigener Abschnitt oben).

Bewusst **nicht** enthalten (siehe Build Plan v1 + v2): Tarif-Auswertung (`TA_KOMB_EB1`,
weder als eigene Analyse noch als Gruppierungsdimension der Grund-Kombinationen) und
Bewertung der Hellwerdegründe nach Automatisierbarkeit — beides auf ausdrücklichen Wunsch
für eine spätere Ausbaustufe zurückgestellt. Ebenfalls optional und noch offen:
Altersstruktur des Rückstands (aus `EING_DAT`).

## Tests

`scripts/` (im Build-Verzeichnis, nicht Teil der ausgelieferten HTML-Datei) enthält die
Playwright- und Node-Testskripte, die alle Sollwerte aus dem Build Plan gegen die reale
Beispieldatei prüfen.
