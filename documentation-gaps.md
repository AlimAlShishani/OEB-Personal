# Dokumentationslücken – was vor einer verlässlichen technischen Spezifikation noch fehlt

> **Status Phase 2.1 (2026-11-12):** Punkte A1 und A2 sowie die Korrekturen aus G2, I3 und E3 sind in Revision 2 von `technical-handover.md` und `OEB-Analyse-IT.md` erledigt. D1, D3, D4, G1, G3 und G4 sind dort als Datenschutz-/Sicherheitsabschnitt dokumentiert (Fakten); die zugehörigen Laufzeit- und Rechtsfragen bleiben offen. Alle übrigen Punkte sind unverändert offen. Laufzeit-, Browserkompatibilitäts- und Netzwerktests wurden weiterhin **nicht** durchgeführt.

**Bezug:** Phase-2-Verifikation (`verification-report.md`), geprüft gegen ZIP `OEB-2.4.1`.
**Zweck:** Übersichtliche Liste der Informationen, die in `technical-handover.md` und `OEB-Analyse-IT.md` fehlen, unklar oder nur als Annahme vorhanden sind.
**Abgrenzung:** Die Liste dient der **Bewertung, Erkennung und Datenschutz-Einordnung**. Die in Phase 1 bewusst weggelassenen Inhalte (Rechenvorschriften der Header-/Schlüsselberechnung, Konstanten, Profildaten, Prompt-Texte, Seitenstruktur-Selektoren) gelten **nicht** als Lücke und sollen nicht nachdokumentiert werden.

**Priorität:** **P1** = vor weiterer technischer Planung klären · **P2** = vor einer vollständigen Spezifikation klären · **P3** = nice to have
**Art:** **K** = aus dem Code klärbar (statisch) · **T** = braucht Laufzeittest in Testumgebung · **E** = braucht externe Quelle/Entscheidung
Verweise (DF-/UA-/TR-/DS-/E-) beziehen sich auf `verification-report.md`.

---

## A. Dokumentenstand und Herkunft

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| A1 | **Welcher Stand ist maßgeblich?** Die hochgeladene `OEB-Analyse-IT.md` ist älter als die Repo-Version (K1–K3 fehlen). Nötig: ein Versionsvermerk (Datum/Commit) in beiden Dokumenten. | P1 | K | DF-13 |
| A2 | **Korrigierte Fundstellen** aus DF-01 und DF-08 (11 Stück) und die Zahl „20 Nachrichtentypen" (DF-02) sind noch nicht eingearbeitet. | P1 | K | DF-01, DF-02, DF-08 |
| A3 | **Widerspruch Version `2.4.0` ↔ Ordner `OEB-2.4.1`:** unklar, ob 2.4.1 inhaltlich abweicht. Es gibt weder Changelog noch Commit-Historie im ZIP. | P2 | E | BF-02 |
| A4 | **Herkunft und Lizenz:** README nennt „Based on SebBypass" (zwei Quellen). Es gibt **keine Lizenzdatei**. Unklar, welche Teile abgeleitet sind und unter welchen Bedingungen. | P3 | E | TR-08 |
| A5 | **Release-Artefakte:** README verweist auf GitHub-Releases. Ob die Release-ZIPs mit einem Build aus diesem Quellstand identisch sind, ist **nicht geprüft** (Build wurde nicht ausgeführt). | P2 | T | – |

## B. Zuständigkeiten und Architektur

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| B1 | **Serialisierungsmodell vollständig beschreiben:** Welche Zustandsänderungen laufen in der Warteschlange, welche nicht (`unlockPending`, `unloadConfiguration`)? Erwartetes Verhalten bei gleichzeitigem Entsperren/Entladen/Ersetzen. | P1 | K+T | DF-03, TR-02 |
| B2 | **Zustandsautomat unvollständig:** Header-Zustand (an/aus) ist nicht Teil des Automaten in Handover §7.2. Unbeschrieben: Übergänge bei Teilfehlern (Header an, Zustand „error"), Verhalten bei `runtime.onInstalled` (setzt jedes Mal „idle", auch bei Browser-/Erweiterungs-Update). | P1 | K | TR-01, TR-10 |
| B3 | **Tab-Semantik:** Was passiert, wenn der Prüfungs-Tab navigiert, dupliziert oder in ein anderes Fenster verschoben wird? Wie wirkt `presentationTabs` bei mehreren Tabs/Fenstern? Wie verhalten sich Unter-Frames (nur oberster Frame bekommt `SEB_*`)? | P2 | T | – |
| B4 | **Datei-Import-Pfad:** Dateiauswahl läuft in separatem Tab (`popup.html?file=1&target=…`). Zielreiter-Auswahl und Lebensdauer dieses Tabs nicht beschrieben. | P3 | K | DF-09 |
| B5 | **Popup→Content-Direktpfad** (`OEB_AI_EXTRACT` aus `assistant/popup.js:413`) fehlt im Kommunikationsdiagramm. | P3 | K | DF-09 |
| B6 | **Kopplung Plist-Parser ↔ Overlay-IDs:** `config-key.js:95-106` toleriert eigene Overlay-Frames im kopierten XML. Diese zusätzliche ID-Kopplung steht nicht in Handover §8.2. | P3 | K | – |
| B7 | **Zweiter Ort der hartkodierten Build-Angabe:** inerter `text/plain`-Block in `startup.html` (Z. 238 ff.) enthält eine Kopie der Start-Logik. Handover §14 nennt ihn „nicht untersucht". Zweck/Altlast ungeklärt. | P3 | K | BF-01 |
| B8 | **Taskleisten-Funktionsumfang** unvollständig (Akku-Anzeige, Sprachkürzel-Zuordnung, Reload-Bestätigung). | P3 | K | DF-10 |

## C. Schnittstellen und Datenmodelle

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| C1 | **Nachrichtenschemata:** Für alle 20 Nachrichtentypen (und 7 Richtung Content) fehlen Felder der Anfrage und der Antwort (`{ok, state, error, …}`). Bisher nur Namen und Zweck. | P2 | K | DF-02 |
| C2 | **Vollständiges Datenmodell** der Speicherobjekte mit Typen und Pflichtfeldern: `sebInspectorState`, `sebInspectorPending` (inkl. `metadata`-Felder), `oebAiProviders`-Konto, `oebAiLastRun`, `oebBekCatalogCache`, `oebBekSettings`. | P1 | K | DS-02 |
| C3 | **`postMessage`-Protokolle** (Kanal `oeb-download-capture-v1`; Content↔UI-Frames) mit Feldern, Richtung, Validierung und Zeitlimit (15 s in `download-page.js:69`). | P2 | K | E-01 |
| C4 | **Migrationen/Altformate:** `schemaVersion 2` bei BEK-Einstellungen, Legacy `showSebBar`, automatische Migration „provided" → Standard. Kein Migrationskonzept dokumentiert. | P2 | K | DF-05 |
| C5 | **Fehlercode-Katalog** über alle Module (nicht nur `SebParseError`) mit Benutzertext und Auswirkung (Badge, Zustand). | P3 | K | – |

## D. Persistenz und Datenlebenszyklen

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| D1 | **Lebenszyklus je Schlüssel:** Wann geschrieben, wann gelöscht, wann überschrieben? Löschbar per UI sind einzelne KI-Konten („Remove" + „Save"), Domain-Freigaben (Schalter) und die Notiz (leeren). Für `oebAiLastRun` und `oebBekCatalogCache` gibt es keine UI-Löschfunktion; es gibt keine „Alle Daten löschen"-Funktion. *(Berichtigt in Phase 2.1.)* | P1 | K | DS-01, DS-03 |
| D2 | **Verhalten von `storage.session` je Browser** (Verfügbarkeit, Lebensdauer bei Reload/Neustart, Fallback auf `local`). | P1 | T | UA-02, UA-08 |
| D3 | **Inhalt des Pending-Datensatzes** (inkl. `method`, `body`, `pageUrl`, `tabId`) und seine Aufbewahrungsdauer. | P1 | K | DS-02 |
| D4 | **Was von `oebAiLastRun` gespeichert wird** (inkl. Bild-`src`-URLs, Fragetexte, Antworten) und wie lange. | P1 | K | DS-03 |
| D5 | **Speicherorte auf dem Endgerät** je Browser (Profilpfade der Erweiterungs-Datenbanken), Sichtbarkeit für Forensik/Inventarisierung. | P2 | T+E | E-05 |
| D6 | **Verhalten bei Deinstallation/Profilwechsel** (was bleibt zurück?). | P3 | T | – |

## E. Fehlerbehandlung und Einschränkungen

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| E1 | **Liste der ~28 stillen Fehlerstellen** mit jeweiliger Folge (Datenverlust, ausbleibende UI-Meldung, stiller Abbruch). Bisher nur „rund 27". | P2 | K | – |
| E2 | **Teilfehler-Verhalten:** `activateLaunch`, Download-Erkennung (Fail-open), KI-Lauf mit teilweise gefüllten Antworten, `setBadge` im Fehlerpfad. | P2 | K+T | TR-01, TR-06 |
| E3 | **Korrigierte Grenzen-Tabelle:** welche Größen-/Zeitgrenze gilt auf welchem Pfad (8 MB nur Auto-Pfade; keine Grenze für entpackte Größe; 4 Gzip-Schichten). | P1 | K | DF-04, TR-03 |
| E4 | **Verhalten bei Ratenlimits/Fehlercodes der KI-Anbieter** (HTTP 429, 5xx) und Reihenfolge der Fallback-Konten. | P3 | K+T | – |
| E5 | **Robustheit des Plist-Parsers** gegen reale Konfigurationen. | P2 | T | UA-07 |

## F. Browserkompatibilität

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| F1 | **Kompatibilitätsmatrix** je Browser/Version: MV2-Verfügbarkeit, `webRequestBlocking`, `storage.session`, `runtime.getBrowserInfo` (Content-Script), `filterResponseData`, `world: MAIN`, `data_collection_permissions`, Android. | P1 | T+E | UA-01 bis UA-03 |
| F2 | **Vollständigkeit der Chrome-Fassade** `browser-api.js` (nur ausgewählte APIs): Gegenprobe, ob alle im Code genutzten `browser.*`-Aufrufe in Chromium existieren (`commands.update/getAll/reset` sind geschützt). | P2 | K+T | – |
| F3 | **Chromium-Derivate** (Edge, Brave, Opera, Vivaldi): unterstützt, getestet, MV2-Politik? Nichts dokumentiert. | P2 | E | UA-03 |
| F4 | **Verhalten der Firefox-Doppelregistrierung** (MAIN-Skript + ggf. Skript-Tag-Injektion). | P2 | T | UA-01 |

## G. Externe Verbindungen, Berechtigungen, Datenschutz

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| G1 | **Vollständige Verbindungsliste** je Ziel: Auslöser, Häufigkeit, Cookies/Credentials, Nutzdaten. Fehlend: Bild-Abrufe von Fremdhosts (`extract.js:51`), Modelllisten-Abrufe, Custom-Endpunkt, zweiter Abruf der Konfiguration (Probe). | P1 | K | DS-05, DS-01 |
| G2 | **Korrigierte Katalog-Abrufbedingungen** (nicht „bei Start" garantiert; 6-h-Schwelle, Popup-/Launch-Auslöser). | P1 | K | DF-06 |
| G3 | **Datenfluss-Diagramm KI-Lauf** mit Datenkategorien (Fragetext, Optionen, Bilder, Notiz) und Entfernungen vor dem Versand; Aufbewahrung beim Anbieter ist anbieterspezifisch und **nicht im Code festgelegt** (nur `store:false` bei OpenAI). | P1 | K+E | DS-04 |
| G4 | **Umgang mit API-Schlüsseln:** Klartext in `storage.local`, einzelnes Entfernen per UI möglich, kein Export, Übertragung an frei wählbaren Host (Custom). Risikobewertung und Empfehlung (z. B. Schlüsselrotation bei Verlust des Geräts) fehlen. | P1 | E | DS-01 |
| G5 | **Berechtigungsbegründung:** Wofür wird jede Berechtigung genutzt (`downloads`, `tabs`, `<all_urls>`, `webRequestBlocking`)? Fehlt als Tabelle. | P2 | K | DS-07 |
| G6 | **Exponierte Ressourcen (`web_accessible_resources`)** und Auswirkung auf Sichtbarkeit der Erweiterung für Webseiten. | P2 | K+T | E-02 |
| G7 | **Datenschutzrechtliche Einordnung** (Art der Daten, Rolle, Meldewege). Nur als Hinweis in IT-Analyse §6, keine Bewertung. | P1 | E | DS-01 bis DS-05 |
| G8 | **Authentizität der Fernaktualisierung:** Quelle ist ein `main`-Branch eines Drittrepos, Prüfsumme nur selbstreferenziell. Bedrohungsmodell fehlt. | P2 | E | TR-07 |

## H. Tests und Qualitätssicherung

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| H1 | **Es existieren keine Tests, keine CI.** Es fehlt ein Testkonzept für die rein funktionalen Module (`parser`, `launch-response`, `settings`, `download-check`, Katalog-Manager). | P1 | – | TR-08 |
| H2 | **Testdaten:** synthetische, selbst erzeugte Konfigurationsdateien (Klartext, gzip, passwortgeschützt, defekt) und ein Test-Moodle; **keine** realen Prüfungskonfigurationen. | P2 | E | – |
| H3 | **Build-Reproduzierbarkeit:** Node-Mindestversion nicht festgelegt (kein `engines`/`.nvmrc`); Lockfile umfasst 25 Pakete. Build wurde nicht ausgeführt. | P3 | T | – |
| H4 | **Abnahmekriterien** für „Dokumentation ist verlässlich" (z. B. Zeilenangaben als automatisch geprüfte Referenzen statt Hand-Angaben). | P3 | E | DF-08 |

## I. Erkennung (nur für die IT-Analyse)

| # | Fehlende Information | Prio | Art | Bezug |
|---|---|---|---|---|
| I1 | **Referenzdaten echter SEB-Clients** der eigenen Umgebung (UA, Header-Reihenfolge, Client-Hints, TLS-Fingerprint). Ohne sie bleiben alle Netzwerk-Vergleiche Hypothesen. | P1 | E | UA-05 |
| I2 | **Beobachtbarkeit im Moodle** (Ereignisse mit `isTrusted=false`, `postMessage`-Marker) mit der eingesetzten Moodle-Version testen. | P2 | T | UA-06, E-01 |
| I3 | **Korrektur der Erkennungsmerkmale** DF-06 (Abrufzeitpunkt), DF-07 (nur dreiteilige UA-Version), DF-12 (Konfig-Download-UA nicht immer „normal"). | P1 | K | DF-06, DF-07, DF-12 |
| I4 | **Ergänzende Merkmale** in IT-Analyse einarbeiten: E-01 (`postMessage`), E-03 (fester OpenRouter-Header), E-04 (Weiter-Kürzel nicht domain-gebunden). | P2 | K | E-01 bis E-04 |
| I5 | **Falsch-Positiv-Betrachtung:** Welche anderen Erweiterungen/Hilfsmittel erzeugen ähnliche Spuren? Nicht untersucht. | P3 | E | – |

---

## Bewusst nicht als Lücke geführt

- Rechenvorschriften und Konstanten der Header-/Schlüsselberechnung, Inhalt von `bek.json`, Prompt-Texte, Seitenstruktur-Selektoren: absichtlich nicht dokumentiert (Abgrenzung Phase 1). Ihre Korrektheit gegenüber SEB wurde in Phase 2 auch **nicht** bewertet.
- Tests gegen reale Prüfungsplattformen: ausdrücklich ausgeschlossen.

## Nicht gelesen in Phase 2 (eigene Lücke dieses Berichts)

`popup.css`; Markup von `taskbar.html`, `quit.html`, `startup.html` (nur gezielt); `assistant/popup.js` und `popup.html` (nur gezielt, UI-Rendering nicht vollständig); Bild- und GIF-Dateien; Inhalt von `bek.json`.
