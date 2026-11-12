# Wiederverwendbarkeitsbewertung der OEB-Codebasis (Phase 3)

**An:** IT-Leitung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Klassifizierung:** intern
**Grundlage:** `security-privacy-audit.md` (Befund-IDs SEC-xx, NF-xx), `technical-handover.md` (Revision 2 plus Nachtrag 0.6), ZIP `OEB-2.4.1` (Paketversion 2.4.0).
**Kennzeichnung:** BESTÄTIGT (Quellcode belegt) · ABGELEITET (plausible Schlussfolgerung) · OFFEN · NICHT GEPRÜFT. Es wurden weiterhin keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt.

---

## 1. Ergebnis

1. **Praktisch kein Quellcode ist ohne grundlegende Änderung wiederverwendbar.** Von 6 735 Zeilen Quelltext (JS, HTML, CSS, JSON) fallen **0 Zeilen in die Kategorie „Wiederverwendbar"**, 1 308 (19,4 %) in „Vorher zu überarbeiten" und 5 427 (80,6 %) in „Nicht übernehmen". Berücksichtigt man, dass der größte Teil der Kategorie 2 aus dem nicht geprüften Stylesheet `popup.css` (914 Zeilen) besteht, bleiben **394 Zeilen (6,8 %)** tatsächlich überarbeitbarer Code; 93,2 % des geprüften Codes gehören zum ausgeschlossenen Funktionsumfang (Abschnitt 4).
2. **Wiederverwendbar sind Konzepte und Dokumentationsstruktur**, kein Code (Abschnitt 5).
3. **Der Funktionskern der Codebasis ist vom neuen Projektauftrag ausgeschlossen:** Nachahmung eines SEB-Clients gegenüber Prüfungsservern, Nachbau der SEB-Oberfläche, Erfassung von Prüfungskonfigurationen, KI-gestütztes Beantworten und Eintragen von Prüfungsfragen. Eine Umbenennung oder Teilentfernung würde die übrige Architektur (Hintergrund-Hub, Rechteprofil, Nachrichtenwege) auf diese Funktionen zugeschnitten belassen.
4. **Empfehlung (Abschnitt 6):** Neues Projekt als **eigenständige Neuentwicklung** (Clean-Room) mit Wissensübernahme; die bestehende Codebasis wird **isoliert archiviert**, nicht fortgeführt.

---

## 2. Methode und Kategorien

Bewertet wurden die neun von der IT-Leitung genannten Bereiche. Je Komponente wurde geprüft: (a) fachliche Bindung an den ausgeschlossenen Funktionsumfang, (b) Sicherheits-/Datenschutzbefunde aus `security-privacy-audit.md`, (c) technische Voraussetzungen (Manifest V2, Persistenz), (d) Rechts- und Herkunftsfragen.

| Kategorie | Bedeutung |
|---|---|
| **1 – Wiederverwendbar** | Kann im Wesentlichen unverändert in das neue Projekt eingehen. |
| **2 – Vorher zu überarbeiten** | Idee oder Gerüst brauchbar; Umsetzung muss vor Übernahme neu geschrieben oder grundlegend angepasst werden. |
| **3 – Zu isolieren** | Enthält Wissen oder Material, das erhalten bleiben soll, aber nicht in das Projekt eingehen darf; Aufbewahrung getrennt und zugriffsbeschränkt. |
| **4 – Nicht übernehmen** | Gehört zum ausgeschlossenen Funktionsumfang, ist technisch ungeeignet oder rechtlich nicht übernehmbar. |

**Hinweis zur Gewichtung:** Die Kategorie sagt aus, was mit dem **Code** geschieht. Konzepte, die aus der Analyse gelernt wurden (Abschnitt 5), sind davon unabhängig und werden neu umgesetzt.

---

## 3. Bewertung je Bereich

### 3.1 Hintergrundprozesse

| Komponente | Umfang | Kat. | Begründung | Beleg |
|---|---|---|---|---|
| Hintergrund-Hub `seb/background.js` | 748 Z. | **4** | Vermischt Erkennung, Zustand, Routing, Overlay, KI-Start; persistente MV2-Hintergrundseite; nicht serialisierte Zustandspfade, Teilfehler (SEC-14). Fachlich vollständig an den ausgeschlossenen Zweck gebunden. | BESTÄTIGT `background.js:59-63,212-346,442-491,559-584,615-637`; `manifest: persistent: true` |
| Header-Hook und Schlüsselmodule `headers.js`, `config-key.js`, `browser-exam-key.js`, `bek-catalog.js`, `bek.json` | 121 + 337 + 64 + 143 Z. + 59 KB | **4** | Ausdrücklich ausgeschlossener Kernmechanismus (Nachahmung eines SEB-Clients). Zusätzlich Fernaktualisierung ohne Authentizitätsprüfung (SEC-06). | BESTÄTIGT `headers.js:19-43`; `bek-catalog.js:2,27-29` |
| Erkennung von Konfigurationen `launch-response.js`, `download-check.js`, `download-stream.js`, `download-content.js`, `download-page.js`, `file-import.js` | 23 + 88 + 41 + 78 + 126 + 9 Z. | **4** | Erfasst Prüfungskonfigurationen; Firefox-Spiegelung fremder Antworten (SEC-07); Brücke, die Seiten-JS nutzen kann (SEC-01). | BESTÄTIGT `background.js:95-207`; `download-content.js:16-31` |
| KI-Assistent `assistant/background.js`, `providers.js`, `extract.js`, `apply.js`, `content.js` | 105 + 441 + 178 + 202 + 56 Z. | **4** | Ausdrücklich ausgeschlossen (automatisierte Beantwortung); überträgt Inhalte an Dritte und speichert Schlüssel im Klartext (SEC-03/04). | BESTÄTIGT `providers.js:194-303`; `apply.js:3-202` |
| Parser für SEB-Dateien `parser.js` | 246 Z. | **4** | SEB-spezifisch; für das neue Projekt ohne Bedarf; unbegrenzte Dekompression (SEC-13). | BESTÄTIGT `parser.js:107-125` |
| Katalog-Manager-Muster (Cache, Validierung, Dependency-Injection) | Teil von `bek-catalog.js` | Code **4**, Konzept → 5.1 | Die Injektion von `storage`/`fetcher`/`now` ist ein gutes Testmuster; die Fernaktualisierung selbst ist nicht übernehmbar. | BESTÄTIGT `bek-catalog.js:64` |

### 3.2 Benutzeroberfläche

| Komponente | Umfang | Kat. | Begründung | Beleg |
|---|---|---|---|---|
| Popup-Gerüst `popup.html`, `popup.css` | 207 + 914 Z. | **2** | Reiter-Layout als Anregung brauchbar; Inhalte sind an SEB/KI gebunden. `popup.css` wurde **nicht geprüft** (NICHT GEPRÜFT). Praktisch: neu schreiben. | `popup.html:11-117,203-205` |
| Popup-Logik `seb/popup.js`, `assistant/popup.js` | 351 + 600 Z. | **4** | Fachlich gebunden („Exam connection", API-Schlüsselverwaltung, KI-Lauf). Rendering mit `textContent` (keine HTML-Senken) ist ein Positivbefund → 5.1. | BESTÄTIGT `seb/popup.js:172-190`; `assistant/popup.js:214-218,371` |
| Overlay-Frames `startup.*`, `taskbar.*`, `quit.*`, `colors.css`, `content.js`, `content.css` | 554 + 95 + 111 + 69 + 50 + 34 + 101 + 11 + 316 + 40 Z. | **4** | Nachbau der SEB-Oberfläche (Start-Animation, Taskleiste, Beenden-Dialog); täuschender Zweck; Passwortweitergabe per `postMessage "*"` (SEC-05). | BESTÄTIGT `seb/assets/quit.js:25`; `content.js:163-248` |
| SEB-Logos und Marken-Assets `seb/icons/SebSplashScreen.png`, `seb_logo.png` | 2 Dateien | **4** | Fremde Marken-/Gestaltungsassets; Rechtslage ungeklärt (NICHT GEPRÜFT). | Dateiliste; `startup.html` |
| Vorschau-GIFs `preview/*.gif` | 3 Dateien | **4** | Werbematerial für die ausgeschlossene Funktion. | Dateiliste |

### 3.3 Nachrichtenkommunikation

| Komponente | Kat. | Begründung | Beleg |
|---|---|---|---|
| Router-Muster (zentrale Funktion, Antwort `{ok, error}`, Fehler werden abgefangen) | **2** | Muster sinnvoll. Neu mit Absender-Autorisierung je Nachrichtentyp, Schema-Validierung und Typkatalog (NF-04, SEC-08). | BESTÄTIGT `background.js:65-70,212-346` |
| Seite↔Content-Script-Brücke `download-page.js`/`download-content.js` | **4** | Seiten-JavaScript kann Aktionen anstoßen (SEC-01); gehört zur ausgeschlossenen Erkennung. | BESTÄTIGT `download-content.js:16-31` |
| Content↔UI-Frame-Nachrichten (`SEB_*`) | **4** | `targetOrigin "*"` an allen Stellen, Frame-Zuordnung per DOM-ID, Passwortweitergabe (NF-01, SEC-05, SEC-15). | BESTÄTIGT Suchlauf `postMessage`: 14 Stellen im Code, alle mit `targetOrigin "*"`, davon 10 zwischen Content-Script und UI-Frames |
| Nachrichtenkatalog als Dokumentation (Tabelle Typ/Absender/Zweck) | **1** | Dokumentationsmuster übernehmen. | `technical-handover.md` 6.2 |

### 3.4 Konfigurationsverwaltung

| Komponente | Umfang | Kat. | Begründung | Beleg |
|---|---|---|---|---|
| `settings.js` (Normalisierung, Standardwerte, Altformat) | 16 Z. | **2** | Gutes, kleines Muster; Schlüsselnamen, Schema-Version und Migrationen fehlen. Neuer Code ohnehin kleiner als eine Anpassung. | BESTÄTIGT `settings.js:1-16` |
| Speichernutzung (`storage.local/session`, Schlüssel verstreut) | – | **2** | Zentrale Speicherschicht mit Schema, Aufbewahrungsregeln und Löschfunktion nötig; heute 11 Schlüssel an mehreren Stellen dupliziert. | `technical-handover.md` 7.1, 8.2 |
| Plist-Konfiguration der Prüfsoftware `config-key.js` (Parser), `parser.js` | 337 + 246 Z. | **4** | SEB-spezifisch; handgeschriebener Parser mit NF-06. | BESTÄTIGT `config-key.js:83-259` |
| KI-Konten und Domain-Freigaben (`oebAi*`) | – | **4** (Konzept → 5.1) | Klartext-Schlüssel (SEC-03). Das Konzept „Freigabe nur für explizit gelistete Hosts" ist als Scope-Allowlist übertragbar. | BESTÄTIGT `assistant/background.js:44-46` |

### 3.5 Logging

| Komponente | Kat. | Begründung | Beleg |
|---|---|---|---|
| Ereignisprotokollierung | **4** (nicht vorhanden → Neuentwicklung) | Es gibt keine Logging-Funktion; Fehler liegen nur als letzter Zustand (`errorCode`, `lastError`) vor. Für ein Sicherheitswerkzeug muss ein Ereignisschema, Manipulationsschutz und Datenminimierung neu entworfen werden. | BESTÄTIGT NF-03 (Suchlauf: kein `console.*`) |

### 3.6 Fehlerbehandlung

| Komponente | Kat. | Begründung | Beleg |
|---|---|---|---|
| Muster „Fehler → Zustand, Router liefert `{ok:false}`" | **2** | Übernehmbar als Konvention; Fehlertaxonomie, Ereignisprotokoll und Benutzertexte ergänzen. | BESTÄTIGT `background.js:420-431,479-490,65-70` |
| Etwa 28 stille Fehlerblöcke, „fail-open"-Verhalten bei Erkennung | **4** | Für ein Prüfwerkzeug unzulässig: Fehler müssen sichtbar und protokolliert sein. | BESTÄTIGT (eigene Zählung) `download-check.js:1-2`; `background.js:127,167` |
| Größen- und Zeitlimits | **2** | Idee übernehmen und **auf allen Pfaden** durchsetzen (heute nicht, SEC-13). | BESTÄTIGT `technical-handover.md` 7.6 |

### 3.7 Berechtigungsmodell

| Komponente | Kat. | Begründung | Beleg |
|---|---|---|---|
| Rechteprofil: `<all_urls>`, `webRequest`+`webRequestBlocking`, `tabs`, `downloads`, Content-Scripts in allen Frames | **4** | Auf Eingriff in fremden Webverkehr ausgelegt; MV2; nicht mit minimalen Rechten vereinbar (SEC-02, SEC-11). | BESTÄTIGT Manifeste Z. 11-18 |
| Manifest-Gerüst (zwei Browserziele, Kürzel, Popup) | **2** | Struktur als Vorlage; Manifest V3, minimale und optionale Rechte, gestaffelt neu entwerfen. | BESTÄTIGT Manifeste; Build `build.mjs:69-77` |
| Kompatibilitätsfassade `browser-api.js` | **4** | Ersatz durch Standardlösung (nativ unter MV3 bzw. Polyfill); Erkennungslogik zweifelhaft (UA-01). | BESTÄTIGT `browser-api.js:2`; OFFEN UA-01 |

### 3.8 Build-System

| Komponente | Umfang | Kat. | Begründung | Beleg |
|---|---|---|---|---|
| `build.mjs`, `package.json`, `package-lock.json` | 95 + 14 Z., 25 Dev-Pakete | **2** | Gerüst (zwei Zielpakete, Manifest-Prüfung, ZIP) brauchbar und klein. Anpassen: Abhängigkeit vom Katalogmodul entfernen, Node-Version festlegen, Linting, Tests, SBOM, nachvollziehbare (nicht minifizierte) Prüfbuilds, signierte Releases. | BESTÄTIGT `build.mjs:6,17,31-66,80-95`; kein `engines`-Feld |
| Tests und CI | – | **4** (nicht vorhanden → Neuentwicklung) | Keine Tests, keine CI. | BESTÄTIGT Dateiliste |

### 3.9 Dokumentationsstruktur

| Komponente | Kat. | Begründung |
|---|---|---|
| Struktur (Handover / Verifikationsbericht / Lückenliste) mit Befund-IDs, Kennzeichnungsschema BESTÄTIGT-ABGELEITET-OFFEN-NICHT GEPRÜFT, Änderungsprotokoll | **1** | Eigene, neutrale Arbeitsmethode; direkt auf das neue Projekt übertragbar. |
| `technical-handover.md`, `OEB-Analyse-IT.md` (Inhalt) | **3** | Beschreiben Mechanik und Erkennungsmerkmale der Umgehungsfunktion; als Referenz erhalten, aber getrennt aufbewahren (Zugriff nur für berechtigte Personen). `OEB-Analyse-IT.md` gehört fachlich in einen separaten Arbeitsstrang „Prüfungsintegrität". |
| `verification-report.md`, `documentation-gaps.md` | **3** | Belegsammlung zur Entscheidung; Methode übernehmen, Inhalt archivieren. |
| Ursprungsarchiv `OEB-2.4.1.zip` und README | **3** | Schreibgeschützt, mit Prüfsumme, getrennt vom neuen Repository, nicht ausführen. |

---

## 4. Quantitative Übersicht

Quelltext (JS, HTML, CSS, JSON unter `source/`): 6 735 Zeilen in 37 Dateien. Bilddateien und `icon.svg` nicht eingerechnet. Zugeordnet nach Abschnitt 3.

| Kategorie | Zeilen | Anteil | Dateien |
|---|---|---|---|
| 1 – Wiederverwendbar | 0 | 0,0 % | 0 |
| 2 – Vorher zu überarbeiten | 1 308 | 19,4 % | 5 (`settings.js`, `popup.html`, `popup.css`, beide Manifeste) |
| 3 – Zu isolieren (Code) | 0 | 0,0 % | 0 (nur Dokumente und Archiv) |
| 4 – Nicht übernehmen | 5 427 | 80,6 % | 32 |

Ohne das nicht geprüfte Stylesheet `popup.css` (914 Zeilen): Kategorie 2 = 394 von 5 821 Zeilen (6,8 %), Kategorie 4 = 5 427 (93,2 %). Hinzu kommen `build.mjs` und `package.json` (109 Zeilen, Kategorie 2).

**ABGELEITET:** Der tatsächlich nutzbare Codeanteil liegt im niedrigen einstelligen Prozentbereich und besteht aus Gerüsten, die ohnehin neu entworfen werden müssen.

---

## 5. Was wirklich mitgenommen wird

### 5.1 Konzepte (neu umzusetzen, nicht zu kopieren)

| Konzept | Quelle | Nutzen im neuen Projekt |
|---|---|---|
| Zentraler Router mit einheitlichem Antwortformat `{ok, error}` | `background.js:65-70` | Wird um Schema, Absenderprüfung, Ereignisprotokoll ergänzt |
| Zustandsautomat mit dokumentierten Übergängen und getrennter Behandlung von Teilfehlern | `technical-handover.md` 7.2/7.3 | Vorlage; Fehler der Vorlage (SEC-14) vermeiden |
| Injizierbare Abhängigkeiten für Netzwerk/Speicher/Zeit | `bek-catalog.js:64`; `download-check.js:79` | Grundlage für deterministische, reproduzierbare Tests |
| Gesonderte „Scope-Allowlist": Funktion nur für explizit freigeschaltete Hosts | `assistant/background.js:44-46` | Technische Durchsetzung des Autorisierungsrahmens |
| Rendering ausschließlich über `textContent`, keine HTML-Senken | Suchlauf | Anforderung SR-NF-03 |
| Größen- und Zeitlimits an Netzwerkgrenzen, HTTPS-Pflicht, Abrufe ohne Cookies/Weiterleitungen | `download-check.js:3`; `providers.js:394-398`; `bek-catalog.js:100` | Anforderung SR-NF-04 |
| Zwei Zielpakete aus einer Quelle mit Prüfung des Manifests im Build | `build.mjs:69-77` | Build-Gerüst |
| Dokumentation: Befund-IDs, Kennzeichnungsschema, Änderungsprotokoll | Vorarbeiten | Prüfberichte |

### 5.2 Negative Lehren (als Anforderungen formuliert)

Keine Klartext-Geheimnisse; keine Persistenz von Seiteninhalten; keine Fremdübertragung; kein Mitschnitt fremder Antworten; kein `postMessage "*"` mit sensiblen Daten; kein Router ohne Absenderprüfung; kein Verzicht auf Logging; keine stillen Fehler; keine Tests auslassen; Lizenz und Herkunft vor Beginn klären. Siehe `security-testing-requirements.md`.

### 5.3 Prüfobjekt statt Bauteil (Hinweis)

Für Tests der Browserrichtlinien (z. B. „wird das Laden nicht freigegebener Erweiterungen verhindert?") sollte eine **eigens entwickelte, harmlose Testerweiterung** („Canary") verwendet werden und **nicht** OEB. OEB enthält Funktionen, die in der Laborumgebung keinen Zweck haben und ein Missbrauchsrisiko darstellen.

---

## 6. Empfehlung

| Frage | Antwort |
|---|---|
| Ist die Codebasis als Ausgangspunkt geeignet? | **Nein.** 93 % des geprüften Codes gehören zum ausgeschlossenen Funktionsumfang; die Architektur (Rechteprofil, Hintergrund-Hub, Nachrichtenwege, MV2) ist darauf zugeschnitten. |
| Was wird übernommen? | Konzepte (5.1), das Build-Gerüst als Vorlage, die Dokumentationsstruktur und das Kennzeichnungsschema. |
| Was wird isoliert? | Das Ursprungsarchiv, `technical-handover.md`, `OEB-Analyse-IT.md`, `verification-report.md`, `documentation-gaps.md`. |
| Was wird neu entwickelt? | Alles Funktionale: Hintergrundlogik, Oberfläche, Nachrichtenschicht, Konfiguration, Logging, Fehlerbehandlung, Berechtigungsmodell, Tests, Berichtswesen. |
| Was wird nicht übernommen? | Header-/Schlüsselmodule, Konfigurationserkennung, Overlay-Nachbau, KI-Assistent, SEB-Assets, Vorschaumaterial. |

**Begründung der Neuentwicklung gegenüber einem Umbau (ABGELEITET):** Ein Umbau müsste fast alle Dateien entfernen oder neu schreiben, die verbleibenden Gerüste entsprechen nicht den Mindestanforderungen (Manifest V3, minimale Rechte, Logging), und das Herkunfts- und Lizenzrisiko (SEC-10) bliebe bestehen. Die Neuentwicklung ist deshalb der kürzere und sauberere Weg.

**Zu klären bevor irgendein Code übernommen wird:** Lizenz der Vorlageprojekte (`README.md:7`), Rechtslage der SEB-Assets, Freigabe durch Datenschutz und Rechtsabteilung (alle NICHT GEPRÜFT).
