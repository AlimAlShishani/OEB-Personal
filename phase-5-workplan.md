# Phase-5-Arbeitsplan: Architektur und technische Spezifikation (Vorbereitung)

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** Entwurf, **nicht freigegeben** · **Klassifizierung:** intern
**Grundlage:** `project-charter.md` · `oeb-transformation-concept.md` · `target-architecture.md` · `transformation-decision-matrix.md` · `project-roadmap.md` · `phase-4-status.md` · `security-testing-requirements.md` · `security-privacy-audit.md` · `migration-risk-assessment.md`

**Kennzeichnung:** BESTÄTIGT (belegt) · ABGELEITET (Schlussfolgerung) · OFFEN (Entscheidung oder Information fehlt) · NICHT GEPRÜFT (nicht untersucht) · **VORSCHLAG** (Planungs- oder Entwurfsvorschlag, noch nicht entschieden).
**Prüfstatus:** Dieses Dokument ist reine Planung. Es wurden keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt, nichts implementiert und keine Quell-, Konfigurations- oder Build-Dateien verändert. Aussagen zu Browsern, Plattformen und Technologien beruhen auf allgemeinem Fachwissen und sind **nicht** in dieser Umgebung geprüft; sie sind in den Arbeitspaketen zu verifizieren.

---

## 1. Ausgangslage und Planungsgrundlage

### 1.1 Entscheidungsstand

| Punkt | Stand | Bewertung |
|---|---|---|
| Arbeitsrichtung Option B (neues Repository, keine ungeprüfte Übernahme von Altcode/Assets, Übernahme zulässiger Dokumentation/Anforderungen/Erkenntnisse, unabhängiger Aufbau, vollständige Trennung von ausgeschlossenen Funktionsbereichen) | **Planungsentscheidung der Projektleitung** (Mitteilung zum Phasenübergang) | Gilt als Grundlage dieses Plans. Sie ist ausdrücklich **keine** Freigabe. |
| Gate G0 (Optionsentscheidung, dokumentiert) | Formal nicht dokumentiert | **OFFEN.** Empfehlung: Planungsentscheidung als Beschluss festhalten (PE-1) |
| Gate G1 (Governance, Rechtsbewertung, Datenschutz-Plan) | Nicht erreicht | OFFEN |
| Freigabe der Phase-4-Entwürfe (F-1) | Nicht erteilt; Dokumentation wurde zur Kenntnis genommen | OFFEN |
| Rechts-, Datenschutz- und Sicherheitsfreigaben (F-3 bis F-6) | Nicht erteilt | OFFEN |
| Wiederverwendung von Quellcode (K-C) | Gesperrt | Keine schriftliche Lizenz- und Sicherheitsfreigabe liegt vor |

**`phase-4-status.md` wurde nicht geändert,** da sich der tatsächliche Freigabestatus nicht geändert hat.

### 1.2 Abweichung von der Roadmap (transparent)
`project-roadmap.md` nennt als Eingangsbedingungen für Phase 5 unter anderem G0 und G1. Die Projektleitung hat die **Vorbereitung** von Phase 5 angewiesen, bevor diese formal erfüllt sind. Dieser Plan geht deshalb wie folgt vor:
- Die Arbeit ist **Vorbereitung unter Vorbehalt:** Alle Ergebnisse tragen den Status „Entwurf" und werden erst nach Erfüllung von G0 und G1 als Phase-5-Ergebnisse eingereicht.
- Nicht begonnen werden Arbeiten, die von offenen Freigaben abhängen (z. B. rechtsabhängige Festlegungen, Produktname, Lizenz).
- Keine Aussage dieses Plans ersetzt eine Freigabe.

### 1.3 Randbedingungen (unverändert)
Ausschlusskatalog AK-01 bis AK-09; Datenschutz-Voraussetzungen P-1 bis P-7; Befunde SEC-01 bis SEC-15 und NF-01 bis NF-06; Stop-Kriterien der Charta (7.2). Die Spezifikation entsteht **ohne Einsicht in Altcode**; Wissen aus der Altbasis fließt ausschließlich über die abgeleiteten Anforderungen (EK-1 bis EK-12) ein (`oeb-transformation-concept.md` 3, 6).

---

## 2. Überblick: Arbeitspakete

Die Pakete decken die zehn geforderten Themen ab (Spalte „Thema") und die Phase-5-Ergebnisse E5-1 bis E5-9 der Roadmap.

| WP | Titel | Thema | Roadmap-Ergebnis | Geplantes Ergebnisdokument (noch nicht erstellt) |
|---|---|---|---|---|
| WP5-00 | Voraussetzungen und Entscheidungen der Projektleitung | 9 | – | Entscheidungsprotokoll |
| WP5-01 | Architekturentscheidungen (Entscheidungsregister) | 1 | E5-4 | `architecture-decision-register.md` |
| WP5-02 | Technologie-Alternativen | 2 | E5-4 | `technology-options.md` |
| WP5-03 | Browser- und Plattformanforderungen | 3 | E5-4 | `platform-requirements.md` |
| WP5-04 | Modulverantwortlichkeiten und Schnittstellen | 4 | E5-1, E5-3 | `technical-specification.md` (Teil A) |
| WP5-05 | Datenmodell und Speicherstrategie | 5 | E5-3 | `technical-specification.md` (Teil B) |
| WP5-06 | Berechtigungs- und Sicherheitsmodell, Bedrohungsmodell | 6 | E5-2 | `security-model.md` |
| WP5-07 | Teststrategie und Qualitätskriterien | 7 | E5-5 | `test-strategy.md` |
| WP5-08 | Datenschutz- und Reporting-Konzept | 8 | E5-7 | `privacy-reporting-concept.md` |
| WP5-09 | Lieferketten-, Lizenz- und Compliance-Konzept | 9 | E5-6 | `dependency-license-plan.md` |
| WP5-10 | Aufwands- und Terminschätzung | 9 | E5-8 | `estimate-and-schedule.md` |
| WP5-11 | Rückverfolgbarkeit, Konsistenz, Review | 10 | alle | Rückverfolgbarkeitsmatrix, Reviewprotokolle |

---

## 3. Konkrete Architekturentscheidungen (Thema 1)

Das Register führt die Entwurfsentscheidungen DD-1 bis DD-9 aus `target-architecture.md` 11 weiter und ergänzt sie. **Entscheider:** PL = Projektleitung, TL = Technische Leitung, IS = Informationssicherheit, DS = Datenschutz, RA = Rechtsabteilung. Der Analyst bereitet fachlich vor (Alternativen, Kriterien, Empfehlung); die Entscheidung trifft die genannte Stelle.

| Nr. | Entscheidung | Entspricht | Alternativen (Abschnitt 4) | Entscheider | Vorbereitung durch Analyst | Zeitpunkt | Status |
|---|---|---|---|---|---|---|---|
| AD-01 | Ausführungsmodell der Tests | DD-1 | 4.1 | TL, IS (PL bei Rechtefolgen) | Bewertungsmatrix, Abhängigkeit von Browser-Matrix, vorläufige Empfehlung | vor G3 | OFFEN |
| AD-02 | Plattform für Browser-seitige Komponenten (falls erforderlich) | DD-2 | 4.2 | TL, IS | Prüfplan zur Plattformunterstützung | vor G3 | OFFEN |
| AD-03 | Implementierungssprache und Laufzeitumgebung | neu | 4.3 | TL | Kriterienliste, Vergleich | vor G3 | OFFEN |
| AD-04 | Nachrichtentransport und Authentisierung zwischen Modulen | DD-3 | 4.4 | TL, IS | Sicherheitsvergleich | vor G3 | OFFEN |
| AD-05 | Erweiterungsmodell der Prüfmodule | DD-4 | 4.5 | TL, IS | Fähigkeitsmodell, Ausschlusskontrolle | vor G3 | OFFEN |
| AD-06 | Nachweisspeicher (Format, Verschlüsselung, Schlüsselverwaltung) | DD-5 | 4.6 | TL, IS, DS | Datenklassen, Integritäts- und Löschanforderungen | vor G3 | OFFEN |
| AD-07 | Testfall- und Datenformat | DD-6 | 4.7 | TL | Schema-Entwurf | vor G3 | OFFEN |
| AD-08 | Berichtsformate und Signaturverfahren | DD-7 | 4.8 | TL, IS | Anforderungen aus REP-01 bis REP-07 | vor G3 | OFFEN |
| AD-09 | Art der Benutzeroberfläche und Bediener-Authentisierung | DD-8 | 4.9 | TL, IS | Rollenmatrix, Risiken | vor G3 | OFFEN |
| AD-10 | Umgang mit internen Dienstzugangsdaten | DD-9 | 4.10 | IS | Bedrohungsbetrachtung | vor G3 | OFFEN |
| AD-11 | Isolationsplattform des Labors (virtuell/Container/andere) und Netzgrenze | neu | 4.11 | TL, IS | Anforderungen LAB-01 bis LAB-08 | vor G3 | OFFEN |
| AD-12 | Repository-, Branch-, Review- und Release-Modell (inkl. Signierung) | neu | – | TL | Vorschlag zu Vier-Augen, geschützter Hauptlinie, Negativtests | vor Phase 6 | OFFEN |
| AD-13 | Zeitquelle und Protokoll-Backend | neu | – | TL, IS | Anforderungen LAB-05, LOG-01 bis LOG-06 | vor G3 | OFFEN |
| AD-14 | Betriebsmodell: wer betreibt Labor und Framework, wo laufen Nachweise | neu | – | **PL** | Betriebsvarianten mit Aufwand/Risiko | vor G2 | OFFEN |
| AD-15 | Produktname und Lizenz des neuen Projekts | – | – | **PL** mit RA | Kriterien (Charta 5), Prüfliste | vor Phase 6 | OFFEN |

Reihenfolge nach Abhängigkeit (VORSCHLAG): AD-14, AD-11 und die Browser-Matrix (WP5-03) zuerst, weil sie AD-01/AD-02 bestimmen; danach AD-01, AD-02, AD-03; danach AD-04 bis AD-10, AD-13; zuletzt AD-12.

---

## 4. Technologische Alternativen (Thema 2)

**Wichtig:** Die Tabellen zeigen Alternativen ohne Vorauswahl. Beurteilungen sind qualitativ (ABGELEITET) und nicht in einer Laborumgebung geprüft. Produktentscheidungen entstehen erst in WP5-02 mit Kriterienbewertung. Eine vorläufige Neigung (VORSCHLAG) ist gekennzeichnet.

### 4.1 Ausführungsmodell (AD-01)

| Alternative | Vorteile | Nachteile / Risiken |
|---|---|---|
| **A) Externer Prüfstand** steuert Browser über standardisierte Automatisierungsschnittstellen (z. B. WebDriver-Standard und dessen Weiterentwicklungen) | Browser bleibt weitgehend unverändert; keine zusätzlichen Browser-Rechte nötig; gute Trennung Steuerung/Ausführung; reproduzierbar | Reichweite der Beobachtung (z. B. interne Browserspeicher, Erweiterungsinterna) begrenzt; Reife und Browserunterstützung neuerer Schnittstellen zu verifizieren (NICHT GEPRÜFT) |
| **B) Browserspezifische Debug-Schnittstelle** (z. B. DevTools-Protokoll von Chromium-Browsern) | Tiefe Beobachtungsmöglichkeiten | Nicht browserübergreifend; Stabilitäts- und Versionsabhängigkeit; breite Fähigkeiten → strengere Kontrolle nötig |
| **C) Instrumentierung per Browser-Erweiterung** („Probe") | Zugriff auf Erweiterungs-/Browserinterna innerhalb des Browsers; kann Berechtigungsverhalten direkt messen | Erfordert Rechte im Zielbrowser; Plattformabhängigkeit (Erweiterungsmodelle ändern sich); größere Angriffsfläche; wiederholt Risiken der Altbasis (Rechteprofil) |
| **D) Hybrid:** Prüfstand A (und ggf. B) plus minimale, deklarierte Probe C nur wo nötig | Rechte nur dort, wo erforderlich; flexible Abdeckung | Mehr Komponenten; Schnittstelle zwischen Prüfstand und Probe muss streng validiert werden |
| **E) Rein statische Analyse** (Manifeste, Richtlinien, Konfigurationen) für Teile des Umfangs | Kein Ausführungsrisiko; einfach reproduzierbar | Deckt dynamisches Verhalten (Kommunikation, Speicher) nicht ab |

**Vorläufige Neigung (VORSCHLAG):** D (A plus E als Basis, minimale Probe nur bei nachgewiesenem Bedarf), da sie dem Prinzip der minimalen Rechte (SR-NF-02) am besten entspricht. Endgültig erst nach WP5-03.

### 4.2 Plattform für Browser-seitige Komponenten (AD-02)

| Alternative | Vorteile | Nachteile / Risiken |
|---|---|---|
| Keine Browser-Erweiterung (nur Alternative A/E) | Keine Plattformabhängigkeit der Erweiterungsmodelle; kleinste Angriffsfläche | Eingeschränkte Messung von Interna |
| Erweiterung auf dem aktuellen, unterstützten Manifest-Modell (Manifest V3, falls Browser dies verlangen) | Zukunftsfähig; strengere Plattformvorgaben (Rechte, kein Remote-Code) passen zu den Anforderungen | Funktionsgrenzen (z. B. weniger Eingriffsmöglichkeiten in Netzwerkverkehr, kurzlebige Hintergrundkontexte); Unterschiede zwischen Browsern (zu verifizieren) |
| Ältere Erweiterungsmodelle (z. B. Manifest V2) | Mehr Eingriffsmöglichkeiten | **Nicht empfohlen:** auslaufend (allgemeines Wissen, nicht geprüft), widerspricht SR-NF-01 und minimalen Rechten |

### 4.3 Implementierungssprache und Laufzeit (AD-03)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Typisierte Skriptsprache (z. B. TypeScript) | Gleiche Sprache für Prüfstand und ggf. Probe; breite Browser-Werkzeuge; statische Typprüfung | Laufzeit- und Lieferkettenumfang (viele Pakete) |
| Dynamische Skriptsprache (z. B. Python) | Schnelle Entwicklung; breites Testwerkzeug-Angebot | Typsicherheit schwächer; Pakete der Lieferkette; getrennte Sprache für eine Probe |
| Kompilierte Sprache (z. B. Go oder Rust) | Statische Binärdateien, kleine Angriffsfläche, klare Typisierung | Weniger direkte Browser-Werkzeuge; Kompetenzaufbau |

Kriterien: Team-Kompetenz (OFFEN), Lieferkettenumfang, Testbarkeit, Wartbarkeit, Signierbarkeit.

### 4.4 Nachrichtentransport und Authentisierung (AD-04)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Direkte Schnittstellenaufrufe im selben Prozess | Einfach; geringe Angriffsfläche | Schwache Isolation zwischen Modulen |
| Lokale Prozesskommunikation mit Schema-Prüfung | Prozessgrenzen; klare Verträge | Mehr Infrastruktur |
| Netzwerkdienst mit gegenseitiger Authentisierung | Verteilt betreibbar; starke Trennung der Zonen | Betriebsaufwand; Zertifikatsverwaltung |
| Nachrichtenwarteschlange | Entkopplung; Wiederholbarkeit | Zusätzliche Komponente; Konfigurationsrisiko |

Pflicht in allen Varianten: versioniertes Schema, authentifizierter Absender, laufbezogene Berechtigungsnachweise, kein offener Zielursprung (SR-NF-07).

### 4.5 Erweiterungsmodell der Prüfmodule (AD-05)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Festes Verzeichnis geprüfter Module im selben Repository | Einfache Ausschlusskontrolle und Herkunft | Weniger Erweiterbarkeit |
| Signierte Plug-ins mit Fähigkeitsdeklaration | Erweiterbar; prüfbar | Signier-/Verwaltungsaufwand; höhere Angriffsfläche |

### 4.6 Nachweisspeicher (AD-06)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Anhängende Dateien mit Hash-Verkettung (z. B. zeilenorientiertes Format) | Einfach, transparent, gut prüfbar | Abfragen/Indizes begrenzt |
| Eingebettete Datenbank mit Integritäts-Zusatz | Abfragefähig | Unveränderlichkeit muss zusätzlich sichergestellt werden |
| Objektspeicher mit Schreibsperre | Starke Unveränderlichkeit | Betriebsaufwand, Abhängigkeit von Infrastruktur |

Gemeinsam zu klären: Verschlüsselung ruhender Daten, Schlüsselverwaltung, Löschsemantik (Löschnachweis, z. B. durch Schlüsselvernichtung), Zugriffsklassen (WP5-05, WP5-08).

### 4.7 Testfall- und Datenformat (AD-07)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Deklarative Dateien (strukturierte Textformate) mit Schema | Maschinenlesbar, versionierbar, prüfbar, getrennt von der Logik | Komplexe Abläufe schwer ausdrückbar |
| Testfälle als Code | Ausdrucksstark | Weniger prüfbar; Ausschlusskontrolle schwieriger |
| Kombination: deklarativ für Metadaten/Erwartung, Code nur für zugelassene Prüfmodule | Balance | Zwei Mechanismen |

### 4.8 Berichtsformate und Signatur (AD-08)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Textformat → lesbare Ausgabe (Dokument) plus strukturierte Datei | Lesbar und maschinenlesbar | Konvertierungswerkzeuge in der Lieferkette |
| Nur lesbares Dokument | Einfach | Keine automatisierte Weiterverarbeitung |
| Zusätzliche Signatur der Berichte (abgesetzt) | Nachprüfbarkeit | Schlüsselverwaltung (AD-10) |

### 4.9 Benutzeroberfläche und Bediener-Authentisierung (AD-09)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Kommandozeile als Basis | Kleine Angriffsfläche, skriptbar | Weniger komfortabel für Berichtsleser |
| Lokale Weboberfläche | Komfort, Rollenansichten | Zusätzliche Angriffsfläche (Eingabe-/Ausgabesicherheit, SR-NF-03) |
| Kombination (Kommandozeile für Prüfer, Weboberfläche nur lesend für Berichte) | Risiko begrenzt | Zwei Oberflächen |

Authentisierung der Bediener (OFFEN): Anbindung an vorhandene Identitätsdienste der Organisation oder lokale Konten.

### 4.10 Umgang mit internen Dienstzugangsdaten (AD-10)

Alternativen: Geheimnisverwaltung der Organisation; Betriebssystem-Schlüsselbund; kurzlebige Berechtigungsnachweise. **Nicht zulässig:** Klartext im Profil oder in Konfigurationsdateien (SR-NF-08). Ein Fremd-API-Schlüssel wird nicht benötigt (SR-NF-05).

### 4.11 Isolationsplattform des Labors (AD-11)

| Alternative | Vorteile | Nachteile |
|---|---|---|
| Virtuelle Maschinen mit Snapshots | Starke Isolation; saubere Rücksetzung | Ressourcenbedarf |
| Container | Leicht; schnell zu erzeugen | Schwächere Isolation; Browserfunktionen (z. B. Richtlinien, GUI) je Betriebssystem eingeschränkt zu prüfen |
| Dedizierte Prüfgeräte | Realistisch (echte Geräte/Betriebssysteme) | Aufwändig; Rücksetzung schwieriger |
| Kombination | Passgenau je Browser/Betriebssystem | Mehr Betriebsaufwand |

---

## 5. Browser- und Plattformanforderungen (Thema 3, WP5-03)

### 5.1 Heutiger Kenntnisstand
- **OFFEN / NICHT GEPRÜFT:** Zielbrowser und -versionen sind nicht benannt (ED-2). Es existiert **keine** Browser-Matrix.
- Aus der Analyse der Altbasis folgt nur eine Warnung (BESTÄTIGT als Befund SEC-02): Plattformabhängigkeiten (Erweiterungsmodell, blockierende Netzwerkeingriffe) können wegfallen. Welche Browser im Haus welche Modelle unterstützen, ist **NICHT GEPRÜFT**.

### 5.2 Zu klärende Anforderungen

| Nr. | Anforderung / Frage | Prüfweg | Status |
|---|---|---|---|
| BP-01 | Welche Browser (Familien, Versionen, Kanäle) sind Prüfobjekte? | Entscheidung PL (ED-2), Inventar der IT | OFFEN |
| BP-02 | Welche Betriebssysteme und Gerätearten sind abzudecken? | Inventar der IT | OFFEN |
| BP-03 | Welche Richtlinienmechanismen stellt jeder Zielbrowser bereit, und wie werden sie in der Organisation verwaltet? | Herstellerdokumentation, Inventar | NICHT GEPRÜFT |
| BP-04 | Welche Automatisierungs-/Beobachtungsschnittstellen bietet jeder Zielbrowser, und wie ausgereift sind sie? | Laborprüfung (Phase 6/7), Herstellerdokumentation | NICHT GEPRÜFT |
| BP-05 | Welches Erweiterungsmodell wird je Zielbrowser unterstützt (falls Erweiterung nötig, AD-02)? | Herstellerdokumentation, Laborprüfung | NICHT GEPRÜFT |
| BP-06 | Wie werden Browserversionen im Labor festgelegt und gesperrt (Pinning, Auto-Update aus)? | Konzept (LAB-02) | OFFEN |
| BP-07 | Welche Lizenz- und Nutzungsbedingungen gelten für die Browser-Distributionen im Labor? | Rechtsabteilung | NICHT GEPRÜFT |
| BP-08 | Welche Ressourcen (Rechenleistung, Speicher, Netz) braucht das Labor? | Dimensionierung in WP5-10 | OFFEN |
| BP-09 | Wie verhalten sich Sitzungsspeicher, Erweiterungsspeicher und Löschfunktionen in den Zielbrowsern (Lebensdauer über Neustart/Reload)? | Laborprüfung (Q-1) | NICHT GEPRÜFT |
| BP-10 | Welche Unterschiede zwischen den Zielbrowsern beeinflussen die Reproduzierbarkeit (Konfiguration, Profile, Zeitverhalten)? | Laborprüfung | NICHT GEPRÜFT |

### 5.3 Ergebnis von WP5-03
Eine **Browser-Matrix** (Browser × Version × Betriebssystem × Richtlinienstand × verfügbare Schnittstellen × Eignung für AD-01/AD-02), die in Phase 7 durch Läufe verifiziert wird (die ersten Laufzeit-Browsertests des Projekts). Bis dahin bleiben alle Zellen mit Status OFFEN oder NICHT GEPRÜFT gekennzeichnet.

---

## 6. Modulverantwortlichkeiten und Schnittstellen (Thema 4, WP5-04)

### 6.1 Vorgehen
Die konzeptionellen Module aus `target-architecture.md` 5 (Browser Environment, Security Checks, Permission Analysis, Communication Security, Privacy Analysis, Test Runner, Evidence Manager, Reporting) und Bausteine (Governance Gate, Nachrichtenschicht, Ereignisprotokoll, Benutzeroberfläche) werden zu **prüfbaren Modulspezifikationen** ausgearbeitet.

| Modul | Ergebnis der Spezifikation | Wesentliche offene Punkte |
|---|---|---|
| Browser Environment (MOD-01) | Umgebungsprofile, Richtlinien, Netzgrenze, Rücksetzung, Beobachtungsschnittstelle | AD-11, BP-01 bis BP-06 |
| Security Checks (MOD-02) | Katalog der Prüfpunkte mit Erwartungswerten; Check-Vertrag | AD-01, AD-05 |
| Permission Analysis (MOD-03) | Eingabeartefakte, Baselines, Differenzlogik | Baseline-Inhalte (durch TL/IS festzulegen) |
| Communication Security (MOD-04) | Kanalmodell, Testnachrichtenkatalog (synthetisch), Beobachtungsregeln | AD-01, AD-04 |
| Privacy Analysis (MOD-05) | Canary-Verfahren, Speicher-/Egress-Inventur, Löschprüfung | AD-06, BP-09 |
| Test Runner (MOD-06) | Laufzustand, Serialisierung, Katalogvalidierung, Not-Aus | AD-03, AD-07 |
| Evidence Manager (MOD-07) | Speichermodell, Versiegelung, Aufbewahrung | AD-06, ED-4 |
| Reporting (MOD-08) | Vorlagen, Schweregrad-Maßstab, Exportformate, Signatur | AD-08 |
| Governance Gate (QB-01) | Format von Prüfauftrag und Scope, Signaturverfahren, Zeitfenster | AD-14, ED-1 |
| Nachrichtenschicht (QB-02) | Schemaregister, Versionierung, Fehlerformate | AD-04 |
| Ereignisprotokoll (QB-03) | Ereignisschema, Verkettung, Aufbewahrung | AD-13 |
| Benutzeroberfläche (PL-01) | Funktionen je Rolle, Eingabe-/Ausgabesicherheit | AD-09 |

### 6.2 Schnittstellenverträge
Für jede Schnittstelle I-01 bis I-13 (`target-architecture.md` 6) wird festgelegt: Zweck, Richtung, Schema (versioniert), Authentisierung und Autorisierung, Fehlerfälle und Fehlercodes, Zeit-/Größen-/Ratenlimits, Idempotenz, Protokollierung, Datenklassifikation, Testfälle.

**Abnahme (Auszug):** Kein Schnittstellenentwurf enthält sensible Daten mit offenem Zielursprung; jede Schnittstelle hat mindestens einen Negativtest (ungültiger Absender, ungültiges Schema, Übergröße).

---

## 7. Datenmodell und Speicherstrategie (Thema 5, WP5-05)

### 7.1 Zu spezifizierende Entitäten
Prüfauftrag, Scope, TestCase, TestRun, Observation, Finding, EvidenceItem, Event, Report (`target-architecture.md` 7.1) mit: Pflichtfeldern, Typen, Kennungsschema, Versionierung, Beziehungen, Zustandswerten (Laufzustand 7.3 der Architektur), Validierungsregeln.

### 7.2 Speicherstrategie (zu entscheiden)

| Thema | Frage | Entscheider |
|---|---|---|
| Datenklassen | Einteilung (z. B. öffentlich / intern / vertraulich / streng vertraulich) und Zuordnung der Entitäten | IS, DS |
| Speicherorte je Zone | Wo liegen Steuerdaten (Z1), Beobachtungsdaten (Z2, nur flüchtig), Nachweise (Z3), Berichte (Z4)? | TL, IS |
| Flüchtige vs. dauerhafte Daten | Beobachtungsdaten in Z2 sind flüchtig und werden am Laufende gelöscht, außer sie wurden als Nachweis aufgenommen | TL |
| Verschlüsselung | Ruhende und übertragene Daten; Schlüsselverwaltung | IS |
| Integrität | Hash-Verkettung, Versiegelung, Prüfroutinen | TL, IS |
| Aufbewahrung und Löschung | Fristen je Datenklasse (ED-4); Löschverfahren mit Nachweis; Behandlung von Sicherungen | **PL** (Fristen), DS, TL |
| Redaktion/Schwärzung | Regeln, wann und wie Inhalte entfernt werden | DS, IS |
| Canary-Daten | Erzeugung, Verwaltung, Auffindungsregeln ohne Rohinhalt | TL |
| Testdatenbestand | Synthetische Daten, Versionierung, Herkunft | TL |
| Datensparsamkeit | Welche Felder sind zwingend; welche entfallen | DS |

### 7.3 Abnahme (Auszug)
Jede Datenkategorie hat Klassifikation, Speicherort, Schutzmaßnahme, Aufbewahrungsfrist und Löschverfahren; keine Entität sieht Geheimnisse oder Rohinhalte realer Nutzer vor.

---

## 8. Berechtigungs- und Sicherheitsmodell (Thema 6, WP5-06)

### 8.1 Zu spezifizieren

| Bereich | Inhalt |
|---|---|
| Rollen und Rechte | Prüfer, Freigeber, Berichtsleser, Administrator; Berechtigungsmatrix je Funktion und Datenklasse; Vier-Augen-Regeln (GOV-04, GOV-07) |
| Identität und Authentisierung | Bediener und Dienste; Anbindung (OFFEN, AD-09); laufbezogene Berechtigungsnachweise (Tokens) |
| Fähigkeitsmodell der Prüfmodule | Katalog zulässiger Fähigkeiten (z. B. lesend beobachten, Labordienste ansteuern); Liste **nicht zulässiger** Fähigkeiten gemäß Ausschlusskatalog; Prüfung bei Laden und zur Laufzeit |
| Rechte in Zielbrowsern | Falls Probe (AD-02): minimale Rechte, optionale Rechte, keine pauschalen Host-Rechte (SR-NF-02) |
| Vertrauensgrenzen | B1 bis B5 (`target-architecture.md` 9.1) mit Datenfluss und Kontrollen |
| Scope-Durchsetzung | Gate und Netzgrenze (AP-2); Verhalten bei Fehlern (fail-closed) |
| Geheimnisse | Inventar, Speicherung, Lebensdauer (AD-10) |
| Lieferkette | Gesperrte Abhängigkeiten, SBOM, Signierung (WP5-09) |
| Bedrohungsmodell | Methodisches Vorgehen (z. B. strukturierte Bedrohungsanalyse je Vertrauensgrenze), Auswertung der Liste TH-01 bis TH-12, Missbrauchsfälle, Restrisiken |
| Verifikation | Je Sicherheitsanforderung eine Nachweismethode (Test, Review, Analyse) |

### 8.2 Abnahme (Auszug)
Jede Bedrohung TH-xx hat Gegenmaßnahme, Verantwortlichen und Verifikationsmethode; kein offenes Risiko der Schwere „hoch" ohne Akzeptanz durch die Projektleitung; Informationssicherheit hat das Modell reviewt.

---

## 9. Teststrategie und Qualitätskriterien (Thema 7, WP5-07)

### 9.1 Testebenen (VORSCHLAG)

| Ebene | Zweck | Beispiele |
|---|---|---|
| Modultests | Logik der Module isoliert | Parser, Validierung, Zustandsautomat, Limits |
| Vertragstests | Schnittstellen halten ihr Schema ein | Schema-Prüfung, Fehlerfälle, Versionierung |
| Integrationstests im Labor | Zusammenspiel | Lauf von Anmeldung bis Versiegelung |
| Reproduzierbarkeitstests | Gleiche Eingabe → gleiches Ergebnis | Zwei Läufe in frischer Umgebung |
| Selbsttest (FR-ST-01) | Framework prüft sich mit eigenen Verfahren | Berechtigungen, Nachrichten, Speicher, Egress |
| Negativtests (NT-01 bis NT-06) | Ausschlusskatalog technisch durchgesetzt | Fehlen verbotener Fähigkeiten |
| Robustheit | Grenzwerte, manipulierte Eingaben, Ressourcenlimits | Fuzzing der eigenen Parser |
| Regressionstests | Änderungen brechen nichts | Vergleich mit früheren Ergebnissen |
| Browser-Matrix-Läufe | Verhalten je Zielbrowser | Phase 7 |

### 9.2 Qualitätskriterien (messbar zu machen in WP5-07)
- Testabdeckung: **Zielwert OFFEN (ED-7)**, nach Festlegung als Gate-Kriterium.
- Jede MUSS-Anforderung hat mindestens einen Testfall und eine Komponente (Rückverfolgbarkeit ohne Lücke).
- Statische Analyse und Linting ohne offene Befunde ab einer festzulegenden Schwere.
- Jeder Merge erfolgt nach Vier-Augen-Review; Negativtests sind Teil jedes Builds.
- Reproduzierbare, signierte Builds; SBOM und Schwachstellenprüfung der Abhängigkeiten pro Release.
- Definition of Done je Arbeitspaket in Phase 6 (Code, Tests, Dokumentation, Herkunftsprotokoll, Review).
- Testdatenregeln: ausschließlich synthetische Daten; keine Geheimnisse in Testartefakten.

---

## 10. Datenschutz- und Reporting-Konzept (Thema 8, WP5-08)

### 10.1 Datenschutzkonzept – Inhalte
- Verzeichnis der Datenkategorien und Verarbeitungen des Frameworks (auch Protokolle und Mitschnitte) mit Zweck, Rechtsgrundlage (**RA/DS**, NICHT GEPRÜFT), Speicherort, Frist, Zugriff.
- Prüfung, ob eine Datenschutz-Folgenabschätzung nötig ist (**Entscheidung DS**).
- Protokollinhalte: Datenminimierung, keine Geheimnisse, keine Rohinhalte (LOG-02); Pseudonymisierung von Bedienerkennungen (zu entscheiden).
- Löschkonzept mit Nachweis (FR-DP-03), einschließlich Zwischenablage und temporärer Dateien.
- Zugriffskonzept auf Nachweise und Berichte nach Rolle; Verteilerlisten.
- Beteiligungspflichten (Personalvertretung, **NICHT GEPRÜFT**).
- Datenschutz im Betrieb: Umgang mit Vorfällen und Meldewegen.

### 10.2 Reporting-Konzept – Inhalte
- Berichtsvorlagen (REP-01): Auftrag, Umfang, Umgebung, Methode, Ergebnisse, Befunde, Empfehlungen, offene Punkte.
- Befundmodell: ID-Schema, Evidenzklassen (BESTÄTIGT/ABGELEITET/OFFEN/NICHT GEPRÜFT), Schweregrad nach dokumentiertem Maßstab (**Maßstab: Entscheidung IS und PL**), Reproduktionsangaben.
- Belegverweise statt Rohinhalten; Schwärzungsregeln.
- Formate (AD-08) und Signatur; Versionierung und Freigabestatus.
- Qualitätskontrolle: Canary-Prüfung vor Export auf Geheimnisse/Rohinhalte; Review durch Datenschutz vor Erstveröffentlichung.
- Berichtsbeispiele auf Basis synthetischer Daten (für Phase 7).

---

## 11. Abhängigkeiten und Freigabepunkte (Thema 9)

### 11.1 Abhängigkeiten zwischen Arbeitspaketen

| Paket | benötigt zuvor | Externe Abhängigkeit |
|---|---|---|
| WP5-00 | – | Projektleitung |
| WP5-03 | ED-2 (PL) | Inventar der IT |
| WP5-01, WP5-02 | WP5-03 (für AD-01/AD-02), AD-14 und AD-11 | Technische Leitung |
| WP5-04 | WP5-01 (Eckdaten), WP5-02 | – |
| WP5-05 | WP5-04, ED-4 (PL) | Datenschutz |
| WP5-06 | WP5-04, WP5-05 | Informationssicherheit |
| WP5-07 | WP5-04 bis WP5-06 | ED-7 (PL) |
| WP5-08 | WP5-05, WP5-06 | Datenschutz, Rechtsabteilung |
| WP5-09 | WP5-02 | Rechtsabteilung |
| WP5-10 | WP5-01 bis WP5-09 | Vorgabe des Ressourcen- und Zeitrahmens (PL) |
| WP5-11 | alle | – |

Kritischer Pfad (ABGELEITET): ED-2 → WP5-03 → AD-01/AD-02 → WP5-04 → WP5-06 → WP5-11 → G3.

### 11.2 Prüfpunkte innerhalb von Phase 5

| Nr. | Prüfpunkt | Gegenstand | Beteiligte |
|---|---|---|---|
| CP5-1 | Entscheidungsregister und Browser-Matrix-Rahmen abgestimmt | WP5-00, WP5-01, WP5-03 | PL, TL |
| CP5-2 | Technologieoptionen bewertet, Vorauswahl begründet | WP5-02, WP5-09 | TL, IS, RA |
| CP5-3 | Modul-, Schnittstellen- und Datenmodell-Entwurf | WP5-04, WP5-05 | TL |
| CP5-4 | Sicherheits- und Bedrohungsmodell-Review | WP5-06 | IS |
| CP5-5 | Teststrategie und Qualitätskriterien | WP5-07 | TL |
| CP5-6 | Datenschutz- und Reporting-Konzept | WP5-08 | DS, RA |
| CP5-7 | Gesamtkonsistenz und Rückverfolgbarkeit | WP5-11 | TL, Analyst |

### 11.3 Gates (aus Roadmap)

| Gate | Status | Bedeutung für Phase 5 |
|---|---|---|
| G0 | OFFEN (nicht dokumentiert) | Formaler Beschluss der Planungsentscheidung nötig |
| G1 | OFFEN | Verantwortliche Stelle, Rechtsbewertung (OP-1 bis OP-4, OP-8), Datenschutz-Plan, Beteiligungspflichten |
| G2 | Ziel von Phase 5 | Anforderungen freigegeben, ED-1 bis ED-7 beantwortet |
| G3 | Ziel von Phase 5 | Architektur- und Bedrohungsmodell, Plattform- und Browser-Matrix |

Phase 6 darf erst nach G2 und G3 beginnen.

---

## 12. Messbare Abnahmekriterien

### 12.1 Je Arbeitspaket

| WP | Abnahmekriterium (messbar) |
|---|---|
| WP5-00 | Alle Projektleitungsentscheidungen PE-1 bis PE-13 sind protokolliert oder mit Begründung zurückgestellt (100 %) |
| WP5-01 | Jede Entscheidung AD-01 bis AD-15 hat Status „entschieden", „zurückgestellt mit Begründung und Frist" oder „entfallen"; keine mit Status „OFFEN" bei G3 (Ausnahme: AD-12, AD-15 bis Phase 6) |
| WP5-02 | Zu jeder Entscheidung mit Technologiebezug liegen mindestens zwei Alternativen mit dokumentierten Kriterien und Bewertung vor; Vorauswahl ist begründet |
| WP5-03 | Browser-Matrix enthält alle in ED-2 benannten Browser; jede Zelle trägt einen Status; Verifikationsplan für Phase 7 liegt vor |
| WP5-04 | 100 % der Schnittstellen I-01 bis I-13 haben Schema-Entwurf, Authentisierungs- und Fehlerfalldefinition sowie Limits; 100 % der Module haben Aufgaben, Ein-/Ausgaben, Abhängigkeiten, Sicherheitsanforderungen, Abgrenzung |
| WP5-05 | 100 % der Entitäten haben Klassifikation, Speicherort, Schutzmaßnahme, Frist (ED-4) und Löschverfahren |
| WP5-06 | 100 % der Bedrohungen TH-xx haben Gegenmaßnahme, Verantwortlichen und Verifikationsmethode; Review durch Informationssicherheit dokumentiert |
| WP5-07 | 100 % der MUSS-Anforderungen sind einer Komponente und mindestens einem Testfallentwurf zugeordnet; NT-01 bis NT-06 als Testfälle spezifiziert; Abdeckungsziel (ED-7) eingetragen |
| WP5-08 | Datenschutz hat das Konzept geprüft (Vermerk); Berichtsvorlage und Beispiel auf synthetischen Daten liegen vor |
| WP5-09 | SBOM-Verfahren und Lizenzprüfregeln definiert; Liste der geplanten Abhängigkeiten mit Lizenzfeldern; Herkunftsprotokoll-Format festgelegt |
| WP5-10 | Schätzung mit offen gelegten Annahmen und Unsicherheiten; Ressourcen- und Zeitrahmen der Projektleitung ist eingearbeitet |
| WP5-11 | Rückverfolgbarkeitsmatrix ohne Lücken; Suchlauf der Spezifikation nach Kennungen und Bezeichnern der Altbasis: 0 Treffer; Querverweis-Abgleich: 0 undefinierte Kennungen |

### 12.2 Für Phase 5 insgesamt
1. G2 und G3 sind erreicht (Projektleitung bzw. TL/IS bestätigen schriftlich).
2. Kein Schnittstellen-, Modul- oder Datenmodellentwurf bildet eine Fähigkeit des Ausschlusskatalogs ab (schriftliche Bestätigung nach Review).
3. Spezifikation und Konzepte enthalten keine Übernahme von Altcode oder -assets; Herkunft jedes übernommenen Inhalts ist nachvollziehbar.
4. Jeder in der Phase gemachte Annahmen- oder Technologievorbehalt ist als OFFEN oder NICHT GEPRÜFT gekennzeichnet; es wird keine nicht erteilte Freigabe als erteilt dargestellt.
5. Statusbericht an die Projektleitung mit offenen Punkten liegt vor.

---

## 13. Entscheidungs- und Freigabeübersicht

### 13.1 Entscheidungen, die die Projektleitung selbst treffen muss

| Nr. | Entscheidung | Bezug | Bemerkung |
|---|---|---|---|
| PE-1 | Formaler Beschluss der Optionsentscheidung (Option B) – Gate G0 | Roadmap, Matrix | Planungsentscheidung ist noch nicht protokolliert |
| PE-2 | Freigabe der Phase-4-Entwürfe (F-1) | Phase-4-Status | Aussteht |
| PE-3 | Benennung der verantwortlichen Stelle für Prüfgenehmigungen (ED-1) | Charta 6 | Voraussetzung für G1 |
| PE-4 | Zielbrowser, -versionen und Betriebssysteme (ED-2) | WP5-03 | Bestimmt die Plattformentscheidung |
| PE-5 | Aufbewahrungsfristen (ED-4) | WP5-05, WP5-08 | Mit Datenschutz |
| PE-6 | Umgang mit dem Arbeitsstrang „Prüfungsintegrität" und mit `OEB-Analyse-IT.md` (ED-5) | Charta 3.2 | Wissensschutz |
| PE-7 | Beteiligung von Datenschutz, Rechtsabteilung, Personalvertretung (ED-6) | Charta 6 | Voraussetzung für G1 |
| PE-8 | Zielwert der Testabdeckung (ED-7) | WP5-07 | |
| PE-9 | Ressourcen-, Zeit- und Budgetrahmen; Betriebsmodell (AD-14) | WP5-10 | Ohne Rahmen keine belastbare Schätzung |
| PE-10 | Produktname und Lizenz des neuen Projekts (AD-15) | Charta 5 | Mit Rechtsabteilung |
| PE-11 | Compliance-Entscheidung (OP-8) und Umgang mit dem Altmaterial | Matrix 3.3 | Mit Rechtsabteilung |
| PE-12 | Akzeptanz verbleibender Restrisiken (am Phasenende) | WP5-06 | |
| PE-13 | Freigabe der Anforderungen – Gate G2 | Anforderungen | |

### 13.2 Technische Entscheidungen, die ich fachlich vorbereiten kann
Entscheiden: Technische Leitung (TL) bzw. die genannte Stelle. Meine Vorbereitung: Alternativenanalyse, Bewertungskriterien, Risiken, vorläufige Empfehlung, Entscheidungsvorlage.

| Entscheidung | Entscheider | Vorbereitung im Arbeitspaket |
|---|---|---|
| AD-01 Ausführungsmodell, AD-02 Plattform, AD-03 Sprache/Laufzeit | TL (IS) | WP5-01/02/03 |
| AD-04 Nachrichtentransport, AD-05 Erweiterungsmodell | TL, IS | WP5-01/02/06 |
| AD-06 Nachweisspeicher, AD-07 Testfallformat, AD-08 Berichte/Signatur | TL (IS, DS) | WP5-02/05/08 |
| AD-09 Oberfläche, AD-10 Geheimnisverwaltung, AD-11 Laborisolation, AD-13 Zeit/Protokoll | TL, IS | WP5-02/06 |
| AD-12 Repository-/Review-/Release-Modell | TL | WP5-07/09 |
| Modul- und Schnittstellenspezifikation, Datenmodell | TL | WP5-04/05 |
| Bedrohungsmodell, Berechtigungsmodell | IS (TL) | WP5-06 |
| Teststrategie, Qualitätskriterien | TL | WP5-07 |

### 13.3 Freigaben (nicht erteilt)

| Freigabe | Gegenstand | Wer | Stand |
|---|---|---|---|
| F-1 | Phase-4-Entwürfe | Projektleitung | OFFEN |
| F-2 | Anforderungen (G2) | Projektleitung | OFFEN |
| F-3 | Lizenz-, Marken-, Herkunftsbewertung, Compliance | Rechtsabteilung (mit PL) | OFFEN |
| F-4 | Datenschutzfreigabe-Plan und Aufbewahrungskonzept | Datenschutz | OFFEN |
| F-5 | Zugriffsschutz und Ablage des Altmaterials | Informationssicherheit | OFFEN |
| F-6 | Beteiligung der Personalvertretung (falls erforderlich) | Projektleitung, Rechtsabteilung | NICHT GEPRÜFT, ob erforderlich |
| F-7 | Codeübernahme (nicht eingeplant) | Rechtsabteilung, Informationssicherheit, Projektleitung | gesperrt |
| F-8 | Architektur- und Bedrohungsmodell (G3) | Technische Leitung, Informationssicherheit | OFFEN (Ergebnis von Phase 5) |

---

## 14. Risiken der Phase 5

| Nr. | Risiko | Gegenmaßnahme | Bezug |
|---|---|---|---|
| R5-1 | Beginn vor Erfüllung von G0/G1 führt zu Ergebnissen, die später verworfen werden müssen | Status „Entwurf unter Vorbehalt"; rechtsabhängige Festlegungen zurückstellen | 1.2 |
| R5-2 | Technologieentscheidungen ohne Browser-Matrix | WP5-03 vor AD-01/AD-02 | 3 |
| R5-3 | Funktionsdrift in Richtung ausgeschlossener Fähigkeiten durch Anforderungen „aus Bequemlichkeit" | Review gegen Ausschlusskatalog je Checkpoint; Stop-Kriterien | M-02 |
| R5-4 | Überladene Spezifikation, die in Phase 6 nicht umsetzbar ist | Priorisierung nach MUSS/SOLL; Inkrementplan | M-14 |
| R5-5 | Offene Entscheidungen der Projektleitung verzögern die Kette | Entscheidungsvorlagen mit Frist; Eskalation an PL | PE-1 bis PE-13 |
| R5-6 | Spezifikationsautoren greifen versehentlich auf Altcode zurück | Zugriffstrennung, Herkunftsprotokoll, Suchlauf-Kontrolle (WP5-11) | M-07 |
| R5-7 | Unzureichende Datenschutzklärung führt zu Nacharbeit | DS frühzeitig einbinden (CP5-6) | P-7 |
| R5-8 | Aufwandsschätzung ohne Rahmen unbrauchbar | PE-9 | OP-6 |
| R5-9 | Dokumentationsdrift zwischen den vielen Dokumenten | Querverweis-Abgleich je Checkpoint; Änderungsprotokoll | M-15 |

---

## 15. Nicht-Ziele und offene Punkte

**Nicht-Ziele dieses Plans:** Implementierung; Untersuchung oder Wiedereinführung ausgeschlossener Funktionen; Übernahme von Altcode oder -assets; Darstellung nicht erteilter Freigaben als erteilt; Festlegung von Produktnamen oder Lizenz.

| Nr. | Offener Punkt | Status |
|---|---|---|
| OF5-1 | Formaler Beschluss G0, Freigabe F-1 | OFFEN |
| OF5-2 | Zielbrowser (ED-2), Browser-Matrix | OFFEN / NICHT GEPRÜFT |
| OF5-3 | Aufwand, Dauer, Personal (OP-6) | OFFEN |
| OF5-4 | Lizenz-, Herkunfts- und Compliance-Klärung (OP-1 bis OP-4, OP-8) | OFFEN |
| OF5-5 | Technologieentscheidungen AD-01 bis AD-15 | OFFEN |
| OF5-6 | Laufzeit-, Browserkompatibilitäts- und Netzwerktests | NICHT GEPRÜFT (frühestens Phase 6/7) |
