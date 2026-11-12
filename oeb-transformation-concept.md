# Transformationskonzept: Vom OEB-Projekt zu einem eigenständigen defensiven Security-Testing-Framework (Phase 4)

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** Entwurf zur Entscheidung, **nicht freigegeben** · **Klassifizierung:** intern
**Zugehörig:** `project-charter.md` · `transformation-decision-matrix.md` · `target-architecture.md` · `project-roadmap.md` · `phase-4-status.md` · Phase-3-Dokumente (`security-privacy-audit.md`, `reuse-assessment.md`, `security-testing-requirements.md`, `migration-risk-assessment.md`) · Vorarbeiten (`technical-handover.md`, `OEB-Analyse-IT.md`, `verification-report.md`, `documentation-gaps.md`)

**Kennzeichnung:** BESTÄTIGT (durch Quellcode oder Dokumente belegt) · ABGELEITET (Schlussfolgerung) · OFFEN (Klärung nötig; Grund angegeben) · NICHT GEPRÜFT (außerhalb des Prüfumfangs). Entwurfsaussagen sind als **VORSCHLAG** gekennzeichnet; sie werden in Phase 5 bestätigt oder verworfen.
**Prüfstatus:** Es wurden keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt. Dieses Dokument ist reine Konzeptarbeit. Es wurden keine Quell-, Konfigurations- oder Build-Dateien verändert und keine ausgeschlossenen Funktionen untersucht oder rekonstruiert.

---

## 1. Zweck und Rahmen

Dieses Dokument beschreibt, wie aus dem bisherigen OEB-Projekt ein **eigenständiges, defensives** Framework entstehen kann, **ohne dass Altcode vorausgesetzt wird**. Es respektiert die offene Lizenzlage (`transformation-decision-matrix.md` 3): Es wird keine Übernahme von Quellcode, Assets, Logos oder fremden Projektbestandteilen vorausgesetzt.

**Verbindliche Rahmenbedingungen** (unverändert aus Phase 3/4): Ausschlusskatalog AK-01 bis AK-09 (`security-testing-requirements.md` 1.3), Datenschutz-Voraussetzungen P-1 bis P-7, Sicherheitsbefunde SEC-01 bis SEC-15 und NF-01 bis NF-06, Stop-Kriterien der Charta (7.2). Funktionen zur Umgehung von Prüfungssoftware oder zur automatisierten Erfassung und Beantwortung von Prüfungsfragen werden nicht entwickelt, wiederhergestellt, verbessert oder vorbereitet.

**Was dieses Dokument nicht tut:** Es beschreibt die Funktionsweise der ausgeschlossenen Bereiche nicht, und es bewertet keine Komponente der Altbasis nach ihrer Funktionstauglichkeit. Der Ausschlussbereich wird nur als Gesamtmenge behandelt (Abschnitt 5).

---

## 2. Die drei Kategorien der „Wiederverwendung"

Alle Aussagen dieses Konzepts ordnen sich einer von drei Kategorien zu. Sie haben unterschiedliche Voraussetzungen und werden strikt getrennt behandelt.

| Kat. | Bezeichnung | Gegenstand | Voraussetzung | Status |
|---|---|---|---|---|
| **K-D** | Wiederverwendung von **Dokumentation und Erkenntnissen** | Eigene Analyse- und Planungsdokumente, Befunde, Anforderungen, Methodik, Kennzeichnungsschema | Interne Verwendung; Zugriffsbeschränkung für Dokumente, die die Altbasis beschreiben | **Zulässig** (eigenes Werk) |
| **K-K** | **Wiederaufbau allgemeiner Konzepte** | Allgemeine Muster und Ideen, die **unabhängig neu spezifiziert und neu implementiert** werden; es wird kein Code und kein Text kopiert | Neuimplementierung auf Grundlage einer Anforderung/Spezifikation; Herkunftsprotokoll; rechtliche Bestätigung, dass allgemeine Muster frei sind (OP-4) | **Zulässig unter Vorbehalt:** Nach Einschätzung des Analysten unkritisch (ABGELEITET), rechtliche Bestätigung OFFEN |
| **K-C** | **Tatsächliche Wiederverwendung von Quellcode** (einschließlich Assets, Logos, Texten, Konfigurations- und Build-Dateien der Altbasis) | Datei- oder zeilenweise Übernahme | **Schriftliche Lizenzfreigabe UND schriftliche Sicherheitsfreigabe**, je Datei (Abschnitt 7.3) | **GESPERRT.** Bleibt gesperrt, bis beide Freigaben schriftlich vorliegen. Heute liegt keine vor. |

**Konsequenz:** Dieses Konzept plant mit K-D und K-K. K-C wird nicht eingeplant. Sollte eine Freigabe später erteilt werden, wäre sie ein **Änderungsantrag** an die Projektleitung (Abschnitt 7.4), kein Bestandteil des Basisplans.

---

## 3. Übernehmbare Erkenntnisse und Anforderungen (Kategorie K-D)

Die folgenden Erkenntnisse stammen aus Phase 1 bis 3 und gehen als **Anforderungen oder Entwurfsvorgaben** in das neue Projekt ein. Sie beschreiben, **was zu vermeiden oder sicherzustellen ist**, nicht wie die Altbasis arbeitet.

| Nr. | Erkenntnis | Quelle | Wird zu |
|---|---|---|---|
| EK-1 | Zugangsdaten gehören nicht im Klartext in den Browserspeicher; das neue Werkzeug braucht keine Fremd-API-Schlüssel | SEC-03, DS-01 | SR-NF-05, SR-NF-08 |
| EK-2 | Seiteninhalte, Formularfelder und Prüfungsinhalte dürfen nicht dauerhaft gespeichert werden; Testdaten sind synthetisch | SEC-04, DS-02, DS-03, DS-06 | SR-NF-12, LAB-03, FR-DP-02 |
| EK-3 | Nachrichten dürfen sensible Inhalte nicht an offene Ziele senden; Empfänger prüfen Absender und Schema; Frames/Elemente nicht allein per Kennung vertrauen | NF-01, NF-04, SEC-05, SEC-15 | SR-NF-07, FR-MC-02 bis FR-MC-04 |
| EK-4 | Das Rechteprofil muss minimal und begründet sein; pauschale Host-Rechte und Eingriff in fremden Verkehr entfallen | SEC-02, SEC-11 | SR-NF-02, FR-PM-01, FR-PM-02 |
| EK-5 | Auf **allen** Eingabepfaden gelten Größen-, Zeit- und Tiefenlimits | SEC-13, NF-06 | SR-NF-04, FR-IV-02 |
| EK-6 | Ein Prüfwerkzeug braucht Ereignisprotokoll, Fehlertransparenz und kein „fail-open" | NF-03 | LOG-01 bis LOG-06, SR-NF-13 |
| EK-7 | Zustandsänderungen müssen konsistent sein: Teilfehler dürfen keinen halb aktiven Zustand hinterlassen; konkurrierende Pfade sind zu serialisieren | TR-01, TR-02, TR-10 (SEC-14) | Zustandsmodell in `target-architecture.md` 7.3; FR-TC-02 |
| EK-8 | Fernaktualisierung nur authentifiziert und aus eigener Infrastruktur | SEC-06 | SR-NF-06 |
| EK-9 | Lösch- und Aufbewahrungsfunktionen müssen nachweislich wirken, einschließlich Zwischenablage | Audit Abschnitt 7, NF-05 | SR-NF-12, LAB-08, FR-DP-03 |
| EK-10 | Plattform- und Browserabhängigkeiten (auslaufende Schnittstellen, Unterschiede zwischen Browsern, Verfügbarkeit von Sitzungsspeichern) sind früh zu klären | SEC-02; UA-01 bis UA-03 | Browser-Matrix LAB-04, SR-NF-01 |
| EK-11 | Qualität: Tests, CI, Lizenzdatei, Changelog, SBOM von Beginn an | TR-08, SEC-10 | SR-NF-09 bis SR-NF-11 |
| EK-12 | Methodik: Befund-IDs, Evidenzklassen (BESTÄTIGT/ABGELEITET/OFFEN/NICHT GEPRÜFT), Änderungsprotokoll, Querverweisprüfung, Gate-Modell | Phase 1–4 | REP-02, SR-NF-14, Charta 7 |

**Schutz des Wissens (K-D):** Dokumente, die die ausgeschlossenen Bereiche der Altbasis beschreiben (`technical-handover.md`, `OEB-Analyse-IT.md`, `verification-report.md`, `documentation-gaps.md`), bleiben zugriffsbeschränkt und gehören nicht in das Repository des neuen Projekts. In das neue Projekt gehen **nur die daraus abgeleiteten Anforderungen** (Tabelle oben) ein (M-07, `migration-risk-assessment.md`).

---

## 4. Neue, eigenständige Projektstruktur (VORSCHLAG)

Technologieneutraler Vorschlag für das Repository des neuen Projekts. Er enthält **keinen** Verzeichnis- oder Dateinamen der Altbasis und ist von deren Gliederung unabhängig. Die endgültige Struktur folgt aus der Architekturentscheidung in Phase 5.

| Bereich (Verzeichnis, Arbeitsname) | Inhalt | Entsprechung in `target-architecture.md` |
|---|---|---|
| `docs/` | Anforderungen, Architektur, Entwurfsentscheidungen (ADR), Bedrohungsmodell, Betriebs- und Sicherheitsdokumentation, Berichtsvorlagen | Reporting, Dokumentation |
| `governance/` | Schemata und Vorlagen für Prüfauftrag und Scope-Allowlist, Negativliste, Rollen- und Freigabeprozess | Governance Gate |
| `core/` | Test Runner (Teststeuerung), Laufzustand, Nachrichtenschicht mit Schemata, Ereignisprotokoll | Test Runner, Querschnitt |
| `environment/` | Beschreibung und Bereitstellung der Laborumgebung und der Browserprofile | Browser Environment |
| `checks/` | Prüfmodule je Familie: Sicherheitsmechanismen, Berechtigungen, Kommunikation, Datenschutz | Security Checks, Permission Analysis, Communication Security, Privacy Analysis |
| `evidence/` | Nachweisspeicher, Integritätsschutz, Aufbewahrung | Evidence Manager |
| `reporting/` | Berichtsgenerator, Vorlagen, Exportformate | Reporting |
| `ui/` | Bedienoberfläche (Präsentationsschicht, ohne Prüflogik) | Benutzeroberfläche |
| `testcases/` | Testkatalog (maschinenlesbar, versioniert) und synthetische Testdaten | Test Runner |
| `policy-tests/` | Negativtests zur Ausschlusskontrolle (NT-01 bis NT-06) und Lizenz-/Herkunftsprüfung | Querschnitt |
| `build/` | Build, CI, SBOM-Erzeugung, Signierung | – |
| Wurzel | `LICENSE` (Lizenzentscheidung der Organisation, OFFEN), `NOTICE`, `CHANGELOG`, `SECURITY`, `CODEOWNERS`, `PROVENANCE` (Herkunftsprotokoll, Abschnitt 6) | – |

**Strukturregeln (VORSCHLAG):**
1. Eigener Produktname ohne Bezug zur Altbasis (Charta 5; Name OFFEN, Marken- und Namensprüfung durch die Rechtsabteilung).
2. Keine Bezeichner, Speicherschlüssel, Kennungen oder Konstanten der Altbasis.
3. Jede Datei hat ein Herkunftsprotokoll-Eintrag (Autor, Datum, Spezifikationsbezug, bei Fremdbestandteilen Lizenznachweis).
4. Vier-Augen-Prinzip für Änderungen; geschützte Hauptlinie; automatische Negativtests bei jedem Build.
5. Das Repository enthält **keine** Dateien der Altbasis; keine Historie der Altbasis.

---

## 5. Bereiche, die vollständig entfallen

Aus dem neuen Projekt entfallen **ersatzlos**:

1. **Alle funktionalen Kernbereiche der Altbasis**, die dem Ausschlusskatalog AK-01 bis AK-09 unterliegen (nach der Phase-4-Zählung 30 Dateien mit 5 346 Zeilen; `reuse-assessment.md` 3.1 bis 3.4, `transformation-decision-matrix.md` 2). Das Konzept beschreibt sie nicht weiter und überführt aus ihnen **keine Funktion**.
2. **Funktionsgebundene Projektstruktur** und alle Bezeichner, Kennungen und Speicherschlüssel der Altbasis (Abschnitt 4, Regel 2).
3. **Fremde Assets und Marken-/Gestaltungselemente**, Vorschaumaterial und deren Texte (Herkunft ungeklärt, `transformation-decision-matrix.md` 3.1, V-4 bis V-7; teilweise Ausschlussbereich).
4. **Persistente Klartext-Konten und Prüfungsdaten**, Fernaktualisierung ohne Authentizitätsprüfung, Mitschnitt/Zweitabruf fremder Antworten (EK-1, EK-2, EK-8, AK-07).
5. **Technisch ungeeignete Gerüste**, die nicht ausgeschlossen, aber zu ersetzen sind (Kategorie 3 der Phase-4-Zählung: 4 Dateien, 304 Zeilen): Sie werden neu entworfen.

**Nicht „entfallen", sondern isoliert:** Das Ursprungsarchiv und die Dokumente zur Altbasis bleiben als Referenz zugriffsbeschränkt erhalten (`migration-risk-assessment.md` 5).

---

## 6. Allgemeine Konzepte, die unabhängig neu umgesetzt werden können (Kategorie K-K)

Jedes Konzept wird als **Anforderung beschrieben und neu implementiert**. Das Vorgehen: (1) Der Konzeptgedanke wird in der Spezifikation (Phase 5) in eigenen Worten und mit eigenen Anforderungen festgelegt; (2) Implementierer arbeiten ausschließlich nach dieser Spezifikation und erhalten keinen Altcode; (3) ein Reviewer prüft die Herkunft und trägt sie in das Herkunftsprotokoll ein. **Vorbehalt:** Bestätigung der Rechtsabteilung zu OP-4.

| Nr. | Allgemeines Konzept | Neue Umsetzung (VORSCHLAG) | Verantwortungsbereich |
|---|---|---|---|
| KO-1 | Eine zentrale Stelle nimmt interne Nachrichten entgegen und antwortet in einem einheitlichen Format | Nachrichtenschicht mit versioniertem Schema, Absender-Autorisierung je Typ, einheitlichem Antwort- und Fehlerformat | Querschnitt / Test Runner |
| KO-2 | Dokumentierter Zustandsautomat mit definierten Übergängen | Laufzustandsmodell mit Teilfehlerbehandlung und Serialisierung (EK-7) | Test Runner |
| KO-3 | Abhängigkeiten (Netzwerk, Speicher, Zeit) sind injizierbar, damit Logik testbar ist | Schnittstellen statt fester Aufrufe; Test-Doppel für deterministische Läufe | alle Module |
| KO-4 | Ein Merkmal wirkt nur für ausdrücklich freigegebene Ziele | Scope-Allowlist mit Prüfauftrag und Negativliste (GOV-02, GOV-03) | Governance Gate, Browser Environment |
| KO-5 | Darstellung ausschließlich über sichere Textzuweisung | UI-Regeln und statische Prüfung (SR-NF-03) | Benutzeroberfläche |
| KO-6 | Größen-, Zeit- und Tiefenlimits an Systemgrenzen | Zentrale Limit-Konfiguration, Pflicht an jedem Eingabepfad (SR-NF-04) | alle Module |
| KO-7 | Eine Quelle, mehrere Zielpakete, Prüfung im Build | Neues Build-Konzept mit Lint, Tests, SBOM, Signierung (SR-NF-09) | `build/` |
| KO-8 | Normalisierung von Einstellungen mit Standardwerten | Konfigurationsschicht mit Schema-Version, Validierung, Migration | Querschnitt |
| KO-9 | Reiter-Oberfläche mit Statusanzeige | Bedienkonzept der Präsentationsschicht (ohne Prüflogik) | Benutzeroberfläche |
| KO-10 | Evidenzklassen und Befund-IDs | Berichts- und Befundmodell (REP-02, REP-03) | Reporting, Evidence Manager |

**Hinweis:** Diese Konzepte sind allgemeiner Natur und kommen in sehr vielen Software-Projekten vor. Ob sie rechtlich als frei anzusehen sind, ist **nicht Gegenstand einer Analystenentscheidung**, sondern der Rechtsabteilung (OP-4).

---

## 7. Lizenzklärung und Compliance-Freigabe im Projektablauf

### 7.1 Klärungsschritte

| Nr. | Schritt | Phase | Verantwortlich | Nachweis | Folge bei negativem/fehlendem Ergebnis |
|---|---|---|---|---|---|
| LC-1 | Beauftragung der Rechtsabteilung mit OP-1 bis OP-4 und OP-8 (`transformation-decision-matrix.md` 6, `phase-4-status.md` 5) | 4 | Projektleitung | Schriftlicher Auftrag | Keine Aussagen zur Lizenzlage möglich; K-C bleibt gesperrt |
| LC-2 | Einsicht in die Originalrepositories der beiden Vorlageprojekte und Ermittlung ihrer Lizenzen (OP-1) | 4 | Rechtsabteilung | Lizenztexte und Fundstelle mit Datum | K-C bleibt gesperrt |
| LC-3 | Ermittlung des Rechteinhabers und der Weitergabebedingungen der Altbasis (OP-2) | 4 | Rechtsabteilung | Schriftliche Feststellung | K-C bleibt gesperrt |
| LC-4 | Herkunftsklärung für Assets, Logos, Symbole, Vorschaudateien, Fremdvermerke (OP-3) | 4 | Rechtsabteilung | Je Datei: Herkunft, Rechte | Betroffene Dateien bleiben gesperrt; Ausschlussbereiche ohnehin |
| LC-5 | Compliance-Bewertung der Weiterverwendung eines Projekts, dessen Zweck die Umgehung von Prüfungssoftware ist (OP-8/H-6) | 4 | Projektleitung mit Rechtsabteilung | Schriftliche Entscheidung | Bei negativer Entscheidung: K-C dauerhaft ausgeschlossen |
| LC-6 | Rechtliche Bestätigung zu K-K (OP-4) | 4/5 | Rechtsabteilung | Schriftliche Einschätzung | K-K wird auf K-D und eigene Konzepte ohne Bezug zur Altbasis beschränkt (Option C) |
| LC-7 | Lizenzprüfung der **neuen** Abhängigkeiten und Werkzeuge | 5 | Technische Leitung, Rechtsabteilung | SBOM mit Lizenzfeldern, Prüfvermerk | Abhängigkeit wird ersetzt |
| LC-8 | Herkunftsprotokoll fortlaufend führen und je Inkrement prüfen | 6 | Entwicklung, Reviewer | `PROVENANCE`-Einträge | Inkrement wird nicht abgenommen |
| LC-9 | Abschließender Nachweis aller Lizenzen, Marken und Herkünfte | 8 | Rechtsabteilung | Freigabevermerk | Kein Release (G6) |

### 7.2 Einbindung in die Gates

| Gate | Lizenz-/Compliance-Anteil |
|---|---|
| G0 | Optionsentscheidung berücksichtigt den Stand von LC-1 bis LC-6 (auch wenn offen) |
| G1 | LC-1 bis LC-6 bewertet; Datenschutz-/Rechtsfreigabeplan vorhanden |
| G3 | LC-7 im Konzept eingeplant |
| G4 | LC-8: Herkunftsprotokoll lückenlos |
| G6 | LC-9 abgeschlossen |

### 7.3 Freigabeverfahren für K-C (nur falls beantragt)

K-C ist **gesperrt**. Eine Entsperrung erfordert für **jede einzelne Datei** kumulativ:

| Nr. | Anforderung | Aussteller |
|---|---|---|
| CF-1 | Schriftliche **Lizenzfreigabe**: Nachweis der Rechte (Lizenztext, Rechteinhaber) und Zulässigkeit der konkreten Verwendung | Rechtsabteilung |
| CF-2 | Schriftliche **Compliance-Freigabe** (LC-5) | Projektleitung |
| CF-3 | Schriftliche **Sicherheitsfreigabe**: Review auf bekannte Befunde (SEC-xx), Abwesenheit von Ausschlussfähigkeiten, Prüfung auf verdeckte Kopplungen | Informationssicherheit |
| CF-4 | Bestehen der Negativtests NT-01 bis NT-06 für die Datei im Zielkontext | Technische Leitung |
| CF-5 | Dokumentierter Zweck der Übernahme (welches Anforderungsziel wird nur durch diese Datei erreicht) | Antragsteller |
| CF-6 | Eintrag im Herkunftsprotokoll | Reviewer |

Fehlt eine dieser Voraussetzungen, bleibt die Datei gesperrt. Eine Freigabe für eine Datei gilt nicht für andere Dateien. Eine mündliche oder stillschweigende Freigabe gilt nicht.

### 7.4 Änderungsantrag
Jede Entsperrung von K-C ist ein **Änderungsantrag** an die Projektleitung mit Auswirkung auf Roadmap (Zusatzschritte Z-A1 bis Z-A6, `project-roadmap.md` 4), Risikoregister und Matrix.

---

## 8. Vor- und Nachteile: OEB-basierter Projektstart gegenüber vollständiger Neuentwicklung

„OEB-basierter Projektstart" bedeutet hier zwei unterscheidbare Dinge:
- **(i) Erkenntnis-basierter Start** (K-D und K-K): Das neue Projekt beginnt mit dem dokumentierten Wissen aus Phase 1–4, aber ohne Altcode. Entspricht Option B der Matrix.
- **(ii) Code-basierter Start** (K-C): Das neue Projekt beginnt auf Altbestand. Entspricht Option A; **gesperrt**.

Option C der Matrix („vollständige Neuentwicklung") verzichtet auf die dokumentierte Wissensübernahme.

| Aspekt | (i) Erkenntnis-basierter Start (B) | (ii) Code-basierter Start (A, gesperrt) | Vollständige Neuentwicklung (C) |
|---|---|---|---|
| **Vorteile** | Nutzt vorhandene, geprüfte Erkenntnisse und Methodik; vermeidet bekannte Fehler früh; keine Codeübernahme, daher keine Abhängigkeit von Lizenzklärung (außer OP-4); Kontinuität im Team | Möglicher Zeitgewinn nur bei Gerüsten (rund 296 Zeilen, ca. 4 % des Bestands, `transformation-decision-matrix.md` 2); Kontinuität der Dateistruktur | Maximale Unabhängigkeit; keine Anknüpfung an die Altbasis; geringster Prüfaufwand für Herkunft |
| **Nachteile** | Zusatzaufwand für das Führen des Herkunftsprotokolls und für die Trennung Spezifikation/Implementierung; Wissensschutz verlangt Zugriffssteuerung | Lizenz- und Herkunftsrisiko; Import und Rückbau des Ausschlussbereichs; Nachweis der Abwesenheit; Altlasten (Plattform, Struktur, fehlende Tests); belastete Produktidentität; Reputations-/Compliance-Risiko | Verzicht auf bereits erarbeitete Erkenntnisse (Risiko gleichartiger Fehler); eigenes Gerüst muss neu entworfen werden |
| **Lizenzrisiko** | gering bis mittel (OP-4 offen) | hoch (alle OP offen) | gering |
| **Sicherheitsrisiko** | gering | mittel bis hoch | gering bis mittel (Eigenfehler) |
| **Aufwand** | mittel | ungünstig (OFFEN, keine Basis) | mittel bis günstig |
| **Wartbarkeit/Erweiterbarkeit** | günstig | ungünstig | günstig |

**Fazit (ABGELEITET):** Der Mehrwert eines „OEB-basierten" Starts liegt **in den Erkenntnissen, nicht im Code**. Ein code-basierter Start hat nach heutiger Kenntnis keinen belastbaren wirtschaftlichen Vorteil; seine Voraussetzungen liegen teilweise außerhalb des Projekteinflusses. Die Empfehlung (Option B, hilfsweise C) bleibt unverändert; die Entscheidung trifft die Projektleitung an Gate G0.

---

## 9. Wirtschaftliche Einordnung

Es liegt keine belastbare Aufwands- oder Kostenschätzung vor (OFFEN, `project-roadmap.md` RP-1). Qualitativ:

- **Gleich in allen Optionen:** Entwicklung der Prüfmodule, des Test Runners, des Nachweis- und Berichtswesens, der Governance und der Qualitätssicherung (die Altbasis liefert dafür keine Funktion).
- **Nur bei Option A zusätzlich:** Lizenz- und Compliance-Klärung als Vorbedingung, Quarantäne, Rückbau mit Nachweis, Bezeichnerbereinigung.
- **Möglicher Gewinn bei Option A:** begrenzt auf Gerüste (rund 4 % des Bestands).
- **Bei Option B/C zusätzlich:** Herkunftsprotokoll und Spezifikations-/Implementierungstrennung (geringer Zusatzaufwand).

Eine monetäre Bewertung ist nach der Architekturentscheidung (Phase 5) möglich.

---

## 10. Übergangsplan (Kurzfassung)

| Schritt | Inhalt | Phase |
|---|---|---|
| US-1 | Entscheidung über Option; Beauftragung LC-1 | 4 |
| US-2 | Isolierung des Altmaterials (Ablage, Zugriff, Prüfsumme) | 4/5 |
| US-3 | Ableitung der Anforderungen aus den Erkenntnissen (Tabelle Abschnitt 3, EK-1 bis EK-12) in die freigegebene Anforderungsliste | 5 |
| US-4 | Spezifikation der K-K-Konzepte (Abschnitt 6) | 5 |
| US-5 | Anlage des neuen Repositories nach Abschnitt 4, ohne Altbestand | 6 |
| US-6 | Aufbau nach Freigabe, mit Herkunftsprotokoll und Negativtests | 6 |
| US-7 | Prüfung, Test, Bericht, Release | 7/8 |

---

## 11. Konsistenz und offene Punkte

- **Konsistenz:** Die Optionsbezeichnungen A/B/C folgen `transformation-decision-matrix.md` 4; die Zuordnung zu Phase 3 steht dort. Gates G0 bis G6, Entscheidungen ED-1 bis ED-7 und Rollen folgen `project-charter.md` und `project-roadmap.md`. Bestandszahlen: 6 735 Zeilen Quelltext in 37 Dateien; Ausschlussbereich 30 Dateien/5 346 Zeilen; ersetzbare Gerüste 4 Dateien/304 Zeilen; Gerüste für Überarbeitung 296 Zeilen (ohne das ungeprüfte Stylesheet).
- **Offen:**

| Nr. | Punkt | Status |
|---|---|---|
| OF-1 | Lizenzen der Vorlageprojekte, Rechteinhaber, Asset-Herkunft (OP-1 bis OP-3) | OFFEN |
| OF-2 | Rechtliche Bestätigung zu K-K (OP-4) | OFFEN |
| OF-3 | Compliance-Bewertung (OP-8) | OFFEN |
| OF-4 | Lizenz des neuen Projekts und Produktname | OFFEN (Entscheidung der Organisation) |
| OF-5 | Aufwands-, Zeit- und Kostenbasis | OFFEN |
| OF-6 | Laufzeit-, Browser- und Netzwerktests | NICHT GEPRÜFT |
