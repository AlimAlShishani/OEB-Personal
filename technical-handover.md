# Technical Handover: „OEB — Open Exam Browser" (Architektur- und Aufbauübersicht)

**Stand:** 2026-11-12 · **Revision 2** (Dokumentationsbereinigung nach Phase-2-Verifikation)
**Zielgruppe:** neue Mitglieder des IT-Teams, die sich in kurzer Zeit einen fundierten Überblick verschaffen sollen.
**Begleitdokumente:** `OEB-Analyse-IT.md` (Erkennung und Gegenmaßnahmen) · `verification-report.md` (Ergebnis der Verifikation, enthält die Befund-IDs DF/BF/TR/DS/UA/E) · `documentation-gaps.md` (offene Informationen)

---

## 0. Dokumentstand, Kennzeichnung, Prüfstatus

### 0.1 Untersuchter Stand und Versionsbezeichnungen

| Bezeichnung | Wert | Fundstelle |
|---|---|---|
| ZIP-Ordner / Archiv | `OEB-2.4.1` | Dateiname des Archivs |
| Interne Paketversion | `2.4.0` | `package.json:3`; beide Manifeste Z. 5; `README.md:5` |

**Verwendete Schreibweise in diesen Dokumenten:** „ZIP `OEB-2.4.1` (Paketversion 2.4.0)". Ob die Bezeichnung 2.4.1 inhaltliche Änderungen gegenüber 2.4.0 enthält, ist **OFFEN**: Es gibt weder Changelog noch Versionshistorie im ZIP. Andere Versionen oder Forks können abweichende IDs, Frame-IDs, Hosts und Speicherschlüssel haben. Zu den im Code vorkommenden SEB-Build-Angaben siehe Abschnitt 7.7.

Pfade beziehen sich auf `OEB-2.4.1/`, Quellcode-Pfade auf `OEB-2.4.1/source/`. Zeilenangaben gelten für diesen Stand und wurden in Revision 2 gegen den Quelltext geprüft.

### 0.2 Kennzeichnung der Aussagen

| Kennzeichnung | Bedeutung |
|---|---|
| **BESTÄTIGT** | Direkt durch statische Quellcodeanalyse belegt (Datei:Zeile). |
| **ABGELEITET** | Technisch plausible Schlussfolgerung aus dem Code, nicht zur Laufzeit geprüft. |
| **OFFEN** | Benötigt weitere Prüfung (Laufzeittest, externe Quelle oder Entscheidung). |
| **NICHT GEPRÜFT** | Außerhalb des bisherigen Prüfungsumfangs. |

### 0.3 Prüfstatus

**Es wurden bisher keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt.** Die Erweiterung wurde nicht installiert, nicht ausgeführt und nicht gebaut. Es gab keinen Netzwerkmitschnitt und keine Tests gegen eine Moodle-Instanz oder eine Prüfungsplattform. Alle Aussagen zu Laufzeitverhalten, Browserunterstützung und Netzwerkverkehr sind höchstens ABGELEITET.

### 0.4 Bewusste Abgrenzung (Detailtiefe)

Dieses Dokument beschreibt Aufbau, Zuständigkeiten, Kommunikation, Zustände, Lebenszyklen sowie Datenschutz- und Sicherheitsaspekte. Es enthält **keine** Rechenvorschriften, Konstanten, Profildaten, Prompt-Texte, Seitenstruktur-Selektoren oder Schritt-für-Schritt-Abläufe der Header-Fälschung bzw. des KI-Autofills und keine Hinweise, wie diese Teile gepflegt oder verbessert werden könnten. Ob die Header-/Schlüsselberechnung gegenüber einer SEB-Spezifikation korrekt ist, wurde **nicht** geprüft (NICHT GEPRÜFT, bewusst ausgeklammert).

### 0.5 Änderungsprotokoll Revision 2

| Thema | Änderung | Referenz |
|---|---|---|
| Zeilenangaben | 11 abweichende Fundstellen korrigiert, darunter `assistant/content.js` (17-24 statt 122-129) | DF-01, DF-08 |
| Message-Typen | Der Router kennt 20 Typen (nicht „≈ 25"), vollständiger Katalog in 6.2 | DF-02 |
| Queue / Zustand | Nur das Ersetzen einer Konfiguration ist serialisiert; Entsperren, Entladen und Header-Schalten laufen daran vorbei (7.3) | DF-03, TR-01, TR-02, TR-10 |
| 8-MB-Grenze | Gilt nur für die automatischen Erkennungswege, Tabelle in 7.6 | DF-04, TR-03 |
| BEK-Eingabe | Per Popup nur „Off"/Versionsauswahl; Modus „provided" nicht über die Oberfläche erreichbar | DF-05, BF-04 |
| Katalog-Abruf | Bedingungen präzisiert (6-h-Schwelle, mehrere Auslöser) | DF-06 |
| Versionen / UA | Tabelle der Versionsangaben (7.7); UA enthält nur dreiteilige Version | DF-07, BF-01, BF-02 |
| Popup | Dateiauswahl in eigenem Tab, direkter Popup→Content-Pfad, `localStorage` | DF-09 |
| Taskleiste | Batterieanzeige ergänzt | DF-10 |
| Web-APIs | Trennung WebExtension-APIs / Web-Plattform-APIs (2.2) | DF-11 |
| Konfigurations-Abruf | Hook kann auch Hintergrund-Abrufe erfassen (ABGELEITET, 10.3) | DF-12 |
| Dokumentstand | Beide Hauptdokumente auf einen gemeinsamen Stand gebracht | DF-13 |
| Datenschutz / Sicherheit | Neuer Abschnitt 11; Risiken in Abschnitt 12 | DS-01 bis DS-07, TR-01 bis TR-10, E-05 |
| Berichtigung | DS-01 im Verifikationsbericht behauptete fälschlich „keine Löschfunktion"; einzelne KI-Konten sind per „Remove" + „Save" entfernbar (`assistant/popup.js:183,359-372`) | DS-01 |
| Frühere Korrekturen K1–K5 | integriert (K1: 5.3/10.3; K2: 6.2; K3: 5.4/6.2; K4: 7.7; K5: 7.1/11) | – |

---

## 1. Systemüberblick in einem Absatz

OEB ist eine **Browser-Erweiterung (Manifest V2) ohne eigenen Server**. Sie besteht aus (a) einer dauerhaft laufenden **Hintergrundseite** („Background Page") als Steuerzentrale, (b) **Content-Scripts**, die in jede besuchte Seite und jeden Frame geladen werden, (c) einer **Popup-Oberfläche** zur Bedienung, (d) mehreren **eingebetteten Oberflächen-Seiten** (Start-Animation, Taskleiste, Beenden-Dialog), die als iframes in die Prüfungsseite gelegt werden, und (e) einem **Build-Skript**, das daraus zwei Pakete (Firefox, Chromium) erzeugt. Fachlich hat die Erweiterung zwei weitgehend getrennte Funktionsblöcke: **„SEB-Block"** (Konfigurationsdatei erkennen → laden → Anfragen an den Prüfungsserver modifizieren → SEB-ähnliche Oberfläche einblenden) und **„Assistant-Block"** (Fragen auslesen → externen KI-Dienst fragen → Antworten eintragen). Beide teilen sich nur die Hintergrundseite, die Browser-API-Schicht und das Popup.

---

## 2. Komponenten, Ausführungsumgebungen, verwendete APIs

### 2.1 Ausführungsumgebungen

Eine Browser-Erweiterung läuft in **getrennten Umgebungen**, die nur über Nachrichten kommunizieren.

| Umgebung | Läuft wo | Lebensdauer | Dateien | Zugriff |
|---|---|---|---|---|
| **Background Page** | unsichtbare Dauerseite (`persistent: true`) | solange der Browser läuft | `background.html` → `seb/background.js` und alles, was es importiert | Voller Zugriff auf Erweiterungs-APIs (`tabs`, `webRequest`, `downloads`, `storage`), Netzwerk ohne Seiten-Einschränkung |
| **Content-Script** (isolierte Welt) | in jeder Webseite, jedem Frame, ab `document_start` | solange die Seite geladen ist | `browser-api.js`, `assistant/extract.js`, `assistant/apply.js`, `assistant/content.js`, `seb/download-content.js`, `seb/content.js` (+ `seb/content.css`) | DOM der Seite, aber nicht deren JS-Variablen; wenige Erweiterungs-APIs (`runtime`, `storage`) |
| **Page-Script** („MAIN world") | im JS-Kontext der Webseite | solange die Seite geladen ist | `seb/download-page.js` | Sieht und verändert Seiten-JS-Objekte (z. B. `URL.createObjectURL`); **kein** Zugriff auf Erweiterungs-APIs |
| **Popup / Import-Tab** | Fenster beim Klick aufs Symbol; die Dateiauswahl öffnet dieselbe Seite in einem **eigenen Tab** (`popup.html?file=1&target=<tabId>`, `seb/popup.js:140-151`) | solange geöffnet | `popup.html`, `popup.css`, `seb/popup.js`, `assistant/popup.js` | Erweiterungs-APIs. Eigener Zustand nur die zuletzt gewählte Ansicht in `localStorage` (`assistant/popup.js:109,553`); Daten liegen sonst im Erweiterungsspeicher |
| **Eingebettete UI-Frames** | iframes in der Prüfungsseite, geladen von der Erweiterung | solange der Content-Script sie vorhält | `seb/assets/startup.html/.js`, `taskbar.html/.js`, `quit.html/.js`, `colors.css`, `quit.css` | Eigene Seite der Erweiterung, kommunizieren per `postMessage` mit dem Content-Script |

**BESTÄTIGT:** Content-Scripts sind **klassische Skripte** (kein `import`/`export`); sie teilen Funktionen über das globale Objekt `globalThis.OEBAssistant` (`assistant/extract.js:2`, `assistant/apply.js:2`). Background und Popup nutzen **ES-Module** (`background.html:6`, `popup.html:204-205`).

### 2.2 Verwendete APIs (Einordnung)

Der Code nutzt zwei grundverschiedene API-Arten. Nur die erste ist Erweiterungs-spezifisch und browserabhängig.

**A. WebExtension-APIs** (Namespace `browser.*`; in Chromium über die Fassade `browser-api.js` auf `chrome.*` abgebildet). Auflistung nach Durchsuchen des Quellcodes (BESTÄTIGT):

| Namespace | Genutzt |
|---|---|
| `runtime` | `sendMessage`, `onMessage`, `getURL`, `onInstalled`; `getBrowserInfo` nur als Erkennungsmerkmal für Firefox (`background.js:102`, `headers.js:40`, `download-content.js:59`, `browser-api.js:2`) |
| `storage` | `local`, `session` (Fallback auf `local`, `browser-api.js:3,28`), `onChanged` |
| `tabs` | `query`, `get`, `create`, `update`, `reload`, `remove`, `sendMessage`, `onRemoved`, `onUpdated` |
| `webRequest` | `onBeforeSendHeaders` (blocking), `onHeadersReceived` (blocking), `onCompleted`, `onErrorOccurred`; `filterResponseData` nur Firefox |
| `downloads` | nur `onCreated` |
| `commands` | `onCommand`; `update`/`getAll`/`reset` nur wenn vorhanden (`assistant/popup.js`, Zeile mit Prüfung vor Verwendung) |
| `browserAction` (bzw. `action`) | `setBadgeText`, `setBadgeBackgroundColor` |

`browser-api.js` bildet zusätzlich `windows` ab; im Code wird `browser.windows` **nicht** verwendet (BESTÄTIGT durch Suche).

**B. Web-Plattform-APIs** (Standard-Browserfunktionen, kein Erweiterungsbezug; Verfügbarkeit hängt von der Browserversion ab, nicht von der Erweiterung):

| API | Verwendung | Fundstelle |
|---|---|---|
| `crypto.subtle` (SHA-256, HMAC, AES-CBC, PBKDF2) | Konfigurations-Hash, Entschlüsselung `.seb`, Katalog-Prüfsumme, Schlüsselableitung | `parser.js:154-213`, `config-key.js:261-264`, `bek-catalog.js:27`, `browser-exam-key.js` |
| `DecompressionStream`, `Blob`, `Response` | Gzip entpacken | `parser.js:117-118`, `download-check.js:73` |
| `fetch`, `AbortSignal.timeout`, `ReadableStream` | alle Netzwerkabrufe | siehe 6.3 |
| `postMessage`, `MutationObserver` | Seite↔Content-Script, Skript-Injektion | `download-content.js`, `content.js` |
| `FormData`, `FileReader` | Formularabfangung, Bild→Base64 | `seb/content.js:57`, `assistant/extract.js:41` |
| `navigator.clipboard`, `localStorage` | Kopieren im Popup, Ansichtsmerker | `seb/popup.js:193`, `assistant/popup.js:109` |
| `navigator.getBattery` (Battery Status API) | Batterieanzeige der Taskleiste | `seb/assets/taskbar.js:44-57` |
| `Intl.Collator`, `BigInt`, `TextEncoder/Decoder`, `atob` | Hash-Berechnung, Kodierung | `config-key.js:3-4,273` u. a. |

`parser.js` verwendet **keine WebExtension-APIs**, wohl aber Web-Plattform-APIs (Zeile oben). Die Aussage „reiner Datenbaustein" bezieht sich nur auf die Erweiterungs-Unabhängigkeit.

---

## 3. Dateiinventar mit Zweck und Zusammenhang

**BESTÄTIGT:** Das ZIP enthält 55 Dateien; alle Code-, Konfigurations- und Oberflächendateien sind unten einzeln oder als Gruppe aufgeführt. Alle angegebenen Zeilenzahlen wurden mit `wc -l` gegengeprüft.

### 3.1 Wurzelverzeichnis

| Datei | Aufgabe | Hängt zusammen mit |
|---|---|---|
| `README.md` | Projektbeschreibung des Autors, Installations- und Build-Hinweise; nennt Herkunft („Based on SebBypass"), Haftungsausschluss | – |
| `package.json` | Name `oeb`, Paketversion `2.4.0`, Skript `build`, Dev-Abhängigkeiten, `"type": "module"` | `build.mjs`, `package-lock.json` |
| `package-lock.json` | Lockfile v3, 25 Pakete (alle Dev) | `package.json` |
| `build.mjs` | Build und Paketierung (Abschnitt 14) | `source/**`, `bek.json`, `source/seb/bek-catalog.js` |
| `bek.json` | **Datenkatalog** mit buildspezifischen Profilen (≈ 59 KB; `schemaVersion: 1`, `defaultVersion`, `profiles` mit derzeit einem Eintrag). Wird vom Build ins Paket kopiert **und** zur Laufzeit von GitHub nachgeladen. Inhalt wird hier bewusst nicht dokumentiert | `seb/bek-catalog.js`, `build.mjs` |
| `.gitignore` | Ignoriert `/release/`, `/node_modules/`, `/source/manifest.json`, `*.log`, `.DS_Store` | `build.mjs` |
| `preview/*.gif` | Demo-Animationen (nur Dokumentation) | `README.md` |

**BESTÄTIGT (Negativbefund):** Es gibt kein Testverzeichnis, keine Testkonfiguration, kein Test-Skript in `package.json`, keine CI-Konfiguration, keine Lizenzdatei, keinen Changelog, kein `engines`-Feld und keine `.nvmrc`.

### 3.2 `source/` – Wurzel der Erweiterung

| Datei | Aufgabe | Hängt zusammen mit |
|---|---|---|
| `chrome.manifest.json` | Manifest V2 für Chromium: Berechtigungen, Background-Seite, Popup, Kürzel, Content-Script-Liste, `web_accessible_resources` (inkl. `seb/download-page.js`), Mindestversion 111 | `build.mjs` |
| `firefox.manifest.json` | Manifest V2 für Firefox: ID `oeb@open-exam-browser.local`, `strict_min_version 140` (Android 142), `data_collection_permissions` (`websiteContent`, `authenticationInfo`), zweiter Content-Script-Block mit `world: "MAIN"` | `build.mjs` |
| `background.html` | 8 Zeilen: lädt `browser-api.js`, dann `seb/background.js` als Modul | – |
| `browser-api.js` | **Kompatibilitätsschicht**: Wenn `browser.runtime.getBrowserInfo` fehlt, wird ein `browser`-Objekt auf Basis von `chrome.*` mit Promises aufgebaut; fehlt `storage.session`, wird `storage.local` untergeschoben | Wird in Hintergrund, Popup und Content-Scripts als **erstes** geladen |
| `popup.html` / `popup.css` | Struktur (207 Z.) und Stil (914 Z.) des Popups mit Reitern „SEB" und „Assistant" | `seb/popup.js`, `assistant/popup.js` |
| `icon.svg`, `assets/icons/icon-{16,32,48,128}.png` | Symbole (Firefox: SVG, Chromium: PNG) | Manifeste |

### 3.3 `source/seb/` – SEB-Block

| Datei (Zeilen) | Aufgabe | Importiert / genutzt von |
|---|---|---|
| `background.js` (748) | **Steuerzentrale**: Nachrichten-Router, Zustandsverwaltung, Download-Erkennung, Konfigurationswechsel, Start/Beenden, Badge, Einstellungen | Importiert `parser`, `headers`, `config-key`, `assistant/background`, `browser-exam-key`, `launch-response`, `settings`, `download-check`, `download-stream`, `bek-catalog` |
| `headers.js` (121) | Registriert den `webRequest`-Hook und hält den **Header-Zustand**. Wirkt nur auf Anfragen an den Ursprung der Prüfungs-Start-URL | Nur `background.js`; importiert `config-key`, `browser-exam-key` |
| `config-key.js` (337) | (1) Eigener **Plist-XML-Parser**, (2) Auslesen einzelner Konfigurationswerte (Start-URL, Quit-Einstellungen, Taskleiste, Salt), (3) Hash-Berechnung, (4) eigene **synchrone SHA-256-Implementierung** (Z. 266-337) | `background.js`, `headers.js`, `browser-exam-key.js`, `seb/popup.js` |
| `browser-exam-key.js` (64) | Normalisierung der BEK-Einstellungen, Bereitstellung des Schlüssels für den Header-Hook. Enthält ungenutzten Code (9.4) | `headers.js`, `background.js`; nutzt `config-key`, `bek-catalog` |
| `bek-catalog.js` (143) | Profilkatalog: Validierung, Laden aus Cache/Paket, Aktualisierung von GitHub, Zeitsteuerung. `createBekCatalogManager` mit injizierbarem `storage`/`fetcher`/`now` | `browser-exam-key.js`, `background.js`, `build.mjs` |
| `bek-selection.js` (7) | `bekSettingsForVersion`: aus Auswahlfeld-Wert („off" oder Version) ein Einstellungsobjekt (immer Modus „automatic") | `seb/popup.js` |
| `parser.js` (246) | Formaterkennung der `.seb`-Datei, Entpacken, Passwort-Entschlüsselung; Fehlerklasse `SebParseError`. Zertifikat-verschlüsselte Dateien werden abgelehnt | `background.js` |
| `settings.js` (16) | Schlüssel, Standardwerte und `normalizeSettings` (Download-Prüfung, Tab beim Beenden schließen, Start-Animation, Taskleistenmodus und -größe 50-300; Altformat `showSebBar`) | `background.js` |
| `file-import.js` (9) | `importSelectedFile`: liest eine gewählte Datei und sendet sie als `SEB_PROCESS_BYTES` | `seb/popup.js` |
| `launch-response.js` (23) | `isSebConfigurationUrl`, `classifySebResponse`: erkennt anhand URL, `Content-Disposition`, MIME-Typ, ob eine HTTP-Antwort eine SEB-Konfiguration ist | `background.js` |
| `download-check.js` (88) | Begrenztes, **nur lesendes** Prüfen von Downloads unbekannten Typs (≤ 8 MB, Streaming, 3 s Timeout) auf SEB-Signaturen | `background.js` |
| `download-stream.js` (41) | Firefox: `mirrorDownloadResponse` spiegelt eine Antwort über `filterResponseData` unverändert weiter und prüft eine begrenzte Kopie; `isExtensionlessResponse` | `background.js` |
| `download-content.js` (78) | Content-Script-Brücke zwischen Seiten-Skript (`postMessage`) und Hintergrund (`runtime.sendMessage`); beantwortet `SEB_PROBE_DOWNLOAD_URL` (Abruf mit den **Cookies der Seite**); injiziert in Chromium `download-page.js` per `<script>`-Tag | `download-page.js`, `background.js` |
| `download-page.js` (126) | Läuft im Seitenkontext: merkt sich Blob-URLs, ersetzt `URL.createObjectURL`/`revokeObjectURL`/`HTMLAnchorElement.prototype.click`, prüft Download-Links, übergibt Kandidaten an die Brücke | `download-content.js` |
| `content.js` (316) | Content-Script des SEB-Blocks: fängt Klicks/Formulare auf SEB-Links ab, zeigt Status, steuert Overlay-Frames, reagiert auf Taskleisten-Nachrichten, löst die Moodle-Rücksetzung aus | `background.js` (Nachrichten), UI-Frames (`postMessage`) |
| `content.css` (40) | Stil der Overlay-Frames (`position: fixed`, höchster `z-index`, `!important`) – auf **jeder** Seite eingebunden | `content.js` |
| `popup.js` (351) | Logik des SEB-Reiters: Status, Datei öffnen (eigener Tab), Passwort, BEK-Version, Einstellungen, XML anzeigen/kopieren/speichern | `background.js` (Nachrichten), `config-key.js`, `file-import.js`, `bek-selection.js` |
| `assets/startup.html/.js` (554/95) | **Start-Animation** im Stil des SEB-Startbildschirms. `startup.html` enthält zusätzlich einen inerten `<script type="text/plain">`-Block (ab Z. 238), der nie ausgeführt wird und eine ältere Fassung der Animationslogik samt Build-Angabe enthält | `content.js` |
| `assets/taskbar.html/.js` (111/69) | **Taskleiste**: Uhr, Tastatursprache, Netzstatus, Audio, Reload, Beenden und eine **Batterieanzeige** (`navigator.getBattery`, nur wo die API existiert, `taskbar.js:44-57`) | `content.js` |
| `assets/quit.html/.js/.css` (50/34/101) | **Beenden-Dialog** (mit/ohne Passwortfeld) | `content.js` |
| `assets/colors.css` (11) | gemeinsame Farbvariablen von `startup.html` und `taskbar.html` | – |
| `icons/*.png/svg` | Logos/Symbole der Overlay-Frames | `startup.html`, `taskbar.html` |

### 3.4 `source/assistant/` – Assistant-Block

| Datei (Zeilen) | Aufgabe | Zusammenhang |
|---|---|---|
| `background.js` (105) | **Orchestrierung** eines KI-Laufs: Vorbedingungen prüfen, Seite auslesen lassen, Anbieter fragen, Seite erneut prüfen, Antworten eintragen lassen, Ergebnis speichern; Registrierung des Kürzel-Listeners; Badge | Importiert `providers.js`; eingebunden von `seb/background.js` (`answerTab`, `installAssistantCommands`) |
| `providers.js` (441) | Katalog der Dienste (8 feste + 1 frei konfigurierbar), Konto-Datenstruktur, Validierung, Modelllisten, Anfrage/Antwort je Dienst-Familie, Fallback über mehrere Konten. Prompt-Inhalt hier nicht dokumentiert | `assistant/background.js`, `assistant/popup.js` |
| `extract.js` (178) | Content-Script-Modul: liest die Fragen der Seite in eine strukturierte Liste (inkl. begrenzter Bilddaten) | `OEBAssistant.extractQuestions` |
| `apply.js` (202) | Content-Script-Modul: trägt Antworten in Formularelemente ein; liefert pro Frage „filled"/„skipped" | `OEBAssistant.applyAnswers` |
| `content.js` (56) | Content-Script-Brücke, **nur im obersten Frame** (Z. 2): beantwortet `OEB_AI_EXTRACT`/`OEB_AI_APPLY` (Z. 26-42), hält das „Nächste Seite"-Kürzel (Z. 17-24) | `extract.js`, `apply.js` |
| `popup.js` (600) | Logik des Assistant-Reiters: KI-Konten, Domain-Freigabe, Fragen anzeigen, Lauf starten, Kürzel | `providers.js`; sendet `OEB_AI_RUN` an den Hintergrund und `OEB_AI_EXTRACT` **direkt** an den Tab (Z. 413, 436) |

---

## 4. Funktionen nach Komponente

Nur Funktionsnamen, Zweck und Fundstelle.

### 4.1 `seb/background.js`

| Funktion | Zeile | Zweck |
|---|---|---|
| (Listener) `runtime.onInstalled` | 59 | Schreibt Standard-Einstellungen (falls fehlend) und setzt den Zustand auf „idle" – **bei jeder Installation und jedem Update** |
| (Listener) `runtime.onMessage` → `handleMessage` | 65, 212 | **Zentraler Nachrichten-Router** (20 Typen, Katalog in 6.2) |
| (Listener) `downloads.onCreated` | 72 | Fallback der Erkennung bei Browser-Downloads |
| (Listener) `webRequest.onHeadersReceived` | 95 | Haupt-Erkennungsweg für HTTP-Antworten; delegiert an `inspectDownloadResponse` |
| `inspectDownloadResponse` | 109 | Pro Antwort: spiegeln (Firefox), als SEB erkennen, prüfen oder ignorieren; kann abbrechen („cancel") |
| `probeDownloadOnce` / `inspectUnknownDownload` | 153 / 176 | Inhalt unbekannter Downloads begrenzt prüfen; Doppelabrufe vermeiden |
| `captureAutomaticBytes` | 188 | Automatisch erkannte Bytes validieren, Duplikate unterdrücken, an `processBytes` übergeben |
| `processUrl` / `processUrlOnce` | 348 / 359 | Konfigurations-URL (auch `seb://`, `sebs://`, eingebettete Base64-Daten) holen und weitergeben |
| `processBytes` | 436 | **Serialisiert das Ersetzen einer Konfiguration** über `configurationQueue` (435-440) |
| `replaceConfiguration` | 442 | Parsen → alten Zustand entladen → neuen Zustand speichern → ggf. starten |
| `unlockPending` | 493 | Passwortgeschützte Datei mit eingegebenem Passwort öffnen (**nicht** über die Queue) |
| `activateLaunch` | 559 | Start-URL bestimmen, Header-Hook aktivieren, Zielreiter überblenden/neu laden |
| `findTargetTab` / `transitionToDestination` / `beginPendingReload` | 586 / 593 / 607 | Zielreiter; Übergang Start-Animation → Neuladen |
| `unloadConfiguration` | 615 | Zurücksetzen: Moodle-Rücksetzung, Header aus, Zustand leeren, Overlay entfernen, Tabs optional schließen/neuladen |
| `saveState` / `loadState` | 540 / 544 | Zustandsspeicher |
| `loadSettings` / `saveSettings` / `loadBekSettings` / `enableLoadedHeaders` | 639 / 662 / 644 / 654 | Einstellungen und Schlüsselmodus; Header-Hook aus geladenem Zustand aktivieren |
| `buildPresentationSettings` / `broadcastPresentation` / `broadcastMessage` | 675 / 669 / 683 | Darstellungseinstellungen zusammensetzen und an Tabs senden |
| `setBadge` | 553 | Badge: `…` lädt, `✓` bereit, `?` Passwort nötig, `!` Fehler |
| Hilfsfunktionen | 690-748 | `isWebUrl`, `getMoodleCmid`, `normalizeSebUrl`, `isSebUrl`, `isSebMime`, `fileNameFrom*` |

### 4.2 `seb/headers.js` (Rolle)

| Funktion | Zeile | Zweck |
|---|---|---|
| (Listener) `webRequest.onBeforeSendHeaders` | 19 | Blockierender Hook; ändert Anfrage-Header nur für den Prüfungs-Ursprung (Prüfung Z. 21); in nicht-Firefox-Browsern mit zusätzlicher Option `extraHeaders` (Z. 40-42) |
| `enableHeaderInjection` | 53 | Validiert Eingaben, löst den Schlüssel auf, setzt Modulzustand |
| `disableHeaderInjection` | 73 | Setzt den Modulzustand zurück |
| `reloadActiveHeaderTab` / `getHeaderStatus` | 85 / 93 | Aktiven Tab neu laden / Status fürs Popup |
| `refreshMatchingTabs`, `isAllowedUrl`, `recordError` | 106-121 | Zähler, URL-Prüfung, Fehlermerker |

### 4.3 `seb/config-key.js`

| Funktion | Zeile | Zweck |
|---|---|---|
| `calculateSebConfigHash` | 6 | Konfigurations-Hash |
| `calculateRequestConfigKey` / `…Sync` | 13 / 17 | Anfragebezogener Hash (async **ungenutzt**, sync genutzt) |
| `getSebExamSalt`, `getSebStartUrl`, `getSebQuitSettings`, `getSebPresentationSettings` | 21-68 | Einzelwerte aus der Konfiguration |
| `verifySebQuitPassword` | 70 | Prüft das Beenden-Passwort **lokal** gegen den Hash in der Konfiguration |
| `serializeSebConfiguration` | 76 | **Ungenutzt** |
| `parsePlist` und Hilfsfunktionen | 83-259 | Eigener Plist-Parser/Serialisierer (toleriert eigene Overlay-Frames im kopierten XML, Z. 95-106) |
| `sha256Hex`, `sha256HexSync`, `rotateRight` | 261-337 | Hashfunktionen |

### 4.4 `seb/browser-exam-key.js` und `seb/bek-catalog.js`

| Funktion | Datei:Zeile | Zweck |
|---|---|---|
| `normalizeBekSettings` | browser-exam-key:9 | Validiert Schlüsselmodus („automatic"/„provided") und Build-Angabe |
| `resolveBrowserExamKey` | :23 | Liefert je Modus den Schlüssel bzw. `undefined` |
| `calculateNativeBrowserExamKey` | :34 | Automatische Berechnung aus Konfiguration und Katalogprofil |
| `calculateFallbackBrowserExamKey` | :49 | **Ungenutzt** |
| `calculateRequestBrowserExamKeySync` | :61 | Synchrone Ableitung je Anfrage |
| `validateBekCatalog` | bek-catalog:9 | Schema- und Prüfsummenvalidierung (Prüfsumme steht in derselben Datei, siehe TR-07 in Abschnitt 12) |
| `createBekCatalogManager` (`initialize`, `info`, `refresh`, `get`) | :64 | Katalogspeicher mit Cache, Zeitsteuerung, Fehlerzustand |
| `getBekCatalogInfo` / `refreshBekCatalog` / `getBekProfile` | :136-143 | Öffentliche Schnittstelle |

### 4.5 `seb/parser.js`

`decodeEmbeddedSebUrl` (19), `inspectSebFile` (37), `unwrapPlainPayload` (83), `unwrapGzip` (107; höchstens 4 Schichten, **keine Größenbegrenzung der entpackten Daten**), `decryptRNCryptor` (127), `deriveKey` (191), `sha256Hex` (212), `readyResult`, `looksLikeXml`, `isGzip`, `ascii`, `toBytes`.

### 4.6 `seb/content.js`

| Funktion | Zeile | Zweck |
|---|---|---|
| Klick-Listener (Capture) | 34 | Fängt Klicks auf SEB-Links/Buttons ab |
| Submit-Listener (Capture) | 49 | Dasselbe für Formulare (GET/POST); Dateiuploads ausgenommen; Formulardaten werden erst gelesen, wenn das Ziel eine SEB-URL ist (Z. 55-62) |
| `findUrl`, `isSebUrl`, `sendUrl`, `showLinkStatus` | 79-146 | URL ermitteln, prüfen, an Hintergrund senden (`SEB_CAPTURE_URL`), Statusmeldung |
| `initializePresentation` | 148 | Beim Laden den Hintergrund fragen, ob das Overlay erscheinen soll |
| `showStartup`, `applyPresentation`, `showSebBar`, `removePresentation`, `openQuitDialog` | 163-248 | Overlay-Frames anlegen/aktualisieren/entfernen |
| `handlePresentationMessage` | 250 | Nachrichten der Frames (Beenden, Reload, Audio) |
| `resetMoodleSebAccess` | 285 | Sendet die Rücksetz-Anfrage an Moodle im Namen der angemeldeten Sitzung |

### 4.7 `seb/popup.js`

Handler für Datei-Öffnen (140), Datei-Auswahl (153), Verbinden/Trennen (172), Kopieren (192), XML speichern (198), Entladen (208), Einstellungen (213-225); `storage.onChanged` (227-233); `refresh` (235), `refreshSettings` (241), `saveDisplaySettings` (254, serialisiert über `displaySaveQueue`), `refreshBekSettings` (69), `render` (292), `formatMode`, `formatXml`.

### 4.8 `assistant/*`

| Funktion | Datei:Zeile | Zweck |
|---|---|---|
| `installAssistantCommands` | background:17 | Registriert den Listener für das Kürzel `answer-questions` |
| `answerTab` | background:23 | Einstieg; verhindert parallele Läufe pro Tab (`runningTabs`) |
| `runAssistant` | background:38 | Gesamtablauf (5.4) |
| `saveRun`, `setBadge`, `questionFingerprint`, `withoutImageData` | background:101, 96, 9, 13 | Ergebnis speichern, Badge, Vergleich/Bereinigung |
| `askWithFallback` | providers:194 | Versucht aktivierte Konten der Reihe nach |
| `askProvider` | providers:216 | Eine Anfrage je Dienst-Familie (vier Varianten) |
| `fetchProviderModels`, `validateProviderAccount`, `normalizeProviderAccounts`, `newProviderAccount`, `providerDisplayName`, `modelImageCapability` | providers | Kontoverwaltung und Modellliste |
| `requestJson` | providers:417 | HTTP-Hilfsfunktion mit 45-s-Timeout |
| `getBaseUrl` | providers:385 | Endpunkt auflösen; entfernte Endpunkte nur über HTTPS (Ausnahme: `localhost`, `127.0.0.1`, `[::1]`) |
| `extractQuestions` / `applyAnswers` | extract:3 / apply:3 | Seite lesen / Seite beschreiben |
| `removeProvider` / `saveProviders` | popup:359 / 366 | Konto aus der Liste entfernen / Liste speichern |

---

## 5. Zusammenspiel: die wichtigsten Abläufe

### 5.1 Konfiguration erkennen und laden

```
Auslöser (einer von mehreren Wegen)
 ├─ Klick/Formular auf SEB-Link ......... seb/content.js → SEB_CAPTURE_URL ──┐
 ├─ HTTP-Antwort ist SEB-Konfiguration .. webRequest.onHeadersReceived ──────┤
 ├─ Download ohne Endung (Firefox) ...... download-stream.js (Spiegelung) ───┤
 ├─ Blob-Download auf der Seite ......... download-page.js → download-content.js → SEB_CHECK_DOWNLOAD_BYTES
 ├─ Browser-Download-Ereignis ........... downloads.onCreated ──────────────┤
 └─ Nutzer wählt Datei (Import-Tab) ..... file-import.js → SEB_PROCESS_BYTES ┘
                                       ▼
                 background.js: processUrl / captureAutomaticBytes
                                       ▼
                 processBytes (Queue configurationQueue)
                                       ▼
                 replaceConfiguration:
                   1. parser.inspectSebFile → Klartext-XML | "Passwort nötig" | Fehler
                   2. config-key: Konfigurations-Hash berechenbar?
                   3. unloadConfiguration (alter Zustand weg)
                   4. saveState({status: "ready" | "password-needed" | "error", …})
                   5. Badge setzen
                   6. wenn autoLaunch → activateLaunch (5.2)
```

Wichtige Eigenschaften:
- **BESTÄTIGT:** Alle Erkennungs- und Ladewege führen über `processBytes`; dadurch wird immer nur **ein Ersetzen** gleichzeitig ausgeführt (`background.js:435-440`). **Ausnahme:** Das Öffnen einer zuvor als „Passwort nötig" abgelegten Datei (`unlockPending`, `:493-538`) wird nicht über die Queue geführt. Auch das Entladen ist außerhalb von `replaceConfiguration` nicht serialisiert (Details in 7.3).
- **BESTÄTIGT:** Doppelerkennungen werden mehrfach unterdrückt: Zeitfenster 8 s für eingebettete Daten (`duplicateRequests`, `:372`), Schlüssel aus URL+Hash (`automaticCaptures`, `:194`), Hash der aktuell geladenen Bytes (`loadedBytesDigest`, `:198`). Kommentar im Code: A → B → A muss A wieder laden (`:197`).
- **BESTÄTIGT:** Chromium kann Antworten nur synchron beantworten und hat kein `filterResponseData`; dort gibt es Ersatzwege (Probe-Abruf mit Seiten-Cookies, Blob-Hook). Firefox darf asynchron antworten und spiegeln (`background.js:101-102`).

### 5.2 Start („Launch")

```
activateLaunch(xml, metadata)
 1. Konfigurations-Hash berechnen; Start-URL aus der Konfiguration (Fallback: URL der Seite)
 2. Darstellungseinstellungen aus Nutzereinstellungen + Konfiguration zusammenführen
 3. Zielreiter bestimmen (bevorzugt der auslösende, sonst aktiver, sonst erster Web-Tab)
 4. headers.enableHeaderInjection(...)       → ab jetzt wird jede Anfrage an den Prüfungs-Ursprung verändert
 5. SEB_UNLOAD_PRESENTATION an alle; presentationTabs = {Zielreiter}
 6. transitionToDestination:
      Animation an → pageTransitions[tab]={phase:"startup"} → SEB_SHOW_STARTUP
                    → (Frame meldet Ende) → SEB_STARTUP_FINISHED → beginPendingReload → tabs.update(url)
      Animation aus (oder Seite ohne Content-Script) → direkt tabs.update(url)
 7. Seite lädt neu → content.js:initializePresentation → SEB_GET_PAGE_PRESENTATION
      → applyPresentation (Taskleiste) → SEB_TRANSITION_FINISHED
 8. cmid aus Start-URL im Zustand ablegen (für Rücksetzung beim Entladen)
```

**BESTÄTIGT:** Reihenfolge nach `background.js:559-584`. Schlägt ein Schritt **nach** Schritt 4 fehl, wird der Zustand auf „error" gesetzt, der Header-Hook aber nicht deaktiviert (siehe TR-01 in 12).

### 5.3 Beenden / Entladen

```
Nutzer klickt "Beenden" in der Taskleiste (iframe)
  → postMessage SEB_QUIT_REQUEST → content.js.openQuitDialog
  → SEB_GET_QUIT_REQUIREMENTS (Konfiguration erlaubt Beenden? Passwort nötig?)
  → Dialog (quit.html) → postMessage SEB_QUIT_SUBMIT → content.js → SEB_QUIT
  → background: Prüfungen (Konfiguration geladen? Tab gehört dazu? erlaubt? bestätigt? Passwort passt lokal?)
  → unloadConfiguration({closeTabs: Einstellung closeTabOnQuit})
       Moodle-Rücksetzung (SEB_RESET_MOODLE_ACCESS) → Header-Hook aus → Zustand "idle" → Overlay weg → Tab schließen/neuladen
```

- **BESTÄTIGT:** Das Beenden-Passwort wird **lokal** gegen den Hash in der Konfiguration geprüft (`config-key.js:70-74`); es gibt keine Serverprüfung.
- **BESTÄTIGT (K1):** Die Moodle-Rücksetzung (POST an `/mod/quiz/accessrule/seb/clear_session.php`, `seb/content.js:285-306`) erfolgt bei **jedem Entladen** (Beenden, Popup-„Unload", **und beim Ersetzen durch eine neue Konfiguration**), aber nur, wenn aus der Start-URL eine `cmid` ermittelt werden konnte (Pfad enthält `/mod/quiz/`, Parameter `id`): `background.js:615-623`, `:448`, `:698-707`.

### 5.4 KI-Lauf (Phasenübersicht)

```
Auslöser: Tastenkürzel (Standard Ctrl+Shift+Y) oder Popup-Button → OEB_AI_RUN
 → assistant/background.answerTab → runAssistant
    1. Vorbedingungen: Domain im Popup freigeschaltet? ≥ 1 aktiviertes KI-Konto? (sonst Fehler)
    2. OEB_AI_EXTRACT  → Content-Script liest Fragen        (extract.js)
    3. askWithFallback → externer Dienst, Konten der Reihe nach (providers.js)
    4. Plausibilität: gleiche URL? nochmals auslesen – Fragen unverändert?
    5. OEB_AI_APPLY    → Content-Script trägt Antworten ein (apply.js)
    6. Ergebnis zählen, Badge setzen, Lauf unter oebAiLastRun speichern
```

- **BESTÄTIGT:** Der Block ist vom SEB-Block unabhängig: Er prüft nicht, ob eine Konfiguration geladen ist (`assistant/background.js:38-94`).
- **BESTÄTIGT (K3):** Das Kürzel „Nächste Seite" (Standard Ctrl+Shift+X) ist ein `keydown`-Listener in der **Capture-Phase** (`assistant/content.js:17-24`). Existiert ein aktiver „Weiter"-Button, wird das Ereignis verschluckt (`preventDefault` + `stopImmediatePropagation`) und der Button per `click()` ausgelöst. Im Unterschied zum KI-Lauf gibt es **keine Domain-Freigabe**; nur der oberste Frame ist aktiv.
- **BESTÄTIGT:** Das Absenden der Prüfung wird nirgends ausgelöst (keine `submit`/`requestSubmit`-Aufrufe im Quellcode).
- **BESTÄTIGT:** Die Popup-Anzeige der Fragen (`assistant/popup.js:413`) fragt den Tab **direkt** per `tabs.sendMessage(OEB_AI_EXTRACT)`; nur der Lauf selbst geht über den Hintergrund (`:436`).

---

## 6. Kommunikationswege

### 6.1 Überblick

```
                  ┌──────────────── Popup / Import-Tab (popup.html) ───────────────┐
                  │ seb/popup.js            assistant/popup.js                     │
                  └───────┬──────────────────────────┬──────────────────────────────┘
  runtime.sendMessage     │                          │  storage.local (direkt lesen/schreiben:
  (SEB_*, OEB_AI_RUN)     │                          │  KI-Konten, Domains, Notiz, Kürzel)
                          ▼                          │  tabs.sendMessage (OEB_AI_EXTRACT, direkt)
 ┌──────────────── Background Page ──────────────────┼────────────────────────────┐
 │ seb/background.js ── importiert ──► headers, config-key, parser, bek-catalog,   │
 │   (Router, Zustand)                  download-*, …                              │
 │        └── ruft ──► assistant/background.js ──► providers.js ──► HTTPS zu KI-Diensten
 │   webRequest-Hooks (Antworten erkennen, Anfragen verändern)  ──► HTTPS zum Prüfungsserver (fetch mit Cookies)
 │   Katalog-Aktualisierung ──────────────────────────────────────► HTTPS GitHub (bek.json)
 └───────▲──────────────────────────┬──────────────────────────────────────────────┘
         │ runtime.sendMessage      │ tabs.sendMessage (SEB_SHOW_STARTUP, SEB_APPLY_PRESENTATION,
         │ (SEB_CAPTURE_URL, …)     │ SEB_UNLOAD_PRESENTATION, SEB_RESET_MOODLE_ACCESS,
         │                          │ SEB_PROBE_DOWNLOAD_URL, OEB_AI_EXTRACT, OEB_AI_APPLY)
         │                          ▼
 ┌───────┴────────────── Content-Scripts je Seite/Frame ───────────────────┐
 │ seb/content.js   seb/download-content.js   assistant/content.js         │
 │   (+ extract.js, apply.js als globale Module)                           │
 └───────┬───────────────────────────────┬─────────────────────────────────┘
         │ window.postMessage            │ iframe + postMessage
         │ Kanal "oeb-download-capture-v1"│ (SEB_QUIT_REQUEST, SEB_RELOAD_REQUEST, …)
         ▼                               ▼
   seb/download-page.js           startup.html / taskbar.html / quit.html
   (Seitenkontext)                (Erweiterungs-Seiten im iframe)
```

### 6.2 Nachrichtenkatalog (erweiterungsintern)

**Richtung „→ Hintergrund"** (`runtime.sendMessage`, beantwortet von `handleMessage`, `background.js:212-346`). **BESTÄTIGT: genau 20 Typen** (`background.js:213-344`):

| Nachricht | Absender (Fundstelle) | Zweck |
|---|---|---|
| `SEB_CAPTURE_URL` | `seb/content.js:110` | Link/Formular auf Konfiguration abgefangen → herunterladen und laden |
| `SEB_CHECK_DOWNLOAD_BYTES` | `download-content.js:24` | Von der Seite übergebene Bytes auf SEB-Konfiguration prüfen |
| `SEB_PROCESS_BYTES` | `file-import.js:5` (Import-Tab) | Vom Nutzer gewählte Datei laden |
| `SEB_UNLOCK` | `seb/popup.js:127` | Passwort für geschützte Datei |
| `SEB_GET_STATE` | `seb/popup.js:236` | Zustand lesen |
| `SEB_GET_SETTINGS` | `seb/popup.js:242`; `download-content.js:6` | Einstellungen lesen |
| `SEB_SET_SETTINGS` | `seb/popup.js:269` | Einstellungen speichern |
| `SEB_GET_BEK_SETTINGS`, `SEB_REFRESH_BEK_CATALOG` | `seb/popup.js:70` | Schlüsselmodus lesen / Katalog erzwungen aktualisieren |
| `SEB_SET_BEK_SETTINGS` | `seb/popup.js:54,75` | Schlüsselmodus speichern (UI sendet immer Modus „automatic") |
| `SEB_HEADER_ENABLE`, `SEB_HEADER_DISABLE` | `seb/popup.js:180,177` | „Exam connection" manuell verbinden/trennen |
| `SEB_UNLOAD` | `seb/popup.js:209` | Entladen mit Neuladen der Tabs |
| `SEB_CLEAR` | **kein Absender** | Entladen ohne Neuladen (ungenutzt) |
| `SEB_GET_PAGE_PRESENTATION`, `SEB_STARTUP_FINISHED`, `SEB_TRANSITION_FINISHED` | `seb/content.js:150,180,156` | Overlay-Steuerung |
| `SEB_GET_QUIT_REQUIREMENTS`, `SEB_QUIT` | `seb/content.js:236,275` | Beenden-Fluss |
| `OEB_AI_RUN` | `assistant/popup.js:436` | KI-Lauf starten |

**Richtung „→ Content-Script"** (7 Typen, `tabs.sendMessage`): `SEB_SHOW_STARTUP`, `SEB_APPLY_PRESENTATION`, `SEB_UNLOAD_PRESENTATION`, `SEB_RESET_MOODLE_ACCESS`, `SEB_PROBE_DOWNLOAD_URL`, `OEB_AI_EXTRACT`, `OEB_AI_APPLY`. **BESTÄTIGT:** `SEB_*`-Nachrichten werden nur vom obersten Frame verarbeitet (`seb/content.js:11`).

**Seite ↔ Content-Script (`window.postMessage`)**, Kanal `oeb-download-capture-v1`: Typen `settings`, `ready`, `bytes`, `result`. **BESTÄTIGT:** Geprüft wird nur `event.source === window` (`download-page.js:25`, `download-content.js:17`). Das Page-Script sendet beim Laden `ready` (`download-page.js:125`), die Brücke antwortet mit `settings` (`download-content.js:5,18`).
**ABGELEITET:** Da die Seite dieselbe Quelle ist, kann beliebiges Seiten-JavaScript diese Nachrichten senden und der Erweiterung Bytes „unterschieben" (Details TR-04 in 12).

**Content-Script ↔ UI-Frames (`postMessage`)**: `SEB_STARTUP_COMPLETE` (Start-Animation → Content), `SEB_QUIT_REQUEST`, `SEB_RELOAD_REQUEST`, `SEB_AUDIO_TOGGLE` (Taskleiste → Content), `SEB_AUDIO_STATE` (Content → Taskleiste), `SEB_QUIT_SUBMIT`, `SEB_QUIT_CANCEL` (Dialog → Content), `SEB_QUIT_ERROR` (Content → Dialog). **BESTÄTIGT:** Der Content-Script vergleicht die Quelle mit `frame.contentWindow` (`content.js:184, 253-268`); die Frames prüfen `event.source === parent`. **BESTÄTIGT (K2):** Content-Scripts teilen das **DOM** mit der Seite; die von der Erweiterung eingefügten Elemente (iframes, Statusmeldung, Marker) sind für Seiten-JavaScript sichtbar, nur die JS-Objekte der Erweiterung sind getrennt (Standardverhalten, ABGELEITET). Die Frames werden per DOM-ID gefunden (`content.js:251-252`), siehe TR-05.

**Event-Kanäle:** `storage.onChanged` (`seb/popup.js:227-233`, `background.js:37-41`), `tabs.onRemoved` (`background.js:54`), `commands.onCommand`, `webRequest.*`.

**BESTÄTIGT:** Das Manifest enthält weder `externally_connectable` noch Handler für externe Nachrichten; der Router erhält Nachrichten nur von Erweiterungs-Kontexten (ABGELEITET aus dem Standardverhalten der Browser).

### 6.3 Externe Netzwerkverbindungen

| Ziel | Auslöser / Häufigkeit | Credentials | Fundstelle |
|---|---|---|---|
| **Prüfungsserver**: Abruf der Konfiguration | Erkennung eines SEB-Links/Downloads | Cookies (`credentials: "include"`) | `background.js:399-411` |
| Prüfungsserver: zweiter Abruf eines Downloads (Probe) | unbekannte Downloads, wenn Prüfung aktiv | Cookies | `download-check.js:79-84`; `download-content.js:37`; `download-page.js:42` |
| Prüfungsserver: Moodle-Rücksetzung | Entladen | Cookies (`same-origin`) | `seb/content.js:295` |
| Prüfungsserver: normale Seitenanfragen mit veränderten Headern | nach Launch | Browser-Cookies | `headers.js:19-43` |
| **GitHub Raw** (`raw.githubusercontent.com/63kitsune/OEB/refs/heads/main/bek.json`) | siehe 6.4 | `credentials: "omit"`, `redirect: "error"` | `bek-catalog.js:2,100-101` |
| **KI-Anbieter** (8 feste Hosts + frei wählbarer „Custom"-Host) | Nutzeraktion (Lauf, Modellliste) | API-Schlüssel im Header | `assistant/providers.js:1-68,142-183,216-290` |
| **Bild-URLs der Seite** (jeder Host, den die Seite einbettet) | beim Auslesen der Fragen, bis zu 8 Bilder | Cookies (`credentials: 'include'`) | `assistant/extract.js:46-51` |

**BESTÄTIGT:** Keine eigene Telemetrie der Autoren, kein `eval`/`new Function`/dynamisches Nachladen von Code; `innerHTML` wird nur gelesen (`seb/content.js:290`). **NICHT GEPRÜFT:** tatsächlicher Netzwerkverkehr.

### 6.4 Aktualisierung des Profilkatalogs (Catalog-Refresh)

**BESTÄTIGT (Code):**
- Die Netzwerkprüfung läuft nur, wenn seit der letzten Prüfung mindestens 6 h vergangen sind (`CHECK_INTERVAL`, `bek-catalog.js:4,96`). Der Zeitpunkt `checkedAt` wird im Cache `oebBekCatalogCache` gespeichert (`:78,110,114`) und überdauert also einen Browserneustart.
- Auch ein **fehlgeschlagener** Versuch setzt den Zeitpunkt (`:98,114`); der nächste Versuch folgt frühestens nach 6 h.
- Ohne Cache (Erststart) ist `checkedAt` leer, und die Prüfung läuft sofort.
- **Auslöser** für die Prüfung (jeweils mit 6-h-Schwelle): Start der Hintergrundseite und stündlicher Timer (`background.js:27-29`); jedes Laden eines Profils, also jeder Launch und jedes Setzen der BEK-Version (`bek-catalog.js:121-125,138-140`); Öffnen des Popups (`background.js:263-266`). **Erzwungen** (ohne Schwelle): Schaltfläche „Check for updates" (`SEB_REFRESH_BEK_CATALOG`, `bek-catalog.js:137`).
- Anfrage: GET, `Accept: application/json`, `credentials: "omit"`, `redirect: "error"`, `cache: "no-store"`, Timeout 8 s (`:100-101`).
- Ein „Abruf bei jedem Browserstart" ist damit **nicht** garantiert.

**NICHT GEPRÜFT:** tatsächliche Abrufzeitpunkte im Netzwerk.

---

## 7. Daten, Zustände, Grenzen

### 7.1 Persistenter Speicher (`browser.storage`)

| Schlüssel | Bereich | Inhalt (Struktur) | Schreiber → Leser | Löschung / Lebensdauer |
|---|---|---|---|---|
| `sebInspectorSettings` | local | `{checkDownloadsForSeb, closeTabOnQuit, showStartupAnimation, taskbarMode, taskbarScale}` | Hintergrund → Hintergrund, Popup, `download-content.js` | bleibt; Standardwerte bei Installation (`background.js:60-61`) |
| `oebBekSettings` | local | `{enabled, buildVersion, mode, browserExamKey, schemaVersion:2}` | Hintergrund → Hintergrund | bleibt; UI schreibt nur Modus „automatic" |
| `oebBekCatalogCache` | local | `{catalog, checkedAt, error}` | `bek-catalog.js` → `bek-catalog.js`, Popup (Änderungsereignis) | bleibt dauerhaft |
| `sebInspectorState` | **session** (Fallback `local`) | Zustandsobjekt (7.2), enthält bei „ready" das **Klartext-XML** | Hintergrund → Hintergrund, Popup | wird bei Entladen auf „idle" ohne XML überschrieben (`background.js:628`) |
| `sebInspectorPending` | **session** (Fallback `local`) | `{bytes, metadata, mode}`: Rohbytes der **verschlüsselten** Datei und Metadaten (7.5) | Hintergrund | entfernt beim Ersetzen, Entsperren, Fehler, Entladen (`:466,500,526,627`) |
| `oebAiProviders` | local | Liste von KI-Konten (Dienst, Modell, **API-Schlüssel im Klartext**, Basis-URL, aktiviert) | `assistant/popup.js` → `assistant/background.js` | bleibt bis zum Entfernen im Popup („Remove" + „Save", `assistant/popup.js:183,359-372`) |
| `oebAiEnabledDomains` | local | Karte `Hostname → true` | Popup → Assistant-Hintergrund | pro Host per Schalter entfernbar (`:390-394`) |
| `oebAiNote` | local | Freitext-Zusatzanweisung | Popup → Assistant-Hintergrund | bleibt; im Popup editierbar |
| `oebAiLastRun` | local | Ergebnis des **letzten** Laufs (7.5) | Assistant-Hintergrund → Popup | überschrieben beim nächsten Lauf (auch bei Fehler); **keine** UI-Löschfunktion |
| `oebNextPageShortcut` | local | Text des Tastenkürzels | Popup → `assistant/content.js` | bleibt |
| `oebPopupView` | `localStorage` des Popups | zuletzt gewählter Reiter | Popup → Popup | bleibt |

**BESTÄTIGT:** Es gibt keine weiteren Speicherschlüssel (Suche über den gesamten Quellcode). Es gibt keine „Alle Daten löschen"-Funktion.
**ABGELEITET:** `storage.session` ist ein flüchtiger Speicher (leert sich nach dem Browserende). `browser-api.js` ersetzt ihn durch `storage.local`, wenn er fehlt (`:3,28`).
**OFFEN:** Ob `storage.session` in Chromium mit Manifest V2 verfügbar ist, und wie lange es Daten hält (Reload der Erweiterung, Neustart). Fällt es auf `local` zurück, liegen Zustand (inkl. Klartext-XML) und Pending-Datensatz **dauerhaft** im Profil. Auf einem Testgerät zu prüfen (NICHT GEPRÜFT).

### 7.2 Zustandsautomat der Konfiguration (`sebInspectorState`)

```
        ┌────────┐  Erkennung (URL-Weg)  ┌─────────────┐  Datei geladen  ┌───────┐
  Start │  idle  │──────────────────────►│ downloading │────────────────►│ ready │◄──────────┐
        └───▲────┘                       └──────┬──────┘ (Klartext)      └───┬───┘           │
            │ unloadConfiguration               │ Fehler                     │               │ Passwort ok
            │                                   ▼                            ▼               │
            │                               ┌───────┐                  (unload/quit)  ┌──────┴──────────┐
            └───────────────────────────────│ error │◄── Fehler in jedem Schritt      │ password-needed │
                                            └───────┘                                 └─────────────────┘
```

Felder: `status`, `mode` (Dateiformat), `xml` (Klartext, nur bei `ready`), `sourceUrl`, `filename`, `detectedAt`, `message`, `errorCode`, `passwordError`, `cmid`. Das Popup erhält zusätzlich `headerDemo` (Status des Header-Moduls, `background.js:549-551`).

**BESTÄTIGT:** Der Header-Zustand (an/aus) ist **nicht** Teil dieses Automaten, sondern liegt getrennt in den Modulvariablen von `headers.js` (7.4). Beide können auseinanderlaufen (siehe 7.3 und TR-01).

### 7.3 Queue und Nebenläufigkeit

| Pfad | Serialisiert (über `configurationQueue`)? | Fundstelle |
|---|---|---|
| Erkennen → `processBytes` → `replaceConfiguration` (einschließlich des darin enthaltenen Entladens und `activateLaunch`) | **Ja** | `background.js:436-440` |
| `unlockPending` (Nachricht `SEB_UNLOCK`): setzt Zustand und ruft `activateLaunch` | **Nein** | `:493-538` |
| `unloadConfiguration` aus `SEB_UNLOAD`, `SEB_CLEAR`, `SEB_QUIT` | **Nein** | `:315-342`, `:615-637` |
| `SEB_HEADER_ENABLE` / `SEB_HEADER_DISABLE` | **Nein** | `:329-336` |
| `SEB_SET_BEK_SETTINGS` (kann bei geladenem Zustand die Header neu aktivieren) | **Nein** | `:271-281` |
| `runtime.onInstalled`: setzt Zustand „idle" | **Nein** | `:59-63` |

**ABGELEITET:** Gleichzeitiges Entsperren, Entladen, Header-Schalten und Ersetzen kann Zustand und Header-Modul gegeneinander überschreiben. Nicht getestet (TR-02).
**BESTÄTIGT:** `runtime.onInstalled` setzt den Zustand bei **jeder** Installation und jedem Update auf „idle" und verwirft damit eine geladene Konfiguration (TR-10).
**ABGELEITET:** Schlägt in `activateLaunch` ein Schritt nach `enableHeaderInjection` fehl, steht der Zustand auf „error", während das Header-Modul aktiv bleibt (`background.js:572-584`, `:479-490`; TR-01).

### 7.4 Laufzeit-Variablen im Hintergrund (gehen bei Neustart verloren)

| Variable | Typ | Zweck |
|---|---|---|
| `automaticDownloads` | bool | Kopie der Einstellung für synchrone Hooks |
| `settingsReady` | Promise | Hooks warten, bis Einstellungen geladen sind |
| `presentationTabs` | `Set<tabId>` | Tabs mit Prüfungsoberfläche (gemeinsam genutzt von Overlay-, Beenden-, Rücksetzlogik) |
| `pageTransitions` | `Map<tabId, {phase, destination}>` | Übergangszustand „startup" → „reloading" |
| `configurationQueue` | Promise | Serialisierung des Ersetzens |
| `loadedBytesDigest` | string | Fingerabdruck der geladenen Bytes |
| `duplicateRequests`, `inFlightCaptures`, `inFlightProbes`, `automaticCaptures`, `responseCaptures`, `sebResponseRequests` | `Map`/`Set` | Entdoppelung und Merker laufender Anfragen |
| Modulvariablen `headers.js` (Z. 9-17): `enabled`, `configHash`, `browserExamKey`, `buildVersion`, `examOrigin`, `bekMode`, `requestsWithBek`, `matchingTabs`, `lastError` | verschiedene | Zustand des Header-Hooks |
| `catalog`, `source`, `checkedAt`, `lastError`, `initializing`, `updating` | im `createBekCatalogManager`-Closure | Katalogverwaltung |
| `runningTabs` | `Map<tabId, Promise>` | Verhindert parallele KI-Läufe pro Tab |

**ABGELEITET:** Weil die Header-Variablen nur im Arbeitsspeicher stehen, endet die Manipulation bei Browserende oder Reload der Erweiterung. Zeigt `sebInspectorState` dank `local`-Fallback weiter „ready", würde das Popup „Exam loaded" anzeigen, ohne dass Header gesetzt werden. **OFFEN:** nur relevant, wenn der Fallback in einem Browser tatsächlich eintritt.

### 7.5 Datenstrukturen zwischen Komponenten

- **Parse-Ergebnis** (`parser.js` → `background.js`): `{status:"ready", mode, xml}` oder `{status:"password-needed", mode, message}`; Fehler als `SebParseError{code, message}`. Codes (10): `INVALID_DATA_URL`, `TOO_MANY_LAYERS`, `INVALID_GZIP`, `INVALID_ENCRYPTED_DATA`, `UNSUPPORTED_ENCRYPTION`, `INVALID_PASSWORD`, `CERTIFICATE_REQUIRED`, `INVALID_SEB`, `BINARY_PLIST_UNSUPPORTED`, `INVALID_XML`.
- **Pending-Datensatz** (`sebInspectorPending.metadata`): je nach Auslöser `source`, `filename`, `sourceUrl`, `pageUrl`, `tabId`, `autoLaunch` und, wenn die Datei über ein abgefangenes **Formular** kam, zusätzlich `method` und **`body`** (URL-kodierte Formularfelder). **BESTÄTIGT:** `seb/content.js:57-76` → `background.js:217-225,415-419` → `:451-453`. Nur der Pending-Datensatz enthält den `body`; der Zustand `sebInspectorState` nicht (`:467-475`).
- **Darstellungs-Einstellungen**: Nutzereinstellungen + `config.taskbar` (`visible, height, showClock, showKeyboard, showNetwork, showAudio, showReload, reloadWarning, showQuit`), abgeleitet aus der Konfiguration (`config-key.js:51-67`).
- **Frage-Liste** (`extract.js` → Hintergrund → `providers.js`): Objekte mit `number`, `type` und typabhängigen Feldern (Text, Optionen, Lücken, optionale Bilder mit `id`, `alt`, `src`, Größe und `dataUrl`).
- **Antwort-Liste** (`providers.js` → `apply.js`): `[{number, values:[…]}]`; Rückgabe von `apply.js`: `[{number, status:"filled"|"skipped", reason?}]`.
- **KI-Konto** (`providers.js`): `id`, `provider`, `enabled`, `label`, `apiKey`, `model`, `baseUrl` (`normalizeProviderAccounts`, `:98-109`).
- **Letzter KI-Lauf** (`oebAiLastRun`): `tabId`, `url`, `questions` (ohne `dataUrl`, aber **mit** Bild-`src`), `answers`, `results`, `provider`, `attempts`, `message`, `time`; bei Fehler `error`, `attempts`, `message` ohne `questions` (`assistant/background.js:76-92,101-105`).

### 7.6 Grenzen und Zeitlimits – wo gilt was

**Größengrenzen**

| Pfad | Grenze | Fundstelle |
|---|---|---|
| Bytes vom Seiten-Skript (Blob/Link) | 8 MB | `download-page.js:4,33,54,64`; `download-content.js:22`; `background.js:231` |
| Probe-Abruf im Content-Script / Hintergrund | 8 MB | `download-content.js:47`; `background.js:163`; `download-check.js:3,83` |
| Entpackte Daten bei der **Erkennung** (`containsSebConfiguration`) | 8 MB je Schicht, max. 4 Schichten | `download-check.js:69-76` |
| Firefox-Spiegelung | 8 MB (Standardwert) | `download-stream.js:3` |
| **Keine Grenze**: Datei aus dem Import-Tab (`SEB_PROCESS_BYTES`) | – | `file-import.js`, `background.js:246-252` |
| **Keine Grenze**: Abruf der Konfiguration in `processUrlOnce` (`response.arrayBuffer()`) | – | `background.js:411-414` |
| **Keine Grenze**: Dekompression im Parser (nur max. 4 Gzip-Schichten) | – | `parser.js:107-125` |
| Katalog-JSON / Profil-Prefix | 4 MB / 512 KB | `bek-catalog.js:5,26` |
| KI-Bilder | max. 8 Bilder, 4 MB je Bild, 16 MB gesamt | `extract.js:4-6` |
| Taskleistengröße | 50–300 % | `settings.js` |

Die „8-MB-Grenze" gilt somit nur für die **automatischen Erkennungswege**, nicht für jeden Weg, auf dem eine Konfiguration geladen werden kann (ABGELEITET: Speicherbedarf bei manipulierten Dateien, TR-03).

**Zeitlimits:** 30 s Konfigurations-Download (`background.js:404`); 3 s Probe im Hintergrund (`download-check.js:79`); 5 s Probe im Content-Script (`download-content.js:37`, `download-page.js:42`); 15 s Wartezeit auf das Ergebnis der Brücke (`download-page.js:69`); 45 s KI-Anfrage (`providers.js:419`); 8 s Katalog-Abruf (`bek-catalog.js:101`); 6 h Katalog-Intervall; 1 h Hintergrund-Timer (`background.js:28`); 350 ms zwischen simulierten Tasten beim Drag-&-Drop-Eintrag (`apply.js:177`).

### 7.7 Versionsangaben und angezeigte SEB-Builds

| Angabe | Wert | Fundstelle | Wirkung |
|---|---|---|---|
| Paketversion der Erweiterung | `2.4.0` | `package.json:3`; Manifeste Z. 5; `README.md:5` | Add-on-Version |
| ZIP-Bezeichnung | `OEB-2.4.1` | Archivname | Abweichung unerklärt (OFFEN) |
| Standard-Profil / Build für die lokale Schlüsselberechnung (vierteilig) | `3.10.2.920` | `browser-exam-key.js:6`; `bek.json` (`defaultVersion`) | Nur lokale Berechnung; erscheint **nicht** im Netzwerk |
| `SEB/…`-Token im `User-Agent` (**dreiteilig**: erste drei Teile der **gewählten** Profilversion) | Standard `SEB/3.10.2 (x64)` | `headers.js:27,64` | Netzwerkmerkmal; folgt der Profilauswahl |
| Von der Start-Animation angezeigter Build | `3.10.1.864` / Version `3.10.1 (x64)` | `seb/assets/startup.js:1-2`; zusätzlich `startup.html:243` (inerter Block) | Nur Optik; **widerspricht** dem Standard-Profil (BF-01) |

**BESTÄTIGT:** Die vierteilige Build-Nummer geht nie in eine Netzwerkanfrage ein; sichtbar ist nur die dreiteilige Version im User-Agent.

---

## 8. Abhängigkeiten zwischen Modulen

### 8.1 Import-Graph (nur `import`-Beziehungen)

```
background.html
 └─ seb/background.js
     ├─ seb/parser.js                      (keine Abhängigkeit)
     ├─ seb/headers.js
     │    ├─ seb/config-key.js             (keine Abhängigkeit)
     │    └─ seb/browser-exam-key.js
     │         ├─ seb/config-key.js
     │         └─ seb/bek-catalog.js       (keine Abhängigkeit)
     ├─ seb/config-key.js
     ├─ seb/browser-exam-key.js
     ├─ seb/bek-catalog.js
     ├─ seb/launch-response.js             (keine Abhängigkeit)
     ├─ seb/settings.js                    (keine Abhängigkeit)
     ├─ seb/download-check.js              (keine Abhängigkeit)
     ├─ seb/download-stream.js             (keine Abhängigkeit)
     └─ assistant/background.js
          └─ assistant/providers.js        (keine Abhängigkeit)

popup.html
 ├─ seb/popup.js ── config-key.js, file-import.js, bek-selection.js
 └─ assistant/popup.js ── providers.js

build.mjs ── seb/bek-catalog.js (validateBekCatalog)
```
**BESTÄTIGT:** Keine Zirkelbezüge; `seb/background.js` hat 10 Importe.

### 8.2 Abhängigkeiten ohne `import` (Laufzeit-Kopplung)

| Kopplung | Art | Auswirkung einer Änderung |
|---|---|---|
| Nachrichtentypen (Strings) zwischen Popup/Content/Hintergrund | implizit, nur durch Namensgleichheit | Umbenennen bricht still (kein Compilerfehler) |
| `OEBAssistant`-Globalobjekt | Reihenfolge der Content-Script-Liste im Manifest | `extract.js`, `apply.js` müssen **vor** `assistant/content.js` geladen werden |
| Overlay-IDs `seb-local-*-frame` | mehrfach definiert: `seb/content.js:3-5`, `content.css`, **und** in der Toleranzregel des Plist-Parsers `config-key.js:95-106` | Änderung muss alle Stellen treffen |
| `popup.html` Element-IDs ↔ Popup-Skripte | `querySelector("#…")` | Umbenennen bricht die Oberfläche |
| Storage-Schlüssel | als Textkonstanten an mehreren Orten (z. B. `sebInspectorSettings` auch in `download-content.js:11-12`; `oebAi*` in Hintergrund **und** Popup) | Jede Änderung muss alle Orte treffen |
| `web_accessible_resources` ↔ iframe-Ziele | Manifest | Fehlt ein Eintrag, laden Overlay-Frames nicht |
| Firefox-MAIN-Skript ↔ Chromium-Script-Tag | zwei Wege zum selben Ziel (`firefox.manifest.json:55-65` vs. `download-content.js:57-77`) | Beide pflegen |
| `bek.json` ↔ `bek-catalog.js` ↔ `browser-exam-key.js` ↔ Build | Daten + Schema + Verbraucher | Schemaänderung betrifft alle |
| `build.mjs` ↔ Dateistruktur | Build kopiert den gesamten `source/`-Baum | Neue Dateien automatisch aufgenommen; Manifest-Listen manuell |

### 8.3 Unabhängige Bereiche

- **Assistant-Block** ↔ **SEB-Block**: keine Importe, kein gemeinsamer Zustand; Berührungspunkte sind `seb/background.js:15,52,214-215` und das gemeinsame Popup.
- **Download-Erkennung** (`download-*`, `launch-response`) ↔ **Header-Modul**: unabhängig; verbunden nur über `processBytes`/Zustand.
- **Overlay-Oberfläche** ↔ **Header-/Schlüsselmodul**: unabhängig.
- **`parser.js`**: keine Erweiterungs-APIs (nutzt Web-Plattform-APIs, 2.2).

### 8.4 Bereiche mit den meisten Abhängigkeiten

1. `seb/background.js` (Hub; 10 Importe, 20 Nachrichtentypen, die meisten Zustandsvariablen).
2. `seb/config-key.js` (von vier Modulen genutzt; enthält Parser **und** Hashing).
3. `popup.html` (Struktur für beide Popup-Module).

---

## 9. Initialisierung und Lebenszyklen

### 9.1 Start der Erweiterung (Hintergrund)

Reihenfolge in `seb/background.js`:
1. `browser-api.js` läuft zuerst.
2. Module werden importiert; **beim Import** registriert `headers.js` bereits seinen `webRequest`-Hook (`:19`), anfänglich inaktiv (`enabled = false`).
3. Katalog: `getBekCatalogInfo()` sofort und alle 60 Minuten (`:27-29`); die Netzwerkprüfung unterliegt der 6-h-Schwelle (6.4).
4. `settingsReady` lädt Einstellungen asynchron (`:36`); `storage.onChanged` hält `automaticDownloads` aktuell (`:37-41`).
5. `installAssistantCommands()` (`:52`).
6. Weitere Listener: `tabs.onRemoved`, `runtime.onInstalled`, `runtime.onMessage`, `downloads.onCreated`, `webRequest.onHeadersReceived`, `onCompleted`/`onErrorOccurred`.
7. Bei Installation/Update: `onInstalled` schreibt Standard-Einstellungen (falls fehlend) und setzt den Zustand „idle".

### 9.2 Laden einer Seite (Content-Scripts)

Pro Seite und Frame bei `document_start`: Registrierung der Nachrichten-Listener, Klick-/Submit-Abfänger, Download-Brücke, Tastenkürzel-Listener (nur oberster Frame). Der oberste Frame fragt den Hintergrund via `SEB_GET_PAGE_PRESENTATION`, ob hier die Prüfungsoberfläche erscheinen soll (`content.js:313-316`).
**BESTÄTIGT:** Reiter, die vor dem Installieren/Neuladen der Erweiterung geöffnet wurden, haben keine Content-Scripts; Fehler wie „Receiving end does not exist" werden im Popup in „Reload this tab" übersetzt (`assistant/popup.js:546-550`). Das README rät ebenfalls zum Neuladen nach Updates.

### 9.3 Lebensdauer von Komponenten

| Komponente | Entsteht | Endet |
|---|---|---|
| Background Page | Browserstart / Erweiterung geladen | Browserende / Erweiterung entfernt; alle Laufzeitvariablen (7.4) gehen verloren |
| Header-Manipulation | `enableHeaderInjection` (Launch) | `disableHeaderInjection` (Entladen) oder Verlust der Background Page |
| Konfigurationszustand | `replaceConfiguration` | `unloadConfiguration`; ggf. Browserende (nur bei echtem `session`-Speicher, OFFEN) |
| Overlay-Frames | Launch / Seiten-Neuladen im Prüfungs-Tab | Entladen, Tab-Schließen, `tabs.onRemoved` |
| Popup | Klick aufs Symbol | Schließen; **ein laufender KI-Lauf läuft im Hintergrund weiter** (Hinweistext `assistant/popup.js:433`) |
| Katalog-Cache | erster erfolgreicher Abruf | bleibt dauerhaft in `storage.local` |

### 9.4 Totcode und Unstimmigkeiten

| Fund | Fundstelle | Status |
|---|---|---|
| Ungenutzte Funktion `calculateFallbackBrowserExamKey` und Konstante `CERTIFICATE_THUMBPRINT` (nur dort verwendet) | `seb/browser-exam-key.js:5,49-59` | BESTÄTIGT |
| Ungenutzte Funktionen `calculateRequestConfigKey` (async), `serializeSebConfiguration` | `seb/config-key.js:13,76` | BESTÄTIGT |
| Nachricht `SEB_CLEAR` ohne Absender | `seb/background.js:340` | BESTÄTIGT |
| Popup sendet bei `SEB_HEADER_ENABLE` einen Parameter `configHash`, den der Hintergrund ignoriert | `seb/popup.js:179-180` vs. `background.js:329-334` | BESTÄTIGT |
| Bezeichnung „headerDemo" in Code und Oberfläche, obwohl es der echte Betriebsmodus ist | `background.js:549`, `popup.html:113-117` | BESTÄTIGT |
| Modus „provided" (manueller BEK): im Hintergrund implementiert, per Oberfläche nicht erreichbar, gespeicherte Werte werden migriert | `browser-exam-key.js:9-30`; `popup.html:67-77`; `seb/popup.js:74-79` | BESTÄTIGT |
| Start-Animation zeigt anderen Build als das Standardprofil (zwei Stellen) | siehe 7.7 | BESTÄTIGT |
| Paketversion 2.4.0 vs. ZIP-Bezeichnung 2.4.1 | siehe 7.7 | BESTÄTIGT |
| `browser-api.js` definiert `windows`, im Code ungenutzt | `browser-api.js:62-66` | BESTÄTIGT |

---

## 10. Fehlerbehandlung, Fallbacks, Einschränkungen

### 10.1 Muster

| Muster | Beispiele |
|---|---|
| **Fehler werden in Zustand übersetzt, nicht geworfen.** Die meisten enden in `saveState({status:"error", message, errorCode})` plus rotem Badge `!`. | `replaceConfiguration:479-490`, `processUrlOnce:420-431` |
| **Nachrichten-Router fängt alles:** Fehler in `handleMessage` werden als `{ok:false, error}` zurückgegeben. | `background.js:65-70` |
| **„Fail-open" bei Erkennung:** Schlägt die Prüfung eines unbekannten Downloads fehl, bleibt der Download unberührt. Leere Fehlerblöcke (`catch {}` / `.catch(() => {})`) kommen an etwa 28 Stellen vor (eigene Zählung per Muster, ±1), überwiegend bei optionalen Schritten. | `download-check.js:1-2`; `background.js:127,167` u. a. |
| **Fallback-Ketten** | Firefox: Spiegelung → bei Fehler Probe-Abruf (`:111-128`). Probe: Abruf über den Tab mit Seiten-Cookies → bei Misserfolg Hintergrund-`fetch` (`:153-170`). Overlay: Start-Animation → bei fehlendem Content-Script direktes Neuladen (`:593-605`). Katalog: Cache → Paket-Datei (`bek-catalog.js:71-86`). KI: Konten der Reihe nach (`providers.js:194-214`). |
| **Zeit- und Größenbegrenzungen** | Auflistung in 7.6 (nicht auf allen Pfaden) |
| **Validierung vor Übernahme:** Katalog nur nach Schema- und Prüfsummenkontrolle; ungültige Updates werden verworfen (`bek-catalog.js:100-115`). Die Prüfsumme stammt aus derselben Datei (siehe TR-07). | – |
| **Header-Hook schluckt Fehler:** im Fehlerfall wird die Anfrage unverändert weitergeleitet, `lastError` gesetzt. | `headers.js:34-37` |
| **Vorher/Nachher-Prüfung beim KI-Lauf:** Abbruch, wenn sich URL oder Fragen geändert haben. | `assistant/background.js:58-65` |

### 10.2 Bekannte technische Einschränkungen

1. **Manifest V2 + blockierendes `webRequest`** ist Voraussetzung (README). **OFFEN:** Ob aktuelle Chromium-Versionen MV2 noch unterstützen, wurde nicht geprüft (NICHT GEPRÜFT).
2. Zertifikat-verschlüsselte `.seb`-Dateien (Präfixe `pkhs`/`phsk`) und Apple-Binär-Plists werden nicht unterstützt (`parser.js:69-74,92-96`).
3. Nur der Profiltyp `windows-x64` und genau **ein** Algorithmus werden vom Katalog akzeptiert (`bek-catalog.js:16`).
4. **BEK-Eingabe:** Das Popup bietet nur „Off" oder die Auswahl einer Profilversion (`popup.html:67-77`); der Modus „provided" (eigener 64-stelliger Hex-Schlüssel) ist im Hintergrund implementiert, aber nicht über die Oberfläche erreichbar, und ein dort gespeicherter Wert wird beim Öffnen des Popups auf den Standard zurückgesetzt (`seb/popup.js:74-79`). **OFFEN:** Ob der Pfad auf anderem Weg erreichbar ist (z. B. über Entwicklerwerkzeuge der Erweiterung).
5. **Keine** Betriebssystem-Sperre (README). Taskleiste und Dialog sind reine Web-Optik; Reload/Audio/Beenden wirken nur auf die Seite bzw. die Erweiterung (`content.js:250-283`).
6. Die Header-Änderung gilt nur für **einen** Ursprung zur Zeit und nur nach geladener Konfiguration (`headers.js:21`).
7. Größen- und Zeitgrenzen gelten pfadabhängig, siehe 7.6.
8. Katalogänderungen wirken nicht rückwirkend auf die aktive Prüfung (`bek-catalog.js:103-104`; README).
9. Der Plist-Parser ist handgeschrieben (`config-key.js:83-259`); unbekannte Elemente werfen einen Fehler (`:170-171`). **OFFEN:** Robustheit gegen reale Konfigurationen.
10. Download-Erkennung ist browserabhängig: Spiegelung nur Firefox; Chromium nutzt einen Probe-Abruf (zweiter Abruf; kann bei einmal gültigen Download-Links scheitern, README „Some downloads may need manual import").
11. Content-Scripts müssen nach Installation/Update per Seiten-Neuladen aktiviert werden.
12. KI-Lauf und „Nächste Seite" arbeiten nur im obersten Frame (`assistant/content.js:2`).

### 10.3 Mögliche Fehlerquellen (aus Betreibersicht)

- Zustand „ready" im Popup bedeutet nicht zwingend, dass der Header-Hook aktiv ist (7.3, 7.4).
- Mehrere Prüfungs-Tabs: `presentationTabs` wird bei jedem Launch geleert und nur der Zieltab gesetzt (`background.js:574-575`); gleichzeitige Prüfungen werden nicht unterstützt (ABGELEITET).
- **ABGELEITET (DF-12):** Der Header-Hook filtert nur nach Ursprung, nicht nach Tab oder Initiator (`headers.js:19-21`). Wird eine weitere Konfiguration für denselben Ursprung geladen, solange die vorige noch aktiv ist, kann auch der Hintergrund-`fetch` (`background.js:411`) die veränderten Header tragen; das Entladen geschieht erst danach (`:448`). Nicht getestet.
- Pflege-Risiko durch implizite Kopplungen (8.2).

---

## 11. Datenschutz und Sicherheit

**Einordnung.** Dieser Abschnitt trennt je Befund **nachgewiesene technische Fakten** (BESTÄTIGT, aus dem Quellcode), **mögliche Auswirkungen** (ABGELEITET) und **offene Fragen** (OFFEN / NICHT GEPRÜFT). Er enthält keine rechtliche Bewertung. Es wurde kein Laufzeitverhalten und kein Netzwerkverkehr geprüft; Aufbewahrung und Verarbeitung bei externen Anbietern liegen außerhalb des Codes und sind **NICHT GEPRÜFT**. Die IDs verweisen auf `verification-report.md`.

### 11.1 Datenbestände im Überblick

| Datenart | Wo | Sensibel, weil | Details |
|---|---|---|---|
| API-Schlüssel der KI-Dienste | `storage.local` (`oebAiProviders`) | Zugangsdaten, Kostenrisiko | 11.2 |
| Formularinhalte (POST/GET) abgefangener SEB-Formulare | Pending-Datensatz, ggf. URL im Zustand | können Sitzungsdaten enthalten | 11.3 |
| Fragen, Antworten, Bild-URLs des letzten KI-Laufs | `storage.local` (`oebAiLastRun`) | Prüfungsinhalte | 11.4 |
| Fragen/Bilder/Notiz beim Versand | KI-Anbieter | Prüfungsinhalte, ggf. personenbezogene Daten | 11.5 |
| Abgerufene Bilder | Hintergrundspeicher der Seite, Netzwerk | Cookies an Bildhosts | 11.6 |
| Klartext-Konfiguration | `sebInspectorState`, Popup | enthält u. a. Hash des Beenden-Passworts | 11.7 |

### 11.2 DS-01 – Speicherung von API-Schlüsseln

| | |
|---|---|
| **BESTÄTIGT** | Schlüssel liegen im Klartext in `storage.local` unter `oebAiProviders` (`assistant/popup.js:371`; `providers.js:98-109`). Eingabefeld `type=password`, `autocomplete=off` (`assistant/popup.js:217-218`). Im Code wurde keine Verschlüsselung der Schlüssel gefunden. Sie werden bei jeder Anfrage übertragen: Header `x-goog-api-key` (Gemini), `Authorization: Bearer` (OpenAI-kompatible und OpenAI), `x-api-key` (Anthropic) (`providers.js:151,235,404,411`), auch bei Modelllisten-Abrufen (`:142-183`). Beim Typ „Custom" ist die Basis-URL frei wählbar; unverschlüsseltes HTTP ist nur für `localhost`, `127.0.0.1`, `[::1]` erlaubt (`:385-400`). Einzelne Konten lassen sich per „Remove" + „Save" entfernen (`assistant/popup.js:183,359-372`). Es gibt keine Export- und keine „Alles löschen"-Funktion. README: „API keys stay in local browser storage". |
| **ABGELEITET** | Wer Lesezugriff auf das Browserprofil bzw. den Erweiterungsspeicher hat, kann die Schlüssel lesen. Schlüssel bleiben über Sitzungen hinweg erhalten. Ein Custom-Endpunkt erhält den Schlüssel im Header. Ein abhandengekommener Schlüssel kann Kosten und Datenzugriff beim Anbieter verursachen. |
| **OFFEN / NICHT GEPRÜFT** | Speicherformat und -ort der Erweiterungsdaten im Profil je Browser (NICHT GEPRÜFT). Ob Reste nach dem Entfernen im Profil verbleiben (OFFEN). Ob `storage.local` im jeweiligen Browser synchronisiert wird (OFFEN; der Code verwendet keinen Sync-Speicher). |

### 11.3 DS-02 – Speicherung von POST-Daten und Formularinhalten

| | |
|---|---|
| **BESTÄTIGT** | Ein abgefangenes Formular mit SEB-Ziel wird vollständig ausgelesen (`FormData`); Dateiuploads werden ausgenommen (`seb/content.js:49-62`). Bei **POST** wird der URL-kodierte `body` mit `method` an den Hintergrund gesendet und beim Konfigurations-Abruf mitgesendet (`content.js:68-69,76`; `background.js:216-225,399-411`). Bei **GET** werden die Felder an die URL angehängt (`content.js:66-67`) und landen in `sourceUrl`, damit im Zustand und im Popup (`seb/popup.js:303-304`). Der `body` wird nur dann gespeichert, wenn die Datei passwortgeschützt ist: er liegt dann im Pending-Datensatz (`background.js:451-453`). Der Zustand `sebInspectorState` enthält den `body` nicht (`:467-475`). Das **Dateipasswort** wird nicht gespeichert (`seb/popup.js:122-124`; README). |
| **ABGELEITET** | Ob Formularfelder reale Sitzungs- oder Sicherheitswerte enthalten, hängt von der Seite ab (bei Moodle-Formularen möglich). Die URL-Parameter bei GET können in der Popup-Anzeige sichtbar sein. |
| **OFFEN** | Welche Felder reale Start-Formulare enthalten (NICHT GEPRÜFT). Wie lange der Pending-Datensatz im Browser verbleibt, hängt von `storage.session` ab (7.1, OFFEN). |

### 11.4 Persistierte KI-Anfragen und deren Inhalte (zu DS-03)

| | |
|---|---|
| **BESTÄTIGT** | Nach jedem Lauf – auch nach einem Fehler – wird `oebAiLastRun` geschrieben (`assistant/background.js:76-92,101-105`): URL, Tab, Fragen (Text, Optionen; `dataUrl` entfernt, aber Bild-`src` bleibt, `:9-15,78`; `extract.js:64`), Antworten, Ergebnis je Frage, Anbietername, Fehlerliste, Meldung, Zeit. Der Datensatz wird nur beim nächsten Lauf überschrieben; das Popup zeigt ihn für denselben Tab und dieselbe URL erneut an (`assistant/popup.js:579-587`). Es gibt keine Löschfunktion im UI. Zusätzlich bleiben `oebAiNote` (Freitext) und `oebAiEnabledDomains` (freigeschaltete Hosts) gespeichert. |
| **ABGELEITET** | Prüfungsinhalte (Fragetexte, gegebene KI-Antworten) bleiben nach der Prüfung lokal im Profil; das ist eine **forensisch auswertbare Spur** (E-05). Die Domain-Liste zeigt, auf welchen Hosts der Assistent freigeschaltet wurde. |
| **OFFEN** | Verbleib nach Deinstallation oder Profilwechsel (NICHT GEPRÜFT). |

### 11.5 Übertragung von Daten an externe KI-Anbieter (zu DS-04)

| | |
|---|---|
| **BESTÄTIGT** | Pro Lauf wird an den gewählten Anbieter gesendet: ein JSON mit den Fragen (Typ, Text, Optionen, Lückenstruktur, Bild-Metadaten `alt`/Größe), die Bilder als Base64 (bis 8, 4 MB je Bild, 16 MB gesamt) und eine optionale Nutzernotiz im System-Prompt (`providers.js:216-222,292-303`). Fragenummer und Bild-`src` werden vor dem Versand entfernt (`:294-299`). Schlägt ein Anbieter fehl, wird derselbe Inhalt an das **nächste aktivierte Konto** gesendet (`:194-214`); Daten können so nacheinander an mehrere Anbieter gehen. Bei OpenAI wird `store: false` gesetzt (`:252`); für die übrigen Anbieter gibt es keine entsprechende Steuerung im Code. Der Code fügt keine Nutzer- oder Sitzungskennungen hinzu. Zwei Anbieter erhalten feste, werkzeugbezogene Header: `X-OpenRouter-Title` mit dem Erweiterungsnamen (`:22,404-406`) und für Anthropic `anthropic-dangerous-direct-browser-access: true` (`:413`). |
| **ABGELEITET** | Fragetexte und Bilder können personenbezogene oder vertrauliche Inhalte enthalten (Fallstudien, Namen, Prüfungsmaterial). Die Weitergabe kann je nach Organisation ein Vertraulichkeits- bzw. Datenschutzereignis sein. |
| **OFFEN / NICHT GEPRÜFT** | Aufbewahrung, Protokollierung und Nutzung der Daten bei den Anbietern (NICHT GEPRÜFT; anbieterspezifisch, nicht im Code festgelegt). Rechtsgrundlage und Vertragslage (NICHT GEPRÜFT). Tatsächlicher Netzwerkverkehr (NICHT GEPRÜFT). |

### 11.6 Umgang mit Bildquellen und externen Bild-URLs (zu DS-05)

| | |
|---|---|
| **BESTÄTIGT** | Beim Auslesen ruft der Content-Script jedes Bild der Frage selbst per `fetch(src, {credentials:'include'})` ab (`extract.js:46-51`); `src` ist `currentSrc` bzw. `src` des Bildes. Eine Beschränkung auf den Prüfungsserver gibt es im Code nicht; ausgeschlossen werden nur bestimmte Theme-Symbole (`:49`). Geprüft werden Typ `image/*` und Größe (`:55-56`). Bei Fehlern wird das Bild mit `src` und Fehlermeldung vermerkt (`:70`). Die Base64-Bilddaten gehen an den KI-Anbieter (11.5), die `src`-URLs verbleiben im lokalen `oebAiLastRun` (11.4). |
| **ABGELEITET** | Einbettet die Seite Bilder von Fremdhosts, entstehen zusätzliche Abrufe an diese Hosts; Cookies werden gemäß den Cookie-Regeln des Browsers mitgesendet. Der Abruf per `fetch` ist von einem normalen Bildladen unterscheidbar, soweit Browser entsprechende Anfrage-Metadaten setzen (nicht geprüft). |
| **OFFEN** | Welche Cookies/Header tatsächlich übertragen werden (NICHT GEPRÜFT, Netzwerktest nötig). |

### 11.7 Speicherung und Darstellung ausgelesener Konfigurationsdaten (zu DS-06)

| | |
|---|---|
| **BESTÄTIGT** | Die entschlüsselte Konfiguration liegt als Klartext-XML in `sebInspectorState.xml` (`background.js:467-475`). Das Popup zeigt sie formatiert an und bietet **Kopieren** (Zwischenablage) und **Speichern** als `.xml`-Datei (`seb/popup.js:192-206,310`). Das XML enthält u. a. Start-URL, Quit-Einstellungen und den Hash des Beenden-Passworts, falls gesetzt (`config-key.js:32-49`). Das eingegebene Dateipasswort wird nicht gespeichert. Die verschlüsselten Rohbytes liegen nur im Pending-Datensatz (7.1). |
| **ABGELEITET** | Eine Prüfungskonfiguration (u. U. mit Passwort-Hash und Administrations-/Start-URL) liegt unverschlüsselt im Erweiterungsspeicher und ist im Popup lesbar; exportierte `.xml`-Dateien liegen außerhalb der Kontrolle der Erweiterung. |
| **OFFEN** | Welche Felder reale Konfigurationen enthalten (NICHT GEPRÜFT). Dauerhaftigkeit je Browser (7.1, OFFEN). |

### 11.8 Berechtigungen der Erweiterung (zu DS-07)

| | |
|---|---|
| **BESTÄTIGT** | `downloads`, `storage`, `tabs`, `webRequest`, `webRequestBlocking`, `<all_urls>`; Content-Scripts in **allen Frames** aller Seiten ab `document_start`; Firefox deklariert `websiteContent` und `authenticationInfo` als erforderlich (`firefox.manifest.json:85-90`). Genutzt wird von `downloads` nur `onCreated`. Die Hooks `onBeforeSendHeaders` und `onHeadersReceived` sind blockierend und für alle URLs registriert; die Einschränkung auf den Prüfungs-Ursprung erfolgt erst im Code (`headers.js:21`). `onHeadersReceived` bewertet jede Antwort und kann geeignete Downloads erneut abrufen (`background.js:95-107,153-170`). Klick- und Submit-Abfänger laufen auf jeder Seite (`seb/content.js:34-77`); Formulardaten werden erst gelesen, wenn das Ziel eine SEB-URL ist. Auslesen und Eintragen von Fragen erfolgt nur auf Anforderung (Nachrichten `OEB_AI_EXTRACT`/`OEB_AI_APPLY`). Zwölf Ressourcen (Chromium: dreizehn) sind als `web_accessible_resources` freigegeben. Es gibt keine `externally_connectable`-Deklaration. |
| **ABGELEITET** | Die Erweiterung hat technisch die Möglichkeit, Anfrage-Header und Seiteninhalte aller besuchten Webseiten zu lesen bzw. zu verändern; der Code beschränkt sich nach Lektüre auf die oben genannten Fälle. Die Fremdinstallation (temporär oder entpackt) erfordert Browser-Entwicklerfunktionen (README). |
| **OFFEN** | Ob das Laufzeitverhalten dem Code entspricht (NICHT GEPRÜFT). Auswirkung der freigegebenen Ressourcen auf die Sichtbarkeit für Webseiten (OFFEN). |

### 11.9 Lokale forensische Spuren und gespeicherte Zustände (E-05)

| | |
|---|---|
| **BESTÄTIGT** | Im Erweiterungsspeicher liegen dauerhaft: Einstellungen, BEK-Einstellungen, Katalog-Cache mit Zeitstempel und letztem Fehler, KI-Konten (mit Schlüsseln), freigeschaltete Domains, Notiz, Kürzel, **letzter KI-Lauf** (11.4). Je nach Browser in `session` oder `local`: Zustand mit Klartext-XML und Pending-Datensatz. Im Popup-`localStorage`: Ansichtsmerker. Eine vom Nutzer exportierte `.xml`-Datei (11.7) und die Zwischenablage liegen außerhalb der Erweiterung. |
| **ABGELEITET** | Die Erweiterungs-ID (Firefox: `oeb@open-exam-browser.local`) und die Speicherdaten sind bei Zugriff auf das Profil auswertbar; bei temporärer/entpackter Installation tauchen sie ggf. nicht in zentralen Store-Listen auf (siehe `OEB-Analyse-IT.md` 4.2). |
| **NICHT GEPRÜFT** | Konkrete Speicherorte und Dateiformate je Browser und Betriebssystem. Verhalten bei Deinstallation/Profilwechsel. |

### 11.10 Zusammenfassung der Datenschutz-/Sicherheitsbefunde

| ID | Thema | Status des Kernbefunds |
|---|---|---|
| DS-01 | API-Schlüssel im Klartext, an Anbieter und frei wählbaren Host übertragen | BESTÄTIGT |
| DS-02 | Formularinhalte (POST-`body`, GET-Parameter) werden verarbeitet und teilweise gespeichert | BESTÄTIGT (Inhalte der Felder OFFEN) |
| DS-03 | Letzter KI-Lauf (Fragen, Antworten, Bild-URLs) dauerhaft gespeichert | BESTÄTIGT |
| DS-04 | Fragen, Bilder, Notiz an externe KI-Anbieter, ggf. an mehrere nacheinander | BESTÄTIGT (Aufbewahrung NICHT GEPRÜFT) |
| DS-05 | Bildabrufe mit Cookies von beliebigen Hosts | BESTÄTIGT (Netzwerkwirkung NICHT GEPRÜFT) |
| DS-06 | Klartext-Konfiguration im Speicher, Anzeige/Export im Popup | BESTÄTIGT |
| DS-07 | Breites Berechtigungsprofil | BESTÄTIGT |
| E-05 | Lokale Spuren | BESTÄTIGT (Speicherorte NICHT GEPRÜFT) |

---

## 12. Technische Risiken der Erweiterung

| ID | Risiko | Fundstelle | Status |
|---|---|---|---|
| TR-01 | **Teilfehler in `activateLaunch`**: Nach `enableHeaderInjection` fehlschlagende Schritte setzen den Zustand auf „error", deaktivieren aber den Header-Hook nicht. | `background.js:572-584,479-490` | ABGELEITET |
| TR-02 | **Nicht serialisierte Zustandsänderungen** (Entsperren, Entladen, Header-Schalter) können konkurrierend den Zustand überschreiben. | `background.js:493-538,615-637,257-258,315-342` | ABGELEITET |
| TR-03 | **Unbegrenzte Dekompression** (4 Gzip-Schichten ohne Größenlimit) auf den Pfaden ohne Vorprüfung; kann den dauerhaft laufenden Hintergrundprozess speicherseitig belasten. | `parser.js:107-125`; Pfade in 7.6 | ABGELEITET |
| TR-04 | **Seiten-JavaScript kann die Konfigurationsübernahme auslösen.** Kettenglieder BESTÄTIGT: Brücke prüft nur `event.source === window` (`download-content.js:17`) → `SEB_CHECK_DOWNLOAD_BYTES` → Inhaltsprüfung → `captureAutomaticBytes` → `processBytes(autoLaunch: true)` → `activateLaunch` mit dem sendenden Tab als Ziel (`background.js:229-245,188-206`). Die automatische Erkennung ist standardmäßig an (`settings.js:2-4`). Die Herkunft der Bytes wird nicht geprüft, der Inhalt schon. | siehe links | ABGELEITET (Wirkung nicht getestet) |
| TR-05 | **Frame-Zuordnung per DOM-ID**: Taskleisten- und Beenden-Frame werden per `getElementById` gefunden. Ob eine Seite damit Einfluss nehmen kann (z. B. Entladen auslösen), ist nicht getestet. | `seb/content.js:251-253,268-281` | ABGELEITET, gering |
| TR-06 | **Fehlerpfad kann selbst werfen**: `setBadge` im `catch` von `runAssistant` (z. B. Tab inzwischen geschlossen) bricht vor `saveRun` ab; ein Fehlerlauf wird dann nicht gespeichert. | `assistant/background.js:85-87,96-99` | ABGELEITET |
| TR-07 | **Fernaktualisierung ohne Authentizitätsprüfung**: Die Prüfsumme im Katalog stammt aus derselben Datei; Quelle ist der `main`-Branch eines Drittrepositories. Kein Signaturverfahren, kein Pinning. | `bek-catalog.js:2,27-29,100-111` | Mechanismus BESTÄTIGT, Folgen ABGELEITET |
| TR-08 | **Keine Tests, keine CI, keine Lizenz, kein Changelog.** | Dateiliste | BESTÄTIGT |
| TR-09 | **Plattformabhängigkeit**: MV2 + blockierendes `webRequest`; Fortbestand in Chromium ungeklärt. | README; Manifeste | OFFEN |
| TR-10 | **`runtime.onInstalled` verwirft den Zustand** bei jeder Installation/Aktualisierung. | `background.js:59-63` | BESTÄTIGT |

---

## 13. Offene Punkte und Prüfstatus

### 13.1 Prüfstatus nach Bereichen

| Bereich | Status |
|---|---|
| Statische Lektüre des Quellcodes (Abschnitte 2–12) | BESTÄTIGT bzw. ABGELEITET, soweit gekennzeichnet |
| **Laufzeittests** (Erweiterung installiert/ausgeführt) | **NICHT GEPRÜFT** |
| **Browserkompatibilität** (Versionen, Derivate, Android) | **NICHT GEPRÜFT** |
| **Netzwerktests** (Mitschnitt, Header, Abrufzeitpunkte) | **NICHT GEPRÜFT** |
| Moodle-/Prüfungsplattform-Verhalten | **NICHT GEPRÜFT** (und nicht Teil des Auftrags) |
| Build (`npm ci`, `npm run build`) | **NICHT GEPRÜFT** |
| Korrektheit der Header-/Schlüsselberechnung gegenüber SEB | **NICHT GEPRÜFT** (bewusst ausgeklammert) |
| Inhalt von `bek.json` | **NICHT GEPRÜFT** (nur Struktur) |
| `popup.css`, Bilder/GIFs; Markup von `taskbar.html`, `quit.html`, `startup.html`, `popup.html` und `assistant/popup.js` (nur gezielt) | **NICHT GEPRÜFT** bzw. nur teilweise |

### 13.2 Offene Fragen

| ID | Frage | Status |
|---|---|---|
| UA-01 | Ist `runtime.getBrowserInfo` in **Firefox-Content-Scripts** verfügbar? Davon hängt ab, ob dort die Chromium-Fassade (`browser-api.js:2`) und die Skript-Injektion (`download-content.js:59`) zusätzlich greifen. | OFFEN |
| UA-02 | Ist `storage.session` in Chromium mit MV2 verfügbar; wie lange hält es Daten? | OFFEN |
| UA-03 | Unterstützen aktuelle Chromium-Versionen/Derivate noch MV2 mit blockierendem `webRequest`? | OFFEN |
| UA-04 | Begründungen für den eigenen Plist-Parser und die synchrone SHA-256 stehen nicht im Code. Die Notwendigkeit synchroner Antworten gilt belegbar nur für Chromium (`background.js:101-103`). | OFFEN |
| UA-05 | Weichen User-Agent, Client-Hints, Header-Reihenfolge und TLS-Merkmale von einem echten SEB-Client ab? (Referenzdaten fehlen.) | OFFEN |
| UA-06 | Sind Ereignisse mit `isTrusted=false` auf der eingesetzten Moodle-Version beobachtbar? | OFFEN |
| UA-07 | Robustheit des Plist-Parsers gegen reale Dateien. | OFFEN |
| UA-08 | Tritt der `local`-Fallback tatsächlich auf (Zustandsanzeige trotz inaktiver Header)? | OFFEN |
| – | Welche Änderungen enthält die Bezeichnung 2.4.1 gegenüber der Paketversion 2.4.0? | OFFEN |
| – | Aufbewahrung der Daten bei den KI-Anbietern; Rechtsgrundlagen. | NICHT GEPRÜFT |
| – | Speicherorte der Erweiterungsdaten je Browser/Betriebssystem. | NICHT GEPRÜFT |

Vollständige Liste der Dokumentationslücken: `documentation-gaps.md`.

---

## 14. Build-Prozess und Entwicklungsumgebung

### 14.1 Voraussetzungen

| Punkt | Wert | Quelle |
|---|---|---|
| Sprache | JavaScript (ES-Module), HTML, CSS; kein TypeScript, kein Framework | gesamter Code |
| Laufzeit für Build | Node.js mit ESM und `top-level await` | `build.mjs`, `package.json:"type":"module"` |
| Paketmanager | npm (`package-lock.json` vorhanden) | README |
| Entwicklungsabhängigkeiten | `terser 5.47.1`, `lightningcss 1.31.1`, `fflate 0.8.2` (Lockfile: 25 Pakete) | `package.json`, `package-lock.json` |
| Laufzeitabhängigkeiten der Erweiterung | **keine** Drittbibliotheken | Quellcode |
| Zielbrowser | Firefox ≥ 140 (Android ≥ 142); Chromium ≥ 111 mit MV2-Unterstützung | Manifeste |

Eine Mindest-Node-Version ist nicht festgelegt (kein `engines`-Feld, keine `.nvmrc`).

### 14.2 Ablauf von `npm run build` (`build.mjs`)

1. Importiert `validateBekCatalog` aus dem Erweiterungsquellcode (`:6`) und prüft `bek.json`; ein Fehler stoppt den Build (`:17`).
2. Stellt sicher, dass beide Manifest-Quelldateien existieren (`:19-23`).
3. Leert und legt `release/chrome` und `release/firefox` an (`:27-30`).
4. Durchläuft `source/` rekursiv (`buildDirectory`, `:31-66`): Manifeste übersprungen; `.js` mit Terser (Modul-Modus per Regex auf `import`/`export` am Zeilenanfang, `:50`); `.css` mit `lightningcss`; `.json` neu serialisiert; alle anderen Dateien (HTML, PNG, SVG) unverändert kopiert.
5. Schreibt pro Browser `manifest.json` und das geprüfte `bek.json` (`:69-77`); Chrome-MV2-Manifest darf kein `world` in den Content-Scripts enthalten (`:71-74`).
6. Erstellt ZIPs `release/OEB-<Version>-chrome.zip` und `…-firefox.zip` mit `fflate` (Stufe 9, `manifest.json` im Wurzelverzeichnis), Version aus `package.json` (`:80-95`).

**NICHT GEPRÜFT:** Der Build wurde nicht ausgeführt.

### 14.3 Installation zu Testzwecken (laut README)

- **Firefox:** `about:debugging#/runtime/this-firefox` → „Temporäres Add-on laden" → `manifest.json`; verschwindet beim Beenden von Firefox.
- **Chromium:** `chrome://extensions` → Entwicklermodus → „Entpackte Erweiterung laden".
- Nach Änderungen: Erweiterung neu laden **und** Seite neu laden.

### 14.4 Hinweise

- `source/manifest.json` steht in `.gitignore`, wird vom Build übersprungen und nie verwendet (Altlast, OFFEN).
- `bek.json` liegt im Projektwurzelverzeichnis, damit es gleichzeitig als Fern-Datenquelle im GitHub-Repo dient.
- `release/` und `node_modules/` sind nicht versioniert.

### 14.5 Tests

**BESTÄTIGT:** Es existieren keine automatisierten Tests und keine CI. Die Struktur (injizierbare Abhängigkeiten in `createBekCatalogManager`, `probeSebDownload(url, {fetchImpl, signal})`, reine Funktionen in `launch-response.js`, `settings.js`, `parser.js`) wäre für Tests geeignet (ABGELEITET), ist aber nicht genutzt. Vorschau-GIFs im Repo deuten auf manuelles Testen hin (ABGELEITET).

---

## 15. Wartbarkeit und Erweiterbarkeit (allgemein)

**Bewertung (ABGELEITET):**

| Stärke | Schwäche |
|---|---|
| Klare Trennung in zwei Funktionsblöcke, viele kleine, abhängigkeitsarme Module | Zentraler Hub `seb/background.js` (748 Zeilen) vermischt Erkennung, Zustand, Routing und Overlay-Steuerung |
| Keine Drittbibliotheken zur Laufzeit | Keine Tests, keine statische Typisierung; Nachrichtennamen und Speicherschlüssel sind verstreute Strings |
| Defensive Fehlerbehandlung, Größen- und Zeitlimits (nicht auf allen Pfaden) | Doppelte Wege für Firefox/Chromium; Totcode (9.4); nicht serialisierte Zustandspfade (7.3) |
| Datenkatalog vom Code getrennt | Mehrere Konstanten doppelt (IDs, Storage-Schlüssel, Versionsangaben) |

**Für ein Untersuchungsvorhaben im IT-Team:** Wenn die Erweiterung als Untersuchungsobjekt zerlegt werden soll (z. B. um Erkennungsregeln zu validieren), nur **lesen und beobachten**, in einem **isolierten Testprofil**, gegen eine **eigene Test-Moodle-Instanz** und nie gegen produktive Prüfungen. Eine Anleitung zum Erweitern der Umgehungs- oder Autofill-Funktionen ist **nicht** Teil dieses Dokuments.

---

## 16. Einarbeitung in 30 Minuten: Lesereihenfolge

1. `OEB-Analyse-IT.md` Abschnitte 1–3 (was ist das, was kann es, was nicht).
2. Dieses Dokument: Abschnitte 0, 1, 6.1, 5.1–5.3 (Gesamtbild).
3. Quelldateien: `chrome.manifest.json` → `seb/background.js` ab Zeile 212 (`handleMessage`) → `seb/launch-response.js` und `seb/parser.js` → `seb/content.js` → `assistant/background.js`.
4. Bei Bedarf: `seb/headers.js`, `config-key.js`, `bek-catalog.js`, `download-*`.
5. Abschnitte 7.3, 9.4, 10.2 für bekannte Besonderheiten; Abschnitt 11 für Datenschutz und Sicherheit.
