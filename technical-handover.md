# Technical Handover: „OEB — Open Exam Browser" (Architektur- und Aufbauübersicht)

**Ergänzung zu:** `OEB-Analyse-IT.md` (Sicherheits- und Erkennungsanalyse). Diese Datei ersetzt sie nicht. Beide Dokumente zusammen bilden das Gesamtbild: *Analyse* = Risiken und Erkennung, *Handover* = innerer Aufbau.
**Zielgruppe:** neue Mitglieder des IT-Teams, die sich in kurzer Zeit einen fundierten Überblick verschaffen sollen.
**Analysierter Stand:** ZIP `OEB-2.4.1` (interne Versionsnummer in `package.json` und beiden Manifesten: `2.4.0`).
**Methode:** statische Quellcode-Lektüre; die Erweiterung wurde nicht ausgeführt. Pfade beziehen sich auf `OEB-2.4.1/`, Zeilenangaben auf diesen Stand.

**Kennzeichnung:** **[Fakt]** direkt im Code belegt · **[Schluss]** Folgerung aus dem Code, nicht getestet · **[Vermutung]** plausibel, nicht belegt.

**Bewusste Abgrenzung (Detailtiefe):** Dieses Dokument beschreibt Aufbau, Zuständigkeiten, Kommunikation, Zustände und Lebenszyklen. Es enthält **keine** Rechenvorschriften, Konstanten, Profildaten, Prompt-Texte, Seitenstruktur-Selektoren oder Schritt-für-Schritt-Abläufe der Header-Fälschung bzw. des KI-Autofills, und keine Hinweise, wie diese Teile gepflegt oder verbessert werden könnten. Für diese Bereiche steht nur, *dass* sie existieren, *wo* sie liegen und *womit* sie zusammenhängen.

---

## 0. Korrekturen an der bisherigen Analyse (`OEB-Analyse-IT.md`)

Beim erneuten Abgleich mit dem Quellcode wurden diese Punkte präzisiert oder korrigiert. Die Datei `OEB-Analyse-IT.md` wurde entsprechend angepasst.

| # | Bisherige Aussage | Korrektur | Beleg |
|---|---|---|---|
| K1 | Der Aufruf `POST …/clear_session.php` erfolge „beim Beenden". | Er erfolgt bei **jedem Entladen** der Konfiguration (Beenden, „Close exam"-Button, Entladen per Popup, **und beim Ersetzen durch eine neue Konfiguration**), aber nur, wenn die Erweiterung eine Moodle-Kurs-Modul-ID (`cmid`) aus der Start-URL ermitteln konnte (Pfad enthält `/mod/quiz/`, Parameter `id`). | `seb/background.js:615-623` (`unloadConfiguration`), `:442-448` (`replaceConfiguration` ruft `unloadConfiguration`), `:698-707` (`getMoodleCmid`), `seb/content.js:285-306` |
| K2 | „Das Seiten-JavaScript kann Elemente aus Content-Scripts nicht direkt ‚sehen'". | Missverständlich. Content-Scripts teilen sich das **DOM** mit der Seite. Von der Erweiterung eingefügte Elemente (iframes, Statusmeldung, Marker, Script-Tag) sind für Seiten-JavaScript sichtbar. Nur die **JavaScript-Objekte** (Variablen, Funktionen) der Erweiterung sind getrennt. | `seb/content.js:168-171`, Isolation ist Standardverhalten von Content-Scripts [Schluss] |
| K3 | Ctrl+Shift+X werde „im Capture-Modus abgefangen". | Genauer: Ein `keydown`-Listener in der **Capture-Phase** (dritter Parameter `true`) vergleicht die Tastenkombination mit dem gespeicherten Kürzel. Existiert ein aktiver „Weiter"-Button, wird das Ereignis **verschluckt** (`preventDefault` + `stopImmediatePropagation`) und der Button per `click()` ausgelöst. | `assistant/content.js:122-129` |
| K4 | Start-Animation zeige den „gefälschten" SEB-Build. | Die Animation zeigt fest verdrahtet Build `3.10.1.864` / Version `3.10.1`. Das im Netzwerk behauptete Standard-Profil ist dagegen `3.10.2.920`. Das ist ein interner Widerspruch (nur kosmetisch, aber ein Hinweis auf nicht synchron gepflegte Teile). | `seb/assets/startup.js:1-2` vs. `seb/browser-exam-key.js:6`, `seb/headers.js:27` |
| K5 | (neu) Lokale Spuren auf dem Endgerät. | Die Erweiterung speichert den **letzten KI-Lauf** (Fragen ohne Bilddaten, Antworten, verwendeter Anbieter, Zeitstempel, URL) im lokalen Erweiterungs-Speicher unter dem Schlüssel `oebAiLastRun`. Dies ist ein forensisch relevantes Artefakt. | `assistant/background.js:6, 101-105` |

---

## 1. Systemüberblick in einem Absatz

OEB ist eine **Browser-Erweiterung (Manifest V2) ohne eigenen Server**. Sie besteht aus (a) einer dauerhaft laufenden **Hintergrundseite** („Background Page") als Steuerzentrale, (b) **Content-Scripts**, die in jede besuchte Seite und jeden Frame geladen werden, (c) einer **Popup-Oberfläche** zur Bedienung, (d) mehreren **eingebetteten Oberflächen-Seiten** (Start-Animation, Taskleiste, Beenden-Dialog), die als iframes in die Prüfungsseite gelegt werden, und (e) einem **Build-Skript**, das daraus zwei Pakete (Firefox, Chromium) erzeugt. Fachlich hat die Erweiterung zwei weitgehend getrennte Funktionsblöcke: **„SEB-Block"** (Konfigurationsdatei erkennen → laden → Anfragen an den Prüfungsserver modifizieren → SEB-ähnliche Oberfläche einblenden) und **„Assistant-Block"** (Fragen auslesen → externen KI-Dienst fragen → Antworten eintragen). Beide teilen sich nur die Hintergrundseite, die Browser-API-Schicht und das Popup.

---

## 2. Komponenten und Ausführungsumgebungen

Eine Browser-Erweiterung läuft in **getrennten Umgebungen**, die nur über Nachrichten kommunizieren. Das ist der wichtigste Schlüssel zum Verständnis.

| Umgebung | Läuft wo | Lebensdauer | Dateien | Zugriff |
|---|---|---|---|---|
| **Background Page** | unsichtbare Dauerseite der Erweiterung (`persistent: true`) | solange der Browser läuft | `background.html` → `seb/background.js` und alles, was es importiert | Voller Zugriff auf Browser-APIs (`tabs`, `webRequest`, `downloads`, `storage`), Netzwerk ohne Seiten-Einschränkung |
| **Content-Script** (isolierte Welt) | in jeder Webseite, jedem Frame, ab `document_start` | solange die Seite geladen ist | `browser-api.js`, `assistant/extract.js`, `assistant/apply.js`, `assistant/content.js`, `seb/download-content.js`, `seb/content.js` | Zugriff auf das DOM der Seite, aber nicht auf deren JS-Variablen; wenige Browser-APIs (`runtime`, `storage`) |
| **Page-Script** („MAIN world") | im JS-Kontext der Webseite selbst | solange die Seite geladen ist | `seb/download-page.js` | Sieht und verändert Seiten-JS-Objekte (z. B. `URL.createObjectURL`); **kein** Zugriff auf Browser-APIs |
| **Popup** | kleines Fenster beim Klick aufs Symbol | nur solange geöffnet | `popup.html`, `popup.css`, `seb/popup.js`, `assistant/popup.js` | Erweiterungs-APIs; kein dauerhafter Zustand |
| **Eingebettete UI-Frames** | iframes in der Prüfungsseite, geladen von der Erweiterung | solange der Content-Script sie vorhält | `seb/assets/startup.html/.js`, `taskbar.html/.js`, `quit.html/.js`, `colors.css`, `quit.css` | Eigene Seite der Erweiterung, kommunizieren per `postMessage` mit dem Content-Script |

**[Fakt]** Content-Scripts sind **klassische Skripte** (kein `import`/`export`); sie teilen Funktionen über das globale Objekt `globalThis.OEBAssistant` (`assistant/extract.js:2`, `assistant/apply.js:2`). Background und Popup nutzen dagegen **ES-Module** (`background.html:6`, `popup.html:204-205`). Dieser Unterschied erklärt viele Struktur-Entscheidungen.

---

## 3. Vollständiges Dateiinventar mit Zweck und „Warum existiert diese Datei?"

### 3.1 Wurzelverzeichnis

| Datei | Typ | Aufgabe | Warum es sie gibt | Hängt zusammen mit |
|---|---|---|---|---|
| `README.md` | Doku | Projektbeschreibung des Autors, Installations- und Build-Hinweise | Öffentliche Projektseite; nennt Herkunft („Based on SebBypass") und Haftungsausschluss | — |
| `package.json` | Konfig | Name `oeb`, Version `2.4.0`, Skript `build`, Dev-Abhängigkeiten, `"type": "module"` | Build-Definition. Durch `"type":"module"` wird `build.mjs` als ES-Modul ausgeführt | `build.mjs`, `package-lock.json` |
| `package-lock.json` | Konfig | Fixiert exakte Versionen der Build-Werkzeuge | Reproduzierbarer Build per `npm ci` | `package.json` |
| `build.mjs` | Skript | Build und Paketierung (siehe Abschnitt 11) | Erzeugt aus einer Quelle zwei browserspezifische Pakete | `source/**`, `bek.json`, `source/seb/bek-catalog.js` |
| `bek.json` | Daten (≈59 KB) | **Datenkatalog** mit buildspezifischen Profilen für die automatische Schlüsselberechnung; Schema `schemaVersion: 1`, `defaultVersion`, Liste `profiles` | Trennt *wechselnde Daten* (Profile) von *festem Code* (Algorithmus), damit neue Profile ohne Code-Release verteilt werden können. Wird vom Build ins Paket kopiert **und** vom Autor im GitHub-Repo zur Laufzeit ausgeliefert | `source/seb/bek-catalog.js`, `build.mjs`. Inhalt wird hier bewusst nicht dokumentiert |
| `.gitignore` | Konfig | Ignoriert `/release/`, `/node_modules/`, `/source/manifest.json`, `*.log`, `.DS_Store` | Build-Artefakte nicht versionieren; `source/manifest.json` wird nie von Hand gepflegt (siehe 11.3) | `build.mjs` |
| `preview/*.gif` | Medien | Demo-Animationen (Start, Beenden, Autofill) für das README | Nur Dokumentation. Nicht Teil des Pakets | `README.md` |

**[Fakt]** Es gibt **kein** `tests/`-Verzeichnis, keine Testkonfiguration und kein Test-Skript in `package.json`.

### 3.2 `source/` — Wurzel der Erweiterung

| Datei | Aufgabe | Warum es sie gibt | Hängt zusammen mit |
|---|---|---|---|
| `chrome.manifest.json` | Manifest (V2) für Chromium: Berechtigungen, Background-Seite, Popup, Content-Script-Liste, `web_accessible_resources` inkl. `seb/download-page.js`, Mindestversion Chrome 111 | Das Manifest ist der „Personalausweis" der Erweiterung; Browser lesen daraus, was geladen wird. Zwei Dateien, weil sich Firefox und Chromium bei Icons, MAIN-World-Skripten und Identität unterscheiden | `build.mjs` (wird beim Build als `manifest.json` ins Paket kopiert) |
| `firefox.manifest.json` | Manifest (V2) für Firefox: fest vergebene ID `oeb@open-exam-browser.local`, `strict_min_version 140`, Android ab 142, deklarierte Datenerfassung (`websiteContent`, `authenticationInfo`), **zweiter Content-Script-Block mit `world: "MAIN"`** für `seb/download-page.js` | siehe oben; die ID macht die Erweiterung im Inventar auffindbar | `build.mjs` |
| `background.html` | 8 Zeilen: lädt `browser-api.js`, dann `seb/background.js` als Modul | Manifest V2 erwartet für Hintergrundseiten eine HTML-Seite | `browser-api.js`, `seb/background.js` |
| `browser-api.js` | **Kompatibilitätsschicht**: Falls kein Firefox-`browser`-Objekt existiert, baut sie ein `browser`-Objekt auf Basis von `chrome.*` mit Promises nach | Chromium (MV2) hat callback-basierte `chrome.*`-APIs, Firefox promise-basierte `browser.*`. So kann der gesamte restliche Code einheitlich `browser.*` mit `await` nutzen. Außerdem: Falls `storage.session` fehlt, wird `storage.local` untergeschoben | Wird in Hintergrund, Popup und allen Content-Scripts als **erstes** geladen |
| `popup.html` / `popup.css` | Struktur und Stil des Popups mit zwei Reitern („SEB" und „Assistant") | Bedienoberfläche | `seb/popup.js`, `assistant/popup.js` |
| `icon.svg`, `assets/icons/icon-{16,32,48,128}.png` | Symbole | Firefox nutzt `icon.svg`, Chromium die PNGs | Manifeste |

### 3.3 `source/seb/` — SEB-Block

| Datei (Zeilen) | Aufgabe | Warum es sie gibt | Importiert / wird genutzt von |
|---|---|---|---|
| `background.js` (748) | **Steuerzentrale**: Nachrichten-Router, Zustandsverwaltung, Download-Erkennung, Konfigurationswechsel, Start/Beenden, Badge, Einstellungen | Ein einziger Ort, der den Gesamtzustand besitzt. Alle anderen Komponenten fragen ihn per Nachricht | Importiert `parser`, `headers`, `config-key`, `assistant/background`, `browser-exam-key`, `launch-response`, `settings`, `download-check`, `download-stream`, `bek-catalog` |
| `headers.js` (121) | Registriert den `webRequest`-Hook und hält den **Header-Zustand** (an/aus, Ursprung, behaupteter Build, Zähler). Wirkt ausschließlich auf Anfragen an den Ursprung der Prüfungs-Start-URL | Kapselt den sicherheitsrelevantesten Eingriff an einer Stelle; bietet `enableHeaderInjection`, `disableHeaderInjection`, `getHeaderStatus`, `reloadActiveHeaderTab` | Genutzt nur von `background.js`; importiert `config-key`, `browser-exam-key` |
| `config-key.js` (337) | (1) **Eigener Parser für das SEB-Plist-XML** (kein DOMParser, im Hintergrund nicht verfügbar), (2) Auslesen einzelner Konfigurationswerte (Start-URL, Quit-Einstellungen, Taskleisten-Anzeige, Salt), (3) Berechnung des Konfigurations-Hashes und des anfragebezogenen Hashes, (4) eine eigene **synchrone SHA-256-Implementierung** | Der `webRequest`-Hook muss *synchron* antworten; die WebCrypto-API ist asynchron. Deshalb gibt es eine zusätzliche synchrone Implementierung (Zeilen ≈ 261-337). Das erklärt die ungewöhnliche Dateigröße | Genutzt von `background.js`, `headers.js`, `browser-exam-key.js`, `seb/popup.js` |
| `browser-exam-key.js` (64) | Normalisierung der BEK-Einstellungen (Modus „automatic"/„provided", Build-Version) und Bereitstellung des Schlüssels für den Header-Hook | Trennt die „Schlüssel"-Frage von der „Header setzen"-Frage. **Enthält ungenutzten Code** (siehe 9.4) | `headers.js`, `background.js`; nutzt `config-key`, `bek-catalog` |
| `bek-catalog.js` (143) | Verwaltung des Profilkatalogs: Validierung, Laden aus Cache/Paket, Aktualisierung von GitHub, Zeitsteuerung. Enthält eine Fabrikfunktion `createBekCatalogManager` mit **injizierbarem** `storage`/`fetcher`/`now` | Fernaktualisierung der Daten ohne Code-Update. Die Dependency-Injection macht die Logik prüfbar, auch wenn keine Tests vorhanden sind | `browser-exam-key.js`, `background.js`, `build.mjs` (importiert `validateBekCatalog`) |
| `bek-selection.js` (7) | Hilfsfunktion `bekSettingsForVersion` für das Popup | Aus Auswahlfeld-Wert („off" oder Version) ein Einstellungsobjekt machen | `seb/popup.js` |
| `parser.js` (246) | Erkennt Format einer `.seb`-Datei und macht daraus Klartext-XML (Entpacken, Passwort-Entschlüsselung). Fehlerklasse `SebParseError` mit Codes | Trennt „Dateiformat" von „Inhalt". Zertifikat-verschlüsselte Dateien werden ausdrücklich abgelehnt | `background.js` |
| `settings.js` (16) | Schlüssel, Standardwerte und `normalizeSettings` für Benutzereinstellungen (Download-Prüfung, Tab beim Beenden schließen, Start-Animation, Taskleistenmodus und -größe 50-300) | Eine Stelle für Einstellungs-Validierung, auch für Altformate (`showSebBar`) | `background.js` |
| `file-import.js` (9) | `importSelectedFile`: liest eine vom Nutzer gewählte Datei und schickt sie als `SEB_PROCESS_BYTES` an den Hintergrund | Entkopplung des Popups vom Hintergrund-Protokoll | `seb/popup.js` |
| `launch-response.js` (23) | `isSebConfigurationUrl`, `classifySebResponse`: erkennt anhand URL, `Content-Disposition`, MIME-Typ, ob eine HTTP-Antwort eine SEB-Konfiguration ist, und ob sie abgefangen („consume") werden soll | Zentrale Erkennungsregel für Navigationsantworten | `background.js` |
| `download-check.js` (88) | Begrenztes, **nur lesendes** Prüfen von Downloads unbekannten Typs (≤ 8 MB, Streaming, 3 s Timeout) auf SEB-Signaturen | Moodle liefert die Konfiguration teils ohne `.seb`-Endung; ohne Inhaltsprüfung würde man sie nicht erkennen | `background.js` |
| `download-stream.js` (41) | Firefox-spezifisch: `mirrorDownloadResponse` spiegelt eine Antwort über `filterResponseData`, gibt sie unverändert weiter und prüft eine begrenzte Kopie; `isExtensionlessResponse` | Nur Firefox kann den Antwortstrom live abgreifen (auch bei einmaligen/POST-Downloads) | `background.js` |
| `download-content.js` (78) | Content-Script-Brücke: spricht per `window.postMessage` mit `download-page.js` und per `runtime.sendMessage` mit dem Hintergrund; beantwortet `SEB_PROBE_DOWNLOAD_URL` (lädt die URL mit den **Cookies der Seite**); injiziert in Chromium `download-page.js` per `<script>`-Tag | Content-Script darf das Seiten-JS nicht direkt erreichen, `download-page.js` darf keine Browser-APIs nutzen: die Brücke verbindet beides | `download-page.js`, `background.js` |
| `download-page.js` (126) | Läuft im Seiten-Kontext: merkt sich Blob-URLs, ersetzt `URL.createObjectURL`/`revokeObjectURL`/`HTMLAnchorElement.prototype.click`, prüft Download-Links und übergibt Kandidaten an die Brücke | Blob-Downloads sind nur im Seiten-Kontext sichtbar und die URL wird oft sofort widerrufen | `download-content.js` |
| `content.js` (316) | Content-Script des SEB-Blocks: fängt Klicks/Formulare auf SEB-Links ab, zeigt Status, steuert Overlay-Frames (Start/Taskleiste/Beenden), reagiert auf Taskleisten-Nachrichten, löst die Moodle-Rücksetzung aus | Verbindungsglied zwischen Seite und Hintergrund | `background.js` (per Nachricht), UI-Frames (per `postMessage`) |
| `content.css` (≈ 30+ Zeilen) | Stil der Overlay-Frames (`position: fixed`, höchste `z-index`, `!important`) | Overlay muss über der Moodle-Seite liegen und sich nicht von Seitenstilen beeinflussen lassen | `content.js` |
| `popup.js` (351) | Logik des SEB-Reiters: Status anzeigen, Datei öffnen, Passwort eingeben, BEK-Version wählen, Einstellungen speichern, XML anzeigen/kopieren/speichern | Bedienung | `background.js` (Nachrichten), `config-key.js`, `file-import.js`, `bek-selection.js` |
| `assets/startup.html/.js` (554/95) | Eingebettete **Start-Animation** im Stil des SEB-Startbildschirms | Optik | `content.js` |
| `assets/taskbar.html/.js` (111/69) | Eingebettete **Taskleiste** (Uhr, Tastatursprache, Netz, Audio, Reload, Beenden) | Optik, erzeugt Nachrichten an `content.js` | `content.js` |
| `assets/quit.html/.js/.css` (50/34/101) | Eingebetteter **Beenden-Dialog** (mit/ohne Passwortfeld) | Optik und Beenden-Fluss | `content.js` |
| `assets/colors.css` (11) | gemeinsame Farbvariablen von Start-Animation und Taskleiste | Gemeinsame Stilbasis | `startup.html`, `taskbar.html` |
| `icons/*.png/svg` | Logos/Symbole für die Overlay-Frames | Optik | `startup.html`, `taskbar.html` |

### 3.4 `source/assistant/` — Assistant-Block

| Datei (Zeilen) | Aufgabe | Warum es sie gibt | Zusammenhang |
|---|---|---|---|
| `background.js` (105) | **Orchestrierung** eines KI-Laufs im Hintergrund: Vorbedingungen prüfen, Seite auslesen lassen, Anbieter fragen, Seite erneut prüfen, Antworten eintragen lassen, Ergebnis speichern; Tastenkürzel-Registrierung; Badge | Der Ablauf braucht Browser-APIs (Tabs, Storage) und Netzwerk, die Content-Scripts nicht haben | Importiert `providers.js`; wird von `seb/background.js` eingebunden (`answerTab`, `installAssistantCommands`) |
| `providers.js` (441) | Katalog der unterstützten KI-Dienste (8 feste + 1 frei konfigurierbar), Konto-Datenstruktur, Validierung, Modelllisten abrufen, Anfrage/Antwort je Dienst-Familie, Fallback über mehrere Konten | Abstraktion: das übrige Programm kennt nur „fragen" und „Antworten"; die Dienste unterscheiden sich in API-Format. **Inhalt der Anweisungen an die KI wird hier nicht dokumentiert** | `assistant/background.js`, `assistant/popup.js` |
| `extract.js` (178) | Content-Script-Modul: liest die Fragen der aktuellen Moodle-Seite in eine strukturierte Liste (inkl. begrenzter Bilddaten) | Trennung „Seite lesen" von „KI fragen" | Bereitgestellt als `OEBAssistant.extractQuestions`; aufgerufen durch `assistant/content.js` |
| `apply.js` (202) | Content-Script-Modul: trägt strukturierte Antworten in die Formularelemente ein; liefert pro Frage „filled"/„skipped" | Trennung „Antwort bestimmen" von „Seite verändern" | Bereitgestellt als `OEBAssistant.applyAnswers` |
| `content.js` (56) | Content-Script-Brücke: beantwortet `OEB_AI_EXTRACT`/`OEB_AI_APPLY`, hält das „Nächste Seite"-Tastenkürzel (nur im obersten Frame) | Verbindet Hintergrund-Nachrichten mit den beiden Modulen | `extract.js`, `apply.js` |
| `popup.js` (600) | Logik des Assistant-Reiters: KI-Konten verwalten, Domain freischalten, Fragen anzeigen, Lauf starten, Kürzel konfigurieren | Bedienung | `providers.js`, `background.js` (Nachricht `OEB_AI_RUN`) |

---

## 4. Wer gehört zu wem: Funktionen nach Komponente

Nur Funktionsnamen, Zweck in einem Satz und Fundstelle. Schritt-für-Schritt-Ablauf nur dort, wo er architektonisch nötig ist.

### 4.1 `seb/background.js` (Steuerzentrale)

| Funktion | Zeile | Zweck |
|---|---|---|
| (Listener) `runtime.onInstalled` | 59 | Schreibt Standard-Einstellungen und Ruhezustand bei Installation/Update |
| (Listener) `runtime.onMessage` → `handleMessage` | 65, 212 | **Zentraler Nachrichten-Router** (Katalog in Abschnitt 6) |
| (Listener) `downloads.onCreated` | 72 | Reaktion auf Browser-Downloads (Fallback der Erkennung) |
| (Listener) `webRequest.onHeadersReceived` | 95 | Reaktion auf HTTP-Antworten (Haupt-Erkennungsweg); delegiert an `inspectDownloadResponse` |
| `inspectDownloadResponse` | 109 | Entscheidet pro Antwort: spiegeln (Firefox), als SEB erkennen, prüfen oder ignorieren; kann Antwort abbrechen („cancel") |
| `probeDownloadOnce` / `inspectUnknownDownload` | 153 / 176 | Inhalt unbekannter Downloads begrenzt prüfen; Doppelabrufe vermeiden (`inFlightProbes`) |
| `captureAutomaticBytes` | 188 | Automatisch erkannte Bytes validieren, Duplikate unterdrücken, an `processBytes` übergeben |
| `processUrl` / `processUrlOnce` | 348 / 359 | Konfigurations-URL (auch `seb://`, `sebs://`, eingebettete Base64-Daten) holen und weitergeben |
| `processBytes` | 436 | **Serialisiert** Konfigurationswechsel über eine Warteschlange (`configurationQueue`) |
| `replaceConfiguration` | 442 | Parsen → alten Zustand entladen → neuen Zustand speichern → ggf. starten (`activateLaunch`) |
| `unlockPending` | 493 | Passwortgeschützte Datei mit eingegebenem Passwort erneut öffnen |
| `activateLaunch` | 559 | Start-URL bestimmen, Header-Hook aktivieren, Zielreiter überblenden/neu laden |
| `findTargetTab` / `transitionToDestination` / `beginPendingReload` | 586 / 593 / 607 | Zielreiter bestimmen; Übergang Start-Animation → Neuladen |
| `unloadConfiguration` | 615 | Alles zurücksetzen (Moodle-Rücksetzung, Header aus, Zustand leeren, Overlay entfernen, Tabs optional schließen/neuladen) |
| `saveState` / `loadState` | 540 / 544 | Zustandsspeicher (siehe Abschnitt 7) |
| `loadSettings` / `saveSettings` / `loadBekSettings` / `enableLoadedHeaders` | 639 / 662 / 644 / 654 | Einstellungen und Schlüsselmodus laden/speichern; Header-Hook aus geladenem Zustand aktivieren |
| `buildPresentationSettings` / `broadcastPresentation` / `broadcastMessage` | 675 / 669 / 683 | Darstellungseinstellungen aus Nutzer- und Konfigurationswerten zusammensetzen und an Tabs senden |
| `setBadge` | 553 | Symbol-Badge: `…` lädt, `✓` bereit, `?` Passwort nötig, `!` Fehler |
| Hilfsfunktionen | 690-748 | `isWebUrl`, `getMoodleCmid`, `normalizeSebUrl`, `isSebUrl`, `isSebMime`, `fileNameFrom*` |

### 4.2 `seb/headers.js` (Header-Hook; nur Rolle)

| Funktion | Zeile | Zweck |
|---|---|---|
| (Listener) `webRequest.onBeforeSendHeaders` | 19 | Blockierender Hook; ändert Anfrage-Header nur für den Prüfungs-Ursprung |
| `enableHeaderInjection` | 53 | Validiert Eingaben, löst den Schlüssel auf, setzt Modulzustand, Status zurückgeben |
| `disableHeaderInjection` | 73 | Setzt den Modulzustand vollständig zurück |
| `reloadActiveHeaderTab` / `getHeaderStatus` | 85 / 93 | Aktiven Tab neu laden / Status für das Popup |
| `refreshMatchingTabs`, `isAllowedUrl`, `recordError` | 106-121 | Zählen betroffener Tabs, URL-Prüfung, Fehlermerker |

### 4.3 `seb/config-key.js`

| Funktion | Zeile | Zweck |
|---|---|---|
| `calculateSebConfigHash` | 6 | Konfigurations-Hash der geladenen Konfiguration |
| `calculateRequestConfigKey` / `…Sync` | 13 / 17 | Anfragebezogener Hash (async/sync). **Die async-Variante wird nirgends verwendet** (siehe 9.4) |
| `getSebExamSalt`, `getSebStartUrl`, `getSebQuitSettings`, `getSebPresentationSettings` | 21-68 | Einzelwerte aus der Konfiguration lesen |
| `verifySebQuitPassword` | 70 | Prüft das Beenden-Passwort lokal gegen den in der Konfiguration hinterlegten Hash |
| `serializeSebConfiguration` | 76 | **Nicht verwendet** |
| `parsePlist` und Hilfsfunktionen | 83-259 | Eigener Plist-Parser/Serialisierer |
| `sha256Hex`, `sha256HexSync`, `rotateRight` | 261-337 | Hashfunktionen (async/sync) |

### 4.4 `seb/browser-exam-key.js` und `seb/bek-catalog.js` (nur Rollen)

| Funktion | Datei:Zeile | Zweck |
|---|---|---|
| `normalizeBekSettings` | browser-exam-key:9 | Validiert/normalisiert Schlüsselmodus und Build-Angabe |
| `resolveBrowserExamKey` | :23 | Liefert für Modus „aus"/„provided"/„automatic" den Schlüssel bzw. `undefined` |
| `calculateNativeBrowserExamKey` | :34 | Automatische Berechnung aus Konfiguration und Katalogprofil |
| `calculateFallbackBrowserExamKey` | :49 | **Nicht verwendet** |
| `calculateRequestBrowserExamKeySync` | :61 | Synchrone Ableitung für jede Anfrage |
| `validateBekCatalog` | bek-catalog:9 | Strikte Schema- und Prüfsummenvalidierung des Katalogs |
| `createBekCatalogManager` (`initialize`, `info`, `refresh`, `get`) | :64 | Katalogspeicher mit Cache, Zeitsteuerung und Fehlerzustand |
| `getBekCatalogInfo` / `refreshBekCatalog` / `getBekProfile` | :136-143 | Öffentliche Schnittstelle |

### 4.5 `seb/parser.js`

`decodeEmbeddedSebUrl` (19), `inspectSebFile` (37, Hauptfunktion: Formaterkennung → Ergebnisobjekt `{status, mode, xml}` oder `{status:"password-needed"}`), `unwrapPlainPayload` (83), `unwrapGzip` (107, max. 4 Schichten), `decryptRNCryptor` (127), `deriveKey` (191), `sha256Hex` (212), `readyResult`, `looksLikeXml`, `isGzip`, `ascii`, `toBytes`.

### 4.6 `seb/content.js`

| Funktion | Zeile | Zweck |
|---|---|---|
| Klick-Listener (Capture) | 34 | Fängt Klicks auf SEB-Links/Buttons ab und leitet sie an `sendUrl` |
| Submit-Listener (Capture) | 49 | Dasselbe für Formulare (GET/POST); Dateiuploads ausgenommen |
| `findUrl`, `isSebUrl`, `sendUrl`, `showLinkStatus` | 79-146 | URL ermitteln, prüfen, an Hintergrund senden (`SEB_CAPTURE_URL`), Statusleiste |
| `initializePresentation` | 148 | Beim Laden jeder Seite den Hintergrund fragen, ob hier das Overlay erscheinen soll |
| `showStartup`, `applyPresentation`, `showSebBar`, `removePresentation`, `openQuitDialog` | 163-248 | Overlay-Frames anlegen/aktualisieren/entfernen |
| `handlePresentationMessage` | 250 | Nachrichten der Frames (Beenden, Reload, Audio) verarbeiten |
| `resetMoodleSebAccess` | 285 | Sendet Rücksetz-Anfrage an Moodle im Namen der angemeldeten Sitzung |

### 4.7 `seb/popup.js`

Ereignisbehandler für Datei-Öffnen (140), Datei-Auswahl (153), Verbinden/Trennen (172), Kopieren (192), XML speichern (198), Entladen (208), Einstellungen (213-231); `refresh` (235), `refreshSettings` (241), `saveDisplaySettings` (254, serialisiert Speichervorgänge über `displaySaveQueue`), `refreshBekSettings` (69), `render` (292, bildet Zustandsobjekt auf die Oberfläche ab), `formatMode`, `formatXml`.

### 4.8 `assistant/*` (nur Rolle)

| Funktion | Datei:Zeile | Zweck |
|---|---|---|
| `installAssistantCommands` | background:17 | Registriert das Tastenkürzel `answer-questions` |
| `answerTab` | background:23 | Einstieg: prüft aktiven Tab/URL, verhindert parallele Läufe pro Tab (`runningTabs`) |
| `runAssistant` | background:38 | Gesamtablauf eines Laufs (Details Abschnitt 5.4) |
| `saveRun`, `setBadge`, `questionFingerprint`, `withoutImageData` | background | Ergebnis speichern, Badge, Vergleich/Bereinigung |
| `askWithFallback` | providers:194 | Versucht aktivierte Konten der Reihe nach; sammelt Fehlerliste |
| `askProvider` | providers:216 | Eine Anfrage je Dienst-Familie (vier Varianten) |
| `fetchProviderModels`, `validateProviderAccount`, `normalizeProviderAccounts`, `newProviderAccount`, `providerDisplayName`, `modelImageCapability` | providers | Kontoverwaltung und Modellliste |
| `requestJson` | providers:417 | HTTP-Hilfsfunktion mit 45-Sekunden-Timeout |
| `getBaseUrl` | providers:385 | Endpunkt auflösen; entfernte Endpunkte nur über HTTPS erlaubt (Ausnahme: localhost) |
| `extractQuestions` / `applyAnswers` | extract:3 / apply:3 | Seite lesen / Seite beschreiben |

---

## 5. Zusammenspiel: die vier wichtigsten Abläufe (architektonische Ebene)

### 5.1 Konfiguration erkennen und laden („Laden")

```
Auslöser (einer von mehreren Wegen)
 ├─ Klick/Formular auf SEB-Link ......... content.js  → SEB_CAPTURE_URL ──┐
 ├─ HTTP-Antwort ist SEB-Konfiguration .. webRequest.onHeadersReceived ────┤
 ├─ Download ohne Endung (Firefox) ...... download-stream.js (Spiegelung) ─┤
 ├─ Blob-Download auf der Seite ......... download-page.js → download-content.js → SEB_CHECK_DOWNLOAD_BYTES
 ├─ Browser-Download-Ereignis ........... downloads.onCreated ────────────┤
 └─ Nutzer wählt Datei im Popup ......... file-import.js → SEB_PROCESS_BYTES
                                                                          ▼
                              background.js: processUrl / captureAutomaticBytes
                                                                          ▼
                                  processBytes (Warteschlange, serialisiert)
                                                                          ▼
                              replaceConfiguration:
                                1. parser.inspectSebFile  → Klartext-XML | "Passwort nötig" | Fehler
                                2. config-key: Konfigurations-Hash prüfbar?
                                3. unloadConfiguration (alter Zustand weg)
                                4. saveState({status:"ready"| "password-needed" | "error", …})
                                5. Badge setzen
                                6. wenn autoLaunch → activateLaunch (5.2)
```

Wichtige Eigenschaften:
- **[Fakt]** Alle Wege münden in `processBytes`; dadurch wird immer nur **eine** Konfiguration gleichzeitig ersetzt (Warteschlange `configurationQueue`, `background.js:435-440`).
- **[Fakt]** Doppelerkennungen werden mehrfach unterdrückt: Zeitfenster 8 s für eingebettete Daten (`duplicateRequests`), Schlüssel aus URL+Hash (`automaticCaptures`), Hash der aktuell geladenen Bytes (`loadedBytesDigest`). Kommentar im Code: A → B → A muss A wieder laden (`:197`).
- **[Fakt]** Chromium kann Antworten nur synchron beantworten und hat kein `filterResponseData`; deshalb gibt es dort Ersatzwege (Probe-Abruf mit Seiten-Cookies, Blob-Hook). Firefox darf asynchron antworten und spiegeln (`background.js:101-102`).

### 5.2 Start („Launch")

```
activateLaunch(xml, metadata)
 1. Konfigurations-Hash berechnen; Start-URL aus der Konfiguration (Fallback: URL der Seite)
 2. Darstellungseinstellungen aus Nutzereinstellungen + Konfiguration zusammenführen
 3. Zielreiter bestimmen (bevorzugt der auslösende, sonst aktiver, sonst erster Web-Tab)
 4. headers.enableHeaderInjection(...)       → ab jetzt wird jede Anfrage an den Prüfungs-Ursprung verändert
 5. SEB_UNLOAD_PRESENTATION an alle; presentationTabs = {Zielreiter}
 6. transitionToDestination:
      Animation an → pageTransitions[tab]={phase:"startup"} → Nachricht SEB_SHOW_STARTUP
                    → (Frame meldet Ende) → SEB_STARTUP_FINISHED → beginPendingReload → tabs.update(url)
      Animation aus (oder Seite hat noch kein Content-Script) → direkt tabs.update(url)
 7. Seite lädt neu → content.js:initializePresentation → SEB_GET_PAGE_PRESENTATION
      → applyPresentation (Taskleiste) → SEB_TRANSITION_FINISHED
 8. cmid aus Start-URL im Zustand ablegen (für Rücksetzung beim Entladen)
```

### 5.3 Beenden / Entladen

```
Nutzer klickt "Beenden" in der Taskleiste (iframe)
  → postMessage SEB_QUIT_REQUEST → content.js.openQuitDialog
  → SEB_GET_QUIT_REQUIREMENTS (Konfiguration erlaubt Beenden? Passwort nötig?)
  → Dialog (quit.html) → postMessage SEB_QUIT_SUBMIT → content.js → SEB_QUIT
  → background: Prüfungen (Konfiguration geladen? Tab gehört dazu? erlaubt? Passwort passt lokal?)
  → unloadConfiguration({closeTabs: Einstellung closeTabOnQuit})
       Moodle-Rücksetzung (SEB_RESET_MOODLE_ACCESS) → Header-Hook aus → Zustand "idle" → Overlay weg → Tab schließen/neuladen
```
**[Fakt]** Das Beenden-Passwort wird **lokal** gegen den in der Konfiguration enthaltenen Hash geprüft (`config-key.js:70-74`); es gibt keine Serverprüfung.

### 5.4 KI-Lauf (nur Phasenübersicht)

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
Dieser Block ist vom SEB-Block **unabhängig**: er prüft nicht, ob eine Konfiguration geladen ist (`assistant/background.js:38-94` enthält keinen Verweis auf den SEB-Zustand). **[Fakt]**

---

## 6. Kommunikationswege

### 6.1 Überblick

```
                          ┌────────────────────────────── Popup (popup.html) ──────────────┐
                          │ seb/popup.js          assistant/popup.js                       │
                          └───────┬──────────────────────────┬──────────────────────────────┘
          runtime.sendMessage     │                          │   storage.local (direkt lesen/schreiben:
          (SEB_*, OEB_AI_RUN)     │                          │   KI-Konten, Domains, Notiz, Kürzel)
                                  ▼                          ▼
 ┌──────────────────────── Background Page ─────────────────────────────┐
 │ seb/background.js  ── importiert ──►  headers, config-key, parser,    │
 │   (Router, Zustand)                    bek-catalog, download-*, …     │
 │        └── ruft ──►  assistant/background.js ──► providers.js ──► HTTPS zu KI-Diensten
 │   webRequest-Hooks (Antworten erkennen, Anfragen verändern)           │──► HTTPS zu Prüfungsserver (fetch mit Cookies)
 │   Katalog-Aktualisierung ─────────────────────────────────────────────┼──► HTTPS GitHub (bek.json)
 └───────▲──────────────────────────┬────────────────────────────────────┘
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
   (Seiten-Kontext)               (Erweiterungs-Seiten im iframe)
```

### 6.2 Nachrichtenkatalog (Erweiterungs-intern)

**Richtung „→ Hintergrund"** (`runtime.sendMessage`, beantwortet von `handleMessage`, `background.js:212-346`):

| Nachricht | Absender | Zweck |
|---|---|---|
| `SEB_CAPTURE_URL` | `seb/content.js` | Link/Formular auf Konfiguration abgefangen → herunterladen und laden |
| `SEB_CHECK_DOWNLOAD_BYTES` | `download-content.js` | Von der Seite übergebene Bytes auf SEB-Konfiguration prüfen |
| `SEB_PROCESS_BYTES` | Popup (`file-import.js`) | Vom Nutzer gewählte Datei laden |
| `SEB_UNLOCK` | Popup | Passwort für geschützte Datei |
| `SEB_GET_STATE`, `SEB_GET_SETTINGS`, `SEB_SET_SETTINGS` | Popup / Content | Zustand und Einstellungen lesen/schreiben |
| `SEB_GET_BEK_SETTINGS`, `SEB_SET_BEK_SETTINGS`, `SEB_REFRESH_BEK_CATALOG` | Popup | Schlüsselmodus und Katalog |
| `SEB_HEADER_ENABLE`, `SEB_HEADER_DISABLE` | Popup | „Exam connection" manuell verbinden/trennen |
| `SEB_UNLOAD` / `SEB_CLEAR` | Popup / – | Entladen mit Neuladen der Tabs / Entladen ohne Neuladen. **`SEB_CLEAR` wird im Code von keinem Absender verwendet** |
| `SEB_GET_PAGE_PRESENTATION`, `SEB_STARTUP_FINISHED`, `SEB_TRANSITION_FINISHED` | Content | Overlay-Steuerung |
| `SEB_GET_QUIT_REQUIREMENTS`, `SEB_QUIT` | Content | Beenden-Fluss |
| `OEB_AI_RUN` | Popup | KI-Lauf starten |

**Richtung „→ Content-Script"** (`tabs.sendMessage`): `SEB_SHOW_STARTUP`, `SEB_APPLY_PRESENTATION`, `SEB_UNLOAD_PRESENTATION`, `SEB_RESET_MOODLE_ACCESS`, `SEB_PROBE_DOWNLOAD_URL`, `OEB_AI_EXTRACT`, `OEB_AI_APPLY`. **[Fakt]** `SEB_*`-Nachrichten werden nur vom obersten Frame verarbeitet (`seb/content.js:9`).

**Seite ↔ Content-Script (`window.postMessage`)**, Kanal `oeb-download-capture-v1`: Nachrichtentypen `settings`, `ready`, `bytes`, `result`. **[Fakt]** Prüfung nur auf `event.source === window` (`download-page.js:25`, `download-content.js:17`).
**[Schluss]** Da die Seite selbst dieselbe Quelle ist, kann **beliebiges Seiten-JavaScript** diese Nachrichten senden. Folge: Eine Webseite könnte der Erweiterung Bytes einer Konfiguration „unterschieben" und so das Laden anstoßen (die Erweiterung validiert den Inhalt, aber nicht die Herkunft). Das ist eine Eigenschaft der Erweiterung, kein Fund gegen Dritte.

**Content-Script ↔ UI-Frames (`postMessage`)**: `SEB_STARTUP_COMPLETE` (Start-Animation → Content), `SEB_QUIT_REQUEST`, `SEB_RELOAD_REQUEST`, `SEB_AUDIO_TOGGLE` (Taskleiste → Content), `SEB_AUDIO_STATE` (Content → Taskleiste), `SEB_QUIT_SUBMIT`, `SEB_QUIT_CANCEL` (Beenden-Dialog → Content), `SEB_QUIT_ERROR` (Content → Dialog). **[Fakt]** Der Content-Script prüft die Quelle gegen `frame.contentWindow` (`content.js:184, 251-266`); die Frames prüfen `event.source === parent`.

**Event-Kanäle:** `storage.onChanged` hält Popup und Hintergrund synchron (`seb/popup.js:227-233`, `background.js:37-41`), `tabs.onRemoved` räumt Zustand auf (`background.js:54`), `commands.onCommand` für das KI-Kürzel, `webRequest.*` für Netzwerkereignisse.

### 6.3 Externe Netzwerkverbindungen der Erweiterung

1. **Prüfungsserver** (Hintergrund-`fetch` der Konfiguration mit Cookies, Rücksetz-Anfrage vom Content-Script, plus die vom Browser selbst gesendeten Seitenanfragen mit veränderten Headern).
2. **GitHub Raw** (`bek.json`, nur Profildaten).
3. **KI-Dienste** (siehe `OEB-Analyse-IT.md`, Abschnitt 4.1).
Keine eigene Telemetrie des Autors gefunden. **[Fakt]**

---

## 7. Variablen, Datenstrukturen und Zustände (architektonisch)

### 7.1 Persistenter Speicher (`browser.storage`)

| Schlüssel | Bereich | Inhalt (Struktur, ohne Werte) | Besitzer / Schreiber | Leser |
|---|---|---|---|---|
| `sebInspectorSettings` | local | `{checkDownloadsForSeb, closeTabOnQuit, showStartupAnimation, taskbarMode ("auto"/"show"/"hide"), taskbarScale (50-300)}` | Hintergrund (`saveSettings`) | Hintergrund, Popup, `download-content.js` |
| `oebBekSettings` | local | `{enabled, buildVersion, mode ("automatic"/"provided"), browserExamKey, schemaVersion:2}` | Hintergrund | Hintergrund |
| `oebBekCatalogCache` | local | `{catalog, checkedAt, error}` (Profilkatalog, zuletzt validiert) | `bek-catalog.js` | `bek-catalog.js`, Popup (Änderungsereignis) |
| `sebInspectorState` | **session** | Zustandsobjekt (7.2) | Hintergrund | Hintergrund, Popup |
| `sebInspectorPending` | **session** | `{bytes, metadata, mode}` für passwortgeschützte Dateien (Rohbytes der *verschlüsselten* Datei, nicht das Passwort) | Hintergrund | Hintergrund |
| `oebAiProviders` | local | Liste von KI-Konten (Dienst, Modell, **API-Schlüssel im Klartext**, Basis-URL, aktiviert) | `assistant/popup.js` | `assistant/background.js` |
| `oebAiEnabledDomains` | local | Karte `Hostname → true` | Popup | Assistant-Hintergrund |
| `oebAiNote` | local | Freitext-Zusatzanweisung | Popup | Assistant-Hintergrund |
| `oebAiLastRun` | local | Ergebnis des letzten Laufs: URL, Tab, Fragen (ohne Bilddaten), Antworten, Ergebnisstatus pro Frage, Anbieter, Fehlerliste, Zeit | Assistant-Hintergrund | Popup |
| `oebNextPageShortcut` | local | Text des Tastenkürzels | Popup | `assistant/content.js` |
| `oebPopupView` | `localStorage` des Popups | zuletzt gewählter Reiter | Popup | Popup |

**[Schluss]** `storage.session` ist ein flüchtiger Speicher (leert sich beim Browserende). `browser-api.js` ersetzt ihn durch `storage.local`, wenn er fehlt (`browser-api.js:3, 28`). **[Vermutung]** In Chromium mit Manifest V2 ist `storage.session` nicht verfügbar. Dann liegen Zustand (inkl. Klartext-XML der Konfiguration) und gesperrte Dateibytes **dauerhaft** im Profil. Das wäre ein weiteres forensisches Artefakt; auf einem Testgerät prüfen.

### 7.2 Das Zustandsobjekt (`sebInspectorState`)

Zustandsautomat der Konfiguration, geführt in `background.js`:

```
        ┌────────┐  Erkennung    ┌─────────────┐  Datei geladen  ┌───────┐
  Start │  idle  │──────────────►│ downloading │────────────────►│ ready │◄────────────┐
        └───▲────┘ (nur URL-Weg) └──────┬──────┘ (Klartext)      └───┬───┘             │
            │                          │ Fehler                      │ unload/quit     │ Passwort ok
            │ unloadConfiguration      ▼                             ▼                 │
            │                      ┌───────┐                      (zurück idle)   ┌────┴────────────┐
            └──────────────────────│ error │◄── Fehler in jedem Schritt           │ password-needed │
                                   └───────┘                                      └─────────────────┘
```
Felder: `status`, `mode` (Dateiformat), `xml` (Klartext der Konfiguration, nur bei `ready`), `sourceUrl`, `filename`, `detectedAt`, `message`, `errorCode`, `passwordError`, `cmid`. Das Popup erhält zusätzlich `headerDemo` (Status des Header-Moduls, `background.js:549`).

### 7.3 Laufzeit-Variablen im Hintergrund (gehen bei Neustart verloren)

| Variable | Typ | Zweck |
|---|---|---|
| `automaticDownloads` | bool | Kopie der Einstellung, damit synchrone Hooks ohne `await` entscheiden können |
| `settingsReady` | Promise | Hooks warten darauf, bis Einstellungen geladen sind |
| `presentationTabs` | `Set<tabId>` | Tabs, in denen die Prüfungsoberfläche gilt (gemeinsam genutzt von Overlay-, Beenden- und Rücksetzlogik) |
| `pageTransitions` | `Map<tabId, {phase, destination}>` | Übergangszustand „startup" → „reloading" |
| `configurationQueue` | Promise | Serialisierung der Konfigurationswechsel |
| `loadedBytesDigest` | string | Fingerabdruck der aktuell geladenen Bytes (Doppelerkennung) |
| `duplicateRequests`, `inFlightCaptures`, `inFlightProbes`, `automaticCaptures`, `responseCaptures`, `sebResponseRequests` | `Map`/`Set` | Entdoppelung und Merker laufender Anfragen |
| Modulvariablen in `headers.js`: `enabled`, `configHash`, `browserExamKey`, `buildVersion`, `examOrigin`, `bekMode`, `requestsWithBek`, `matchingTabs`, `lastError` | verschiedene | Zustand des Header-Hooks |
| `catalog`, `source`, `checkedAt`, `lastError`, `initializing`, `updating` | im `createBekCatalogManager`-Closure | Zustand der Katalogverwaltung |
| `runningTabs` | `Map<tabId, Promise>` | Verhindert parallele KI-Läufe pro Tab |

**[Schluss]** Weil die Header-Zustandsvariablen nur im Arbeitsspeicher der Hintergrundseite stehen, endet die Manipulation, sobald der Browser beendet oder die Erweiterung neu geladen wird, auch wenn `sebInspectorState` (bei `storage.local`-Ersatz) noch „ready" zeigt. Das Popup würde dann „Exam loaded" anzeigen, ohne dass Header gesetzt werden [Vermutung, nur relevant für Chromium-Variante].

### 7.4 Datenstrukturen zwischen Komponenten

- **Parse-Ergebnis** (`parser.js` → `background.js`): `{status:"ready", mode, xml}` oder `{status:"password-needed", mode, message}`; Fehler als `SebParseError{code, message}`. Codes u. a.: `INVALID_DATA_URL`, `TOO_MANY_LAYERS`, `INVALID_GZIP`, `INVALID_ENCRYPTED_DATA`, `UNSUPPORTED_ENCRYPTION`, `INVALID_PASSWORD`, `CERTIFICATE_REQUIRED`, `INVALID_SEB`, `BINARY_PLIST_UNSUPPORTED`, `INVALID_XML`.
- **Darstellungs-Einstellungen**: Nutzereinstellungen + `config.taskbar` (`visible, height, showClock, showKeyboard, showNetwork, showAudio, showReload, reloadWarning, showQuit`), abgeleitet aus der Konfiguration (`config-key.js:51`).
- **Frage-Liste** (`extract.js` → Hintergrund → `providers.js`): Liste von Objekten mit mindestens `number`, `type` und typabhängigen Feldern (Text, Optionen, Lücken, optionale Bilder).
- **Antwort-Liste** (`providers.js` → `apply.js`): `[{number, values:[…]}]`; Rückgabe von `apply.js`: `[{number, status:"filled"|"skipped", reason?}]`.
- **KI-Konto** (`providers.js`): Dienst-Schlüssel, Modellname, API-Schlüssel, optional Basis-URL, `enabled`, `id`.

### 7.5 Globale Konstanten (Auswahl)

Größenlimits (8 MB Downloads, 4 MB je Bild, 16 MB Bilder gesamt, max. 8 Bilder, 4 MB Katalog-JSON, 512 KB Profil-Prefix), Zeitgrenzen (30 s Konfigurations-Download, 3-5 s Probe, 45 s KI-Anfrage, 8 s Katalog-Abruf, 6 h Katalog-Intervall, 1 h Hintergrund-Timer), Overlay-IDs (`seb-local-*-frame`), Nachrichtenkanal `oeb-download-capture-v1`.

---

## 8. Abhängigkeiten zwischen Modulen

### 8.1 Import-Graph (nur `import`-Beziehungen, alle Hintergrund-/Popup-Module)

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
**[Fakt]** Keine Zirkelbezüge. Blätter (ohne eigene Abhängigkeit): `parser`, `config-key`, `bek-catalog`, `launch-response`, `settings`, `download-*`, `providers`, `file-import`, `bek-selection`.

### 8.2 Abhängigkeiten ohne `import` (Laufzeit-Kopplung)

| Kopplung | Art | Auswirkung einer Änderung |
|---|---|---|
| Nachrichtentypen (Strings) zwischen Popup/Content/Hintergrund | Implizit, nur durch Namensgleichheit | Umbenennen an einer Stelle bricht still, weil JS keine Typprüfung hat (kein Compilerfehler) |
| `OEBAssistant`-Globalobjekt | Reihenfolge der Content-Script-Liste im Manifest | `extract.js` und `apply.js` müssen **vor** `assistant/content.js` geladen werden (`*.manifest.json`, `js`-Liste) |
| Overlay-IDs `seb-local-*-frame` | Doppelt definiert (`content.js` und `content.css`) | Eine Änderung muss in beiden Dateien erfolgen |
| `popup.html` Element-IDs ↔ `popup.js`/`assistant/popup.js` | `querySelector("#…")` | Umbenennen bricht die Oberfläche |
| Storage-Schlüssel | Als Textkonstanten an mehreren Orten wiederholt (z. B. `sebInspectorSettings` auch in `download-content.js:18`, `oebAi*` in Hintergrund **und** Popup) | Jede Änderung muss alle Orte treffen |
| `web_accessible_resources` ↔ iframe-Ziele | Manifest | Fehlt ein Eintrag, laden Overlay-Frames nicht |
| Firefox-MAIN-Skript ↔ Chromium-Script-Tag | Zwei Wege zum selben Ziel (`firefox.manifest.json:55-65` vs. `download-content.js:56-76`) | Beide pflegen |
| `bek.json` ↔ `bek-catalog.js` Validierung ↔ `browser-exam-key.js` | Daten + Schema + Verbraucher | Schemaänderung betrifft alle drei und den Build |
| `build.mjs` ↔ Dateistruktur | Build kopiert den gesamten `source/`-Baum | Neue Dateien werden automatisch aufgenommen; Manifeste aber nicht (Listen manuell) |

### 8.3 Unabhängige Bereiche

- **Assistant-Block** ↔ **SEB-Block**: keine Importe, kein gemeinsamer Zustand; einzige Berührungspunkte sind `seb/background.js:15, 52, 214-215` (Einbindung und Nachricht `OEB_AI_RUN`) und das gemeinsame Popup.
- **Download-Erkennung** (`download-*`, `launch-response`) ↔ **Header-Modul**: unabhängig; verbunden nur über `processBytes`/Zustand.
- **Overlay-Oberfläche** (`assets/`) ↔ **Header-/Schlüsselmodul**: unabhängig.
- **`parser.js`**: reiner Datenbaustein, keine Browser-APIs.

### 8.4 Bereiche mit den meisten Abhängigkeiten

1. `seb/background.js` (Hub; 10 Importe, ≈ 25 Nachrichtentypen, die meisten Zustandsvariablen).
2. `seb/config-key.js` (von vier Modulen genutzt; enthält Parser **und** Hashing).
3. `popup.html` (Struktur für beide Popup-Module).

---

## 9. Initialisierung und Lebenszyklen

### 9.1 Start der Erweiterung (Hintergrund)

Reihenfolge in `seb/background.js` (beim Laden von `background.html`):
1. `browser-api.js` läuft zuerst (Kompatibilitätsschicht).
2. Module werden importiert; **beim Import** registriert `headers.js` bereits seinen `webRequest`-Hook (`headers.js:19`), der anfänglich inaktiv ist (`enabled = false`).
3. Katalog-Vorbereitung: `getBekCatalogInfo()` sofort und danach alle 60 Minuten (`:27-29`); Netzwerkprüfung höchstens alle 6 h.
4. `settingsReady` lädt Einstellungen asynchron (`:36`).
5. `installAssistantCommands()` registriert das KI-Tastenkürzel (`:52`).
6. Listener werden registriert: `tabs.onRemoved`, `runtime.onInstalled`, `runtime.onMessage`, `downloads.onCreated`, `webRequest.onHeadersReceived`, `onCompleted`/`onErrorOccurred`.
7. Nur bei Installation/Update: `onInstalled` schreibt Standard-Einstellungen und Zustand „idle".

### 9.2 Laden einer Seite (Content-Scripts)

Pro Seite und Frame bei `document_start`: Registrierung der Nachrichten-Listener, Klick-/Submit-Abfänger, Download-Brücke, Tastenkürzel-Listener (nur oberster Frame). Der oberste Frame fragt den Hintergrund via `SEB_GET_PAGE_PRESENTATION`, ob hier die Prüfungsoberfläche erscheinen soll (`content.js:313-316`).
**[Fakt]** Reiter, die **vor** dem Installieren/Neuladen der Erweiterung geöffnet wurden, haben keine Content-Scripts; Fehler wie „Receiving end does not exist" werden im Popup in den Hinweis „Reload this tab" übersetzt (`assistant/popup.js:546-550`). Das README rät ebenfalls, nach Updates Erweiterung und Seite neu zu laden.

### 9.3 Lebensdauer von Komponenten

| Komponente | Entsteht | Endet |
|---|---|---|
| Background Page | Browserstart / Erweiterung geladen | Browserende / Erweiterung entfernt; alle Laufzeitvariablen (7.3) gehen verloren |
| Header-Manipulation | `enableHeaderInjection` (Launch) | `disableHeaderInjection` (Entladen) oder Verlust der Background Page |
| Konfigurationszustand | `replaceConfiguration` | `unloadConfiguration`; ggf. Browserende (nur `session`-Speicher) |
| Overlay-Frames | Launch / Seiten-Neuladen im Prüfungs-Tab | Entladen, Tab-Schließen, `tabs.onRemoved` |
| Popup | Klick aufs Symbol | Schließen; **ein laufender KI-Lauf läuft im Hintergrund weiter** (Hinweistext in `assistant/popup.js:437`) |
| Katalog-Cache | erster erfolgreicher Abruf | bleibt dauerhaft in `storage.local` |

### 9.4 Totcode und Unstimmigkeiten (Fakten)

| Fund | Fundstelle |
|---|---|
| Ungenutzte Funktion `calculateFallbackBrowserExamKey` und Konstante `CERTIFICATE_THUMBPRINT` (nur dort verwendet) | `seb/browser-exam-key.js:5, 49-59` |
| Ungenutzte Funktionen `calculateRequestConfigKey` (async), `serializeSebConfiguration` | `seb/config-key.js:13, 76` |
| Nachricht `SEB_CLEAR` ohne Absender | `seb/background.js:340` |
| Popup sendet bei `SEB_HEADER_ENABLE` einen Parameter `configHash`, den der Hintergrund ignoriert (er rechnet selbst aus dem Zustand) | `seb/popup.js:179-180` vs. `background.js:329-334` |
| Bezeichnung „headerDemo" in Code und Oberfläche, obwohl es der echte Betriebsmodus ist | `background.js:549`, `popup.html:113-117` |
| Start-Animation zeigt Build 3.10.1.864, Standardprofil ist 3.10.2.920 | `assets/startup.js:1` vs. `browser-exam-key.js:6` |
| Versionsangabe 2.4.0 in Paket/Manifest, Ordner `OEB-2.4.1` | `package.json:3` |

---

## 10. Fehlerbehandlung, Fallbacks, Einschränkungen

### 10.1 Muster

| Muster | Beispiele |
|---|---|
| **Fehler werden in Zustand übersetzt, nicht geworfen.** Die meisten Fehler enden in `saveState({status:"error", message, errorCode})` plus rotem Badge `!`. Das Popup zeigt die Meldung. | `replaceConfiguration:479-490`, `processUrlOnce:420-431` |
| **Nachrichten-Router fängt alles:** `handleMessage`-Fehler werden als `{ok:false, error}` zurückgegeben (`background.js:65-70`). | — |
| **„Fail-open" bei Erkennung:** Schlägt eine Prüfung eines unbekannten Downloads fehl, bleibt der Download unberührt (Kommentar `download-check.js:2-3`). Leere `catch`-Blöcke bzw. `.catch(() => {})` kommen an rund 27 Stellen vor, überwiegend bei optionalen Schritten (Overlay, Rücksetzung, Probe). | `background.js:127, 167` u. a. |
| **Fallback-Ketten** | Firefox: Spiegelung → bei Fehler Probe-Abruf (`:111-128`). Probe: Abruf über den Tab mit Seiten-Cookies → bei Misserfolg Hintergrund-`fetch` (`:153-170`). Overlay: Start-Animation → bei fehlendem Content-Script direktes Neuladen (`:593-605`). Katalog: Cache → Paket-Datei (`bek-catalog.js:71-86`). KI: Konten der Reihe nach (`providers.js:194-214`). |
| **Zeitbegrenzungen** auf allen Netzwerkaufrufen (siehe 7.5). | — |
| **Größenbegrenzungen** gegen Speicherüberlauf. | — |
| **Validierung vor Übernahme:** Katalog nur nach Schema- und Prüfsummenkontrolle; ungültige Updates werden verworfen und der letzte gültige Katalog bleibt (`bek-catalog.js:100-115`). | — |
| **Header-Hook schluckt Fehler:** im Fehlerfall wird die Anfrage **unverändert** weitergeleitet und nur `lastError` gesetzt (`headers.js:34-37`). | — |
| **Vorher/Nachher-Prüfung beim KI-Lauf:** Abbruch, wenn sich URL oder Fragen während der Wartezeit geändert haben (`assistant/background.js:58-65`). | — |

### 10.2 Bekannte technische Einschränkungen (Fakten aus Code/README)

1. **Manifest V2 + blockierendes `webRequest`** ist Voraussetzung (README). **[Vermutung]** Aktuelle Chrome-Versionen haben die MV2-Unterstützung entfernt; nutzbar wäre dann nur Firefox oder ein Chromium-Derivat, das MV2 behält. Aktuellen Stand der eingesetzten Browser prüfen.
2. Zertifikat-verschlüsselte `.seb`-Dateien (Präfixe `pkhs`/`phsk`) und Apple-Binär-Plists werden **nicht** unterstützt (`parser.js:69-74, 92-96`).
3. Nur der Windows-x64-Profiltyp und genau **ein** Algorithmus werden vom Katalog akzeptiert (`bek-catalog.js:16`).
4. Der Schlüsselmodus „provided" benötigt einen vom Nutzer eingetragenen Wert (64 Hex-Zeichen).
5. **Keine** Betriebssystem-Sperre (README). Taskleiste und Dialog sind reine Web-Optik; Reload/Audio/Beenden wirken nur auf die Seite bzw. die Erweiterung (`content.js:250-283`).
6. Header-Änderung gilt nur für **einen** Ursprung (Origin) zur Zeit und nur nach geladener Konfiguration; wechselt die Seite zu einer anderen Origin, bleibt sie unverändert (`headers.js:21`).
7. Konfigurationen werden begrenzt auf 8 MB; mehr als 4 Gzip-Schichten werden abgelehnt.
8. Änderungen am Katalog wirken **nicht rückwirkend** auf die aktive Prüfung (Kommentar `bek-catalog.js:103-104`; README).
9. Der Plist-Parser ist handgeschrieben (`config-key.js:83-259`); **[Vermutung]** exotische Plist-Konstrukte könnten ihn zum Scheitern bringen; Folge wäre Fehlerzustand beim Laden.
10. Download-Erkennung ist browserabhängig: Spiegelung nur Firefox; Chromium nutzt Probe-Abruf (ein zweiter Abruf der Konfiguration, was bei einmal gültigen Download-Links scheitern kann, siehe README „Some downloads may need manual import").
11. Content-Scripts müssen nach Installation/Update per Seiten-Neuladen aktiviert werden.

### 10.3 Mögliche Fehlerquellen (aus Betreibersicht)

- Zustand `ready` im Popup bedeutet nicht zwingend, dass der Header-Hook noch aktiv ist (siehe 7.3).
- Mehrere Prüfungs-Tabs: `presentationTabs` wird bei jedem Launch **geleert** und nur der Zieltab gesetzt (`background.js:574-575`); gleichzeitige Prüfungen werden nicht unterstützt [Schluss].
- Pflege-Risiko durch implizite Kopplungen (8.2).

---

## 11. Build-Prozess und Entwicklungsumgebung

### 11.1 Voraussetzungen

| Punkt | Wert | Quelle |
|---|---|---|
| Sprache | JavaScript (ES2022+), ES-Module, HTML, CSS; kein TypeScript, kein Framework | gesamter Code |
| Laufzeit für Build | Node.js mit ESM, `top-level await`, `node:`-Präfix-Importe | `build.mjs`, `package.json:"type":"module"` |
| Paketmanager | npm (`package-lock.json` vorhanden → `npm ci`) | README |
| Entwicklungsabhängigkeiten | `terser 5.47.1` (JS-Minimierung), `lightningcss 1.31.1` (CSS-Minimierung), `fflate 0.8.2` (ZIP-Erzeugung) | `package.json` |
| Laufzeitabhängigkeiten der Erweiterung | **keine** Drittbibliotheken; nur Web-/Browser-APIs (WebCrypto, `DecompressionStream`, `fetch`, `webRequest`, …) | Quellcode |
| Zielbrowser | Firefox ≥ 140 (Android ≥ 142); Chromium ≥ 111 mit MV2-Unterstützung | Manifeste |
| Minimale Tools zum Entwickeln | Editor + Browser; kein Bundler, kein Dev-Server | — |

**[Fakt]** Mindest-Node-Version ist im Projekt nicht festgelegt (kein `engines`-Feld, keine `.nvmrc`).

### 11.2 Ablauf von `npm run build` (`build.mjs`)

1. Importiert `validateBekCatalog` direkt aus dem Quellcode der Erweiterung und prüft `bek.json` (Schema, Prüfsummen). Ein Fehler stoppt den Build (`:12-14`).
2. Stellt sicher, dass beide Manifest-Quelldateien existieren (`:16-20`).
3. Leert und legt `release/chrome` und `release/firefox` an.
4. Durchläuft `source/` rekursiv (`buildDirectory`, `:30-65`):
   - Manifestdateien werden übersprungen.
   - `.js`: Terser (Kompression + Variablennamen-Verkürzung, Kommentare „some" bleiben; **Modul-Modus wird per Regex erkannt**: Datei hat am Zeilenanfang `import`/`export`).
   - `.css`: `lightningcss` minimiert.
   - `.json`: neu serialisiert (Leerraum entfernt).
   - alle anderen Dateien (HTML, PNG, SVG): unverändert kopiert. **[Fakt]** HTML wird nicht minimiert.
5. Schreibt pro Browser `manifest.json` aus der jeweiligen Quelle (formatiert) und das geprüfte `bek.json` (`:67-75`). Zusätzliche Absicherung: Chrome-MV2-Manifest darf kein `world` in den Content-Scripts enthalten.
6. Erstellt ZIPs `release/OEB-<Version>-chrome.zip` und `…-firefox.zip` mit `fflate` (Stufe 9, `manifest.json` im Wurzelverzeichnis), Version aus `package.json` (`:78-95`).

### 11.3 Installation zu Testzwecken (laut README)

- **Firefox:** `about:debugging#/runtime/this-firefox` → „Temporäres Add-on laden" → `manifest.json` des entpackten Pakets; verschwindet beim Beenden von Firefox.
- **Chromium:** `chrome://extensions` → Entwicklermodus → „Entpackte Erweiterung laden".
- Nach Änderungen: Erweiterung neu laden **und** Prüfungsseite neu laden.

### 11.4 Hinweise zur Struktur des Builds

- `source/manifest.json` steht in `.gitignore`, wird vom Build übersprungen und nie verwendet: Altlast aus einem früheren Ein-Manifest-Stand [Vermutung].
- Der Build schreibt `bek.json` getrennt von `source/` (Datei liegt im Projektwurzelverzeichnis), damit sie gleichzeitig als Fern-Datenquelle im GitHub-Repo dienen kann.
- Ausgabe `release/` und `node_modules/` sind nicht versioniert.

### 11.5 Tests

**[Fakt]** Es existieren keine automatisierten Tests, keine Testframeworks, keine CI-Konfiguration im ZIP. Die Struktur (z. B. injizierbare Abhängigkeiten in `createBekCatalogManager`, `probeSebDownload(url, {fetchImpl, signal})`, reine Funktionen in `launch-response.js`, `settings.js`, `parser.js`) wäre für Tests geeignet **[Schluss]**, ist aber im Projekt nicht genutzt. Der Autor testet offenbar manuell (Vorschau-GIFs im Repo).

---

## 12. Wartbarkeit und Erweiterbarkeit (allgemein, ohne Umgehungsbezug)

**Bewertung [Schluss]:**

| Stärke | Schwäche |
|---|---|
| Klare Trennung in zwei Funktionsblöcke und viele kleine, abhängigkeitsarme Module | Zentraler Hub `seb/background.js` (748 Zeilen) vermischt Erkennung, Zustand, Routing und Overlay-Steuerung |
| Keine Drittbibliotheken zur Laufzeit; klein und überschaubar | Keine Tests, keine statische Typisierung; Nachrichtennamen und Speicherschlüssel sind verstreute Strings |
| Defensive Fehlerbehandlung, Größen- und Zeitlimits | Doppelte Wege für Firefox/Chromium; Totcode (9.4) |
| Datenkatalog vom Code getrennt | Mehrere Konstanten doppelt (IDs, Storage-Schlüssel, Versionsangaben) |

**Für Ihr Team relevant:** Falls die Erweiterung als Untersuchungsobjekt in einer Testumgebung zerlegt werden soll (z. B. um Erkennungsregeln zu validieren), ist die sicherste Vorgehensweise:
1. Nur **lesen und instrumentieren** (Protokollausgaben), nicht funktional ändern.
2. In einem **isolierten Testprofil** und gegen eine **Test-Moodle-Instanz** arbeiten, niemals gegen produktive Prüfungen.
3. Für Erkennungstests die Spuren aus `OEB-Analyse-IT.md`, Abschnitt 4, einzeln nachstellen und Server-/Netzwerk-Logs vergleichen.

Eine Anleitung zum Erweitern der Umgehungs- oder Autofill-Funktionen ist **nicht** Teil dieses Dokuments.

---

## 13. Einarbeitung in 30 Minuten: Lesereihenfolge

1. `OEB-Analyse-IT.md` Abschnitte 1-3 (was ist das, was kann es, was nicht).
2. Dieses Dokument: Abschnitte 1, 2, 6.1, 5.1-5.3 (Gesamtbild).
3. Quelldateien in dieser Reihenfolge: `chrome.manifest.json` (Berechtigungen, Content-Script-Liste) → `seb/background.js` ab Zeile 212 (`handleMessage`, Router) → `seb/launch-response.js` und `seb/parser.js` (klein, rein) → `seb/content.js` → `assistant/background.js` (kurz).
4. Erst danach bei Bedarf: `seb/headers.js`, `config-key.js`, `bek-catalog.js`, `download-*`.
5. Abschnitt 9.4 und 10.2 für bekannte Besonderheiten.

---

## 14. Selbstkontrolle dieser Dokumentation

| Frage | Ergebnis |
|---|---|
| Alle Dateien erfasst? | Ja: 55 Dateien im ZIP; alle Code-, Konfigurations- und Oberflächendateien sind im Inventar einzeln behandelt, der Rest sind Medien/Icons (`preview/*`, `assets/icons/*`, `seb/icons/*`), im Inventar als Gruppe genannt. |
| Alle Funktionen im Detail? | Bewusst **nicht**: Funktionsnamen und Rollen sind vollständig für `background.js`, `headers.js`, `content.js`, `parser.js`, `popup.js`; für `config-key.js`, `providers.js`, `extract.js`, `apply.js`, `browser-exam-key.js` nur auf Rollenebene (siehe Abgrenzung oben). |
| Zeilenangaben geprüft? | Stichprobenartig gegen den Quelltext; die Angaben stammen aus dem Lesen der Dateien mit Zeilennummern. Kleine Abweichungen bei anderer Version sind möglich. |
| Nicht untersucht | `seb/assets/startup.html` (554 Zeilen, überwiegend Markup/Stil und ein eingebetteter `text/plain`-Block), `popup.css` (914 Zeilen), Inhalte von `bek.json`, vollständige Typ-Handhabung in `extract.js`/`apply.js`. |
| Widersprüche zur Sicherheitsanalyse? | Fünf Präzisierungen (Abschnitt 0), in `OEB-Analyse-IT.md` eingearbeitet. |
| Dynamisch getestet? | Nein. Alle Aussagen zu Laufzeitverhalten sind aus dem Code abgeleitet. |
