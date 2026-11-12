# Technische Analyse: Browser-Erweiterung „OEB — Open Exam Browser" (Version 2.4.x)

**Zweck dieses Dokuments:** Information für das IT-Team zur Erkennung, Bewertung und Eindämmung der Nutzung dieser Erweiterung bei Online-Prüfungen (Moodle + Safe Exam Browser, SEB).
**Nicht Zweck:** Dieses Dokument ist keine Anleitung zum Nachbauen, Pflegen oder Verbessern der Erweiterung. Konstanten, Profildaten und Berechnungsdetails der Fälschungsfunktion sind bewusst weggelassen.

**Kennzeichnung der Aussagen**
- **[Fakt]** direkt im analysierten Quellcode belegt (Dateipfad/Zeile angegeben, bezogen auf `OEB-2.4.1/source/`).
- **[Schluss]** technische Schlussfolgerung aus dem Code, nicht dynamisch getestet.
- **[Vermutung]** plausibel, aber nicht belegt; vor Verwendung prüfen.

**Umfang der Analyse:** reine statische Quellcode-Analyse des ZIP `OEB-2.4.1`. Die Erweiterung wurde **nicht** ausgeführt, es wurde kein Netzwerkverkehr mitgeschnitten und keine Moodle-Instanz getestet. Alle Erkennungsansätze unten sind daher vor dem produktiven Einsatz zu validieren.

---

## 1. Kurzfassung (Management-Summary)

1. **[Fakt]** OEB ist eine Browser-Erweiterung (Firefox ≥ 140 und Chromium mit Manifest V2), die sich gegenüber dem Prüfungsserver als echter Safe Exam Browser ausgibt. Das README nennt als Vorlage „SebBypass".
2. **[Fakt]** Zusätzlich enthält sie einen KI-Assistenten, der Moodle-Fragen auf der Seite ausliest, an externe KI-Dienste sendet und die Antworten automatisch in die Formularfelder einträgt.
3. **[Fakt]** Die Erweiterung erzwingt **keine** Betriebssystem-Sperre (README: „OEB does not enforce operating-system kiosk restrictions"). Es läuft ein normaler Browser auf einem normalen, uneingeschränkten System.
4. **[Schluss]** Eine Prüfung, die nur darauf vertraut, dass der Server die SEB-Header (Config Key / Browser Exam Key / User-Agent) bestätigt, ist gegen dieses Werkzeug **nicht belastbar geschützt**. Die Header werden für die geladene Konfiguration passend berechnet.
5. **[Schluss]** Es gibt aber mehrere **Spuren** (Netzwerk, Browser-Inventar, DOM, Ereignisse, Antwortverhalten), über die sich der Einsatz erkennen oder erschweren lässt (Abschnitte 4 und 5).

---

## 2. Funktionsweise (Überblick)

### 2.1 Komponenten

| Komponente | Dateien | Aufgabe |
|---|---|---|
| Hintergrund-Skript (persistente Background-Page, MV2) | `background.html:6`, `seb/background.js` | Zentrale Steuerung: erkennt SEB-Konfigurationen, hält Zustand, aktiviert Header-Manipulation, steuert Overlay und KI-Lauf |
| Erkennung von Konfigurationsdateien | `seb/launch-response.js`, `seb/download-check.js`, `seb/download-stream.js`, `seb/download-content.js`, `seb/download-page.js` | Fängt `.seb`-Dateien, `seb://`/`sebs://`-Links und Downloads ohne Dateiendung ab (auch Blob-Downloads) |
| Parser | `seb/parser.js` | Entpackt/entschlüsselt `.seb`-Dateien (gzip, passwortgeschützt, RNCryptor). Zertifikat-verschlüsselte Dateien werden abgelehnt (`parser.js:69-74`) |
| Header-Manipulation | `seb/headers.js`, `seb/config-key.js`, `seb/browser-exam-key.js`, `seb/bek-catalog.js`, `bek.json` | Setzt SEB-typische Request-Header für den Prüfungsserver (siehe 2.3) |
| Optik/Täuschung | `seb/content.js`, `seb/assets/startup.*`, `taskbar.*`, `quit.*` | Blendet Start-Animation, SEB-ähnliche Taskleiste und Beenden-Dialog ein |
| KI-Assistent | `assistant/background.js`, `providers.js`, `extract.js`, `apply.js`, `content.js`, `popup.js` | Fragen auslesen → KI-Anbieter abfragen → Antworten eintragen |
| Bedienoberfläche | `popup.html`, `popup.css`, `seb/popup.js`, `assistant/popup.js` | Popup der Erweiterung |
| Build | `build.mjs`, `package.json` | Erzeugt `release/chrome` und `release/firefox` als ZIP |

**[Fakt]** Es gibt **keinen eigenen Server** der Autoren, nur zwei Netzwerkziele außerhalb des Prüfungsservers: GitHub (Profil-Datei) und die KI-Anbieter (siehe 4.1).

### 2.2 Typischer Ablauf aus Sicht der nutzenden Person [Fakt, aus dem Code abgeleitet]

1. Nutzer öffnet die Moodle-Prüfungsseite im normalen Browser mit installierter Erweiterung.
2. Klick auf „Launch SEB" bzw. Auswahl der `.seb`-Datei. Die Erweiterung fängt Klick/Download ab (`seb/content.js:34-47`, `seb/background.js:72-107`) und lädt die Konfiguration selbst nach (`processUrlOnce`, `seb/background.js:359-433`).
3. Die Konfiguration wird geparst, daraus werden Config-Key-Hash und Start-URL bestimmt (`replaceConfiguration`, `seb/background.js:442-491`; `activateLaunch`, `:559-584`).
4. Ab jetzt verändert die Erweiterung jede Anfrage an den Prüfungsserver (`seb/headers.js:19-43`).
5. Der Tab wird mit einer SEB-ähnlichen Start-Animation überlagert und danach auf die Prüfung geladen (`transitionToDestination`, `seb/background.js:593-605`); eine nachgebaute SEB-Taskleiste wird eingeblendet.
6. Optional: Tastenkürzel **Ctrl+Shift+Y** startet den KI-Lauf, **Ctrl+Shift+X** klickt auf „Nächste Seite" (`assistant/content.js:122-129`).
7. Beim „Beenden" wird die Konfiguration entladen und ggf. der Tab geschlossen (`unloadConfiguration`, `seb/background.js:615-637`).

### 2.3 Die Header-Fälschung (auf Konzeptebene)

- **[Fakt]** Nur für Anfragen an den **Ursprung (Origin) der Prüfungs-Start-URL** (`headers.js:21`) werden folgende Änderungen vorgenommen:
  - der `User-Agent` wird ersetzt durch `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) SEB/<x.y.z> (x64)` (`headers.js:4, 27`);
  - der Header `X-SafeExamBrowser-ConfigKeyHash` wird gesetzt (`headers.js:5, 28`) — berechnet aus dem Hash der geladenen Konfiguration und der jeweiligen URL;
  - optional der Header `X-SafeExamBrowser-RequestHash` (`headers.js:6, 29-32`) — der Browser Exam Key (BEK).
- **[Fakt]** Der Config Key ergibt sich ausschließlich aus der `.seb`-Datei, die ohnehin der Prüfling/der Server kennt.
- **[Fakt]** Der BEK wird im Modus „automatic" **lokal berechnet** aus der Konfiguration plus buildspezifischen Profildaten (`browser-exam-key.js:34-46`, Profile in `bek.json`, ein Standardprofil für SEB 3.10.2.920 Windows ist mitgeliefert). Alternativ kann ein BEK von Hand eingetragen werden („provided").
- **[Schluss]** Der BEK ist damit **kein Nachweis**, dass tatsächlich ein unveränderter SEB-Client läuft, sofern die Profildaten des betreffenden Builds bekannt sind. Eine reine „BEK/Config-Key in Moodle erlauben"-Prüfung kann mit denselben Eingaben erfüllt werden.
- **[Fakt]** Beim Beenden ruft die Erweiterung auf der Moodle-Seite per POST `/mod/quiz/accessrule/seb/clear_session.php` auf (`seb/content.js:285-306`), um die SEB-Sitzung serverseitig zurückzusetzen.

### 2.4 KI-Assistent [Fakt]

- **Auslesen:** `assistant/extract.js` liest alle `.que`-Elemente (Moodle-Fragen), erkennt den Fragetyp (Multiple Choice, Kurzantwort, Zuordnung, Lückentext, Drag&Drop, Marker u. a. – `extract.js:89-96`) und lädt auch Bilder der Frage nach (bis 8 Bilder, max. 4 MB je Bild, 16 MB gesamt – `extract.js:4-8`).
- **Abfragen:** `assistant/providers.js` / `assistant/background.js:57` – Fragen werden an den konfigurierten KI-Anbieter geschickt, bei Fehlern mit weiterem Anbieter (`askWithFallback`). Der Nutzer kann zusätzlich einen Freitext-Hinweis („Note") mitgeben.
- **Eintragen:** `assistant/apply.js` setzt Auswahlfelder/Textfelder, klickt Radio-/Checkboxen, bedient Drag&Drop per simulierter Pfeiltaste und füllt Rich-Text-Felder (`apply.js:3-202`).
- **Sicherungen der Autoren:** Eintrag nur auf Domains, die im Popup freigeschaltet wurden (`assistant/background.js:44-46`); Abbruch, wenn sich Seite/Fragen während der KI-Antwort ändern (`:58-65`); Hinweistext „Review before submitting" (`:83`). Das Absenden der Prüfung erfolgt **nicht** automatisch [Fakt: kein Submit-Aufruf im Code gefunden; die Navigation „Weiter" erfolgt nur per Tastenkürzel].

---

## 3. Bewertung: Was ist mit der Erweiterung möglich, was nicht?

| Aspekt | Einschätzung |
|---|---|
| Server-seitige SEB-Prüfung per Header (Config Key / BEK / User-Agent) | **[Schluss]** umgehbar, solange der Build bekannt ist, den die Erweiterung nachahmt |
| Betriebssystem-Sperre (Kiosk, Task-Wechsel, Bildschirmaufnahme-Blockade, Prozessüberwachung) | **[Fakt]** wird nicht geleistet, also nicht umgangen, sondern schlicht nicht vorhanden. Wer eine echte Sperre benötigt, braucht Geräte-/Netzwerkkontrolle (Abschnitt 5) |
| Fernzugriff / Datenabfluss | **[Fakt]** Fragetext, Antwortoptionen, Bilder und Nutzer-Hinweis gehen an die gewählten KI-Anbieter. API-Schlüssel liegen lokal im Browser-Storage (README, `assistant/background.js:3-6`) |
| Selbst-Update | **[Fakt]** Es wird nur eine JSON-Profil-Datei nachgeladen (alle 6 h), kein Code (`seb/bek-catalog.js:2-4, 97-103`) |
| Telemetrie an die Autoren | **[Fakt]** im Code nicht gefunden (nur die Profil-Abfrage bei GitHub) |

---

## 4. Erkennungsmöglichkeiten

### 4.1 Netzwerk / Proxy / DNS (am einfachsten, falls Firmenproxy vorhanden)

| Signal | Beleg | Aussagekraft |
|---|---|---|
| Abruf von `https://raw.githubusercontent.com/63kitsune/OEB/refs/heads/main/bek.json` beim Browser-Start und alle ca. 6 Stunden | **[Fakt]** `seb/bek-catalog.js:2, 4`; `seb/background.js:26-29` (Start + stündlicher Timer) | **Sehr spezifisch** wegen des Pfads. Nur sichtbar, wenn HTTPS-Pfade geloggt werden (TLS-Inspektion) — sonst nur der Host `raw.githubusercontent.com`, der allein nicht aussagekräftig ist |
| Verbindungen zu KI-Anbietern, ausgelöst während einer Prüfung | **[Fakt]** `assistant/providers.js`: `generativelanguage.googleapis.com`, `api.groq.com`, `openrouter.ai`, `api.openai.com`, `api.anthropic.com`, `api.mistral.ai`, `api.together.xyz`, `api.cerebras.ai`, zusätzlich ein frei konfigurierbarer „Custom OpenAI-compatible"-Endpunkt | **Mittel**: Hinweis auf KI-Nutzung, aber kein Beweis für diese Erweiterung. Zeitliche Korrelation mit Prüfungsfenstern und Prüfungs-IP wertvoll. Ein eigener Endpunkt lässt sich so nicht erfassen |
| Der „Custom"-Anbieter | **[Fakt]** vorhanden | Blocklisten nach Domain greifen hier nicht vollständig |

**Empfehlung:** Für Prüfungszeiträume KI-API-Domains auf dem Firmen-/Hochschulnetz blockieren oder zumindest protokollieren (nur wirksam im eigenen Netz; auf privaten Geräten/Netzen nicht).

### 4.2 Endgeräte-Inventar (wenn verwaltete Geräte)

- **[Fakt]** Firefox-Erweiterungs-ID: `oeb@open-exam-browser.local` (`firefox.manifest.json:83`); Anzeigename in beiden Browsern: „OEB — Open Exam Browser".
- **[Fakt]** Angeforderte Berechtigungen: `downloads`, `storage`, `tabs`, `webRequest`, `webRequestBlocking`, `<all_urls>`, sowie Content-Scripts auf allen Seiten in allen Frames (`*.manifest.json`). Firefox: `data_collection_permissions` verlangt `websiteContent` und `authenticationInfo`.
- **[Schluss]** Eine Abfrage installierter Erweiterungen (z. B. über Browser-Management/MDM oder Firefox-Profilordner) kann die ID bzw. den Namen finden. Die Erweiterung wird laut README meist als **temporäres Add-on** (about:debugging) bzw. **entpackt** (Chromium „Entpackte Erweiterung laden") installiert, daher erscheint sie ggf. nicht in zentralen Store-Listen.
- **[Vermutung]** Da die Installation temporär bzw. als Entwicklerversion erfolgt, sind Richtlinien, die „Entwicklermodus"/„Temporäre Add-ons" unterbinden, wirksam. Konkrete Richtlinien-Namen bitte in der Herstellerdokumentation der eingesetzten Browser prüfen.

### 4.3 Seiten-seitige Spuren im DOM (JavaScript auf der Prüfungsseite oder Moodle-Plugin)

Die Content-Scripts laufen in jedem Frame jeder Seite und hinterlassen sichtbare Artefakte in der Moodle-Seite:

| Artefakt | Beleg | Bedingung |
|---|---|---|
| `<iframe id="seb-local-startup-frame">`, `id="seb-local-taskbar-frame"`, `id="seb-local-quit-frame"` | **[Fakt]** `seb/content.js:3-5` | Wenn Start-Animation/Taskleiste aktiv; direkt unter `document.documentElement` angehängt |
| `src` dieser Frames beginnt mit `moz-extension://…` bzw. `chrome-extension://…` und enthält `seb/assets/` | **[Fakt]** `seb/content.js:171, 221, 246` | Seiten-JavaScript kann `iframe.src` lesen |
| `<div id="oeb-seb-link-status">` | **[Fakt]** `seb/content.js:132` | Nur bei Statusmeldung |
| Element mit Klasse `oeb-ai-marker` (in Marker-Fragen) | **[Fakt]** `assistant/apply.js:80` | Nur bei Marker-Fragetyp nach KI-Eintrag |
| Zusätzliches Page-Script im „MAIN world" mit Nachrichtenkanal `oeb-download-capture-v1`; überschreibt `URL.createObjectURL`, `URL.revokeObjectURL` (`seb/download-page.js:13-21`) und `HTMLAnchorElement.prototype.click` (`:92`) | **[Fakt]** `seb/download-page.js`; in Firefox als Content-Script mit `world: "MAIN"` (`firefox.manifest.json:55-65`), in Chromium wird kurz ein `<script src="chrome-extension://…/seb/download-page.js">` ins DOM eingefügt und beim Laden wieder entfernt (`seb/download-content.js:61-66`) | Auf manipulierte Prototypen prüfbar (z. B. `URL.createObjectURL.toString()` enthält keinen `[native code]`) **[Schluss]** |
| Tastenkombination Ctrl+Shift+X wird im Capture-Modus abgefangen und führt zum Klick auf „Nächste Seite" | **[Fakt]** `assistant/content.js:122-129` | nur auf dem Top-Frame |
| Hinweis: Das Seiten-JavaScript kann Elemente aus Content-Scripts nicht direkt „sehen", wohl aber das daraus entstehende DOM | **[Schluss]** | — |

**Wichtig [Schluss]:** Die Artefakte sind für einen Nutzer mit Quellcode-Zugriff leicht änderbar (IDs umbenennen). Sie taugen als **Hinweis**, nicht als Beweis oder alleinige Schutzmaßnahme. Ein Fehlen der Artefakte bedeutet nicht, dass die Erweiterung nicht aktiv ist.

### 4.4 Verhaltensspuren bei Eingaben (Moodle-seitig oder Plugin)

- **[Fakt]** Antworten werden **programmatisch** gesetzt: `el.value = …` und synthetische `input`/`change`-Ereignisse (`assistant/apply.js:7-11`), `input.click()` für Radio/Checkbox (`:116-118`), synthetische `KeyboardEvent('keydown', ArrowDown)` für Drag&Drop-Lücken im Abstand von 350 ms (`:174-177`), Direktschreiben in Rich-Text-Felder (`:29-47`).
- **[Schluss]** Solche Ereignisse haben `event.isTrusted === false`. Ein Skript auf der Moodle-Seite, das Ereignisse an den Antwortfeldern beobachtet, könnte Eingaben **ohne echte Tastatur-/Mausereignisse** erkennen. Das gilt auch für Eingaben, die von anderen Automatisierungswerkzeugen stammen. Auf Falschmeldungen achten (z. B. Passwortmanager, Barrierefreiheits-Hilfsmittel). **[Vermutung]** Ob das in der eingesetzten Moodle-Version ohne eigenes Plugin umsetzbar ist, müsste geprüft werden.
- **[Schluss]** In den Moodle-Protokollen/Frageversuchs-Daten typisch: viele Fragen **innerhalb weniger Sekunden** beantwortet, sehr gleichmäßige Abstände, Antworten für lange Freitexte ohne Tippdauer (Freitext-Felder werden komplett in einem Schritt gesetzt, `apply.js:186-190`), und die Antwort auf eine Seite entsteht in einem Block, bevor ein „Weiter" erfolgt.
- **[Schluss]** Das Tastenkürzel „Nächste Seite" erzeugt einen regulären Klick (`next.click()`), also einen untrusted Click.

### 4.5 Server-/Request-seitige Spuren (Reverse-Proxy- oder Moodle-Logs)

| Signal | Beleg | Anmerkung |
|---|---|---|
| User-Agent mit `SEB/3.10.2 (x64)`, aber **ohne** `Chrome/…` oder `Safari/…`-Token (endet nach `Gecko)` direkt mit `SEB/…`) | **[Fakt]** `seb/headers.js:4, 27` | **[Vermutung]** Ob der echte SEB-Client genau diesen UA-Aufbau verwendet, ist hier **nicht verifiziert**. Unbedingt mit Logs echter SEB-Clients im eigenen Haus vergleichen. Ein Abgleich ist die belastbarste Prüfung. |
| Widerspruch zwischen behauptetem UA (Windows, SEB) und anderen Anfrage-Merkmalen des tatsächlichen Browsers (z. B. Client-Hints `Sec-CH-UA*`, `Sec-Fetch-*`, Header-Reihenfolge, TLS-Fingerabdruck, Sprache, Plattform) | **[Schluss]** Die Erweiterung ersetzt nur `User-Agent`, `ConfigKeyHash` und `RequestHash` (`headers.js:25-33`). Andere Header bleiben die des echten Browsers | **[Vermutung]** Bei Chromium könnten `Sec-CH-UA-Platform` oder `Sec-CH-UA` nicht zum behaupteten UA passen; bei Firefox fehlen Client-Hints. Auch TLS-/HTTP2-Fingerprint (JA3/JA4) entspricht dem Firefox/Chrome, nicht dem im echten SEB eingebetteten Browser. Alles ungetestet. |
| Behaupteter Build `3.10.2` / `3.10.2.920` | **[Fakt]** Standard-Profil in `bek.json`/`browser-exam-key.js:6` | Hohe Häufung von genau dieser Buildnummer bei Teilnehmern, die real nur ältere/neuere SEB-Versionen ausgeliefert bekamen, ist ein Hinweis. |
| POST auf `/mod/quiz/accessrule/seb/clear_session.php` nach Prüfungsende, ausgelöst aus dem Browser-Kontext des Teilnehmers | **[Fakt]** `seb/content.js:295` | **[Vermutung]** Ob dieser Aufruf bei echten SEB-Sitzungen anders (anderer Client, anderer Zeitpunkt) auftritt, ist zu prüfen. |
| Nach dem Laden der Konfiguration lädt die Erweiterung die `.seb`-Datei **selbst per `fetch` mit Session-Cookies** nach (`seb/background.js:398-411`) | **[Fakt]** | Zusätzlicher Abruf der Konfigurationsdatei mit „normalem" Browser-UA kurz vor der Prüfung. Bei Moodle ist der Config-Download Teil des regulären „Launch SEB"-Ablaufs, daher schwach aussagekräftig. |

---

## 5. Gegenmaßnahmen (Empfehlungen, nach Wirksamkeit gestuft)

Diese Punkte sind **Empfehlungen aus allgemeinem Fachwissen**, nicht aus dem Code abgeleitet. Prüfen Sie sie für Ihre Umgebung.

1. **Prüfungsumgebung kontrollieren statt nur Header prüfen (wirksamste Maßnahme):** Prüfungen nur von verwalteten Geräten oder in beaufsichtigten Räumen, gekoppelt mit netzwerkseitiger Beschränkung (Prüfungs-VLAN/IP-Bereiche, Zugriff auf das LMS nur von dort). Der Header-Mechanismus sagt nichts über den Zustand des Endgeräts aus.
2. **Verwaltete Browser-Konfiguration:** Installation von Erweiterungen und Entwicklermodus per Unternehmensrichtlinie sperren bzw. Allowlist; auf Prüfungsgeräten keine Nutzer-Admin-Rechte.
3. **Netzwerk:** KI-Anbieter-Domains (4.1) in Prüfungsnetzen sperren oder protokollieren; Alarm auf Zugriffe auf `raw.githubusercontent.com/63kitsune/…` (nur mit Pfad-Sichtbarkeit).
4. **Moodle-seitig:** Einsatz von SEB-Verwaltung mit stärkerer Client-Überprüfung (z. B. SEB Server mit Client-Monitoring) in Betracht ziehen; die reine Header-Prüfung nicht als alleinige Kontrolle verwenden.
5. **Prüfungsdesign:** Zufällige Fragen-/Antwortreihenfolge, Fragenpools, individuelle Zahlenvarianten (Moodle „calculated"), Zeitlimit pro Seite, offene/anwendungsbezogene Aufgaben, Nachgespräche bei Auffälligkeiten. Das reduziert den Nutzen der KI-Antworten unabhängig vom eingesetzten Tool.
6. **Auswertung bereits abgelegter Prüfungen (nachträglich):** Antwortzeiten pro Frage und Seite, Bearbeitungsblöcke, gleichförmige Muster, Abgleich Prüfungs-IP/Netzwerklogs (4.1, 4.5). Befunde gelten nur als **Indizien**; für Konsequenzen sind arbeits-/prüfungsrechtliche und datenschutzrechtliche Vorgaben (Betriebsrat, Datenschutz) vorab zu klären.

---

## 6. Risiko- und Rechtshinweise für das Team

- **[Fakt]** Die Erweiterung kann ein sehr weites Berechtigungsprofil nutzen (alle Webseiten, Request-Header, Downloads). Auf Geräten, auf denen sie installiert ist, hat sie Zugriff auf Seiteninhalte aller besuchten Seiten. Gespeicherte KI-API-Schlüssel liegen im lokalen Browser-Storage.
- **[Schluss]** Wer die Erweiterung nutzt, sendet Prüfungsinhalte an Dritte (KI-Anbieter). Dies kann einen **Vertraulichkeits-/Datenschutzvorfall** darstellen (Prüfungsinhalte, ggf. personenbezogene Daten) und gehört gegebenenfalls in die Meldekette Ihres Unternehmens.
- Nutzen Sie **Zeitstempel- und Protokollquellen nur im Rahmen Ihrer Richtlinien**; eine Zuordnung zu einer Person erfordert in der Regel mehrere unabhängige Indizien.

---

## 7. Grenzen dieser Analyse und offene Prüfpunkte

1. Nicht dynamisch getestet. Insbesondere sind 4.3-4.5 **Hypothesen mit Codebelegen**, keine bestätigten Detektionen.
2. Abgleich mit realen Logs echter SEB-Clients Ihrer Umgebung ist der wichtigste nächste Schritt (UA-Aufbau, Header-Reihenfolge, Client-Hints).
3. Die analysierte Version trägt intern `2.4.0` (`package.json`, `*.manifest.json`), der ZIP-Ordner heißt `OEB-2.4.1`. **[Fakt]** Abweichung; ob 2.4.1 Änderungen enthält, die nicht im Code stehen, ist unbekannt. Andere Versionen/Forks können abweichende IDs, Frame-IDs und Hosts haben.
4. Alle genannten Erkennungsmerkmale sind **leicht änderbar**, wenn jemand den (öffentlichen) Quellcode anpasst. Verlassen Sie sich nicht auf Einzelsignale.
5. Nicht untersucht: tatsächliche Wirksamkeit gegen Ihre konkrete Moodle-Version und -Konfiguration (SEB-Zugriffsregel, SEB Server, Plugins).
6. Nicht geprüft: ob ein Schadcode-/Backdoor-Anteil enthalten ist. Im gelesenen Code wurde keiner gefunden; dennoch gilt unbekannte Software auf Prüfungs- oder Firmengeräten als Risiko.

---

## Anhang A: Schnell-Checkliste für das IT-Team

- [ ] Proxy-/DNS-Logs der Prüfungszeiträume auf KI-API-Domains (4.1) und `raw.githubusercontent.com/63kitsune/` prüfen
- [ ] Moodle-/Reverse-Proxy-Logs: User-Agent-Muster `… Gecko) SEB/3.10.2 (x64)` ohne `Chrome/`/`Safari/` (mit echten SEB-Logs vergleichen)
- [ ] Verwaltete Geräte: Browser-Erweiterungen nach Name „OEB — Open Exam Browser" / ID `oeb@open-exam-browser.local` durchsuchen
- [ ] Moodle-Auswertung: Antwortzeiten pro Frage/Seite, Blockeingaben, Freitext ohne Tippdauer
- [ ] Entscheidung: Erweiterungs-/Entwicklermodus-Richtlinien auf Prüfungsgeräten
- [ ] Entscheidung: Prüfungen nur noch in kontrollierter Umgebung bzw. mit SEB Server
- [ ] Datenschutz/Betriebsrat/Rechtsabteilung vor personenbezogener Auswertung informieren

## Anhang B: Referenz der analysierten Dateien (kurz)

`source/seb/background.js` (748 Zeilen, Hauptsteuerung), `seb/headers.js` (121, Request-Header), `seb/config-key.js` (337), `seb/browser-exam-key.js` (64), `seb/bek-catalog.js` (143, Profilverwaltung/GitHub-Abruf), `seb/parser.js` (246, `.seb`-Entschlüsselung), `seb/content.js` (316, Overlay/Frames/Moodle-Reset), `seb/download-*.js` (Download-Erkennung), `assistant/*.js` (KI-Lauf), `popup.*` (UI), `chrome|firefox.manifest.json` (Berechtigungen), `build.mjs` (ZIP-Build). Es gibt **keine** Testdateien im Projekt [Fakt: keine Testverzeichnisse im ZIP, kein Test-Skript in `package.json`].
