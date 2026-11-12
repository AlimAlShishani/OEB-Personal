# Technische Analyse: Browser-Erweiterung „OEB — Open Exam Browser" (Paketversion 2.4.0, ZIP `OEB-2.4.1`)

**Stand:** 2026-11-12 · **Revision 2** (Dokumentationsbereinigung nach Phase-2-Verifikation)
**Zweck dieses Dokuments:** Information für das IT-Team zur Erkennung, Bewertung und Eindämmung der Nutzung dieser Erweiterung bei Online-Prüfungen (Moodle + Safe Exam Browser, SEB).
**Nicht Zweck:** Dieses Dokument ist keine Anleitung zum Nachbauen, Pflegen oder Verbessern der Erweiterung. Konstanten, Profildaten und Berechnungsdetails der Fälschungsfunktion sind bewusst weggelassen.
**Begleitdokumente:** `technical-handover.md` (Aufbau, Zustände, Datenschutz/Sicherheit im Detail, Änderungsprotokoll) · `verification-report.md` (Verifikation, Befund-IDs) · `documentation-gaps.md` (offene Informationen)

**Versionsbezeichnungen (einheitlich in beiden Dokumenten):** Das analysierte Archiv heißt `OEB-2.4.1`; die interne Paketversion lautet `2.4.0` (`package.json:3`, beide Manifeste Z. 5, `README.md:5`). Ob 2.4.1 inhaltliche Änderungen enthält, ist **OFFEN** (kein Changelog). Andere Versionen oder Forks können abweichende IDs, Frame-IDs und Hosts haben. Die im Code vorkommenden SEB-Build-Angaben sind in `technical-handover.md` 7.7 tabelliert.

**Kennzeichnung der Aussagen**

| Kennzeichnung | Bedeutung |
|---|---|
| **BESTÄTIGT** | Direkt durch statische Quellcodeanalyse belegt (Dateipfad/Zeile, bezogen auf `OEB-2.4.1/source/`). |
| **ABGELEITET** | Technisch plausible Schlussfolgerung, nicht zur Laufzeit geprüft. |
| **OFFEN** | Benötigt weitere Prüfung. |
| **NICHT GEPRÜFT** | Außerhalb des bisherigen Prüfungsumfangs. |

**Prüfstatus:** Reine statische Quellcode-Analyse. **Es wurden bisher keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt.** Die Erweiterung wurde nicht installiert oder ausgeführt, es wurde kein Netzwerkverkehr mitgeschnitten, und es gab keine Tests gegen Moodle oder eine Prüfungsplattform. Alle Erkennungsansätze unten sind daher vor dem produktiven Einsatz zu validieren.

---

## 1. Kurzfassung (Management-Summary)

1. **BESTÄTIGT** – OEB ist eine Browser-Erweiterung (Firefox ≥ 140 und Chromium mit Manifest V2), die sich gegenüber dem Prüfungsserver als echter Safe Exam Browser ausgibt. Das README nennt als Vorlage „SebBypass".
2. **BESTÄTIGT** – Zusätzlich enthält sie einen KI-Assistenten, der Moodle-Fragen auf der Seite ausliest, an externe KI-Dienste sendet und die Antworten in die Formularfelder einträgt.
3. **BESTÄTIGT** – Die Erweiterung erzwingt **keine** Betriebssystem-Sperre (README: „OEB does not enforce operating-system kiosk restrictions"). Es läuft ein normaler Browser auf einem normalen, uneingeschränkten System.
4. **ABGELEITET** – Eine Prüfung, die nur darauf vertraut, dass der Server die SEB-Header (Config Key / Browser Exam Key / User-Agent) bestätigt, ist gegen dieses Werkzeug **nicht belastbar geschützt**. Die Header werden für die geladene Konfiguration passend berechnet.
5. **ABGELEITET** – Es gibt mehrere **Spuren** (Netzwerk, Browser-Inventar, DOM, Ereignisse, Antwortverhalten, lokaler Speicher), über die sich der Einsatz erkennen oder erschweren lässt (Abschnitte 4–6). Keine davon ist bisher in einer Laufzeitumgebung bestätigt.
6. **BESTÄTIGT** – Die Erweiterung verarbeitet und speichert sensible Daten (API-Schlüssel, Prüfungsinhalte, Konfigurationsdaten) lokal im Klartext und sendet Prüfungsinhalte an externe Anbieter (Abschnitt 6).

---

## 2. Funktionsweise (Überblick)

### 2.1 Komponenten

| Komponente | Dateien | Aufgabe |
|---|---|---|
| Hintergrund-Skript (persistente Background-Page, MV2) | `background.html:6`, `seb/background.js` | Zentrale Steuerung: erkennt SEB-Konfigurationen, hält Zustand, aktiviert Header-Manipulation, steuert Overlay und KI-Lauf |
| Erkennung von Konfigurationsdateien | `seb/launch-response.js`, `seb/download-check.js`, `seb/download-stream.js`, `seb/download-content.js`, `seb/download-page.js` | Fängt `.seb`-Dateien, `seb://`/`sebs://`-Links und Downloads ohne Dateiendung ab (auch Blob-Downloads) |
| Parser | `seb/parser.js` | Entpackt/entschlüsselt `.seb`-Dateien (gzip, passwortgeschützt, RNCryptor). Zertifikat-verschlüsselte Dateien werden abgelehnt (`parser.js:69-74`) |
| Header-Manipulation | `seb/headers.js`, `seb/config-key.js`, `seb/browser-exam-key.js`, `seb/bek-catalog.js`, `bek.json` | Setzt SEB-typische Request-Header für den Prüfungsserver (2.3) |
| Optik/Täuschung | `seb/content.js`, `seb/assets/startup.*`, `taskbar.*`, `quit.*` | Blendet Start-Animation, SEB-ähnliche Taskleiste (mit Uhr, Sprache, Netz, Audio, Reload, **Batterieanzeige**, Beenden) und Beenden-Dialog ein |
| KI-Assistent | `assistant/background.js`, `providers.js`, `extract.js`, `apply.js`, `content.js`, `popup.js` | Fragen auslesen → KI-Anbieter abfragen → Antworten eintragen |
| Bedienoberfläche | `popup.html`, `popup.css`, `seb/popup.js`, `assistant/popup.js` | Popup; die Dateiauswahl öffnet dieselbe Seite in einem eigenen Tab (`seb/popup.js:140-151`) |
| Build | `build.mjs`, `package.json` | Erzeugt `release/chrome` und `release/firefox` als ZIP |

**BESTÄTIGT:** Es gibt **keinen eigenen Server** der Autoren und keine Telemetrie an die Autoren. Netzwerkziele außerhalb des Prüfungsservers: GitHub (Profil-Datei), die KI-Anbieter und die Hosts der in Fragen eingebetteten Bilder (Details 4.1 und 6).

### 2.2 Typischer Ablauf aus Sicht der nutzenden Person (BESTÄTIGT, aus dem Code gelesen)

1. Nutzer öffnet die Moodle-Prüfungsseite im normalen Browser mit installierter Erweiterung.
2. Klick auf „Launch SEB" bzw. Auswahl der `.seb`-Datei. Die Erweiterung fängt Klick/Download ab (`seb/content.js:34-47`, `seb/background.js:72-107`) und lädt die Konfiguration selbst nach (`processUrlOnce`, `seb/background.js:359-433`).
3. Die Konfiguration wird geparst, daraus werden Config-Key-Hash und Start-URL bestimmt (`replaceConfiguration`, `seb/background.js:442-491`; `activateLaunch`, `:559-584`).
4. Ab jetzt verändert die Erweiterung jede Anfrage an den Prüfungsserver (`seb/headers.js:19-43`).
5. Der Tab wird mit einer SEB-ähnlichen Start-Animation überlagert und danach auf die Prüfung geladen (`transitionToDestination`, `seb/background.js:593-605`); eine nachgebaute SEB-Taskleiste wird eingeblendet.
6. Optional: Tastenkürzel **Ctrl+Shift+Y** startet den KI-Lauf, **Ctrl+Shift+X** klickt auf „Nächste Seite" (`assistant/content.js:17-24`).
7. Beim „Beenden" wird die Konfiguration entladen und ggf. der Tab geschlossen (`unloadConfiguration`, `seb/background.js:615-637`).

### 2.3 Die Header-Fälschung (auf Konzeptebene)

- **BESTÄTIGT** – Nur für Anfragen an den **Ursprung (Origin) der Prüfungs-Start-URL** (`headers.js:21`) werden folgende Änderungen vorgenommen:
  - der `User-Agent` wird ersetzt durch `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) SEB/<x.y.z> (x64)` (`headers.js:4, 27`). `<x.y.z>` besteht aus den **ersten drei Teilen der gewählten Profilversion** (Standard: `3.10.2`); die vierteilige Build-Nummer (Standard-Profil `3.10.2.920`) erscheint **nicht** im Netzwerk (`headers.js:27, 64`);
  - der Header `X-SafeExamBrowser-ConfigKeyHash` wird gesetzt (`headers.js:5, 28`), berechnet aus dem Hash der geladenen Konfiguration und der jeweiligen URL;
  - optional der Header `X-SafeExamBrowser-RequestHash` (`headers.js:6, 29-32`), der Browser Exam Key (BEK).
- **BESTÄTIGT** – Der Config Key ergibt sich ausschließlich aus der `.seb`-Datei, die ohnehin der Prüfling/der Server kennt.
- **BESTÄTIGT** – Der BEK wird im Modus „automatic" **lokal berechnet** aus der Konfiguration plus buildspezifischen Profildaten (`browser-exam-key.js:34-46`; Profile in `bek.json`; mitgeliefert ist ein Profil für SEB 3.10.2.920 Windows x64). **Über die Oberfläche** wählbar sind nur „Off" oder eine Profilversion (`popup.html:67-77`; Versionsliste aus dem Katalog, `seb/popup.js:69-107`). Ein manuell eingetragener BEK („provided") ist im Hintergrund-Code vorhanden (`browser-exam-key.js:23-30`), aber **nicht** über die Oberfläche erreichbar; gespeicherte „provided"-Werte werden beim Öffnen des Popups auf den Standard migriert (`seb/popup.js:74-79`). **OFFEN:** Ob der Pfad auf anderem Weg erreichbar ist.
- **ABGELEITET** – Der BEK ist damit **kein Nachweis**, dass tatsächlich ein unveränderter SEB-Client läuft, sofern die Profildaten des betreffenden Builds bekannt sind. Eine reine „BEK/Config-Key in Moodle erlauben"-Prüfung kann mit denselben Eingaben erfüllt werden. (Die Korrektheit der Berechnung gegenüber SEB wurde **nicht** geprüft: NICHT GEPRÜFT, bewusst ausgeklammert.)
- **BESTÄTIGT** – Bei **jedem Entladen** der Konfiguration (Beenden, „Close exam", Popup-„Unload", Ersetzen durch eine neue Konfiguration) ruft die Erweiterung auf der Moodle-Seite per POST `/mod/quiz/accessrule/seb/clear_session.php` auf (`seb/content.js:285-306`, ausgelöst aus `seb/background.js:615-623`), sofern aus der Start-URL eine Quiz-ID (`cmid`) ermittelt werden konnte.
- **BESTÄTIGT** – Die Start-Animation zeigt fest Build `3.10.1.864` / Version `3.10.1`, das Standardprofil ist `3.10.2.920`. Das ist ein interner Widerspruch der Erweiterung (BF-01); für die Erkennung ist nur der dreiteilige UA-Token relevant.

### 2.4 KI-Assistent

- **Auslesen (BESTÄTIGT):** `assistant/extract.js` liest alle `.que`-Elemente (Moodle-Fragen), erkennt den Fragetyp (Multiple Choice, Kurzantwort, Zuordnung, Lückentext, Drag&Drop, Marker u. a., `extract.js:89-96`) und ruft auch Bilder der Frage ab (bis 8 Bilder, max. 4 MB je Bild, 16 MB gesamt, `extract.js:4-6`).
- **Abfragen (BESTÄTIGT):** `assistant/providers.js` / `assistant/background.js:57`: Fragen gehen an den konfigurierten KI-Anbieter, bei Fehlern an das nächste aktivierte Konto (`askWithFallback`). Der Nutzer kann einen Freitext-Hinweis („Note") mitgeben.
- **Eintragen (BESTÄTIGT):** `assistant/apply.js` setzt Auswahlfelder/Textfelder, klickt Radio-/Checkboxen, bedient Drag&Drop per simulierter Pfeiltaste und füllt Rich-Text-Felder (`apply.js:3-202`).
- **Sicherungen der Autoren (BESTÄTIGT):** Eintrag nur auf Domains, die im Popup freigeschaltet wurden (`assistant/background.js:44-46`); Abbruch, wenn sich Seite/Fragen während der KI-Antwort ändern (`:58-65`); Hinweistext „Review before submitting" (`:83`). Das Absenden der Prüfung erfolgt **nicht** automatisch (keine `submit`-Aufrufe im Quellcode); die Navigation „Weiter" erfolgt nur per Tastenkürzel. Das Kürzel „Weiter" ist nicht an die Domain-Freigabe gebunden (`assistant/content.js:17-24`).

---

## 3. Bewertung: Was ist mit der Erweiterung möglich, was nicht?

| Aspekt | Einschätzung |
|---|---|
| Server-seitige SEB-Prüfung per Header (Config Key / BEK / User-Agent) | **ABGELEITET** – umgehbar, solange der Build bekannt ist, den die Erweiterung nachahmt |
| Betriebssystem-Sperre (Kiosk, Task-Wechsel, Bildschirmaufnahme-Blockade, Prozessüberwachung) | **BESTÄTIGT** – wird nicht geleistet, also nicht umgangen, sondern schlicht nicht vorhanden. Wer eine echte Sperre benötigt, braucht Geräte-/Netzwerkkontrolle (Abschnitt 5) |
| Datenabfluss | **BESTÄTIGT** – Fragetext, Antwortoptionen, Bilder und Nutzer-Hinweis gehen an die gewählten KI-Anbieter (Abschnitt 6). API-Schlüssel liegen lokal im Browser-Storage |
| Selbst-Update | **BESTÄTIGT** – Es wird nur eine JSON-Profil-Datei nachgeladen, kein Code (`seb/bek-catalog.js:2-4, 97-103`); Abrufbedingungen in 4.1 |
| Telemetrie an die Autoren | **BESTÄTIGT** – im Code nicht gefunden (nur die Profil-Abfrage bei GitHub) |

---

## 4. Erkennungsmöglichkeiten

Alle Signale sind **BESTÄTIGT** nur im Sinn „im Code vorhanden". Ihre Sichtbarkeit und Zuverlässigkeit in einer realen Umgebung ist **NICHT GEPRÜFT**.

### 4.1 Netzwerk / Proxy / DNS (am einfachsten, falls Firmenproxy vorhanden)

| Signal | Beleg | Aussagekraft |
|---|---|---|
| Abruf von `https://raw.githubusercontent.com/63kitsune/OEB/refs/heads/main/bek.json` | **BESTÄTIGT** (Code) `seb/bek-catalog.js:2, 4, 96-101`. **Auslöser und Häufigkeit:** höchstens alle 6 h, wobei der Zeitpunkt der letzten Prüfung gespeichert wird (überdauert Neustarts); Auslöser sind Start der Hintergrundseite, ein stündlicher Timer, jeder Launch, Öffnen des Popups; erzwungen durch „Check for updates". Auch fehlgeschlagene Versuche setzen den Zeitpunkt. Ohne Cache (Erststart) sofort. Anfrage: GET, ohne Cookies | **Sehr spezifisch** wegen des Pfads, aber **unregelmäßig**: ein Abruf „bei jedem Browserstart" ist nicht garantiert. Nur sichtbar, wenn HTTPS-Pfade geloggt werden (TLS-Inspektion); sonst nur der Host `raw.githubusercontent.com`, der allein nicht aussagekräftig ist. Netzwerkverhalten **NICHT GEPRÜFT** |
| Verbindungen zu KI-Anbietern, ausgelöst während einer Prüfung | **BESTÄTIGT** (Code) `assistant/providers.js`: `generativelanguage.googleapis.com`, `api.groq.com`, `openrouter.ai`, `api.openai.com`, `api.anthropic.com`, `api.mistral.ai`, `api.together.xyz`, `api.cerebras.ai`, zusätzlich ein frei konfigurierbarer „Custom OpenAI-compatible"-Endpunkt. Auch Modelllisten-Abrufe im Popup nutzen diese Hosts (`:142-183`) | **Mittel**: Hinweis auf KI-Nutzung, aber kein Beweis für diese Erweiterung. Zeitliche Korrelation mit Prüfungsfenstern und Prüfungs-IP wertvoll |
| Der „Custom"-Anbieter | **BESTÄTIGT** (Code) `providers.js:59-67, 385-400` | Blocklisten nach Domain greifen hier nicht vollständig; ein eigener Endpunkt lässt sich nicht vorab erfassen |
| Fester Zusatz-Header bei OpenRouter-Anfragen: `X-OpenRouter-Title: OEB — Open Exam Browser`; bei Anthropic-Anfragen `anthropic-dangerous-direct-browser-access: true` | **BESTÄTIGT** (Code) `providers.js:22, 404-406, 413` | OpenRouter-Header **werkzeugspezifisch**, aber nur mit TLS-Inspektion oder auf Anbieterseite sichtbar. Der Anthropic-Header ist nicht werkzeugspezifisch. **NICHT GEPRÜFT** |
| Abrufe der Bild-URLs einer Frage durch die Erweiterung (`fetch` mit Cookies, bis zu 8 Bilder) | **BESTÄTIGT** (Code) `assistant/extract.js:46-51` | **OFFEN**: ob sich diese Abrufe in Server-/Proxy-Logs von normalen Bildladevorgängen unterscheiden lassen (zusätzliche Abrufe derselben Bilder; Hosts der Bilder) |

**Empfehlung (aus allgemeinem Fachwissen, nicht aus dem Code):** Für Prüfungszeiträume KI-API-Domains auf dem Firmen-/Hochschulnetz blockieren oder zumindest protokollieren (nur wirksam im eigenen Netz; auf privaten Geräten/Netzen nicht).

### 4.2 Endgeräte-Inventar (wenn verwaltete Geräte)

- **BESTÄTIGT** – Firefox-Erweiterungs-ID: `oeb@open-exam-browser.local` (`firefox.manifest.json:83`); Anzeigename in beiden Browsern: „OEB — Open Exam Browser".
- **BESTÄTIGT** – Angeforderte Berechtigungen: `downloads`, `storage`, `tabs`, `webRequest`, `webRequestBlocking`, `<all_urls>`, sowie Content-Scripts auf allen Seiten in allen Frames (`*.manifest.json`). Firefox: `data_collection_permissions` verlangt `websiteContent` und `authenticationInfo`.
- **ABGELEITET** – Eine Abfrage installierter Erweiterungen (z. B. über Browser-Management/MDM oder Firefox-Profilordner) kann die ID bzw. den Namen finden. Die Erweiterung wird laut README meist als **temporäres Add-on** (about:debugging) bzw. **entpackt** (Chromium „Entpackte Erweiterung laden") installiert und erscheint daher ggf. nicht in zentralen Store-Listen.
- **OFFEN** – Richtlinien, die „Entwicklermodus"/„Temporäre Add-ons" unterbinden, sollten wirksam sein; konkrete Richtlinien-Namen bitte in der Herstellerdokumentation der eingesetzten Browser prüfen. Welche Browserversionen MV2 unterstützen, ist **NICHT GEPRÜFT**.

### 4.3 Seiten-seitige Spuren im DOM und in Seitenereignissen

Die Content-Scripts laufen in jedem Frame jeder Seite und hinterlassen Artefakte in der Moodle-Seite:

| Artefakt | Beleg | Bedingung |
|---|---|---|
| `<iframe id="seb-local-startup-frame">`, `id="seb-local-taskbar-frame"`, `id="seb-local-quit-frame"` | **BESTÄTIGT** `seb/content.js:3-5` | Wenn Start-Animation/Taskleiste aktiv; direkt unter `document.documentElement` angehängt |
| `src` dieser Frames beginnt mit `moz-extension://…` bzw. `chrome-extension://…` und enthält `seb/assets/` | **BESTÄTIGT** `seb/content.js:171, 221, 246` | Seiten-JavaScript kann `iframe.src` lesen |
| `<div id="oeb-seb-link-status">` | **BESTÄTIGT** `seb/content.js:132` | Nur bei Statusmeldung |
| Element mit Klasse `oeb-ai-marker` (in Marker-Fragen) | **BESTÄTIGT** `assistant/apply.js:80` | Nur bei Marker-Fragetyp nach KI-Eintrag |
| Zusätzliches Page-Script im „MAIN world", überschreibt `URL.createObjectURL`, `URL.revokeObjectURL` (`seb/download-page.js:14-22`) und `HTMLAnchorElement.prototype.click` (`:92`). Firefox: Content-Script mit `world: "MAIN"` (`firefox.manifest.json:55-65`); Chromium: kurz eingefügtes `<script src="chrome-extension://…/seb/download-page.js">`, beim Laden wieder entfernt (`seb/download-content.js:57-77`, Erzeugung `:61-66`) | **BESTÄTIGT** (Code) | Auf manipulierte Prototypen prüfbar (z. B. `URL.createObjectURL.toString()` enthält kein `[native code]`) – **ABGELEITET** |
| **`window.postMessage`-Verkehr** mit `{channel: "oeb-download-capture-v1", type: "ready"}` (Page-Script, beim Laden) und `type: "settings"` (Brücke): Seiten-JavaScript kann diese `message`-Ereignisse mitlesen | **BESTÄTIGT** `download-page.js:125`, `download-content.js:5, 18` | Auf jeder Seite, in beiden Browsern. Sichtbarkeit in der Praxis **NICHT GEPRÜFT** |
| Tastenkombination Ctrl+Shift+X (Standard): `keydown`-Listener in der Capture-Phase; ist ein aktiver „Weiter"-Button vorhanden, wird das Ereignis verschluckt und der Button per `click()` ausgelöst | **BESTÄTIGT** `assistant/content.js:17-24` | nur oberster Frame; **keine** Domain-Freigabe nötig |
| Hinweis: Content-Scripts teilen das **DOM** mit der Seite. Von der Erweiterung eingefügte Elemente sind für Seiten-JavaScript sichtbar; nur die JS-Objekte der Erweiterung sind getrennt | **ABGELEITET** | – |

**ABGELEITET:** Die Artefakte sind für jemanden mit Quellcode-Zugriff leicht änderbar (IDs umbenennen). Sie taugen als **Hinweis**, nicht als Beweis oder alleinige Schutzmaßnahme. Ein Fehlen der Artefakte bedeutet nicht, dass die Erweiterung nicht aktiv ist.

### 4.4 Verhaltensspuren bei Eingaben (Moodle-seitig oder Plugin)

- **BESTÄTIGT** – Antworten werden **programmatisch** gesetzt: `el.value = …` und synthetische `input`/`change`-Ereignisse (`assistant/apply.js:7-11`), `input.click()` für Radio/Checkbox (`:116-118`), synthetische `KeyboardEvent('keydown', ArrowDown)` für Drag&Drop-Lücken im Abstand von 350 ms (`:174-177`), Direktschreiben in Rich-Text-Felder (`:29-47`).
- **ABGELEITET** – Solche Ereignisse haben `event.isTrusted === false` (Standardverhalten von `dispatchEvent`/`click()`). Ein Skript auf der Moodle-Seite, das Ereignisse an den Antwortfeldern beobachtet, könnte Eingaben **ohne echte Tastatur-/Mausereignisse** erkennen. Das gilt auch für Eingaben anderer Automatisierungswerkzeuge. Auf Falschmeldungen achten (z. B. Passwortmanager, Barrierefreiheits-Hilfsmittel). **OFFEN:** Ob das in der eingesetzten Moodle-Version ohne eigenes Plugin umsetzbar ist.
- **ABGELEITET** – In den Moodle-Protokollen/Frageversuchs-Daten typisch: viele Fragen **innerhalb weniger Sekunden** beantwortet, sehr gleichmäßige Abstände, Antworten für lange Freitexte ohne Tippdauer (Freitext-Felder werden in einem Schritt gesetzt, `apply.js:186-190`), und die Antwort auf eine Seite entsteht in einem Block, bevor ein „Weiter" erfolgt.
- **ABGELEITET** – Das Tastenkürzel „Nächste Seite" erzeugt einen regulären `click()`, also einen untrusted Click.

### 4.5 Server-/Request-seitige Spuren (Reverse-Proxy- oder Moodle-Logs)

| Signal | Beleg | Anmerkung |
|---|---|---|
| User-Agent mit `SEB/<x.y.z> (x64)` (Standard `SEB/3.10.2 (x64)`), aber **ohne** `Chrome/…` oder `Safari/…`-Token (endet nach `Gecko)` direkt mit `SEB/…`) | **BESTÄTIGT** `seb/headers.js:4, 27` | **OFFEN:** Ob der echte SEB-Client genau diesen UA-Aufbau verwendet, ist **nicht verifiziert**. Unbedingt mit Logs echter SEB-Clients im eigenen Haus vergleichen; dieser Abgleich ist die belastbarste Prüfung |
| Widerspruch zwischen behauptetem UA (Windows, SEB) und anderen Anfrage-Merkmalen des tatsächlichen Browsers (z. B. Client-Hints `Sec-CH-UA*`, `Sec-Fetch-*`, Header-Reihenfolge, TLS-Fingerabdruck, Sprache, Plattform) | **ABGELEITET:** Die Erweiterung ersetzt nur `User-Agent`, `ConfigKeyHash` und `RequestHash` (`headers.js:25-33`); andere Header bleiben die des echten Browsers | **OFFEN:** Bei Chromium könnten `Sec-CH-UA-Platform`/`Sec-CH-UA` nicht zum behaupteten UA passen; bei Firefox fehlen Client-Hints; der TLS-/HTTP2-Fingerprint entspricht Firefox/Chrome, nicht dem im echten SEB eingebetteten Browser. **NICHT GEPRÜFT** |
| **Dreiteilige Version** `3.10.2` im UA-Token, fest bei Standardeinstellung | **BESTÄTIGT** `seb/headers.js:27, 64` | Die vierteilige Build-Nummer (`3.10.2.920`) ist im Netzwerk **nicht** sichtbar. Eine Häufung genau der Version `3.10.2` bei Teilnehmern, die real nur andere SEB-Versionen ausgeliefert bekamen, wäre ein Hinweis; der Token folgt aber der **gewählten** Profilversion, ist also nicht stets gleich |
| POST auf `/mod/quiz/accessrule/seb/clear_session.php` beim Entladen der Konfiguration (Beenden, Ersetzen), ausgelöst aus dem Browser-Kontext des Teilnehmers | **BESTÄTIGT** `seb/content.js:295` | **OFFEN:** Ob dieser Aufruf bei echten SEB-Sitzungen anders (anderer Client, anderer Zeitpunkt) auftritt |
| Die Erweiterung lädt die `.seb`-Datei **selbst per `fetch` mit Session-Cookies** nach (`seb/background.js:399-411`) | **BESTÄTIGT** (Code) | Beim **Erstladen** ein zusätzlicher Abruf mit normalem Browser-UA. **ABGELEITET:** Wird eine weitere Konfiguration für denselben Ursprung geladen, solange die vorige aktiv ist, kann auch dieser Hintergrund-Abruf die veränderten Header tragen (der Hook filtert nur nach Ursprung, `headers.js:19-21`; das Entladen folgt erst danach, `background.js:448`). Bei Moodle ist der Config-Download Teil des regulären „Launch SEB"-Ablaufs, daher insgesamt schwach aussagekräftig. Laufzeitverhalten **NICHT GEPRÜFT** |
| Zusätzlicher Abruf von Downloads durch die Erweiterung (Probe) vor/neben dem Browser-Download | **BESTÄTIGT** `download-check.js:79-84`, `download-content.js:37` | Schwach aussagekräftig; relevant nur für Anomalien bei einmal gültigen Download-Links |

---

## 5. Gegenmaßnahmen (Empfehlungen, nach Wirksamkeit gestuft)

Diese Punkte sind **Empfehlungen aus allgemeinem Fachwissen**, nicht aus dem Code abgeleitet. Prüfen Sie sie für Ihre Umgebung.

1. **Prüfungsumgebung kontrollieren statt nur Header prüfen (wirksamste Maßnahme):** Prüfungen nur von verwalteten Geräten oder in beaufsichtigten Räumen, gekoppelt mit netzwerkseitiger Beschränkung (Prüfungs-VLAN/IP-Bereiche, Zugriff auf das LMS nur von dort). Der Header-Mechanismus sagt nichts über den Zustand des Endgeräts aus.
2. **Verwaltete Browser-Konfiguration:** Installation von Erweiterungen und Entwicklermodus per Unternehmensrichtlinie sperren bzw. Allowlist; auf Prüfungsgeräten keine Nutzer-Admin-Rechte.
3. **Netzwerk:** KI-Anbieter-Domains (4.1) in Prüfungsnetzen sperren oder protokollieren; Alarm auf Zugriffe auf `raw.githubusercontent.com/63kitsune/…` (nur mit Pfad-Sichtbarkeit; die Abrufe sind unregelmäßig, 4.1).
4. **Moodle-seitig:** Einsatz von SEB-Verwaltung mit stärkerer Client-Überprüfung (z. B. SEB Server mit Client-Monitoring) in Betracht ziehen; die reine Header-Prüfung nicht als alleinige Kontrolle verwenden.
5. **Prüfungsdesign:** Zufällige Fragen-/Antwortreihenfolge, Fragenpools, individuelle Zahlenvarianten (Moodle „calculated"), Zeitlimit pro Seite, offene/anwendungsbezogene Aufgaben, Nachgespräche bei Auffälligkeiten. Das reduziert den Nutzen von KI-Antworten unabhängig vom eingesetzten Tool.
6. **Auswertung bereits abgelegter Prüfungen (nachträglich):** Antwortzeiten pro Frage und Seite, Bearbeitungsblöcke, gleichförmige Muster, Abgleich Prüfungs-IP/Netzwerklogs (4.1, 4.5). Befunde gelten nur als **Indizien**; für Konsequenzen sind arbeits-/prüfungsrechtliche und datenschutzrechtliche Vorgaben (Betriebsrat, Datenschutz) vorab zu klären.

---

## 6. Datenschutz- und Sicherheitsrisiken, Rechtshinweise

Dieser Abschnitt fasst die für die IT relevanten Befunde zusammen. Die Einzelheiten mit Fundstellen stehen in `technical-handover.md` Abschnitt 11, die technischen Risiken der Erweiterung in Abschnitt 12 dort. Er enthält keine Rechtsberatung.

### 6.1 Befunde

| ID | Befund (BESTÄTIGT, aus dem Code) | Mögliche Auswirkung (ABGELEITET) | Offen |
|---|---|---|---|
| DS-01 | **API-Schlüssel** der KI-Dienste liegen im Klartext in `storage.local` (`assistant/popup.js:371`); sie werden bei jeder Anfrage an den Anbieter bzw. den frei wählbaren „Custom"-Host gesendet (`providers.js:142-183, 385-415`). Einzelne Konten sind im Popup entfernbar; es gibt keine Export- und keine „Alles löschen"-Funktion | Wer das Browserprofil lesen kann, kann Schlüssel lesen; Schlüssel überdauern Sitzungen | Speicherort/-format je Browser (NICHT GEPRÜFT) |
| DS-02 | **Formularinhalte** abgefangener SEB-Formulare werden gelesen; bei POST wird der Body mitgesendet und bei passwortgeschützten Dateien im Pending-Datensatz gespeichert (`seb/content.js:49-76`, `background.js:451-453`); GET-Parameter landen in der URL im Zustand/Popup | Enthalten Felder Sitzungswerte, liegen sie im Erweiterungsspeicher | Welche Felder real enthalten sind; Dauer der Speicherung (OFFEN) |
| DS-03 | **Letzter KI-Lauf** (URL, Fragetexte, Antworten, Bild-URLs, Anbieter, Zeit) wird dauerhaft gespeichert und nur beim nächsten Lauf überschrieben; keine UI-Löschfunktion (`assistant/background.js:101-105`) | Prüfungsinhalte bleiben lokal zurück – eine **forensische Spur** | Verhalten bei Deinstallation (NICHT GEPRÜFT) |
| DS-04 | **Übertragung an KI-Anbieter:** Fragen (Text, Optionen), bis zu 8 Bilder als Base64 und die Nutzernotiz; bei Fehler zusätzlich an das nächste Konto; nur bei OpenAI `store: false` (`providers.js:194-303, 252`) | Weitergabe vertraulicher Prüfungsinhalte und ggf. personenbezogener Daten an Dritte; gehört ggf. in die Meldekette | Aufbewahrung/Nutzung bei Anbietern, Rechtsgrundlage (NICHT GEPRÜFT) |
| DS-05 | **Bildabrufe** von beliebigen Hosts mit Cookies (`assistant/extract.js:46-51`) | Zusätzliche Abrufe an Fremdhosts | Tatsächlich gesendete Cookies/Header (NICHT GEPRÜFT) |
| DS-06 | **Klartext-Konfiguration** (u. a. Start-URL, Hash des Beenden-Passworts) in `sebInspectorState`, im Popup lesbar, kopier- und als `.xml` speicherbar (`background.js:467-475`, `seb/popup.js:192-206`) | Prüfungskonfiguration liegt unverschlüsselt im Profil; Exporte außerhalb der Kontrolle der Erweiterung | Dauerhaftigkeit je Browser (OFFEN) |
| DS-07 | **Berechtigungen:** `<all_urls>`, `tabs`, `webRequest`+`webRequestBlocking`, `downloads`; Content-Scripts in allen Frames; blockierende Hooks für alle URLs, Einschränkung auf den Prüfungsursprung erst im Code (`headers.js:21`) | Technisch weitreichender Zugriff auf Seiteninhalte und Anfrage-Header; der Code nutzt ihn nach Lektüre nur in den dokumentierten Fällen | Laufzeitverhalten (NICHT GEPRÜFT) |
| E-05 | **Lokale Spuren:** Einstellungen, Katalog-Cache, KI-Konten, Domain-Freigaben, Notiz, letzter Lauf im Erweiterungsspeicher; Zustand/Pending je Browser in `session` oder `local` | Auswertbar bei Profilzugriff (Inventar, Forensik) | Speicherorte je Browser/OS (NICHT GEPRÜFT) |

### 6.2 Hinweise zur Erweiterung als Angriffsfläche

- **ABGELEITET** (Kette im Code gelesen, nicht ausgeführt): Beliebiges Seiten-JavaScript kann über den `postMessage`-Kanal der Erweiterung Konfigurationsbytes übergeben und so das Laden und Starten anstoßen (TR-04 in `technical-handover.md` Abschnitt 12). Die automatische Erkennung ist standardmäßig aktiv.
- **BESTÄTIGT:** Der Katalog wird von einem Drittrepository nachgeladen; die Prüfsumme stammt aus derselben Datei, es gibt keine Signatur (TR-07).

### 6.3 Rechts- und Verfahrenshinweise

- **ABGELEITET** – Wer die Erweiterung nutzt, sendet Prüfungsinhalte an Dritte (KI-Anbieter). Dies kann einen **Vertraulichkeits-/Datenschutzvorfall** darstellen und gehört gegebenenfalls in die Meldekette Ihrer Organisation.
- Nutzen Sie **Zeitstempel- und Protokollquellen nur im Rahmen Ihrer Richtlinien**; eine Zuordnung zu einer Person erfordert in der Regel mehrere unabhängige Indizien. Für personenbezogene Auswertungen sind Betriebsrat/Personalvertretung, Datenschutz und Rechtsabteilung vorab zu beteiligen.

---

## 7. Grenzen dieser Analyse und offene Prüfpunkte

### 7.1 Prüfstatus

| Bereich | Status |
|---|---|
| Statische Lektüre des Quellcodes (Abschnitte 2–6) | BESTÄTIGT bzw. ABGELEITET, wie gekennzeichnet |
| **Laufzeittests** | **NICHT GEPRÜFT** |
| **Browserkompatibilität** (Versionen, Derivate, Android; MV2-Verfügbarkeit) | **NICHT GEPRÜFT** |
| **Netzwerktests** (Mitschnitt, Header, Abrufzeitpunkte, Sichtbarkeit in Proxy-Logs) | **NICHT GEPRÜFT** |
| Wirksamkeit gegen die konkrete Moodle-Version und -Konfiguration (SEB-Zugriffsregel, SEB Server, Plugins) | **NICHT GEPRÜFT** |
| Korrektheit der Header-/Schlüsselberechnung gegenüber SEB | **NICHT GEPRÜFT** (bewusst ausgeklammert) |
| Build der Erweiterung | **NICHT GEPRÜFT** |

### 7.2 Offene Punkte

1. **OFFEN** – Abgleich mit realen Logs echter SEB-Clients Ihrer Umgebung (UA-Aufbau, Header-Reihenfolge, Client-Hints): wichtigster nächster Schritt.
2. **OFFEN** – Abrufzeitpunkte der Katalogaktualisierung und Sichtbarkeit der Merkmale aus 4.1/4.3 in einer Testumgebung.
3. **OFFEN** – Browser-/MV2-Matrix (z. B. Verfügbarkeit von `storage.session` und `runtime.getBrowserInfo` in Content-Scripts, UA-01 bis UA-03 in `technical-handover.md` 13.2).
4. **OFFEN** – Ob Moodle ohne Plugin untrusted Ereignisse beobachten kann (4.4).
5. **OFFEN** – Ob 2.4.1 inhaltlich von 2.4.0 abweicht.
6. Alle genannten Erkennungsmerkmale sind **leicht änderbar**, wenn jemand den (öffentlichen) Quellcode anpasst. Verlassen Sie sich nicht auf Einzelsignale.
7. **BESTÄTIGT (Negativbefund)** – Im gelesenen Code wurde kein Schadcode- oder Backdoor-Anteil gefunden (keine Telemetrie, kein `eval`/dynamisches Nachladen von Code). **NICHT GEPRÜFT:** `popup.css`, Bilddateien, Inhalt von `bek.json`. Dennoch gilt unbekannte Software auf Prüfungs- oder Firmengeräten als Risiko.

---

## Anhang A: Schnell-Checkliste für das IT-Team

- [ ] Proxy-/DNS-Logs der Prüfungszeiträume auf KI-API-Domains (4.1) und `raw.githubusercontent.com/63kitsune/` prüfen (Abrufe sind unregelmäßig, nicht an jeden Browserstart gebunden)
- [ ] Moodle-/Reverse-Proxy-Logs: User-Agent-Muster `… Gecko) SEB/<x.y.z> (x64)` (z. B. `SEB/3.10.2`) ohne `Chrome/`/`Safari/` (mit echten SEB-Logs vergleichen; nur die dreiteilige Version ist sichtbar)
- [ ] Verwaltete Geräte: Browser-Erweiterungen nach Name „OEB — Open Exam Browser" / ID `oeb@open-exam-browser.local` durchsuchen
- [ ] Moodle-Auswertung: Antwortzeiten pro Frage/Seite, Blockeingaben, Freitext ohne Tippdauer
- [ ] Entscheidung: Erweiterungs-/Entwicklermodus-Richtlinien auf Prüfungsgeräten
- [ ] Entscheidung: Prüfungen nur noch in kontrollierter Umgebung bzw. mit SEB Server
- [ ] Datenschutz/Betriebsrat/Rechtsabteilung vor personenbezogener Auswertung informieren; Meldekette für Abfluss von Prüfungsinhalten klären (Abschnitt 6)

## Anhang B: Referenz der analysierten Dateien (kurz)

`source/seb/background.js` (748 Zeilen, Hauptsteuerung), `seb/headers.js` (121, Request-Header), `seb/config-key.js` (337), `seb/browser-exam-key.js` (64), `seb/bek-catalog.js` (143, Profilverwaltung/GitHub-Abruf), `seb/parser.js` (246, `.seb`-Entschlüsselung), `seb/content.js` (316, Overlay/Frames/Moodle-Reset), `seb/download-*.js` (Download-Erkennung), `assistant/*.js` (KI-Lauf), `popup.*` (UI), `chrome|firefox.manifest.json` (Berechtigungen), `build.mjs` (ZIP-Build). **BESTÄTIGT (Negativbefund):** Keine Testdateien und keine CI-Konfiguration im Projekt (keine Testverzeichnisse im ZIP, kein Test-Skript in `package.json`).
