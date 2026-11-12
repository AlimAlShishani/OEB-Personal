# Verifikationsbericht – Phase 2: Abgleich von `technical-handover.md` (und `OEB-Analyse-IT.md`) mit dem Quellcode

> **Status Phase 2.1 (2026-11-12):** Dieser Bericht ist das unveränderte Prüfprotokoll von Phase 2 und beschreibt den Stand **vor** der Bereinigung. DF-01 bis DF-13, BF-01 bis BF-04, TR-01 bis TR-10 und DS-01 bis DS-07 sind in `technical-handover.md` und `OEB-Analyse-IT.md` (Revision 2) berücksichtigt; Zeilenangaben und Aussagen dort sind maßgeblich. Eine Berichtigung betrifft DS-01 (Löschfunktion für KI-Konten ist vorhanden, siehe dort).

**Gegenstand:** Browser-Erweiterung „OEB — Open Exam Browser", ZIP `OEB-2.4.1` (interne Version `2.4.0`).
**Geprüfte Dokumente:** `technical-handover.md` (Repo-Stand, 659 Zeilen) und `OEB-Analyse-IT.md`.
**Methode:** reine statische Quellcode-Lektüre. Nichts wurde ausgeführt, gebaut oder installiert. Es gab keinen Netzwerkverkehr gegen Prüfungsplattformen. Der Quellcode und `technical-handover.md` wurden nicht verändert.
**Kennzeichnung:** **[Fakt]** im Code belegt (Datei:Zeile) · **[Schluss]** logische Folgerung aus dem Code, nicht getestet · **[Vermutung]** plausibel, nicht belegt · **[Offen]** mit den Mitteln dieser Phase nicht entscheidbar.
**Pfade:** relativ zu `OEB-2.4.1/source/`, sofern nicht anders angegeben.

---

## 0. Hinweise zum Prüfumfang

1. **Dokumentenstand.** Der Auftrag nennt `technical-handover.md` als Phase-1-Ergebnis. Hochgeladen war nur `OEB-Analyse-IT.md`. `technical-handover.md` habe ich aus dem Repo gelesen.
   - Die hochgeladene `OEB-Analyse-IT.md` ist **älter** als die Repo-Version. Die Korrekturen K1–K3 aus Handover §0 sind dort noch nicht eingearbeitet (`diff` bestätigt).
   - Ich habe beide Stände berücksichtigt. Soweit eine Aussage nur im älteren Stand steht, ist das vermerkt.
2. **Absichtlich nicht bewertet.** Die Rechenvorschriften der Header-/Schlüsselberechnung (Config Key, BEK, Profildaten, Prompt-Texte, Seitenstruktur-Selektoren) habe ich **nicht** inhaltlich bewertet. Ob sie gegenüber einer SEB-Spezifikation korrekt sind, bleibt ungeprüft.
   - Das wäre Qualitätssicherung für ein Umgehungswerkzeug und liegt außerhalb des Auftrags.
   - Geprüft wurde nur, **ob** die dokumentierten Dateien, Funktionen, Datenflüsse, Grenzen und Speicherorte existieren und stimmen.
   - `bek.json`: nur die Struktur (Schlüsselnamen, Anzahl Profile, `defaultVersion`), nicht die Werte.
3. **Lesetiefe.**
   - **Vollständig gelesen:** `seb/background.js`, `headers.js`, `config-key.js`, `parser.js`, `browser-exam-key.js`, `bek-catalog.js`, `bek-selection.js`, `settings.js`, `file-import.js`, `launch-response.js`, `download-*.js`, `seb/content.js`, `seb/content.css`, `seb/popup.js`, `seb/assets/{startup,taskbar,quit}.js`, `assistant/{background,content,extract,apply,providers}.js`, beide Manifeste, `background.html`, `browser-api.js`, `build.mjs`, `package.json`, `README.md`, `.gitignore`.
   - **Nur gezielt (Suche/Auszüge):** `assistant/popup.js` (600 Z.), `popup.html`, `seb/assets/startup.html`, `taskbar.html`, `quit.html`.
   - **Nicht gelesen:** `popup.css`, Bilder/GIFs, `package-lock.json` (nur Struktur geprüft).
4. **Webrecherche.** Eine Suche zur Verfügbarkeit von `runtime.getBrowserInfo` in Firefox-Content-Scripts blieb ohne belastbares Ergebnis. Der Punkt steht daher als **[Offen]** (UA-01).

---

## 1. Zusammenfassung

**Gesamturteil:** Die Dokumentation ist als **Architektur- und Orientierungsdokument zuverlässig**, als **exakte Spezifikation noch nicht**.

| Bereich | Zuverlässigkeit | Begründung |
|---|---|---|
| Dateiinventar, Zuständigkeiten, Modulgrenzen | **hoch** | 55/55 Dateien erfasst. Alle genannten Zeilenzahlen stimmen. Import-Graph, Nachrichtenkatalog, Speicherschlüssel und Netzwerkziele sind vollständig. |
| Funktionsnamen und Rollen | **hoch** | Keine genannte Funktion fehlt. Totcode-Liste (§9.4) vollständig bestätigt. |
| Korrekturen K1–K5 | **hoch** | K1, K2, K4, K5 voll bestätigt. K3 inhaltlich richtig, aber mit falscher Zeilenangabe. |
| Zeilenangaben | **mittel** | 11 von gut 80 geprüften Fundstellen weichen ab. Eine davon ist gravierend (`assistant/content.js:122-129` – die Datei hat nur 56 Zeilen). |
| Grenzen, Abläufe, Zustandsmodell | **mittel** | 4 inhaltliche Fehler/Ungenauigkeiten (DF-02 bis DF-05). |
| Erkennungsaussagen (`OEB-Analyse-IT.md` §4) | **mittel** | Code-Belege stimmen. Zwei Aussagen sind ungenau (DF-06, DF-07), eine ist nur eingeschränkt gültig (DF-12). Die Laufzeitwirkung bleibt unbestätigt, das war schon in Phase 1 so gekennzeichnet. |
| Datenschutz / Persistenz | **lückenhaft** | Die Hauptaussagen stimmen. Mindestens zwei datenschutzrelevante Speicherinhalte fehlen (DS-02, DS-03) und ein Netzwerkpfad (DS-05). |

**Wichtigste Erkenntnisse**

1. **Keine gravierenden Fehlbeschreibungen der Architektur.** Hub-and-Spoke um `seb/background.js`, zwei weitgehend getrennte Blöcke, Message-Router mit 20 Nachrichtentypen, 10 Imports, keine Zirkelbezüge. All das stimmt.
2. **Eine grobe Fundstellen-Abweichung** (`assistant/content.js:122-129` → richtig `17-24`) steht in Handover (K3) und in beiden Ständen der IT-Analyse (§2.2, §4.3). Das ist der einzige Fall, in dem eine Zeilenangabe ins Leere läuft.
3. **Fünf inhaltliche Korrekturen nötig:**
   - „≈ 25 Nachrichtentypen" → tatsächlich 20.
   - „Alle Wege münden in `processBytes`" → `unlockPending` und `unloadConfiguration` laufen an der Warteschlange vorbei.
   - „Konfigurationen sind auf 8 MB begrenzt" → gilt nur für die automatischen Erkennungswege.
   - Ein BEK „von Hand" ist per UI nicht eintragbar; vorhanden ist nur die Versionsauswahl.
   - „Katalog-Abruf beim Browserstart" gilt nur, wenn die letzte Prüfung über 6 h zurückliegt.
4. **Neue, in Phase 1 fehlende Befunde:**
   - Persistierte POST-Bodies im Pending-Datensatz (DS-02).
   - Dauerhaft gespeicherter KI-Lauf samt Bild-URLs (DS-03).
   - Fremdhost-Bildabrufe mit Cookies (DS-05).
   - Ein fester Header `X-OpenRouter-Title` (Erweiterungsname) bei OpenRouter-Anfragen (E-03). Das ist für die Erkennung relevant.
   - Auslösbarkeit der Konfigurationsübernahme durch beliebiges Seiten-JavaScript (TR-04). Die Kette ist statisch belegt.
5. **Nicht testbar ohne Laufzeitumgebung:** Browserkompatibilität (`storage.session`, `getBrowserInfo`, MV2-Verfügbarkeit), tatsächliche Header-Wirkung, Netzwerk-Timing und Moodle-Verhalten (Abschnitt 5).
6. **Keine automatisierten Tests, keine CI, keine Lizenzdatei** im ZIP (bestätigt).

---

## 2. Bestätigte Aussagen (nach Bereichen)

### 2.1 Inventar und Struktur – bestätigt

| Aussage (Handover) | Befund | Beleg |
|---|---|---|
| 55 Dateien im ZIP | **Bestätigt [Fakt]** | `find` = 55 |
| Manifest V2, `persistent: true`, Firefox ≥ 140 / Android ≥ 142, Chromium ≥ 111 | **Bestätigt** | `chrome.manifest.json:2,19-22,74`; `firefox.manifest.json:2,19-22,84,93` |
| Firefox-ID `oeb@open-exam-browser.local` | **Bestätigt** | `firefox.manifest.json:83` |
| Berechtigungen `downloads, storage, tabs, webRequest, webRequestBlocking, <all_urls>` | **Bestätigt**, keine weitere | beide Manifeste Z. 11-18 |
| `data_collection_permissions`: `websiteContent`, `authenticationInfo` | **Bestätigt** | `firefox.manifest.json:85-90` |
| Content-Scripts in allen Frames ab `document_start`; Firefox mit zusätzlichem `world: "MAIN"`-Block | **Bestätigt** | Chrome 39-58; Firefox 36-66 |
| `download-page.js` nur in Chrome-WAR | **Bestätigt** | `chrome.manifest.json:72`; fehlt in `firefox.manifest.json:67-80` |
| Alle genannten Zeilenzahlen der Module (z. B. `background.js` 748, `config-key.js` 337, `providers.js` 441, `popup.css` 914) | **Bestätigt** | `wc -l` |
| `build.mjs`-Ablauf: Katalogvalidierung, Terser/lightningcss, JSON-Neuserialisierung, HTML unverändert, Chrome-`world`-Prüfung, ZIP Stufe 9 | **Bestätigt** (Zeilenangaben siehe DF-08) | `build.mjs:17,31-66,71-74,80-95` |
| Terser-Modul-Modus per Regex `^(import\|export)\s` | **Bestätigt** | `build.mjs:50` |
| Dev-Abhängigkeiten `terser 5.47.1`, `lightningcss 1.31.1`, `fflate 0.8.2`; keine Laufzeit-Drittbibliotheken | **Bestätigt**; Lockfile v3 mit 25 Paketen, alle `dev` | `package.json:8-12`; `package-lock.json` |
| Kein `engines`-Feld, keine `.nvmrc`, keine Tests, keine CI | **Bestätigt (Negativbefund)** | Dateiliste; keine `.github/`, `test*/`, `*.test.*` |
| `.gitignore`-Inhalt | **Bestätigt** | `.gitignore:1-5` |
| `bek.json`: ≈ 59 KB, `schemaVersion 1`, `defaultVersion`, `profiles` | **Bestätigt** (58 931 Byte; 1 Profil; `defaultVersion` verweist darauf). Inhalt nicht bewertet. | `bek.json` |

### 2.2 Module, Funktionen, Zeilen – bestätigt

- **`seb/background.js`:** alle Funktions-Fundstellen in Handover §4.1 stimmen (59, 65, 72, 95, 109, 153, 176, 188, 212, 348, 359, 436, 442, 493, 540, 544, 553, 559, 586, 593, 607, 615, 639, 644, 654, 662, 669, 675, 683, 690-748).
  - Die Warteschlange liegt auf 435-440, die Doppelerkennungs-Kommentarzeile (A → B → A) auf 197.
  - Die Fallback-Ketten (111-128, 153-170, 593-605) stimmen.
- **`seb/headers.js`:** Hook 19-43, `enableHeaderInjection` 53, `disableHeaderInjection` 73, `reloadActiveHeaderTab` 85, `getHeaderStatus` 93, Hilfsfunktionen 106-121 stimmen.
  - Die 9 Modulvariablen aus §7.3 stimmen genau (Z. 9-17).
  - Fehler im Hook werden verschluckt und die Anfrage unverändert weitergegeben (34-37).
  - Die Beschränkung auf einen Ursprung (21) stimmt.
- **`seb/config-key.js`:** Funktionsliste und Zeilen (6, 13, 17, 21-68, 70, 76, 83-259, 261-337) stimmen.
- **`seb/browser-exam-key.js`:** 5, 6, 9, 23, 34, 49-59, 61 stimmen.
- **`seb/bek-catalog.js`:**
  - 9 (`validateBekCatalog`), 16 (nur `windows-x64` / ein Algorithmus), 64 (Manager), 71-86 (Cache → Paket), 100-115, 103-104 und 136-143 stimmen.
  - Grenzen: 4 MB JSON (5), 512 KB Prefix (26), 8 s Timeout (101), 6 h (4).
- **`seb/parser.js`:**
  - 19, 37, 83, 107 (max. 4 Gzip-Schichten), 127, 191, 212 stimmen; 69-74 (Zertifikat abgelehnt) und 92-96 (Apple-Binär-Plist) stimmen.
  - Alle 10 Fehlercodes aus §7.4 existieren, es gibt keine weiteren.
- **`seb/content.js`:** 34, 49, 79-146, 148, 163-248, 250, 285, 313-316; Quellenprüfung per `contentWindow` bei 184 und 251-266.
  - Dateiuploads sind von der Formularabfangung ausgenommen (58).
- **`seb/popup.js`:** 69, 140, 153, 172, 192, 198, 208, 213-231, 227-233, 235, 241, 254, 292 stimmen; `displaySaveQueue` bei 118/268.
- **Assistant:**
  - `assistant/background.js`: 17, 23, 38 (38-94), 44-46, 58-65, 83, 101-105.
  - `providers.js`: 194, 216, 385, 417; vier Anfragevarianten (`gemini`, `openai-responses`, `anthropic`, `openai`); 45 s Timeout; HTTPS-Pflicht außer localhost (394-398).
  - `extract.js`: 2, 3, 4-8, 89-96.
  - `apply.js`: 2, 3, 7-11, 29-47, 72, 80, 116-118, 174-177, 186-190.
  - Anbieter: 8 feste + 1 frei konfigurierbar. Domains: Gemini, Groq, OpenRouter, OpenAI, Anthropic, Mistral, Together, Cerebras + Custom.
- **Totcode (§9.4) vollständig bestätigt:**
  - `calculateFallbackBrowserExamKey` und `CERTIFICATE_THUMBPRINT` (`browser-exam-key.js:5,49-59`).
  - `calculateRequestConfigKey` (async, `config-key.js:13`) und `serializeSebConfiguration` (`:76`).
  - Nachricht `SEB_CLEAR` ohne Sender (`background.js:340`).
  - Parameter `configHash` bei `SEB_HEADER_ENABLE` wird vom Hintergrund ignoriert (`popup.js:179-180` vs. `background.js:329-334`).
  - Die Bezeichnung „headerDemo" (`background.js:549`, `popup.html:113-117`) bestätigt.
  - Eine Export-Analyse fand **keine weiteren** ungenutzten Exporte.

### 2.3 Korrekturen K1–K5 (Handover §0)

| # | Ergebnis | Anmerkung |
|---|---|---|
| K1 | **Bestätigt** | Rücksetz-Aufruf in `unloadConfiguration` (`background.js:615-623`), nur bei gesetztem `cmid` (`:581`, `:698-707`), auch beim Ersetzen (`:448`). |
| K2 | **Bestätigt** | Standardverhalten von Content-Scripts; Code fügt sichtbare DOM-Elemente ein (`content.js:168-173`). |
| K3 | **Inhaltlich bestätigt, Zeilenangabe falsch** | Der Listener liegt in `assistant/content.js:17-24` (Capture, `preventDefault` + `stopImmediatePropagation`, `click()`), nicht auf `122-129` (siehe DF-01). |
| K4 | **Bestätigt, und eine zweite Fundstelle** | `seb/assets/startup.js:1` = `3.10.1.864`. Dieselbe Konstante steht im inerten `text/plain`-Block `seb/assets/startup.html:243`. Das Standardprofil ist `3.10.2.920` (`browser-exam-key.js:6`). |
| K5 | **Bestätigt** | `oebAiLastRun`, `assistant/background.js:6,101-105`. Zur Vollständigkeit siehe DS-03. |

### 2.4 Datenflüsse, Speicher, externe Verbindungen – bestätigt

- **Nachrichtenkatalog:** Alle im Handover genannten Nachrichten haben die beschriebenen Absender/Empfänger. Es fehlt keine. Der Router hat **20** Typen (siehe DF-02).
- **Speicherschlüssel:** Alle 10 `storage`-Schlüssel plus `localStorage`-Schlüssel `oebPopupView` stimmen. Per `grep` wurden **keine weiteren** gefunden.
  - Bereich `session` für `sebInspectorState` und `sebInspectorPending` ist im Code so angelegt (`background.js:451,466,494,541,545`).
  - Der Fallback auf `storage.local` existiert (`browser-api.js:3,28`).
- **Externe Verbindungen:** vollständig – mit den Ergänzungen in DS-05 und E-03. Per `grep` nach `fetch(`, `XMLHttpRequest`, `sendBeacon`, `WebSocket`, `EventSource`, `new Image` und URL-Literalen:
  - Prüfungsserver (`background.js:411`; `download-content.js:37`; `download-page.js:42`; `content.js:295`).
  - GitHub Raw (`bek-catalog.js:2,100`).
  - KI-Anbieter (`providers.js:422`).
  - Bild-URLs der Seite (`extract.js:51`).
  - **Keine Telemetrie** zu Dritten außer diesen Zielen. **Kein** `eval`, `new Function`, `importScripts`, dynamisches Nachladen von Code. `innerHTML` wird nur gelesen (`content.js:290`).
- **Fehlerbehandlung:**
  - Fehler werden in Zustand übersetzt (`background.js:420-431,479-490`).
  - Der Router fängt alles (`:65-70`).
  - „Fail-open" bei Erkennung ist im Kommentar belegt (`download-check.js:1-2`).
  - Etwa 27 stille Fehlerblöcke: eigene Zählung 28 (musterbasiert, ±1).
  - Zeitlimits und Größenlimits stimmen: 30 s (`background.js:404`), 3 s (`download-check.js:79`), 5 s (`download-content.js:37`, `download-page.js:42`), 45 s, 8 s; 4 MB / 16 MB / 8 Bilder (`extract.js:4-6`).
- **Dependency-Injection** in `createBekCatalogManager` (`bek-catalog.js:64`) und in `probeSebDownload(url, {fetchImpl, signal})` – bestätigt, aber ungenutzt (keine Tests).
- **Das Beenden-Passwort wird nur lokal geprüft** (`config-key.js:70-74`; `background.js:323`) – bestätigt.

---

## 3. Widerlegte bzw. korrekturbedürftige Aussagen

### 3.1 Dokumentationsfehler

| ID | Aussage (Fundstelle im Dokument) | Befund im Code | Beleg | Konfidenz |
|---|---|---|---|---|
| **DF-01** | `assistant/content.js:122-129` (Handover §0 K3; IT-Analyse §2.2, §4.3) | Die Datei hat 56 Zeilen. Der Listener liegt bei **17-24**, der Message-Router bei 26-42. | `assistant/content.js:17-24` | [Fakt] |
| **DF-02** | „≈ 25 Nachrichtentypen" (Handover §8.4) | Der Router `handleMessage` kennt **20** `case`-Zweige. | `background.js:213-344` | [Fakt] |
| **DF-03** | „Alle Wege münden in `processBytes` … immer nur eine Konfiguration gleichzeitig" (§5.1) | `unlockPending` setzt Zustand und startet `activateLaunch` **ohne** Warteschlange. `unloadConfiguration` (über `SEB_UNLOAD`, `SEB_CLEAR`, `SEB_QUIT`) ist ebenfalls nicht eingereiht. Nur Ersetzen läuft serialisiert. | `background.js:493-538`, `:615-637`, `:257-258`, `:315-342` | [Fakt] |
| **DF-04** | „Konfigurationen werden begrenzt auf 8 MB" (§10.2 Nr. 7) | Das Limit gilt nur für automatische Erkennung: Bytes von der Seite, Probe, Spiegelung. **Nicht** für `SEB_PROCESS_BYTES`, nicht für den `fetch` in `processUrlOnce` (`response.arrayBuffer()` ohne Grenze) und nicht für die **entpackte** Größe in `parser.js`. | `background.js:231,246-252,411-414`; `download-check.js:3,69-76,79-84`; `parser.js:107-125` | [Fakt] |
| **DF-05** | Ein BEK kann „von Hand eingetragen werden" („provided") (IT-Analyse §2.3; Handover §10.2 Nr. 4, §7.1) | Das Popup bietet nur eine **Versionsauswahl** (Aus / Version). `bekSettingsForVersion` liefert immer `mode:"automatic"`. Ein gespeicherter „provided"-Wert wird beim Popup-Start **automatisch auf Standard migriert**. Der Codepfad existiert (`browser-exam-key.js:23-27,130-141`), ist per UI aber nicht erreichbar. | `popup.html:67-77`; `bek-selection.js:1-6`; `seb/popup.js:74-79`; README „Off option" | [Fakt] (UI) / [Schluss] (Unerreichbarkeit nur über Erweiterungs-Kontexte) |
| **DF-06** | Profil-Abruf erfolge „beim Browser-Start und alle ca. 6 Stunden" (IT-Analyse §4.1, Anhang A) | Geprüft wird beim Start **nur, wenn der persistierte Zeitstempel > 6 h alt ist** (`checkedAt` bleibt im Cache). Weitere Auslöser: stündlicher Timer, jeder Launch (`getBekProfile` → `get()` → `refresh()`), Öffnen des Popups, Button „Check for updates" (erzwingt). Auch ein **fehlgeschlagener** Versuch setzt den Zeitstempel (nächster Versuch frühestens nach 6 h). Eine Regel „Abruf bei jedem Start" kann Treffer verpassen. | `bek-catalog.js:78,96,98,110,114,121-125,138-140`; `background.js:27-29,263-270` | [Fakt] |
| **DF-07** | „Hohe Häufung von genau dieser Buildnummer (3.10.2.920)" als Indiz (IT-Analyse §4.5) | Im Netz erscheint **nur die dreiteilige Version** (z. B. `SEB/3.10.2`). Die vierteilige Build-Nummer geht ausschließlich in die lokale Berechnung ein. Die UA-Version folgt außerdem der **gewählten** Profilversion, ist also nicht fest `3.10.2`. | `headers.js:27,64`; `browser-exam-key.js:34-46` | [Fakt] |
| **DF-08** | Weitere abweichende Zeilenangaben | Siehe Tabelle 3.1a. | – | [Fakt] |
| **DF-09** | „Datei wird im Popup gewählt"; Popup-Diagramm nur über `runtime.sendMessage`; Popup „ohne dauerhaften Zustand" (§2, §5.1, §6.1) | Die Dateiauswahl läuft in einem **separaten Tab** `popup.html?file=1&target=<tabId>`. Das Assistant-Popup spricht den Content-Script **direkt** an (`tabs.sendMessage OEB_AI_EXTRACT`). Das Popup schreibt `oebPopupView` in `localStorage`. | `seb/popup.js:140-151`; `assistant/popup.js:413,109,553` | [Fakt] |
| **DF-10** | Taskleiste „(Uhr, Tastatursprache, Netz, Audio, Reload, Beenden)" (§3.3) | Zusätzlich **Akku-Anzeige** über `navigator.getBattery` (nur in Browsern mit dieser API). | `seb/assets/taskbar.js:44-57`; `taskbar.html` | [Fakt] |
| **DF-11** | `parser.js`: „keine Browser-APIs" (§8.3) | Gemeint sind WebExtension-APIs (stimmt). Der Parser nutzt aber Web-Plattform-APIs: `crypto.subtle`, `DecompressionStream`, `Blob`, `Response`. | `parser.js:117-118,154-189` | [Fakt] |
| **DF-12** | Konfig-`fetch` erfolge „mit normalem Browser-UA" (IT-Analyse §4.5, letzte Zeile) | Der Hook filtert nicht nach Tab oder Initiator, sondern nur nach Ursprung. Ist beim Laden einer **weiteren** Konfiguration noch eine Konfiguration für denselben Ursprung aktiv, trägt auch der Hintergrund-`fetch` die geänderten Header. Das Entladen passiert erst **nach** dem Abruf (`:448`). Beim Erstladen stimmt die Aussage. | `headers.js:19-21`; `background.js:411,448` | [Schluss], Laufzeittest nötig |
| **DF-13** | Versionsstand der IT-Analyse | Die hochgeladene `OEB-Analyse-IT.md` enthält K1–K3 noch nicht (siehe §0). | `diff` Repo vs. Upload | [Fakt] |

**3.1a Weitere abweichende Zeilenangaben (DF-08):**

| Dokument, Stelle | Angabe | tatsächlich |
|---|---|---|
| Handover §6.2 | `seb/content.js:9` (Top-Frame-Prüfung) | **11** (Definition `IS_TOP_FRAME` in Z. 6) |
| Handover §8.2 | `download-content.js:18` (`sebInspectorSettings`) | **11-12** |
| Handover §8.2 | `download-content.js:56-76` | **57-77** |
| Handover §9.3 | `assistant/popup.js:437` | **433** |
| Handover §10.1 | `download-check.js:2-3` | **1-2** |
| Handover §11.2 | `build.mjs:12-14` (Katalogprüfung) | **17** (Import Z. 6) |
| Handover §11.2 | `build.mjs:16-20` | **19-23** |
| Handover §11.2 | `build.mjs:30-65` | **31-66** |
| Handover §11.2 | `build.mjs:67-75` | **69-77** |
| IT-Analyse §4.3 | `download-page.js:13-21` | **14-22** |

Bewertung: Die Abweichungen sind überwiegend 1-4 Zeilen. Ein Nachvollziehen ist möglich. DF-01 ist der einzige Fall, in dem die Angabe ins Leere führt.

### 3.2 Bestätigte Fehler im Code (Inkonsistenzen)

| ID | Befund | Beleg |
|---|---|---|
| **BF-01** | Die Start-Animation zeigt `3.10.1.864`, das Standardprofil und die UA-Version sind `3.10.2(.920)`. Die Konstante steht doppelt (aktives `startup.js` und inerter Block in `startup.html`). | `seb/assets/startup.js:1`; `startup.html:243`; `browser-exam-key.js:6`; `headers.js:27` |
| **BF-02** | Versionsangabe `2.4.0` in `package.json`, beiden Manifesten und README, aber ZIP-Ordner `OEB-2.4.1`. | `package.json:3`; Manifeste Z. 5; `README.md:5` |
| **BF-03** | Totcode und tote Parameter (siehe 2.2), plus irreführende Bezeichnung „headerDemo" für den Echtbetrieb. | siehe 2.2 |
| **BF-04** | Eine UI-lose Funktion: der „provided"-BEK-Modus ist im Hintergrund implementiert, aber vom Popup abgeschaltet und wird automatisch migriert (siehe DF-05). | `seb/popup.js:74-79` |

---

## 4. Ungeklärte Annahmen, technische Risiken, Datenschutz-/Sicherheitsrisiken

### 4.1 Ungeklärte Annahmen

| ID | Annahme (Fundstelle) | Stand | Prüfung nötig |
|---|---|---|---|
| **UA-01** | Firefox-Erkennung über `runtime.getBrowserInfo` auch in **Content-Scripts** (`browser-api.js:2`, `download-content.js:59`; Handover §3.2, §8.2). | **[Offen]** Meine Recherche blieb ohne belastbares Ergebnis, ob diese API in Firefox-Content-Scripts verfügbar ist. Sollte sie dort fehlen, würde die Chromium-Fassade auch in Firefox installiert und die Chromium-Skript-Injektion zusätzlich ausgeführt. Die Firefox-WAR-Liste enthält `download-page.js` nicht, der Versuch würde am `error`-Handler scheitern (`download-content.js:65`). Wirkung vermutlich harmlos, nicht belegt. | Laufzeittest in Firefox (Testprofil) |
| **UA-02** | `storage.session` fehlt in Chromium-MV2; dann liegen Zustand und Pending dauerhaft im Profil (Handover §7.1). | **[Vermutung]** Fallback-Code vorhanden (`browser-api.js:28`). Laufzeit-Verfügbarkeit nicht geprüft. | Test in Chromium-Variante mit MV2 |
| **UA-03** | Aktuelle Chrome-Versionen haben MV2 entfernt (Handover §10.2 Nr. 1). | **[Vermutung]**, externe Tatsache, nicht verifiziert. | Herstellerdokumentation / Testlauf |
| **UA-04** | Begründungen für den eigenen Plist-Parser („kein DOMParser im Hintergrund") und die synchrone SHA-256 („Hook muss synchron antworten") (§3.3). | **Im Code nicht belegt.** Handover §5.1 räumt selbst ein, dass Firefox asynchron antworten darf. Die Sync-Notwendigkeit gilt belegbar nur für Chromium (`background.js:101-103`). | Entwicklerbefragung / Browser-Doku |
| **UA-05** | UA-Aufbau, Client-Hints, TLS-Fingerprint unterscheiden sich vom echten SEB (IT-Analyse §4.5). | **[Offen]** Es fehlt Referenzmaterial echter SEB-Clients. Code zeigt nur: Es werden nur `User-Agent` und die beiden SEB-Header ersetzt (`headers.js:25-33`). | Abgleich mit Logs echter Clients |
| **UA-06** | `isTrusted=false`-Ereignisse seien in Moodle beobachtbar (IT-Analyse §4.4). | Dass `dispatchEvent`/`click()` untrusted Events erzeugen, ist Standard-JS-Verhalten. Ob Moodle-Seiten das ohne Plugin beobachten können, ist **offen**. | Testinstanz |
| **UA-07** | Der handgeschriebene Plist-Parser scheitert an exotischen Konstrukten (§10.2 Nr. 9). | Teilbeleg: unbekannte Elemente werfen (`config-key.js:170-171`). Robustheit gegen reale Dateien unbekannt. | Synthetische Testdateien |
| **UA-08** | „Popup zeigt ‚Exam loaded', obwohl Header aus" nach Neustart (§7.3, §10.3). | Logisch folgerichtig, **nur relevant**, wenn `storage.session` auf `local` zurückfällt (UA-02). | Chromium-Test |

### 4.2 Technische Risiken (Erweiterung selbst)

| ID | Risiko | Beleg | Einordnung |
|---|---|---|---|
| **TR-01** | **Teilfehler in `activateLaunch`:** Schlägt ein Schritt nach `enableHeaderInjection` fehl (z. B. `tabs.update`), setzt `replaceConfiguration` den Zustand auf „error", ruft aber `disableHeaderInjection` nicht auf. Header bleiben aktiv, `presentationTabs` bleibt gesetzt. | `background.js:572-584`, `:479-490` | [Schluss] |
| **TR-02** | **Nicht serialisierte Zustandsänderungen** (siehe DF-03). Gleichzeitiges Passwort-Entsperren, Entladen und Ersetzen kann den Zustand überschreiben. | `background.js:493-538,615-637` | [Schluss], Race nicht getestet |
| **TR-03** | **Unbegrenzte Dekompression** (4 Gzip-Schichten ohne Größenlimit) auf allen Nicht-Auto-Pfaden. Eine manipulierte Datei kann den persistenten Hintergrundprozess speicherseitig überlasten. | `parser.js:107-125`; Auto-Pfade begrenzt durch `download-check.js:69-76,79-84` | [Schluss] |
| **TR-04** | **Seiten-JavaScript kann die Konfigurationsübernahme auslösen.** Handover §6.2 nennt das als [Schluss]. Die Kette ist jetzt statisch belegt: Die Brücke prüft nur `event.source === window` (`download-content.js:17`). Es folgt `SEB_CHECK_DOWNLOAD_BYTES` → Inhaltsprüfung → `captureAutomaticBytes` → `processBytes(autoLaunch: true)` (`background.js:229-245`, `188-206`) → `activateLaunch` (Zieltab = sendender Tab). Die automatische Erkennung ist standardmäßig **an** (`settings.js:2`). Die Herkunft wird nicht geprüft, der Inhalt schon. | siehe links | [Schluss] (Kette gelesen, nicht ausgeführt) |
| **TR-05** | **Frame-Zuordnung per DOM-ID:** `handlePresentationMessage` ermittelt Taskleisten- und Beenden-Frame über `getElementById` (`content.js:251-252`). Handover §6.2 nennt die Quellenprüfung per `contentWindow`, das ist richtig, aber die Referenz selbst stammt aus einer ID-Suche. Ob eine Seite damit das Entladen anstoßen kann, ist nicht getestet. | `content.js:251-253,268-281` | [Schluss], gering |
| **TR-06** | **Fehlerpfad kann selbst werfen:** `setBadge` im `catch` von `runAssistant` (z. B. Tab inzwischen geschlossen) bricht vor `saveRun` ab. Dann wird kein Fehlerlauf gespeichert. | `assistant/background.js:85-87,96-99` | [Schluss] |
| **TR-07** | **Fernaktualisierung ohne Authentizitätsprüfung:** Die „Prüfsummenvalidierung" des Katalogs (Handover §4.4) vergleicht Daten mit einer Prüfsumme **aus derselben Datei** (`bek-catalog.js:27-29`). Das schützt gegen Beschädigung, nicht gegen Manipulation. Quelle ist der `main`-Branch eines Drittrepos (`:2`). Es gibt keine Signatur, kein Pinning. | `bek-catalog.js:2,27-29,100-111` | [Fakt] (Mechanismus); Folgen [Schluss] |
| **TR-08** | **Keine Tests, keine CI, keine Lizenzdatei, kein Changelog.** Regressionen bei Browser-Updates sind nicht abgesichert. Das ist ein Wartungsrisiko. | Dateiliste | [Fakt] |
| **TR-09** | **Plattformabhängigkeit:** MV2 + blockierendes `webRequest` ist Voraussetzung. Für Chromium ist der Fortbestand unklar (UA-03). | README; Manifeste | [Offen] |
| **TR-10** | **`runtime.onInstalled` setzt bei jeder Installation/Aktualisierung den Zustand auf „idle"** und verwirft damit einen geladenen Prüfungszustand. | `background.js:59-63` | [Fakt] (Zustandsverlust ist dokumentierbar, bislang nicht dokumentiert) |

### 4.3 Datenschutz- und Sicherheitsrisiken

| ID | Befund | Beleg | Einordnung |
|---|---|---|---|
| **DS-01** | **API-Schlüssel im Klartext** in `storage.local` (`oebAiProviders`). Das Eingabefeld ist `type=password`. Einzelne Konten lassen sich über „Remove" + „Save" entfernen (`assistant/popup.js:183,359-372`). Es gibt **keine** Export- oder Verschlüsselungsfunktion und keine „alles löschen"-Funktion. *(Berichtigung Phase 2.1: Die ursprüngliche Aussage „keine Funktion zum Löschen" war falsch.)* Der Schlüssel geht bei jeder Abfrage an den Anbieter, auch bei Modelllisten-Abrufen. Beim Typ „Custom" geht er an einen **frei gewählten Host** (HTTPS Pflicht, außer localhost). | `assistant/popup.js:214-219,371`; `providers.js:142-183,385-400,402-415` | [Fakt] |
| **DS-02** | **Pending-Datensatz enthält mehr als dokumentiert.** Handover §7.1: „`{bytes, metadata, mode}`". `metadata` enthält u. a. `sourceUrl`, `pageUrl`, `tabId`, Dateiname **und bei abgefangenen Formularen `method` und `body`** (Formularfelder als URL-kodierter String). Gespeichert wird bei passwortgeschützten Dateien (`background.js:451-453`). Bei Fallback auf `local` wäre das persistent. | `seb/content.js:57-76`; `background.js:217-225,415-419,451-453` | [Fakt]; Inhalt von `body` je nach Seite [Schluss] |
| **DS-03** | **`oebAiLastRun` bleibt bis zum nächsten Lauf (und über Neustarts) erhalten.** Enthalten: URL, Fragetexte, Antworten, Anbieter, Fehler. Entfernt wird nur `dataUrl` (`assistant/background.js:9-15,78`). Die **Bild-`src`-URLs** bleiben im Datensatz (`extract.js:64`). Handover K5 sagt „ohne Bilddaten" – das ist korrekt, aber unvollständig. Kein Ablauf, kein Löschen. | `assistant/background.js:101-105`; `extract.js:61-67` | [Fakt] |
| **DS-04** | **Was an KI-Anbieter geht:** JSON mit Fragetext, Antwortoptionen, bis zu 8 Bilder als Base64, die Nutzernotiz im System-Prompt. Entfernt werden Fragenummern und Bild-`src`. OpenAI-Responses setzt `store:false`, die anderen Anbieter nicht (keine Aufbewahrungssteuerung im Code). Aufbewahrung beim Anbieter ist nicht dokumentierbar. | `providers.js:216-304`, `:252` | [Fakt] (Inhalt); Aufbewahrung [Offen] |
| **DS-05** | **Bildabruf mit Cookies von beliebigen Hosts:** `extract.js` holt Bild-URLs der Seite mit `credentials:'include'` (auch Fremdhosts, falls die Seite solche einbettet). Das steht nicht in der Verbindungsliste (Handover §6.3). | `extract.js:51` | [Fakt] |
| **DS-06** | **Entschlüsselte Konfiguration im Speicher:** `sebInspectorState.xml` enthält die komplette Klartext-Konfiguration (inkl. Hash des Beenden-Passworts). Das ist im Handover genannt. Zusätzlich zeigt das Popup sie an und bietet Kopieren/Speichern (`seb/popup.js:192-206`). | `background.js:467-475`; `popup.js` | [Fakt] |
| **DS-07** | **Breites Berechtigungsprofil** `<all_urls>` + Content-Scripts in allen Frames. Die Firefox-Deklaration `websiteContent` und `authenticationInfo` als „required" entspricht dem. | Manifeste | [Fakt] |

---

## 5. Ergänzende Befunde für die Erkennungsanalyse (`OEB-Analyse-IT.md`)

Diese Punkte stehen in Phase 1 nicht oder unvollständig. Sie sind **defensiv relevant** und weiterhin ungetestet.

| ID | Befund | Beleg |
|---|---|---|
| **E-01** | **Beobachtbarer `postMessage`-Verkehr auf jeder Seite:** Beim Laden sendet das Page-Script `{channel:"oeb-download-capture-v1", type:"ready"}`, die Brücke antwortet mit `type:"settings"`. Seiten-JavaScript kann diese `message`-Ereignisse mitlesen. Gilt in beiden Browsern. Die IT-Analyse nennt nur den Kanalnamen. | `download-page.js:125`; `download-content.js:5,18` |
| **E-02** | **Weitere DOM-/Protokoll-Spuren:** Bei Chromium wird `<script src="chrome-extension://…/seb/download-page.js">` kurz eingefügt und entfernt. Die Overlay-Frames tragen im `src` Pfad `seb/assets/…`. `content.css` ist auf jeder Seite aktiv (`#seb-local-*`-Regeln), ohne sichtbaren Effekt ohne Elemente. | `download-content.js:59-76`; `content.css:1-40`; Manifeste |
| **E-03** | **Fester Anfrage-Header bei OpenRouter:** `X-OpenRouter-Title: OEB — Open Exam Browser`. Bei Anthropic-Anfragen wird `anthropic-dangerous-direct-browser-access: true` gesetzt (nicht tool-spezifisch). Beides ist nur mit TLS-Inspektion oder auf Anbieterseite sichtbar. | `providers.js:22,402-415` |
| **E-04** | **Das Tastenkürzel „Nächste Seite" ist nicht domain-freigeschaltet**, anders als der KI-Lauf (`assistant/background.js:44-46`). Es greift auf jeder Seite, auf der das Moodle-Element existiert. | `assistant/content.js:17-24` |
| **E-05** | **Spuren auf dem Endgerät:** `oebAiLastRun`, `oebAiProviders`, `oebAiNote`, `oebAiEnabledDomains`, `oebBekCatalogCache`, `sebInspectorSettings` im Erweiterungsspeicher (`storage.local`). Pending/State je nach Browser in `session` oder `local`. Eine Liste der Speicherorte je Browser fehlt (siehe `documentation-gaps.md`). | Speicherschlüssel §2.4 |

---

## 6. Testbarkeit

„Statisch" heißt: durch Lesen des Quellcodes entscheidbar. „Laufzeittest" heißt: braucht eine isolierte Testumgebung (Testprofil, eigene Test-Moodle-Instanz, Netzwerkmitschnitt). **Nicht** gegen reale Prüfungsplattformen.

| Prüfpunkt | Statisch überprüfbar? | Laufzeittest notwendig? | Status |
|---|---|---|---|
| Dateiinventar, Zeilenzahlen, Abhängigkeitsgraph | Ja | Nein | **Bestätigt** |
| Manifest-Berechtigungen, Content-Script-Registrierung, Mindestversionen | Ja | Nein (Annahme der Mindestversion durch Browser: ja) | **Bestätigt** |
| Nachrichtenkatalog, Sender/Empfänger | Ja | Nein | **Bestätigt**, Zahl korrigiert (DF-02) |
| Speicherschlüssel, Struktur, Fallback-Code | Ja | Nein | **Bestätigt** |
| `storage.session` in Chromium-MV2 / Lebensdauer bei Reload und Neustart | Teilweise (Fallback-Code) | **Ja** | **Offen** (UA-02) |
| Firefox-Erkennung in Content-Scripts | Nein | **Ja** | **Offen** (UA-01) |
| MV2-Verfügbarkeit aktueller Chromium-Versionen | Nein | **Ja** | **Offen** (UA-03) |
| Header-Hook: Ursprungsfilter, welche Header geändert werden | Ja | Wirkung: **Ja** (Reihenfolge, Client-Hints, `extraHeaders`) | Code **bestätigt**, Wirkung **offen** |
| Hook erfasst auch den Konfig-`fetch` des Hintergrunds (DF-12) | Teilweise | **Ja** | **Offen** |
| Korrektheit der Schlüssel-/Hash-Berechnung gegenüber SEB | – | – | **Bewusst nicht bewertet** (Abschnitt 0) |
| Erkennungswege Firefox (Spiegelung) / Chromium (Probe) | Logik: Ja | **Ja** (Servers mit einmaligen Links, POST) | Logik **bestätigt** |
| Katalog-Abrufzeitpunkte | Logik: Ja | Netzmitschnitt zur Bestätigung | **Korrigiert** (DF-06) |
| Netzwerkziele und Telemetrie | Ja (`grep`) | Für Vollständigkeit: Netzmitschnitt in Testumgebung | **Statisch bestätigt** |
| DOM-Artefakte (IDs, Frames, Script-Tag) | Ja | Ja (Sichtbarkeit, Zeitpunkte je Browser) | Code **bestätigt** |
| `postMessage`-Marker (E-01) | Ja | Ja | **Neu**, statisch belegt |
| `isTrusted=false` bei Autofill und Beobachtbarkeit in Moodle | Standardverhalten: Ja | **Ja** (Testinstanz) | **Teilweise** (UA-06) |
| Kette „Seite löst Konfigurationsübernahme aus" (TR-04) | Kette gelesen | **Ja** | **Statisch belegt**, Wirkung offen |
| Race-Szenarien (TR-02) / Teilfehler (TR-01) | Teilweise | **Ja** (Fehlerinjektion) | **Offen** |
| Plist-Parser-Robustheit (UA-07) | Teilweise | **Ja** (synthetische Dateien) | **Offen** |
| Gzip-Grenzen (TR-03) | Ja | Optional | **Statisch bestätigt** |
| Moodle-Kompatibilität (Fragetypen, Versionen) | Nein | **Ja** (Test-Moodle) | **Offen**, nicht Teil dieser Phase |
| Build reproduzierbar (`npm ci`, `npm run build`) | Teilweise | **Ja** (nicht ausgeführt, fremder Code und Registry-Zugriff) | **Nicht geprüft** |
| Datenfluss an KI-Anbieter (Inhalt) | Ja | Netzmitschnitt mit Mock-Endpunkt | **Statisch bestätigt** |
| Aufbewahrung bei KI-Anbietern | Nein | Nein (Anbieterdokumentation) | **Nicht prüfbar** |
| Tests/CI vorhanden | Ja | Nein | **Bestätigt:** keine |

---

## 7. Notwendige weitere Prüfungen (Reihenfolge nach Nutzen)

Alle Schritte in **isolierter Testumgebung** (frisches Testprofil, eigene Test-Moodle-Instanz, Mock-KI-Endpunkt), **nicht** gegen produktive Prüfungsplattformen.

1. **Dokumentation korrigieren** (nur Text, kein Code): DF-01 bis DF-13 in `technical-handover.md` und `OEB-Analyse-IT.md` einarbeiten; DS-02, DS-03, DS-05, E-01 bis E-05 ergänzen.
2. **Browser-Matrix klären (UA-01 bis UA-03):** je Zielbrowser/Version prüfen, ob `storage.session`, `getBrowserInfo` (Content-Script), `filterResponseData`, `world: MAIN` und MV2 verfügbar sind. Ergebnis als Tabelle ins Handover.
3. **Netzwerkmitschnitt im Testlabor:** bestätigen, wann der Katalog-Abruf tatsächlich auftritt (DF-06), welche Header auf welchen Anfragen stehen (E-03, DF-12) und dass keine weiteren Ziele auftreten.
4. **Referenzabgleich für Erkennungsregeln (UA-05):** Logs echter SEB-Clients der eigenen Umgebung gegen die Erweiterung in der Testumgebung vergleichen. Das ist die belastbarste Prüfung der Phase-1-Erkennungsansätze.
5. **Fehlerinjektion (TR-01, TR-02):** `tabs.update` fehlschlagen lassen, parallel entsperren/entladen/ersetzen; erwartetes Zustandsmodell festhalten.
6. **Unit-Tests für die rein funktionalen Module** (`parser.js`, `launch-response.js`, `settings.js`, `download-check.js`, `bek-catalog.js`-Manager mit Dependency-Injection). Das ist die kostengünstigste Absicherung der Dokumentation selbst. Dazu synthetische Testdateien.
7. **Datenschutz-Bewertung:** Datenflussplan (DS-01 bis DS-05), Speicherorte je Browser, Bewertung für Meldekette und Datenschutzbeauftragte.
8. **Entscheidung über Tiefe der Spezifikation:** Für Erkennung und Gegenmaßnahmen reicht der jetzige Abstraktionsgrad. Für eine Spezifikation des internen Verhaltens fehlen die Punkte aus `documentation-gaps.md` (P1/P2).

---

## 8. Negativbefunde (ausdrücklich dokumentiert)

- Es existiert **keine** Datei, die das Handover nennt, aber im ZIP fehlt.
- Es existieren **keine** im ZIP vorhandenen Code-Dateien, die im Handover unerwähnt bleiben (die 55 Dateien sind erfasst).
- Es wurde **keine** Telemetrie, kein dynamisches Nachladen von Code, kein `eval`/`new Function` gefunden.
- Es wurde **kein** automatisches Absenden der Prüfung gefunden (keine `submit`/`requestSubmit`-Aufrufe). Das Tastenkürzel klickt nur das „Weiter"-Element (`assistant/content.js:17-24`). Das bestätigt die Aussage der IT-Analyse §2.4.
- Es wurde **kein** Schadcode im Sinne von Backdoor/Exfiltration über die dokumentierten Ziele hinaus gefunden. Das gilt nur für die gelesenen Dateien. Nicht gelesen: `popup.css`, Bilder.
- Die Aussage „Die Erweiterung erzwingt keine Betriebssystem-Sperre" ist im Code bestätigt (Taskleiste und Dialog sind reine Web-Optik; Reload/Audio wirken nur auf die Seite, `content.js:250-283`).
