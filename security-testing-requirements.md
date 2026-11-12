# Anforderungen an das Security-Testing-Framework (Entwurf zur Freigabe)

**An:** IT-Leitung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** Entwurf, **nicht freigegeben** · **Klassifizierung:** intern
**Herleitung:** `security-privacy-audit.md` (Befunde SEC-xx, NF-xx) und `reuse-assessment.md`.
**Wichtiger Hinweis:** Dieses Dokument beschreibt **Anforderungen**, keine Architektur und keine Implementierung. Architektur- und Technologieentscheidungen (z. B. Erweiterung oder externer Prüfstand) werden erst nach Freigabe getroffen (Abschnitt 10).
**Prioritäten:** **MUSS** (Voraussetzung für Freigabe) · **SOLL** (wichtig, begründete Ausnahme möglich) · **KANN** (wünschenswert).
**Kennzeichnung der Herkunft:** *(Befund)* = abgeleitet aus bestätigten Auditbefunden; *(Auftrag)* = aus dem Projektauftrag; *(Vorschlag)* = fachliche Empfehlung des Analysten, noch zu bestätigen.

---

## 1. Zweck, Geltungsbereich, Abgrenzung

### 1.1 Zweck
Das Framework soll Browser-Sicherheitsmaßnahmen der IT-Abteilung **in einer kontrollierten Laborumgebung** systematisch, reproduzierbar und nachvollziehbar überprüfen und die Ergebnisse in Prüfberichten dokumentieren.

### 1.2 Zulässige Prüfobjekte (Geltungsbereich)
Ausschließlich Objekte, die der IT-Abteilung gehören oder für die eine schriftliche Prüfgenehmigung vorliegt:
- (a) zentral verwaltete Browser-Konfigurationen und -Richtlinien (z. B. Erweiterungs-Allowlist, Entwicklermodus, Download-/Cookie-Einstellungen),
- (b) eigene oder freigegebene Browser-Erweiterungen und speziell entwickelte, **harmlose** Testerweiterungen,
- (c) eigene Webanwendungen und Testserver im Labor, die synthetische Daten liefern,
- (d) das Framework selbst (Selbsttest, Abschnitt 4.9).

### 1.3 Ausschlusskatalog (ausdrücklich nicht Teil des Projekts)
Diese Funktionen dürfen **weder implementiert noch vorbereitet** werden *(Auftrag)*:

| Nr. | Ausgeschlossen |
|---|---|
| AK-01 | Nachahmung der Identität oder Integritätsmerkmale fremder Clients oder Prüfsoftware (insbesondere Safe Exam Browser): Fälschen von User-Agent, Config-Key-/Browser-Exam-Key-Headern oder vergleichbarer Nachweise gegenüber fremden Servern |
| AK-02 | Erfassung, Entschlüsselung oder Nutzung von Prüfungskonfigurationen (`.seb` u. ä.) |
| AK-03 | Nachbau von Bedienoberflächen fremder Prüfsoftware (Start-Animation, Taskleiste, Beenden-Dialog) oder Verwendung fremder Marken-/Logo-Assets |
| AK-04 | Automatisches Auslesen, Beantworten oder Eintragen von Prüfungsfragen, Anbindung von KI-Diensten für solche Zwecke |
| AK-05 | Tests gegen reale Prüfungsplattformen oder produktive Systeme Dritter |
| AK-06 | Verschleierung, Tarnung oder Umgehung von Erkennungs-, Protokollierungs- oder Schutzmechanismen |
| AK-07 | Mitschnitt oder Zweitabruf fremder Antworten, Cookies oder Sitzungsdaten außerhalb von Laborzielen |
| AK-08 | Übertragung von Testinhalten an externe Dienste (insbesondere KI-Anbieter) |
| AK-09 | Automatische Ausführung ohne Autorisierungsnachweis (Abschnitt 2) |

Die **Erkennung** der Nutzung von Umgehungswerkzeugen bei Prüfungen (Gegenstand von `OEB-Analyse-IT.md`) ist **kein** Bestandteil dieses Projekts. Sie bildet einen eigenen Arbeitsstrang „Prüfungsintegrität" und bedürfte eines gesonderten Auftrags und Rahmens.

### 1.4 Annahmen und Abhängigkeiten
- **A-1** Die IT-Leitung benennt eine verantwortliche Stelle für Prüfgenehmigungen (Abschnitt 2). *(Vorschlag)*
- **A-2** Datenschutz und Rechtsabteilung sind vor dem ersten Einsatz beteiligt. *(Befund P-7)*
- **A-3** Die eingesetzten Zielbrowser und ihre Versionen sind benannt (heute NICHT GEPRÜFT, Q-6).
- **A-4** Lizenz-, Marken- und Herkunftsfragen der Vorlageprojekte sind geklärt, bevor Code übernommen wird (Q-5).

---

## 2. Governance und Autorisierung

| ID | Anforderung | Priorität | Herkunft |
|---|---|---|---|
| GOV-01 | Jede Testausführung ist an einen **Prüfauftrag** gebunden (Auftraggeber, Zweck, Prüfobjekte, Zeitfenster, Verantwortliche). Ohne gültigen Auftrag startet kein Test. | MUSS | Auftrag |
| GOV-02 | Die erlaubten Ziele sind in einer **Scope-Datei** (Allowlist von Hosts/Ursprüngen/Browserprofilen) festgelegt, versioniert und signiert oder gegengezeichnet. Das Framework verweigert Verbindungen und Aktionen außerhalb der Allowlist. | MUSS | Auftrag; Konzept aus `assistant/background.js:44-46` |
| GOV-03 | Die Scope-Datei enthält ausschließlich Laborziele; **Produktivdomänen und Prüfungsplattformen sind per Regel gesperrt** (Negativliste). | MUSS | Auftrag (AK-05) |
| GOV-04 | Vier-Augen-Prinzip für Freigabe von Scope-Dateien und Testkatalog-Änderungen. | SOLL | Vorschlag |
| GOV-05 | Zeitliche Begrenzung der Prüfaufträge; nach Ablauf sind Tests gesperrt. | SOLL | Vorschlag |
| GOV-06 | Technische **Sperre des Ausschlusskatalogs** (AK-01 bis AK-09): Build- und Review-Prüfungen verhindern das Einbringen entsprechender Fähigkeiten (siehe 6.3, NT-01 bis NT-06). | MUSS | Auftrag |
| GOV-07 | Rollenmodell: Prüfer, Freigeber, Berichtsleser, Administrator; Zugriff auf Ergebnisse nach Rolle. | SOLL | Vorschlag |
| GOV-08 | Not-Aus („kill switch"), der laufende Tests abbricht und Verbindungen schließt. | MUSS | Vorschlag |

---

## 3. Anforderungen an die Laborumgebung

| ID | Anforderung | Priorität | Herkunft |
|---|---|---|---|
| LAB-01 | Isolierte Umgebung (VM oder Container) ohne Zugriff auf Produktivnetze; ausgehender Verkehr nur zu Laborzielen (Egress-Allowlist, DNS nur intern). | MUSS | Auftrag |
| LAB-02 | Jede Testausführung beginnt mit einem **sauberen, definierten Browserprofil** (Snapshot/Wiederherstellung) und festgelegter Browserversion; Version und Konfiguration werden im Ergebnis festgehalten. | MUSS | Auftrag (Reproduzierbarkeit) |
| LAB-03 | Testserver mit **synthetischen** Daten und Benutzerkonten; keine echten personenbezogenen Daten. | MUSS | Befund P-2 |
| LAB-04 | Definierte Browser-Matrix (Browser, Version, Betriebssystem, Richtlinienstand); Pflege und Freigabe der Matrix. | MUSS | Befund Q-6 |
| LAB-05 | Zeitsynchronisation aller Komponenten (für Korrelation von Ereignissen). | SOLL | Vorschlag |
| LAB-06 | Netzwerkmitschnitt im Labor (für Datenschutz- und Egress-Tests), mit Zugriffsschutz und Aufbewahrungsfrist. | SOLL | Befund Q-2 |
| LAB-07 | Ergebnis- und Beweisspeicher mit Zugriffsbeschränkung und Integritätsschutz (Hashes). | MUSS | Vorschlag |
| LAB-08 | Dokumentierte Rücksetzung der Umgebung nach jedem Lauf, einschließlich Löschung von Testdaten und Zwischenablage. | MUSS | Befund NF-05 |

---

## 4. Funktionale Anforderungen

Jede Fähigkeit wird zuerst **am Framework selbst und an harmlosen Testobjekten** nachgewiesen (4.9).

### 4.1 Untersuchung von Browser-Sicherheitsmechanismen

| ID | Anforderung | Priorität |
|---|---|---|
| FR-SM-01 | Prüfen der wirksamen Browserrichtlinien gegen eine Soll-Baseline (z. B. Erweiterungsinstallation, Entwicklermodus, temporäre Add-ons) und Ausgabe der Abweichungen. | MUSS |
| FR-SM-02 | Prüfen, ob Browser-Schutzmechanismen in Testseiten wirken: Content-Security-Policy, Same-Origin-Verhalten, Cookie-Attribute (Secure/HttpOnly/SameSite), Frame-Einbettung, Mixed Content, Berechtigungs-Header. | SOLL |
| FR-SM-03 | Bereitstellung harmloser Testerweiterungen („Canary") zur Prüfung, ob Richtlinien deren Installation/Aktivierung verhindern oder protokollieren. | SOLL |

### 4.2 Berechtigungsprüfung

| ID | Anforderung | Priorität |
|---|---|---|
| FR-PM-01 | Inventarisierung der Berechtigungen einer zu prüfenden Erweiterung aus ihrem Manifest (Host-, API-, optionale Rechte, Content-Script-Reichweite, freigegebene Ressourcen). | MUSS |
| FR-PM-02 | Vergleich mit einer Baseline der Minimalrechte und Ausgabe der Überschreitungen mit Begründungsfeld. | MUSS |
| FR-PM-03 | Erkennen von Rechteänderungen zwischen zwei Versionen (Differenzbericht). | SOLL |
| FR-PM-04 | Hinweis auf Plattformabhängigkeiten (Manifest-Version, auslaufende APIs). | SOLL |

### 4.3 Prüfung der Nachrichtenkommunikation

| ID | Anforderung | Priorität |
|---|---|---|
| FR-MC-01 | Aufzählen der Nachrichtenkanäle eines Prüfobjekts (Laufzeitnachrichten, `postMessage`, externe Verbindungen) mit Typen, Sendern und Empfängern. | MUSS |
| FR-MC-02 | Testfälle für Absender-/Ursprungsprüfung: akzeptiert der Empfänger Nachrichten aus nicht berechtigten Kontexten? | MUSS |
| FR-MC-03 | Testfälle für `targetOrigin`-Verwendung bei sensiblen Nachrichteninhalten (Befund NF-01). | MUSS |
| FR-MC-04 | Testfälle für Schema-Validierung (unbekannte Typen, fehlende Felder, Übergröße). | SOLL |
| FR-MC-05 | Prüfung, ob Seiten-JavaScript Aktionen der Erweiterung auslösen kann (Befund SEC-01) – ausschließlich an Labor-Testobjekten. | SOLL |

### 4.4 Bewertung der Eingabevalidierung

| ID | Anforderung | Priorität |
|---|---|---|
| FR-IV-01 | Grenzwert- und Fuzzing-Tests der Parser und Eingabeschnittstellen des Frameworks und der Prüfobjekte mit **synthetischen Eingaben**. | MUSS |
| FR-IV-02 | Prüfung von Größen-, Zeit- und Schachtelungsgrenzen auf **allen** Pfaden (Befund SEC-13). | MUSS |
| FR-IV-03 | Prüfung auf HTML-/Skript-Senken (`innerHTML` u. ä.) im Prüfobjekt. | SOLL |
| FR-IV-04 | Messung von Laufzeit und Speicherbedarf bei manipulierten Eingaben (Befund NF-06). | SOLL |

### 4.5 Untersuchung von Datenschutz und Speicherverhalten

| ID | Anforderung | Priorität |
|---|---|---|
| FR-DP-01 | Speicherinventur: welche Schlüssel/Dateien legt ein Prüfobjekt an (Erweiterungsspeicher, `localStorage`, Cookies, Cache), mit Inhaltsklassifikation. | MUSS |
| FR-DP-02 | **Canary-Daten-Verfahren:** Einspeisen eindeutiger synthetischer Kennungen und Suche, wo sie gespeichert oder übertragen werden. | MUSS |
| FR-DP-03 | Prüfung der Löschfunktionen: verschwinden die Daten nachweislich (Speicher und, soweit prüfbar, Datenträger)? | MUSS |
| FR-DP-04 | Egress-Inventar: alle Verbindungsziele, Auslöser und übertragenen Datenkategorien (Befund Q-2). | MUSS |
| FR-DP-05 | Prüfung, ob Geheimnisse (Schlüssel, Passwörter, Token) im Klartext persistiert werden (Befund SEC-03). | MUSS |
| FR-DP-06 | Prüfung der Lebensdauer von Sitzungsspeichern über Neustart/Reload der Erweiterung (Befund Q-1). | SOLL |

### 4.6 Reproduzierbare Testfälle

| ID | Anforderung | Priorität |
|---|---|---|
| FR-TC-01 | Testfälle in einem **maschinenlesbaren, versionierten Format** (ID, Titel, Ziel, Voraussetzungen, Schritte, erwartetes Ergebnis, Zuordnung zu Anforderung/Risiko, Prüfobjekt, Schweregrad bei Abweichung). | MUSS |
| FR-TC-02 | Deterministische Ausführung: festgelegte Browserversion, Profil, Eingabedaten, Zufallszahl-Startwerte; Idempotenz (wiederholbare Läufe liefern gleiche Ergebnisse). | MUSS |
| FR-TC-03 | Jeder Lauf erzeugt Metadaten: Framework-Version, Testkatalog-Version, Umgebungsbeschreibung, Prüfauftrag, Start/Ende. | MUSS |
| FR-TC-04 | Trennung von Testkatalog, Testdaten und Ausführungslogik; Testdaten sind synthetisch und versioniert. | MUSS |
| FR-TC-05 | Regressionsläufe gegen frühere Ergebnisse (Differenz). | SOLL |
| FR-TC-06 | Klare Ergebnisstatus je Testfall: bestanden, nicht bestanden, nicht ausführbar, übersprungen (mit Grund). | MUSS |

### 4.7 Protokollierung von Sicherheitsereignissen

| ID | Anforderung | Priorität |
|---|---|---|
| LOG-01 | Strukturiertes Ereignisschema (Zeit, Quelle, Ereignistyp, Schweregrad, Prüfauftrag, Testfall, Korrelations-ID, Ergebnis). | MUSS |
| LOG-02 | **Datenminimierung:** keine Geheimnisse, keine Rohinhalte von Seiten, keine personenbezogenen Daten in Protokollen. | MUSS |
| LOG-03 | Manipulationsschutz (z. B. Verkettung von Prüfsummen) und getrennte Ablage vom Prüfobjekt. | SOLL |
| LOG-04 | Protokollierung auch von Fehlern und verweigerten Aktionen (Scope-Verstöße, Autorisierungsfehler); **keine stillen Fehler** (Befund NF-03). | MUSS |
| LOG-05 | Aufbewahrungsfristen und Löschkonzept für Protokolle. | MUSS |
| LOG-06 | Export in ein gängiges Format zur Weiterverarbeitung (z. B. JSON-Lines). | SOLL |

### 4.8 Prüfberichte

| ID | Anforderung | Priorität |
|---|---|---|
| REP-01 | Berichtsvorlage mit: Auftrag, Umfang, Umgebung, Methode, Ergebnisse je Testfall, Befunde mit Beleg, Risiko, Empfehlung, offene Punkte. | MUSS |
| REP-02 | Befunde tragen eine eindeutige ID und eine **Evidenzklasse** (BESTÄTIGT / ABGELEITET / OFFEN / NICHT GEPRÜFT, wie in den Vorarbeiten). | MUSS |
| REP-03 | Schweregrad nach dokumentiertem Maßstab; die Methode ist im Bericht erläutert. | MUSS |
| REP-04 | Belege (Auszüge, Hashes, Konfiguration) sind dem Bericht beigefügt oder referenziert; sensible Inhalte sind geschwärzt. | MUSS |
| REP-05 | Ausgabeformate: lesbarer Bericht (Dokument) und maschinenlesbare Fassung. | SOLL |
| REP-06 | Reproduktionsangaben: Welche Schritte erzeugen den Befund erneut? | MUSS |
| REP-07 | Versionierung und Freigabestatus des Berichts; Verteilerliste nach Rolle. | SOLL |

### 4.9 Selbsttest des Frameworks

| ID | Anforderung | Priorität |
|---|---|---|
| FR-ST-01 | Das Framework prüft sich selbst mit den Verfahren aus 4.2 bis 4.5 (Berechtigungen, Nachrichten, Speicher, Egress) und veröffentlicht die Ergebnisse. | MUSS |

---

## 5. Nicht-funktionale Anforderungen

| ID | Anforderung | Priorität | Herkunft |
|---|---|---|---|
| SR-NF-01 | Unterstützte, nicht auslaufende Plattform (bei Browser-Erweiterung: Manifest V3); Entscheidung zur Architektur erst nach Freigabe. | MUSS | Befund SEC-02 |
| SR-NF-02 | Minimale Rechte; keine pauschalen Host-Rechte; zusätzliche Rechte nur optional und zur Laufzeit, begründet je Recht. | MUSS | Befund SEC-11 |
| SR-NF-03 | Keine HTML-/Skript-Senken; strikte Inhaltssicherheitsrichtlinie; Darstellung nur über sichere Textzuweisung. | MUSS | Befund 9.3 |
| SR-NF-04 | Größen-, Zeit- und Tiefenlimits auf **jedem** Eingabepfad; Ressourcenbegrenzung (Speicher/CPU). | MUSS | Befund SEC-13 |
| SR-NF-05 | Keine Fremd-API-Schlüssel und keine Anbindung externer KI-Dienste; keine Übertragung von Testinhalten an Dritte. | MUSS | Befund SEC-03, AK-08 |
| SR-NF-06 | Kein Nachladen von Code; Konfigurations-/Datenupdates nur authentifiziert (Signatur) und aus eigener Infrastruktur. | MUSS | Befund SEC-06 |
| SR-NF-07 | Nachrichten: Absender-Autorisierung je Nachrichtentyp, Schema-Validierung, niemals sensible Daten mit offenem `targetOrigin`. | MUSS | Befund SEC-05, SEC-08 |
| SR-NF-08 | Geheimnisse werden nicht im Klartext persistiert; wo unvermeidlich, mit Schutzmechanismus des Betriebssystems und kurzer Lebensdauer. | MUSS | Befund SEC-03 |
| SR-NF-09 | Lieferkette: gesperrte Abhängigkeiten, Software-Stückliste (SBOM), regelmäßige Prüfung auf bekannte Schwachstellen, signierte Releases, nachvollziehbare Builds (nicht nur minifizierte Artefakte). | MUSS | Befund TR-08 |
| SR-NF-10 | Qualitätssicherung: automatisierte Tests, statische Analyse, CI, Code-Review; Mindestabdeckung wird vom Projekt festgelegt (Zielwert OFFEN). | MUSS | Befund SEC-10 |
| SR-NF-11 | Lizenz, Herkunft und Drittassets sind dokumentiert und freigegeben; keine fremden Marken-/Logo-Assets. | MUSS | Befund P-6 |
| SR-NF-12 | Datenschutz durch Technikgestaltung: Datenminimierung, Zweckbindung, Aufbewahrungsfristen, **vollständige Löschfunktion** inkl. Zwischenablage. | MUSS | Befund P-4 |
| SR-NF-13 | Fehler sichtbar und protokolliert; kein „fail-open" bei sicherheitsrelevanten Prüfungen. | MUSS | Befund NF-03 |
| SR-NF-14 | Wartbarkeit: dokumentierte Zustands- und Nachrichtenmodelle, Befund-ID-Schema, Änderungsprotokoll (Vorlage aus den Vorarbeiten). | SOLL | reuse-assessment 3.9 |
| SR-NF-15 | Bedienbarkeit und Barrierefreiheit der Oberfläche. | KANN | Vorschlag |

---

## 6. Qualitätssicherung und Ausschlusskontrolle

### 6.1 Abnahmekriterien (Auszug)
- Alle MUSS-Anforderungen sind umgesetzt und nachgewiesen.
- FR-ST-01 (Selbsttest) ist ohne offene Befunde der Schwere „hoch" abgeschlossen.
- Ein vollständiger Testlauf ist in einer frischen Laborumgebung **reproduzierbar** (gleiche Ergebnisse bei Wiederholung).
- Ein Prüfbericht nach REP-01 bis REP-06 liegt vor und wurde von Datenschutz geprüft.

### 6.2 Rückverfolgbarkeit Auditbefund → Anforderung

| Auditbefund | Anforderungen |
|---|---|
| SEC-01 | FR-MC-05, SR-NF-07 |
| SEC-02 | SR-NF-01 |
| SEC-03 | SR-NF-05, SR-NF-08, FR-DP-05 |
| SEC-04 | SR-NF-12, FR-DP-01, FR-DP-03, LAB-08 |
| SEC-05 (NF-01) | FR-MC-03, SR-NF-07 |
| SEC-06 | SR-NF-06 |
| SEC-07 (NF-02) | AK-07, FR-DP-04 |
| SEC-08 (NF-04) | FR-MC-02, FR-MC-04, SR-NF-07 |
| SEC-09 (NF-03) | LOG-01 bis LOG-06, SR-NF-13 |
| SEC-10 | SR-NF-09, SR-NF-10, SR-NF-11 |
| SEC-11 | SR-NF-02, FR-PM-01, FR-PM-02 |
| SEC-12 | AK-07, FR-DP-04 |
| SEC-13 (NF-06) | FR-IV-01, FR-IV-02, FR-IV-04, SR-NF-04 |
| SEC-14 | SR-NF-13, FR-TC-02 |
| SEC-15 | FR-MC-02, SR-NF-07 |
| P-1 bis P-7 | AK-04, AK-07, AK-08, SR-NF-05, SR-NF-11, SR-NF-12, GOV-01 bis GOV-03, A-2 |

### 6.3 Negativtests (Ausschlusskontrolle)
Der Build und die Review-Pipeline weisen nach, dass verbotene Fähigkeiten fehlen (GOV-06):

| ID | Prüfung | Bezug |
|---|---|---|
| NT-01 | Das Manifest fordert weder blockierendes Netzwerk-Eingreifen noch Host-Rechte außerhalb der Scope-Allowlist. | AK-01, AK-07 |
| NT-02 | Der Quellcode enthält keinen Zugriff auf die Header-Felder `User-Agent` oder herstellerspezifische Integritätsheader im Sinne einer Veränderung ausgehender Anfragen. | AK-01 |
| NT-03 | Keine Netzwerkaufrufe zu Zielen außerhalb der Scope-Allowlist; keine Anbindung bekannter KI-Dienste. | AK-04, AK-08 |
| NT-04 | Kein Code zum Auslesen oder Ausfüllen fremder Formulare oder zum Erfassen von Konfigurationsdateien. | AK-02, AK-04 |
| NT-05 | Keine Marken-/Logo-Assets Dritter im Paket. | AK-03 |
| NT-06 | Kein Nachladen ausführbaren Codes zur Laufzeit. | SR-NF-06 |

Die Negativtests sind selbst Teil des Testkatalogs und werden bei jedem Build ausgeführt.

---

## 7. Offene Entscheidungen für die IT-Leitung

| Nr. | Entscheidung | Wirkung |
|---|---|---|
| ED-1 | Benennung der verantwortlichen Stelle für Prüfgenehmigungen und Freigaben (GOV-01 bis GOV-04) | Voraussetzung für jeden Einsatz |
| ED-2 | Festlegung der Zielbrowser und -versionen (LAB-04) | Prüfumfang, Plattformwahl |
| ED-3 | Architekturrichtung (Browser-Erweiterung, externer Prüfstand oder Kombination) | Entscheidung in der Architekturphase, nicht jetzt |
| ED-4 | Aufbewahrungsfristen für Berichte, Protokolle und Mitschnitte | Datenschutzkonzept |
| ED-5 | Verantwortung für den separaten Arbeitsstrang „Prüfungsintegrität" und den Umgang mit `OEB-Analyse-IT.md` | Wissensschutz |
| ED-6 | Beteiligung von Datenschutz, Rechtsabteilung, Betriebsrat/Personalvertretung | Freigabe |
| ED-7 | Mindestabdeckung der automatisierten Tests (Zielwert) | SR-NF-10 |

**Nicht Gegenstand dieses Dokuments:** konkrete Architektur, Technologieauswahl, Aufwandsschätzung und Zeitplan. Sie folgen nach Freigabe der Anforderungen.
