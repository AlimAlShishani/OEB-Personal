# Migrations- und Risikobewertung: Von OEB zum Security-Testing-Framework (Phase 3)

**An:** IT-Leitung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Klassifizierung:** intern
**Grundlage:** `security-privacy-audit.md`, `reuse-assessment.md`, `security-testing-requirements.md` sowie die Vorarbeiten (`technical-handover.md`, `OEB-Analyse-IT.md`, `verification-report.md`, `documentation-gaps.md`).
**Kennzeichnung:** BESTÄTIGT (Quellcode belegt) · ABGELEITET (Schlussfolgerung) · OFFEN · NICHT GEPRÜFT. Es wurden weiterhin keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt. Rechtliche Einschätzungen sind nicht Teil dieser Bewertung; sie obliegen der Rechtsabteilung.

---

## 1. Management-Zusammenfassung

**1. Ist die bisherige technische Grundlage geeignet?**
**Nein.** Als Wissensbasis ja, als Codebasis nein. 93 % des geprüften Quelltexts (5 427 von 5 821 Zeilen ohne das ungeprüfte Stylesheet) gehören zum ausdrücklich ausgeschlossenen Funktionsumfang: Nachahmung eines SEB-Clients, Nachbau der SEB-Oberfläche, Erfassung von Prüfungskonfigurationen, KI-gestütztes Beantworten von Prüfungsfragen. Kein Code ist unverändert wiederverwendbar (`reuse-assessment.md` 4). Die Dokumentation ist als Bestandsaufnahme ausreichend, aber nicht als Entwurfsgrundlage (`security-privacy-audit.md` 2).

**2. Welche Risiken müssen zuerst beseitigt werden?**
Vor jeder technischen Planung: (a) **Rechts- und Herkunftsklärung** (keine Lizenzdatei, zwei Fremdprojekte als Vorlage, SEB-Marken-Assets), (b) **Governance und Autorisierungsrahmen** für Tests, (c) **Entscheidung gegen die Fortführung der Codebasis**, damit Umgehungsfunktionen nicht unbeabsichtigt in das neue Projekt gelangen. Technisch relevant bei jeder Wiederverwendung: Klartext-Schlüssel (SEC-03), Auslösbarkeit durch Seiten-JavaScript (SEC-01), Plattform Manifest V2 (SEC-02), fehlendes Logging (SEC-09).

**3. Welche Teile sollten übernommen werden?**
Kein Code. Übernommen werden **Konzepte** (zentraler Router mit einheitlichem Antwortformat, Zustandsautomat-Dokumentation, injizierbare Abhängigkeiten für Tests, Scope-Allowlist, Limits an Netzwerkgrenzen), das **Build-Gerüst als Vorlage** (95 Zeilen) und die **Dokumentationsstruktur** samt Kennzeichnungsschema.

**4. Welche Teile sollten vollständig neu entwickelt werden?**
Alles Funktionale: Hintergrundlogik, Oberfläche, Nachrichtenschicht, Konfiguration, Logging, Fehlerbehandlung, Berechtigungsmodell, Tests, Berichtswesen. Das Ursprungsarchiv und die Dokumente zur Umgehungsfunktion werden **isoliert** und zugriffsbeschränkt aufbewahrt.

**5. Welche Anforderungen muss die neue Testumgebung erfüllen?**
Isoliertes Labor ohne Produktivzugriff, Scope-Allowlist mit Prüfauftrag, saubere Profile und festgelegte Browserversionen, synthetische Testdaten, strukturiertes manipulationsgeschütztes Ereignisprotokoll, maschinenlesbare reproduzierbare Testfälle, Berichtsvorlagen mit Evidenzklassen, Selbsttest, technische Sperre des Ausschlusskatalogs durch Negativtests (`security-testing-requirements.md`).

**6. Ist das Projekt bereit für die nächste Planungsphase?**
**Bedingt.** Bereit für die *Konzept- und Voraussetzungsplanung* (Governance, Rechtsklärung, Anforderungsfreigabe, Browser-Matrix). **Nicht bereit** für Architektur und Umsetzung, solange die Gates G1 und G2 (Abschnitt 6) nicht erfüllt sind.

### Empfehlung an die IT-Leitung

1. **Die Option „Umbenennen oder Umbauen von OEB" wird nicht weiterverfolgt.** Das neue Framework wird als **eigenständige Neuentwicklung** geplant (Option C, Abschnitt 3), die auf Anforderungen und Konzepten beruht und keinen Code aus der OEB-Codebasis übernimmt.
2. **Die OEB-Materialien werden isoliert:** Ursprungsarchiv mit Prüfsumme in einem schreibgeschützten, zugriffsbeschränkten Ablageort; die Dokumente `technical-handover.md` und `OEB-Analyse-IT.md` ebenfalls zugriffsbeschränkt. Keine Ausführung auf Produktiv- oder Nutzergeräten.
3. **Vor Beginn der Architekturplanung** werden entschieden und dokumentiert: verantwortliche Stelle für Prüfgenehmigungen, Datenschutz-/Rechtsfreigabe, Zielbrowser, Umgang mit dem Arbeitsstrang „Prüfungsintegrität".
4. **Freigabe** von `security-testing-requirements.md` (nach Anpassung) als Grundlage; Architekturentwurf und Bedrohungsmodell erst danach.
5. **Alternative:** Sollte die Neuentwicklung nicht gewünscht sein, ist Option D (Abschluss mit Prüfbericht, kein Framework) der konsequente Weg; ein Umbau von OEB ist **keine** gleichwertige Alternative.

---

## 2. Ausgangslage

- **BESTÄTIGT:** Das Repository `OEB-Personal` enthält ausschließlich Dokumentation (Markdown). Der Quellcode lag als ZIP `OEB-2.4.1` außerhalb des Repositories vor.
- **BESTÄTIGT:** Die Codebasis stützt sich laut README auf zwei Fremdprojekte (`README.md:7`), enthält keine Lizenzdatei, keine Tests, keine CI und kein Changelog.
- **BESTÄTIGT:** Funktionskern ist die Nachahmung eines Prüfungsclients gegenüber Servern und die automatisierte Beantwortung von Prüfungsfragen (`technical-handover.md` 1, 5; `OEB-Analyse-IT.md` 1–3).
- **ABGELEITET:** Eine Umwandlung ist nur dann „eine echte technische Neuausrichtung", wenn die Architektur neu entworfen wird; ein Austausch von Namen oder Menüs genügt nicht (die IT-Leitung hat dies ausdrücklich verlangt).
- **NICHT GEPRÜFT:** Zielumgebung (Browserversionen, Richtlinien), rechtliche Rahmenbedingungen, Lizenz der Vorlagen.

---

## 3. Optionen

| | A – Umbenennen / Fork | B – Selektive Übernahme (Teile ausbauen, Rest behalten) | **C – Neuentwicklung mit Wissensübernahme** | D – Kein Framework; Abschluss mit Prüfbericht |
|---|---|---|---|---|
| Kurzbeschreibung | OEB bleibt Basis, Funktionen werden umbenannt/umgewidmet | Bestimmte Module bleiben, Umgehungs- und KI-Module werden entfernt | Eigene Codebasis auf Grundlage von Anforderungen; Übernahme nur von Konzepten und Dokumentationsstruktur | Ergebnisse dieser Phase werden als Entscheidungsgrundlage archiviert |
| Konformität mit Ausschlusskatalog | **Nicht gewährleistet**: Fähigkeiten bleiben vorhanden und sind schwer nachweisbar abzustellen | **Fraglich**: Restcode und Rechteprofil sind darauf zugeschnitten (SEC-11) | **Gewährleistet** durch Negativtests (NT-01 bis NT-06) | Gegeben |
| Sicherheitsniveau | Alle Befunde SEC-01 bis SEC-15 bleiben | Teilweise beseitigt; Rest-Architektur bleibt (MV2, Hub, Router) | Nach Anforderungen gestaltbar | – |
| Rechts-/Herkunftslage | Risiko bleibt (M-01) | Risiko bleibt | Klärbar, kein Fremdcode | – |
| Plattformrisiko (MV2) | bleibt (SEC-02) | bleibt | Von Beginn an behandelbar (SR-NF-01) | – |
| Aufwand | Kein verlässlicher Wert; Aufwand für Rückbau und Absicherung wird typischerweise unterschätzt (ABGELEITET) | Hoch: nahezu alle Dateien betroffen (`reuse-assessment.md` 4) | Neuer Aufwand, dafür planbar; Schätzung erst nach Architekturentwurf (OFFEN) | Gering |
| Erreichen des Projektziels | Fraglich | Fraglich | Ja, sofern Voraussetzungen erfüllt | Nein (Ziel entfällt) |
| **Bewertung** | **Nicht empfohlen** | **Nicht empfohlen** | **Empfohlen** | Rückfallposition |

**Begründung (ABGELEITET):** Bei B müssten rund 93 % des geprüften Codes entfernt oder ersetzt werden (`reuse-assessment.md` 4), die verbleibenden Gerüste (Manifest V2, Hub-Struktur, Router ohne Absenderprüfung) entsprechen nicht den Anforderungen aus `security-testing-requirements.md`, und die Herkunftsfragen bleiben bestehen. Die Neuentwicklung ist deshalb nicht nur sicherer, sondern vermutlich auch effizienter.

---

## 4. Risikoregister

**Skalen:** Eintrittswahrscheinlichkeit (EW) und Auswirkung (AW): niedrig / mittel / hoch. Die Einstufungen sind fachliche Einschätzungen des Analysten (ABGELEITET), keine gemessenen Werte.

| ID | Risiko | Kat. | EW | AW | Status | Gegenmaßnahme | Bezug |
|---|---|---|---|---|---|---|---|
| M-01 | **Lizenz-/Herkunftsrisiko:** keine Lizenzdatei, zwei Fremdprojekte als Vorlage, enthaltene SEB-Marken-/Logo-Assets | Rechtlich | hoch | hoch | BESTÄTIGT (Fakten) / NICHT GEPRÜFT (Rechtslage) | Rechtsabteilung einbinden; keine Codeübernahme; keine Fremdassets | `README.md:7`; `seb/icons/*` |
| M-02 | **Zweckentfremdung / Funktionsdrift:** Übernahme von Code oder Rechten schafft Fähigkeiten, die dem Ausschlusskatalog widersprechen | Governance | mittel | hoch | ABGELEITET | Ausschlusskatalog AK-01 bis AK-09; Negativtests NT-01 bis NT-06; Freigabe nach Vier-Augen-Prinzip | `security-testing-requirements.md` 1.3, 6.3 |
| M-03 | **Reputations- und Compliance-Risiko:** ein Sicherheitsprojekt, das aus einem Werkzeug zur Umgehung von Prüfungssoftware hervorgeht, kann intern und extern missverstanden werden | Reputation | mittel | hoch | ABGELEITET | Klare Kommunikation: Neuentwicklung, Ausschlusskatalog, Isolierung des Altmaterials | – |
| M-04 | **Übernahme bekannter Sicherheitsmängel** (Klartext-Schlüssel, Auslösbarkeit durch Seiten-JS, Passwort-Offenlegung, ungeprüfter Router) | Technisch | hoch (bei A/B) | mittel | BESTÄTIGT / ABGELEITET | Option C; Mängel als Anforderungen (SR-NF-05 bis SR-NF-08) | SEC-01, 03, 05, 08 |
| M-05 | **Plattformauslauf Manifest V2 / persistente Hintergrundseite** | Technisch | OFFEN | hoch | OFFEN / NICHT GEPRÜFT | Plattformentscheidung vor Architektur; MV3 oder externer Prüfstand | SEC-02, UA-03 |
| M-06 | **Datenschutz in Entwicklung und Test:** vorhandene Daten (API-Schlüssel, Beispielkonfigurationen, Prüfungsinhalte) in Entwicklerprofilen | Datenschutz | mittel | mittel | ABGELEITET | Keine Realdaten; Profile bereinigen; synthetische Testdaten (LAB-03) | DS-01/03/06 |
| M-07 | **Wissensschutz:** Dokumente und Archiv beschreiben die Umgehungsmechanik und Erkennungsmerkmale | Informationssicherheit | mittel | mittel | ABGELEITET | Zugriffsbeschränkung, Protokollierung des Zugriffs, getrennter Arbeitsstrang „Prüfungsintegrität" | `reuse-assessment.md` 3.9 |
| M-08 | **Fehlende Governance:** kein definierter Prozess für Prüfaufträge, Scope und Freigaben | Governance | hoch | hoch | BESTÄTIGT (nicht vorhanden in den Unterlagen) | GOV-01 bis GOV-08 vor Entwicklungsbeginn | `security-testing-requirements.md` 2 |
| M-09 | **Unklare Zielumgebung** (Browser, Versionen, Richtlinien) | Planung | hoch | mittel | NICHT GEPRÜFT | Browser-Matrix erstellen (LAB-04) | Q-6 |
| M-10 | **Scope-Verletzung im Betrieb:** Tests treffen Produktivsysteme oder Dritte | Betrieb | mittel | hoch | ABGELEITET | Egress-Allowlist, Scope-Datei, Negativliste, Not-Aus | LAB-01, GOV-02/03/08 |
| M-11 | **Falsche Sicherheit:** Ergebnisse werden als Garantie gelesen | Fachlich | mittel | mittel | ABGELEITET | Evidenzklassen, Aussagegrenzen im Bericht, Reproduktionsangaben | REP-02, REP-06 |
| M-12 | **Qualitätsrisiko bei Altcode:** keine Tests, Regressionen beim Umbau | Technisch | hoch (bei A/B) | mittel | BESTÄTIGT (keine Tests) | Option C; Testpflicht SR-NF-10 | TR-08 |
| M-13 | **Lieferkette:** Build-Abhängigkeiten (25 Pakete im Lockfile), Registry-Zugriff | Technisch | niedrig | mittel | NICHT GEPRÜFT (kein Schwachstellen-Scan durchgeführt) | Lockfile, SBOM, Scans, signierte Artefakte | SR-NF-09 |
| M-14 | **Unterschätzter Aufwand der Neuentwicklung** | Projekt | mittel | mittel | OFFEN | Aufwandsschätzung erst nach Architekturentwurf; Meilensteine mit Freigabe-Gates | – |
| M-15 | **Dokumentationsdrift** zwischen mehreren Dokumenten und Befund-IDs | Dokumentation | mittel | niedrig | BESTÄTIGT (in Phase 2/2.1 bereits aufgetreten) | Änderungsprotokoll, Querverweisprüfung bei jeder Revision | `technical-handover.md` 0.5/0.6 |
| M-16 | **Beteiligungspflichten** (Betriebsrat/Personalvertretung, Datenschutz) bei Test- und Auswertungsverfahren | Recht | mittel | mittel | NICHT GEPRÜFT | Frühzeitige Beteiligung | A-2 |
| M-17 | **Neue Befunde in unbekannten Bereichen:** `popup.css` und Laufzeitverhalten wurden nicht geprüft | Fachlich | niedrig | niedrig | NICHT GEPRÜFT | Bei Option C ohne Relevanz; bei A/B vor Weiterverwendung prüfen | – |

---

## 5. Behandlung des vorhandenen Materials

| Material | Maßnahme (Vorschlag) | Kat. |
|---|---|---|
| Ursprungsarchiv `OEB-2.4.1.zip` | Prüfsumme dokumentieren; schreibgeschützt und zugriffsbeschränkt ablegen; nicht im neuen Repository; keine Ausführung auf Produktiv-/Nutzergeräten | Zu isolieren |
| `technical-handover.md`, `OEB-Analyse-IT.md` | Zugriffsbeschränkte Ablage; Verwendung nur im Arbeitsstrang „Prüfungsintegrität" | Zu isolieren |
| `verification-report.md`, `documentation-gaps.md`, die vier Dokumente dieser Phase | Als Entscheidungs- und Belegsammlung aufbewahren | Zu isolieren |
| Struktur und Kennzeichnungsschema der Dokumentation | In das neue Projekt übernehmen | Wiederverwendbar |
| Entwicklerprofile/Kopien mit OEB-Daten (API-Schlüssel, Beispielkonfigurationen) | Bereinigen; Schlüssel beim Anbieter widerrufen, falls je eingetragen (OFFEN, ob vorhanden) | – |

Die Entscheidung über Ablageorte, Aufbewahrungsdauer und eventuelle Löschung trifft die IT-Leitung gemeinsam mit Datenschutz und Rechtsabteilung (offene Entscheidung ED-5/ED-6).

---

## 6. Freigabe-Gates und Bereitschaft

| Gate | Inhalt | Nachweis | Status heute |
|---|---|---|---|
| **G0** | Entscheidung über die Option (empfohlen: C) | Beschluss der IT-Leitung | offen |
| **G1** | Governance und Recht: verantwortliche Stelle benannt, Prüfauftrags-/Scope-Prozess definiert, Lizenz-/Herkunftsklärung, Datenschutz-/Rechtsfreigabe, Beteiligungspflichten geklärt | schriftliche Freigaben | offen |
| **G2** | Anforderungen (`security-testing-requirements.md`) freigegeben, offene Entscheidungen ED-1 bis ED-7 beantwortet | Freigabevermerk | offen |
| **G3** | Architektur- und Bedrohungsmodell erstellt, Plattformentscheidung (SR-NF-01), Browser-Matrix | Entwurf und Review | nicht begonnen |
| **G4** | Erster Prototyp ausschließlich im Labor; Negativtests NT-01 bis NT-06 grün | Testbericht | nicht begonnen |
| **G5** | Selbsttest (FR-ST-01) ohne offene hohe Befunde; erster Prüfbericht freigegeben | Bericht | nicht begonnen |

**Bereitschaft für die nächste Planungsphase:** Die Planung von **G0 bis G2** kann sofort beginnen. Die Planung von **G3 und folgenden** setzt G1 und G2 voraus.

### Wann sich diese Empfehlung ändern würde
- Wenn die Rechtsabteilung die Herkunft klärt **und** die IT-Leitung dennoch eine Teilübernahme wünscht, wäre eine erneute, dateiweise Bewertung nötig – mit dem Ergebnis aus `reuse-assessment.md` 4 als Ausgangspunkt (aus heutiger Sicht ändert sich das Bild dadurch nicht).
- Wenn Laufzeit- und Browsertests (heute NICHT GEPRÜFT) zeigen, dass einzelne Gerüste tragfähiger sind als angenommen: wirkt nur auf Kategorie-2-Komponenten (394 Zeilen), nicht auf die Gesamtempfehlung.

---

## 7. Offene Punkte und Grenzen dieser Bewertung

| Nr. | Punkt | Status |
|---|---|---|
| O-1 | Laufzeit-, Browserkompatibilitäts- und Netzwerktests wurden nicht durchgeführt | NICHT GEPRÜFT |
| O-2 | Rechtslage (Lizenz, Marken, Prüfungs- und Datenschutzrecht) | NICHT GEPRÜFT |
| O-3 | Aufbewahrung und Nutzung der Daten bei KI-Anbietern | NICHT GEPRÜFT (nicht relevant für Option C) |
| O-4 | `popup.css`, Bilddateien, Inhalt von `bek.json` | NICHT GEPRÜFT |
| O-5 | Aufwand, Zeitplan, Ressourcen der Neuentwicklung | OFFEN (nach Architekturentwurf) |
| O-6 | Browser-/MV3-Unterstützung im Haus | OFFEN |
| O-7 | Schwachstellenprüfung der Build-Abhängigkeiten | NICHT GEPRÜFT |
