# Sicherheits- und Datenschutzprüfung der OEB-Codebasis (Phase 3)

**An:** IT-Leitung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Klassifizierung:** intern
**Gegenstand:** Browser-Erweiterung „OEB — Open Exam Browser", ZIP `OEB-2.4.1` (Paketversion 2.4.0), Prüfung auf Eignung als Ausgangspunkt eines Security-Testing-Frameworks.
**Begleitdokumente dieser Phase:** `reuse-assessment.md` · `security-testing-requirements.md` · `migration-risk-assessment.md`
**Vorarbeiten:** `technical-handover.md`, `OEB-Analyse-IT.md`, `verification-report.md`, `documentation-gaps.md` (Revision 2)

**Kennzeichnung:** **BESTÄTIGT** = durch statische Quellcodeanalyse belegt (Datei:Zeile, bezogen auf `OEB-2.4.1/source/`) · **ABGELEITET** = plausible Schlussfolgerung, nicht zur Laufzeit geprüft · **OFFEN** = weitere Prüfung nötig · **NICHT GEPRÜFT** = außerhalb des Prüfumfangs.

**Prüfstatus:** Es wurden weiterhin **keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests** durchgeführt. Die Erweiterung wurde nicht installiert, ausgeführt oder gebaut. Alle Aussagen beruhen auf Lektüre und gezielten Suchläufen im Quellcode.

---

## 1. Ergebnis in Kürze

1. **Die Dokumentation reicht als Bestandsaufnahme, nicht als Entwurfsgrundlage.** Sie beschreibt Aufbau und Datenflüsse zuverlässig (Abschnitt 2). In dieser Phase habe ich sie erneut gegen den Code geprüft und **fünf Lücken bzw. Fehler** gefunden, darunter einen datenschutzrelevanten (Abschnitt 2.2).
2. **Die Codebasis erfüllt in ihrer jetzigen Form keine Mindestanforderungen für ein Sicherheitswerkzeug:** keine Logging- oder Audit-Funktion, keine Tests, keine Lizenzklärung, Manifest V2 mit blockierendem `webRequest`, Klartext-Speicherung von Zugangsdaten und Prüfungsinhalten, nicht autorisierte Nachrichtenpfade (Abschnitte 6–7).
3. **Der Hauptzweck der Codebasis liegt außerhalb des neuen Projektauftrags.** Die zentralen Funktionen (Nachahmung eines SEB-Clients gegenüber Prüfungsservern, Nachbau der SEB-Oberfläche, KI-gestütztes Ausfüllen von Prüfungsfragen) sind genau die Funktionen, die laut Auftrag nicht zum neuen Projekt gehören (siehe `reuse-assessment.md`).
4. **Befunde nach Schweregrad (für den Fall einer Wiederverwendung, Maßstab in 5.1):** 3 hoch, 8 mittel, 4 niedrig. Zusätzlich sind Verfahrensrisiken in `migration-risk-assessment.md` bewertet.

---

## 2. Bewertung der Vorarbeiten (Arbeitsauftrag 1)

### 2.1 Eignung der Dokumente

| Dokument | Eignung für die neue Initiative | Begründung |
|---|---|---|
| `technical-handover.md` | **Als Bestandsaufnahme geeignet**, als Entwurfsgrundlage **nicht ausreichend** | Aufbau, Datenflüsse, Speicherschlüssel, Nachrichtenkatalog und Abhängigkeiten wurden in Phase 2 gegengeprüft und stimmen (Fehlerquote bei Zeilenangaben ca. 11 von gut 80). Es fehlen: Bedrohungsmodell, Logging/Audit, Lizenz- und Herkunftsklärung, MV3-Machbarkeit, Browser-Matrix (alles NICHT GEPRÜFT bzw. OFFEN). |
| `OEB-Analyse-IT.md` | **Nicht Teil der Entscheidungsgrundlage** | Beschreibt Erkennungsmerkmale für Prüfungsumgebungen. Für das neue Framework ohne Bezug; es enthält wegen seines Inhalts Wissen über die Umgehungsfunktion und sollte nur eingeschränkt zugänglich sein (siehe `reuse-assessment.md`, Kategorie „Zu isolieren"). |
| `verification-report.md` | **Teilweise geeignet** | Belegt die Befunde, enthielt aber einen Fehler (DS-01: „keine Löschfunktion" – falsch, in Revision 2 berichtigt) und ist durch diese Phase in Teilen überholt. |
| `documentation-gaps.md` | **Geeignet als Arbeitsliste** | Die offenen Punkte (Browser-Matrix, Speicherorte, Datenschutz) bleiben offen und sind Voraussetzung für Phase 4. |

### 2.2 Neue Fehler und Lücken in den Vorarbeiten (offen dokumentiert)

| ID | Befund | Beleg | Status | Betroffene Stelle in den Vorarbeiten |
|---|---|---|---|---|
| NF-01 | **Das eingegebene Beenden-Passwort wird per `postMessage(…, "*")` an das Elternfenster gesendet.** Jeder `message`-Listener der Seite kann den Wert lesen. Die Quellenprüfung im Content-Script (`content.js:268`) schützt nur den Empfang, nicht den Inhalt der Nachricht. | `seb/assets/quit.js:25` (Senden); `seb/content.js:250,273-277` (Empfang) | BESTÄTIGT (Code); Lesbarkeit durch Seiten-JS ABGELEITET (Standardverhalten von `postMessage`) | Handover 6.2/11 beschreiben die Quellenprüfung, aber nicht die Offenlegung des Passworts. |
| NF-02 | **Die Firefox-Antwortspiegelung betrifft nicht nur „Downloads ohne Endung".** `isExtensionlessResponse` ist wahr für jede Antwort mit Status 200, die ein Anhang **oder eine Haupt-/Unterframe-Navigation** ist, sofern der letzte Pfadteil der URL bzw. der Dateiname keine Endung hat (z. B. `/` oder `/login`). Für diese wird der Antwortstrom mitgeschnitten und bis 8 MB im Speicher der Hintergrundseite gepuffert und untersucht – bei aktivierter Download-Prüfung, die Standard ist. | `seb/background.js:111-127`; `seb/download-stream.js:3-28,30-41`; `seb/settings.js:2-4` | Codepfad BESTÄTIGT; Auswirkung (Speicher/CPU, Inhalte beliebiger Seiten werden lokal ausgewertet, nicht übertragen) ABGELEITET | Handover 5.1/6.2/11.8 und IT-Analyse nennen nur „Download ohne Endung (Firefox)" bzw. „geeignete Downloads". |
| NF-03 | **Es gibt keinerlei Logging oder Ereignisprotokollierung.** Keine `console.*`-Aufrufe, kein Log-Speicher; Fehler werden nur als letzter Zustand bzw. Statusfeld gehalten. | Suchlauf über `source/`: kein `console.`; Fehlerfelder `errorCode`/`lastError` (16 Treffer) | BESTÄTIGT (Negativbefund) | Nicht dokumentiert; für ein Prüfwerkzeug zentral (Anforderung in `security-testing-requirements.md`). |
| NF-04 | **Der zentrale Nachrichten-Router unterscheidet nicht nach Absender.** `sender` dient nur zur Ermittlung von Tab und URL; kein Absender-Kontext, keine Autorisierung je Nachrichtentyp, keine Schema-Validierung. Alle 20 Typen sind von jedem Erweiterungskontext (Popup, Content-Script) aufrufbar, darunter `SEB_QUIT`, `SEB_UNLOAD`, `SEB_HEADER_ENABLE`, `OEB_AI_RUN`. | `seb/background.js:212-346`, `sender` nur in 219-220, 240-241, 250, 289-317 | BESTÄTIGT | Handover 6.2 hält den Router nur für „von Erweiterungs-Kontexten erreichbar" (ABGELEITET); die fehlende Autorisierung innerhalb der Erweiterung war nicht beschrieben. |
| NF-05 | **Das Zwischenablage-Kopieren der Konfiguration wird nicht zurückgenommen.** Nach 1,2 s wird nur die Schaltflächenbeschriftung zurückgesetzt, der Inhalt der Zwischenablage bleibt. | `seb/popup.js:192-196` | BESTÄTIGT | Handover 11.7 nennt Kopieren/Speichern, nicht das Fehlen einer Bereinigung. |
| NF-06 | **Der eigene XML-Tokenizer arbeitet mit nicht verankerten, lazy Regex-Alternativen** (`<!--[\s\S]*?-->` u. a.). Bei manipulierten Eingaben mit vielen unvollständigen Konstrukten ist superlineares Laufzeitverhalten möglich; die Eingabegröße ist auf mehreren Pfaden nicht begrenzt. | `seb/config-key.js:204-207`; Pfade ohne Limit: `seb/background.js:246-252,411-414`, `seb/parser.js:107-125` | ABGELEITET (nicht getestet) | Handover TR-03 betrachtet nur die Dekompression. |

Damit sind die Phase-2-Befunde (DF/BF/TR/DS) nicht widerlegt, aber um diese sechs Punkte zu ergänzen. Handover §0.6 enthält einen entsprechenden Nachtrag.

### 2.3 Folgerung

Die Dokumentation ist **mit den Ergänzungen NF-01 bis NF-06 eine ausreichende Bestandsaufnahme**. Sie ist **keine ausreichende Grundlage für die Architekturplanung** des neuen Frameworks; dafür fehlen die in `security-testing-requirements.md` genannten Vorgaben (Bedrohungsmodell, Governance, Zielumgebungen).

---

## 3. Prüfumfang und Methode

- **Neu in dieser Phase:** gezielte Suchläufe über den gesamten Quellcode nach Logging, CSP-/`externally_connectable`-Deklarationen, allen `postMessage`-Aufrufen und deren `targetOrigin`, Absender-Nutzung im Router, Schreibsenken (`innerHTML`, `eval` u. ä.) und eine erneute Lektüre der Firefox-Spiegelungslogik.
- **Aus Phase 2 übernommen und stichprobenartig erneut belegt:** Speicherschlüssel, Netzwerkziele, Berechtigungen, Löschpfade.
- **Nicht geprüft:** `popup.css` (914 Zeilen), Bilddateien, Inhalt von `bek.json`, Verhalten zur Laufzeit, Browserdaten auf Datenträger, Lizenzlage der Vorlagenprojekte.
- **Nicht bewertet:** Die Korrektheit der Header-/Schlüsselberechnung gegenüber SEB. Sie ist für die neue Initiative ohne Belang.

---

## 4. Lokal gespeicherte Daten

| Datum | Ort / Schlüssel | Inhalt | Beleg | Sensibilität (Einschätzung) |
|---|---|---|---|---|
| API-Schlüssel, Modell, Basis-URL der KI-Dienste | `storage.local` `oebAiProviders` | Klartext | `assistant/popup.js:371`; `providers.js:98-109` | hoch (Zugangsdaten) |
| Letzter KI-Lauf | `storage.local` `oebAiLastRun` | URL, Fragetexte, Antworten, Bild-`src`, Anbieter, Fehler, Zeit | `assistant/background.js:76-92,101-105`; `extract.js:64` | hoch (Prüfungsinhalte) |
| Nutzernotiz, freigeschaltete Hosts, Kürzel | `storage.local` `oebAiNote`, `oebAiEnabledDomains`, `oebNextPageShortcut` | Freitext, Hostnamen, Tastenkombination | `assistant/popup.js:58,390-394,538` | niedrig bis mittel |
| Einstellungen, BEK-Einstellungen | `storage.local` `sebInspectorSettings`, `oebBekSettings` | Schalter, Profilversion | `seb/background.js:60-61,279` | niedrig |
| Katalog-Cache | `storage.local` `oebBekCatalogCache` | Profildaten, Zeitstempel, Fehlertext | `seb/bek-catalog.js:110,114` | niedrig |
| Zustand mit Klartext-Konfiguration | `storage.session` (Fallback `local`) `sebInspectorState` | entschlüsseltes XML (u. a. Start-URL, Hash des Beenden-Passworts), `sourceUrl` | `seb/background.js:467-475`; `config-key.js:32-49` | mittel bis hoch |
| Pending-Datensatz | `storage.session` (Fallback `local`) `sebInspectorPending` | verschlüsselte Rohbytes; Metadaten inkl. POST-`body` abgefangener Formulare | `seb/background.js:451-453,415-419`; `seb/content.js:57-76` | mittel |
| Ansichtsmerker | `localStorage` des Popups `oebPopupView` | Reitername | `assistant/popup.js:109` | niedrig |
| Exportierte XML-Datei | Dateisystem (auf Nutzeraktion) | Klartext-Konfiguration | `seb/popup.js:198-206` | mittel bis hoch |
| Zwischenablage | System | Klartext-Konfiguration (auf Nutzeraktion) | `seb/popup.js:192-196` | mittel |

**BESTÄTIGT:** Es gibt keine weiteren Speicherschlüssel (Suchlauf). Verschlüsselung gespeicherter Daten findet im Code nicht statt.
**OFFEN:** Ob `storage.session` in Chromium-MV2 verfügbar ist; falls nicht, liegen Zeilen 6 und 7 dauerhaft im Profil (`browser-api.js:3,28`). **NICHT GEPRÜFT:** Speicherformat und -ort im Browserprofil.

---

## 5. Externe Übertragungen

| Ziel | Daten | Auslöser | Credentials | Beleg | Status |
|---|---|---|---|---|---|
| KI-Anbieter (8 feste Hosts, 1 frei wählbarer „Custom") | Fragen (Typ, Text, Optionen, Lückenstruktur), Bilder als Base64 (bis 8 / 4 MB je / 16 MB gesamt), Nutzernotiz im System-Prompt; Nummer und Bild-`src` entfernt | Nutzeraktion; bei Fehler Wiederholung beim nächsten Konto | API-Schlüssel im Header | `providers.js:1-68,194-303,402-415`; `extract.js:4-6` | BESTÄTIGT |
| KI-Anbieter, Modelllisten | nur Schlüssel | Nutzeraktion im Popup | API-Schlüssel | `providers.js:142-183` | BESTÄTIGT |
| GitHub Raw (`…/63kitsune/OEB/…/bek.json`) | keine Nutzdaten | Hintergrundprüfung, 6-h-Schwelle | `credentials: "omit"`, `redirect: "error"` | `seb/bek-catalog.js:2,96-101` | BESTÄTIGT |
| Prüfungsserver | Abruf der Konfiguration, Zweitabruf von Downloads, Moodle-Rücksetzung, Seitenanfragen mit veränderten Headern | Erkennung / Launch / Entladen | Cookies | `seb/background.js:399-411`; `download-check.js:79-84`; `seb/content.js:295`; `seb/headers.js:19-43` | BESTÄTIGT |
| Hosts der Frage-Bilder (beliebig) | Abruf der Bild-URL | Auslesen der Fragen | Cookies (`credentials:'include'`) | `assistant/extract.js:46-51` | BESTÄTIGT |

**BESTÄTIGT:** Keine Telemetrie der Autoren; kein `eval`/`new Function`/dynamisches Nachladen von Code; `innerHTML` wird nur gelesen (`seb/content.js:290`). **NICHT GEPRÜFT:** tatsächlicher Netzwerkverkehr.
**ABGELEITET:** Die Weitergabe von Fragen und Bildern an Dritte ist die datenschutzrelevanteste Übertragung. Aufbewahrung und Verwendung beim Anbieter sind nicht im Code festgelegt (nur bei OpenAI `store: false`, `providers.js:252`) und **NICHT GEPRÜFT**.

---

## 6. Zugangsdaten und API-Schlüssel

| Aspekt | Befund | Beleg | Status |
|---|---|---|---|
| Speicherung der KI-Schlüssel | Klartext in `storage.local`; Eingabefeld `type=password`, `autocomplete=off` | `assistant/popup.js:214-218,371` | BESTÄTIGT |
| Übertragung | je Anfrage im Header (`x-goog-api-key`, `Authorization: Bearer`, `x-api-key`); auch für Modelllisten; bei „Custom" an beliebigen Host, HTTP nur für `localhost`/`127.0.0.1`/`[::1]` | `providers.js:151,235,385-415` | BESTÄTIGT |
| Export / Schutz | keine Export-, Verschlüsselungs- oder Sperrfunktion | Suchlauf | BESTÄTIGT (Negativbefund) |
| Dateipasswort (SEB-Datei) | wird nicht gespeichert; Eingabefeld wird nach dem Absenden geleert; Übergabe per Nachricht an den Hintergrund | `seb/popup.js:120-136`; `background.js:257-258,493-499` | BESTÄTIGT |
| Beenden-Passwort | Eingabe im iframe, **Weitergabe per `postMessage` mit Ziel `*`** (NF-01); lokale Prüfung gegen den Hash aus der Konfiguration | `seb/assets/quit.js:25`; `config-key.js:70-74` | BESTÄTIGT |
| Hash des Beenden-Passworts | im Klartext-XML im Zustand und im Popup sichtbar | `config-key.js:41-43`; `seb/popup.js:310` | BESTÄTIGT |
| Moodle-Sitzungsschlüssel (`sesskey`) | wird im Content-Script aus dem DOM gelesen (Formularfeld, Logout-Link, Seitenquelltext) und im POST an denselben Ursprung gesendet | `seb/content.js:285-300` | BESTÄTIGT |
| Weitere Geheimnisse | Formular-`body` abgefangener SEB-Formulare im Pending-Datensatz | `seb/background.js:451-453` | BESTÄTIGT |

**Bewertung (ABGELEITET):** Das Modell „Schlüssel im Profil, Klartext, unbegrenzte Lebensdauer" ist für ein Werkzeug mit Laborbezug nicht tragfähig. Das neue Framework sollte **keine** Fremd-API-Schlüssel verarbeiten (Anforderung SR-NF-05).

---

## 7. Löschfunktionen: Was greift tatsächlich?

| Datum | Löschweg im Code | Greift? | Beleg | Lücke |
|---|---|---|---|---|
| Klartext-Konfiguration (`state.xml`) | `unloadConfiguration` überschreibt den Zustand mit „idle" | **Ja**, ausgelöst durch Beenden, „Unload", Ersetzen, auch durch `onInstalled` | `seb/background.js:615-637,59-63` | Nicht bei Browserende im `local`-Fallback (OFFEN); Reste auf Datenträger NICHT GEPRÜFT |
| Pending-Datensatz | `storage.session.remove` | **Ja**, beim Ersetzen, Entsperren, Fehler, Entladen | `seb/background.js:466,500,526,627` | Bleibt bei falschem Passwort bestehen (gewollt) |
| KI-Konto samt Schlüssel | „Remove" + **„Save"** | **Ja, erst nach „Save"**; schließt man das Popup vorher, bleibt der Eintrag erhalten | `assistant/popup.js:183,359-372` | Kein sicheres Überschreiben, keine Sammel-Löschung |
| Domain-Freigabe | Schalter aus löscht den Eintrag | **Ja** | `assistant/popup.js:390-394` | – |
| Nutzernotiz | Leeren speichert `""` | **Ja** (Wert), Schlüssel bleibt | `assistant/popup.js:58` | – |
| Letzter KI-Lauf | keiner; wird nur beim nächsten Lauf überschrieben | **Nein** | `assistant/background.js:101-105`; `assistant/popup.js` nur lesend (Z. 562, 579) | Prüfungsinhalte bleiben dauerhaft |
| Katalog-Cache, Einstellungen | keiner | **Nein** | – | Unkritisch (öffentliche Profildaten, Schalter) |
| Zwischenablage | nur Beschriftung wird zurückgesetzt | **Nein** | `seb/popup.js:192-196` | NF-05 |
| Exportierte `.xml`-Datei | außerhalb der Kontrolle der Erweiterung | n. a. | `seb/popup.js:198-206` | Nutzerverantwortung |
| Alle Daten gesamt | keine Funktion | **Nein** | Suchlauf | Es gibt keine „Alles löschen"-Funktion |
| Deinstallation | Browserverhalten | NICHT GEPRÜFT | – | – |

---

## 8. Berechtigungen

| Berechtigung | Genutzt für | Bewertung nach dem Prinzip der minimalen Rechte | Beleg |
|---|---|---|---|
| `<all_urls>` | Hooks und Content-Scripts auf allen Seiten | **Zu breit** für ein Laborwerkzeug; das Ziel (Prüfungsursprung) ist erst zur Laufzeit bekannt; Einschränkung erfolgt nur im Code | Manifeste Z. 11-18; `seb/headers.js:21` |
| `webRequest` + `webRequestBlocking` | Header-Änderung (`onBeforeSendHeaders`), Antwortauswertung/Abbruch (`onHeadersReceived`) | **Kritisch**: Änderung und Abbruch fremden Verkehrs; MV2-spezifisch | `seb/headers.js:19-43`; `seb/background.js:95-107` |
| `tabs` | Tabs abfragen, navigieren, neu laden, schließen, Nachrichten senden | Breit; nötig für die Orchestrierung, im Labor enger fassbar | `seb/background.js:157,586-613,615-637` |
| `downloads` | nur `onCreated`-Listener | Eng genutzt; gleichwohl breite Berechtigung | `seb/background.js:72` |
| `storage` | `local`, `session` | Angemessen | – |
| Content-Scripts, `all_frames`, `document_start` | Abfangen von Klicks/Formularen auf **jeder** Seite; Download-Brücke; Tastenkürzel | Breit; Listener laufen dauerhaft auf allen Seiten | Manifeste Z. 39-58 / 36-66; `seb/content.js:34-77` |
| `web_accessible_resources` | 12 (Firefox) bzw. 13 (Chromium) Ressourcen | Jede Webseite kann diese Seiten einbetten bzw. abrufen, sofern sie die Erweiterungs-URL kennt | Manifeste |
| Firefox `data_collection_permissions` | `websiteContent`, `authenticationInfo` als erforderlich deklariert | Entspricht dem Zugriffsumfang | `firefox.manifest.json:85-90` |
| Nicht vorhanden | `content_security_policy`, `externally_connectable`, `optional_permissions`, Host-Beschränkungen | Es gilt die Standard-CSP des Browsers (ABGELEITET); keine zur Laufzeit gestaffelten Rechte | Suchlauf über beide Manifeste (BESTÄTIGT) |

**ABGELEITET:** Das Rechteprofil ist auf Eingriff in fremden Webverkehr ausgelegt. Es ist mit einem Laborwerkzeug nach dem Prinzip minimaler Rechte nicht vereinbar.

---

## 9. Sicherheits- und Architekturrisiken

### 9.1 Bewertungsmaßstab

Die Schweregrade gelten für den Fall einer **Wiederverwendung** der Codebasis (nicht als CVSS-Wert):
- **Hoch:** Offenlegung von Zugangsdaten oder Prüfungsinhalten an Dritte, beziehungsweise Beeinflussung durch Fremdcode ohne Nutzeraktion, oder ein strategisches Ausschlusskriterium.
- **Mittel:** Beeinträchtigung von Vertraulichkeit/Integrität unter zusätzlichen Bedingungen oder erheblicher Mehraufwand.
- **Niedrig:** begrenzte Auswirkung oder geringe Wahrscheinlichkeit.

### 9.2 Risikoregister

| ID | Risiko | Quelle | Schweregrad | Status | Beleg |
|---|---|---|---|---|---|
| SEC-01 | Seiten-JavaScript kann über den `postMessage`-Kanal Konfigurationsbytes übergeben und das Laden/Starten anstoßen; die Herkunft wird nicht geprüft, der Inhalt schon; Standard „an" | TR-04 | **Hoch** | Kette BESTÄTIGT, Wirkung ABGELEITET | `download-content.js:16-31`; `seb/background.js:188-206,229-245`; `settings.js:2-4` |
| SEC-02 | Manifest V2 mit persistenter Hintergrundseite und blockierendem `webRequest`; Fortbestand im Chromium-Umfeld ungeklärt | TR-09 | **Hoch** (strategisch) | OFFEN | Manifeste; `README.md` |
| SEC-03 | API-Schlüssel im Klartext, an frei wählbare Hosts übertragbar | DS-01 | **Hoch** | BESTÄTIGT | siehe Abschnitt 6 |
| SEC-04 | Prüfungsinhalte und Klartext-Konfiguration dauerhaft bzw. ungeschützt gespeichert; keine „Alles löschen"-Funktion | DS-03/06 | Mittel | BESTÄTIGT | Abschnitte 4 und 7 |
| SEC-05 | Beenden-Passwort per `postMessage "*"` offengelegt | NF-01 | Mittel | BESTÄTIGT / ABGELEITET | `quit.js:25` |
| SEC-06 | Fernaktualisierung von Profildaten ohne Authentizitätsprüfung (Prüfsumme in derselben Datei, Drittrepository) | TR-07 | Mittel | Mechanismus BESTÄTIGT | `bek-catalog.js:2,27-29,100-111` |
| SEC-07 | Firefox: Spiegelung und Auswertung von Antwortkörpern aller Navigationen ohne Dateiendung | NF-02 | Mittel | Pfad BESTÄTIGT, Wirkung ABGELEITET | `background.js:111-127`; `download-stream.js:30-41` |
| SEC-08 | Router ohne Absender-Autorisierung und ohne Schema-Validierung | NF-04 | Mittel | BESTÄTIGT | `background.js:212-346` |
| SEC-09 | Keine Logging- und Auditfunktion | NF-03 | Mittel | BESTÄTIGT | Suchlauf |
| SEC-10 | Keine Tests, keine CI, keine Lizenz, kein Changelog; Herkunft aus Vorlageprojekten | TR-08 | Mittel | BESTÄTIGT | Dateiliste; `README.md:7` |
| SEC-11 | Zu breites Rechteprofil (`<all_urls>`, Content-Scripts in allen Frames, Eingriff in Header) | DS-07 | Mittel | BESTÄTIGT | Abschnitt 8 |
| SEC-12 | Bildabrufe mit Cookies von beliebigen Hosts | DS-05 | Niedrig | BESTÄTIGT | `extract.js:46-51` |
| SEC-13 | Unbegrenzte Dekompression / superlineares Parsen bei manipulierten Dateien | TR-03, NF-06 | Niedrig | ABGELEITET | `parser.js:107-125`; `config-key.js:204-207` |
| SEC-14 | Teilfehler und nicht serialisierte Zustandsänderungen; `onInstalled` verwirft Zustand | TR-01/02/10 | Niedrig | ABGELEITET / BESTÄTIGT | `background.js:59-63,493-538,572-584,615-637` |
| SEC-15 | Frame-Zuordnung per DOM-ID | TR-05 | Niedrig | ABGELEITET | `seb/content.js:251-253,268` |

### 9.3 Positive Befunde

Der Code enthält auch Ansätze, die für ein Folgeprojekt als Anforderungen übernommen werden sollten (Konzepte, nicht Code):
- BESTÄTIGT: keine Schreibsenken für HTML (`innerHTML` nur lesend), kein `eval`, keine Drittbibliotheken zur Laufzeit.
- BESTÄTIGT: Größen- und Zeitlimits auf den automatischen Erkennungswegen (`download-check.js:3`, `download-content.js:22,47`).
- BESTÄTIGT: Fehler im Router werden als `{ok:false, error}` zurückgegeben (`background.js:65-70`).
- BESTÄTIGT: HTTPS-Pflicht für entfernte Endpunkte (`providers.js:394-398`); Katalogabruf ohne Cookies und ohne Weiterleitungen (`bek-catalog.js:100`).
- BESTÄTIGT: Abbruch bei geänderter Seite während einer Anfrage (`assistant/background.js:58-65`).
- BESTÄTIGT: injizierbare Abhängigkeiten in `createBekCatalogManager` und `probeSebDownload` (Testbarkeit, `bek-catalog.js:64`, `download-check.js:79`).

---

## 10. Datenschutzprobleme, die vor einer Wiederverwendung beseitigt werden müssten

Voraussetzungen („Blocker") für die Weiterverwendung **irgendeines** Teils der Codebasis:

| Nr. | Voraussetzung | Bezug |
|---|---|---|
| P-1 | Keine Übertragung von Seiteninhalten, Fragen oder Bildern an externe Dienste; keine Fremd-API-Schlüssel im Werkzeug | DS-01, DS-04 |
| P-2 | Keine Speicherung von Seiteninhalten, Formularfeldern, Passwörtern oder Sitzungstoken; Testdaten ausschließlich synthetisch | DS-02, DS-03, DS-06 |
| P-3 | Kein Mitschnitt oder Zweitabruf fremder Antworten und keine Cookie-behafteten Abrufe fremder Hosts | NF-02, DS-05 |
| P-4 | Vollständige Lösch- und Aufbewahrungsfunktionen mit nachweisbarer Wirkung, einschließlich Zwischenablage | Abschnitt 7 |
| P-5 | Ereignisprotokoll mit Datenminimierung (keine Geheimnisse und keine Rohinhalte) | NF-03 |
| P-6 | Klärung der Lizenz-, Marken- und Herkunftslage (Vorlageprojekte, SEB-Logos in `seb/icons/*`) | TR-08 |
| P-7 | Datenschutzfolgenabschätzung bzw. Freigabe durch den Datenschutz vor dem ersten Einsatz | – |

Die Punkte P-1 bis P-3 sind bei der bestehenden Codebasis **nicht durch Änderung einzelner Stellen** erreichbar, weil die Datenverarbeitung ihr Zweck ist (siehe `reuse-assessment.md`).

---

## 11. Offene Fragen

| Nr. | Frage | Status |
|---|---|---|
| Q-1 | Verfügbarkeit und Lebensdauer von `storage.session` je Browser; Folge für 4 | OFFEN |
| Q-2 | Tatsächlicher Netzwerkverkehr (Katalog-Abrufzeitpunkte, Cookies bei Bildabrufen, Header bei Zweitabrufen) | NICHT GEPRÜFT |
| Q-3 | Speicherorte und -formate der Erweiterungsdaten im Browserprofil; Reste nach Löschung/Deinstallation | NICHT GEPRÜFT |
| Q-4 | Aufbewahrung und Nutzung der übertragenen Inhalte bei den KI-Anbietern | NICHT GEPRÜFT (außerhalb des Codes) |
| Q-5 | Lizenz der Vorlageprojekte; Rechtslage der enthaltenen SEB-Marken-/Logodateien | NICHT GEPRÜFT (Rechtsabteilung) |
| Q-6 | MV2-/MV3-Unterstützung der im Haus eingesetzten Browser | NICHT GEPRÜFT |
| Q-7 | Auswirkung von NF-02 auf Leistung und Datenschutz in Firefox im Realbetrieb | OFFEN |
| Q-8 | Robustheit des Parsers gegen manipulierte Dateien (SEC-13) | OFFEN |
