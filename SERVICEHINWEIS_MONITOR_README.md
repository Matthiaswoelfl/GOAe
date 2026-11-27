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
darin automatisch nach `KLTB0570`-Excel-Dateien suchen. Das funktioniert mit **jedem**
Ordner, der diese Dateien enthält — zum Beispiel:
- demselben Ordner, in dem auch diese HTML-Datei selbst liegt (die einfachste Variante, wenn
  HTML-Datei und tägliche KLTB0570-Exporte ohnehin zusammen abgelegt werden), oder
- einer SharePoint-/Teams-Ablage (z. B. „Servicehinweise" unter „DTA KLE/KLS"), die über den
  OneDrive-Sync-Client lokal gespiegelt wird.

**Voraussetzungen:**
- **Chrome oder Edge** (nutzt die File System Access API; in Firefox/Safari ist der Button
  ausgeblendet, das manuelle Hochladen funktioniert dort unverändert).
- Der Ordner muss als **normaler lokaler Ordner** existieren (im Windows-Explorer sichtbar) —
  bei einer SharePoint-Ablage also vorher über „Synchronisieren" im Browser/Teams verbunden
  (typischerweise unter `C:\Users\<Name>\OneDrive - ERGO Group AG\…`).
- Aus Sicherheitsgründen kann keine Webseite automatisch „ihren eigenen Ordner" erkennen —
  der einmalige Klick auf „Ordner verbinden…" mit Auswahl des Ordners lässt sich nicht
  umgehen, auch wenn HTML- und Excel-Dateien zusammen liegen.

**Ablauf:** Beim ersten Klick auf „Ordner verbinden…" öffnet sich die native
Ordnerauswahl des Betriebssystems — dort den Ordner mit den KLTB0570-Dateien auswählen.
Danach liest das Dashboard automatisch **alle** Dateien ein, deren Name mit `KLTB0570`
beginnt und auf `.xlsx` endet (andere Dateien im selben Ordner werden ignoriert): neue
Stichtage werden importiert, bereits vorhandene Stichtage automatisch übersprungen (kein
versehentliches Überschreiben). Eine Zusammenfassung erscheint als Hinweis unten rechts. Ab
dem zweiten Mal genügt ein Klick auf „Ordner synchronisieren" — die Verbindung wird gemerkt
(die Datei selbst verlässt dabei weiterhin nie den Browser).

**Wichtig — pro Person einzurichten:** Diese Verbindung ist nicht an eine bestimmte Person
gebunden, funktioniert aber auch nicht automatisch für alle. Jede Person, die das Dashboard
nutzt, braucht Zugriff auf die SharePoint-Bibliothek, muss den Ordner selbst einmalig über
OneDrive synchronisieren und beim ersten Öffnen des Dashboards einmal selbst auf „Ordner
verbinden…" klicken (Browser-Berechtigungen für Ordnerzugriff sind pro Person/Browser-Profil
und lassen sich nicht zentral vorab erteilen). Jede Person sieht dabei ihren eigenen lokalen
Sync-Pfad — das Dashboard braucht dafür keinen festen Pfad.

## Management Summary (v3) und die Erkenntnis-Engine

Der erste Reiter **„Management Summary"** ist die Startansicht. Er zeigt:
- eine **Kennzahlenleiste** (Helle Belege, mit konkretem Grund, Ø Alter (Median), Δ Vortag),
- **3–5 Kernaussagen**, automatisch ausgewählt und nach betroffener Belegzahl sortiert
  (Kennzahl „Mengengerüst" steht immer an erster Stelle),
- ein **einklappbares Akkordeon „Erkenntnisse je Dok.-Art"** (eine Karte je Dok.-Art, mit der
  gleichen Übersichtstabelle wie im Reiter „Übersicht", gefiltert auf diese Dok.-Art, plus
  bis zu 4 weiteren Erkenntnissen),
- ein freies **Kommentarfeld** je Stichtag (manuell, wird im Snapshot gespeichert und bleibt
  beim JSON-Export erhalten).

**Oberste Regel:** Jede angezeigte Aussage ist aus den geladenen Daten berechnet. Es gibt
keine frei formulierten oder LLM-generierten Texte und keine hart codierten Zahlen — nur
feste Textvorlagen mit Platzhaltern, die ausschließlich aus einer Kennzahlen-Funktion befüllt
werden. Lässt sich eine Vorlage nicht vollständig befüllen, wird sie nicht angezeigt (kein
leerer oder kaputter Text).

**Wie die Erkenntnisse entstehen** (im eingebetteten `core.js`-Block, Suche nach `REGELN`):
- **`computeKennzahlen(snapshot, filter, dokArtCode)`** liefert alle Rohkennzahlen (Belege,
  Anteile, Top-Codes, Feld-Cluster, Altersverteilung, …), wahlweise für den gesamten Bestand
  oder eine einzelne Dok.-Art.
- **`REGELN`** ist eine Liste fester Regeln (R01–R12: Mengengerüst, Kategorie-Mix,
  Konzentration, Einzelhinweis-Hebel, LEA189-Struktur/-Potenzial, Feld-Cluster, ohne
  konkreten Grund, Rechnungsbetrag 0 €, Alter, Vortagsvergleich, Fokus-Dok.-Arten). Jede
  Regel hat eine Bedingung (wann sie greift), eine `werte()`-Funktion und feste
  Text-Vorlagen für Schlagzeile und Befund. Neue Regeln werden hier als weiterer
  Listeneintrag ergänzt — der Fachbereich kann die Texte direkt lesen und anpassen.
- **`MIN_BASIS` (= 30)** unterdrückt eine Regel, wenn zu wenige Belege dahinterstehen;
  zwischen 30 und 99 erscheint der Hinweis „geringe Fallzahl".
- **`HEBEL_ZUORDNUNG`** ordnet einzelnen Servicehinweis-Codes ein Hebel-Label zu (aktuell nur
  `LEA189` → „Datenqualität/Belegerfassung"); weitere Zuordnungen sind einfach ergänzbar.
- Jede Erkenntnis hat ein ⓘ-Symbol (zeigt Regel-ID und Basiszahl beim Hovern) und einen Link
  „Daten ansehen", der automatisch in den passenden Reiter springt und die dazu passenden
  Filter setzt.

**Bewusst nicht umgesetzt** (auf ausdrücklichen Wunsch, siehe Build Plan v3 Abschnitt 9):
eine separate „Einordnung"-Schicht mit fachlich freizugebenden Schlussfolgerungs-Texten und
Schwellenwerten, sowie Hochrechnungen aufs Jahr (z. B. „~63,2k Belege zusätzlich p.a.") — beide
erfordern weitere fachliche Abstimmung und lassen sich bei Bedarf später als zusätzliche
Regel-Eigenschaften ergänzen, ohne die bestehende Struktur zu ändern.

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
die geordnete Folge erkannter Grund-Schlüssel je Detailtextspalte (U–X), deren Anzahl sowie
der Abstand des Eingangsdatums (`EING_DAT`) zum Stichtag in Tagen (für die Altersauswertung,
v3). Optional ein frei eingegebener **Kommentartext des Fachbereichs** je Stichtag
(`kommentar`, nur wenn über das Kommentarfeld der Management Summary explizit gespeichert).
Siehe `ShMonitorCore.buildSnapshot()` im Quelltext für das genaue Schema.

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

## Umfang dieser Version (v3)

Enthalten: Ansicht **„Management Summary"** (Startansicht — Kennzahlenleiste, automatisch
erzeugte Kernaussagen, einklappbares Akkordeon „Erkenntnisse je Dok.-Art", manuelles
Kommentarfeld je Stichtag, siehe eigener Abschnitt oben), Ansicht „Übersicht" (Kennzahlen +
neu gestaltete Übersichtstabelle im Format „Kategorie | Anzahl | Kommentar" mit Abschnitten
„Was haben wir betrachtet", „Verteilung der konkreten Hellwerdegründe auf Belegebene" und
„Welche Angaben fehlen am häufigsten", regelbasierte „Erläuterung" statt statischer
Begriffserklärungen, plus Diagramme inkl. Betragsklassen-Diagramm), Ansicht
„Servicehinweise" (vollständige Tabelle mit Such-/Sortier-/Gruppierfunktion, aufklappbarem
Detailpanel je Servicehinweis mit Verteilung nach Anzahl Hellwerdegründe, CSV-Export),
Ansicht „Konkrete Hellwerdegründe" (Grund-Kombinationen je Servicehinweis, gruppiert nach
Anzahl der Gründe 1–4, mit Top-N-Auswahl, Farblegende und CSV-Export), Ansicht „Verteilung
je Dok.-Art" (Kreuztabelle: Anzahl heller Belege je Dok.-Art × Anzahl konkreter
Hellwerdegründe 1–4, optional auf einen Servicehinweis eingeschränkt — die
Servicehinweis-Auswahl ist mit „Konkrete Hellwerdegründe" synchron, CSV-Export), Ansicht
„Verlauf" (Zeitreihe über mehrere Stichtage, inkl. gestapeltem Balkendiagramm der
Grund-Anzahl je Stichtag), vollständige Sidebar-Filterung (Zeitscheibe, Dok.-Art, Kategorie,
Servicehinweis, Drilldown auf fehlende Felder, Rechnungsbetrag mit optional separat
ausgewiesenen 0-€-Belegen), Altersstruktur des Rückstands aus `EING_DAT` (Median, Klassen
≤ 7/8–30/31–90/> 90 Tage), mehrtägige Snapshot-Verwaltung mit Export/Import, optionale
Ordner-Synchronisierung für automatischen Mehrfach-Import aus einem lokalen oder
SharePoint-/OneDrive-synchronisierten Ordner (Chrome/Edge, siehe eigener Abschnitt oben).

Bewusst **nicht** enthalten (siehe Build Plan v1–v3): Tarif-Auswertung (`TA_KOMB_EB1`, weder
als eigene Analyse noch als Gruppierungsdimension der Grund-Kombinationen), Bewertung der
Hellwerdegründe nach Automatisierbarkeit, eine freigabepflichtige „Einordnung"-Textschicht
mit Schwellenwerten sowie Hochrechnungen aufs Jahr — alle vier auf ausdrücklichen Wunsch
zurückgestellt (siehe Abschnitt „Management Summary" oben für die Details zu den letzten
beiden).

## Tests

`scripts/` (im Build-Verzeichnis, nicht Teil der ausgelieferten HTML-Datei) enthält die
Playwright- und Node-Testskripte, die alle Sollwerte aus den Build Plänen (v1–v3) gegen die
reale Beispieldatei prüfen — darunter Golden-Tests für jede einzelne Regel (inkl. R11/R12, die
im Beispieldatensatz mit nur einem Stichtag/einer Dok.-Art sonst nie aktiv würden, dafür mit
synthetischen Mini-Datensätzen geprüft), Negativtests (zu geringe Fallzahl, fehlendes
`LEA189`/`EING_DAT`/Vortag → Regel entfällt ohne Fehler statt mit kaputtem Text) sowie ein
Konsistenztest je Drill-Link (die im Zielreiter angezeigte Zahl muss exakt der in der
Erkenntnis genannten Zahl entsprechen).
