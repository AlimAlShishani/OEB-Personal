# Projektcharta: Browser-Security-Testing-Framework (Arbeitstitel)

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** **Entwurf, nicht freigegeben** · **Klassifizierung:** intern
**Zugehörig:** `oeb-transformation-concept.md` · `target-architecture.md` · `transformation-decision-matrix.md` · `project-roadmap.md` · `phase-4-status.md` · Phase-3-Dokumente (`security-privacy-audit.md`, `reuse-assessment.md`, `security-testing-requirements.md`, `migration-risk-assessment.md`) · Vorarbeiten (`technical-handover.md`, `OEB-Analyse-IT.md`, `verification-report.md`, `documentation-gaps.md`)

**Kennzeichnung:** BESTÄTIGT (durch Quellcode bzw. Dokumente belegt) · ABGELEITET (Schlussfolgerung) · OFFEN (Klärung nötig) · NICHT GEPRÜFT (außerhalb des Prüfumfangs).
**Prüfstatus:** Es wurden bisher keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt. Dieses Dokument ist reine Planung.

---

## 1. Zweck und Vision

Das Projekt schafft ein **eigenständiges, defensives Browser-Security-Testing-Framework** mit eigener Produktidentität und klar begrenztem Funktionsumfang. Es wird ausschließlich in **autorisierten, isolierten Testumgebungen** eingesetzt, um

- Browser-Sicherheitskonfigurationen zu überprüfen,
- Berechtigungsmodelle zu analysieren,
- die Sicherheit interner Nachrichtenkommunikation zu untersuchen,
- Eingabevalidierung und Fehlerbehandlung zu testen,
- lokale Datenspeicherung und Datenschutz zu bewerten,
- reproduzierbare defensive Testfälle auszuführen,
- Testergebnisse und technische Nachweise zu dokumentieren,
- professionelle Sicherheitsberichte zu erstellen.

Die Funktionalität der bisherigen OEB-Erweiterung (SEB-bezogene Funktionen) soll **nicht** erhalten werden.

## 2. Ziele und Erfolgskriterien

| Nr. | Ziel | Erfolgskriterium (messbar) |
|---|---|---|
| Z-1 | Belastbare Entscheidung zur Ausgangsbasis | Entscheidung der Projektleitung ist dokumentiert (Gate G0), Lizenz- und Herkunftsstand ist schriftlich bewertet |
| Z-2 | Freigegebene Anforderungen | `security-testing-requirements.md` ist freigegeben, offene Entscheidungen ED-1 bis ED-7 sind beantwortet (Gate G2) |
| Z-3 | Prüffähige Architektur | Architektur- und Bedrohungsmodell liegen vor und sind reviewt (Gate G3) |
| Z-4 | Funktionsfähiger Labor-Prototyp | Alle MUSS-Anforderungen sind einem Testfall zugeordnet; Negativtests zum Ausschlusskatalog sind grün (Gate G4) |
| Z-5 | Nachgewiesene Qualität | Selbsttest ohne offene Befunde der Schwere „hoch"; erster Prüfbericht freigegeben (Gate G5) |
| Z-6 | Freigabe zum Einsatz | Dokumentation, Governance und Release-Freigabe abgeschlossen (Gate G6) |

## 3. Abgrenzung

### 3.1 Im Umfang
Prüfobjekte nach `security-testing-requirements.md` 1.2: verwaltete Browser-Konfigurationen und -Richtlinien, eigene oder freigegebene Erweiterungen und harmlose Testerweiterungen, eigene Webanwendungen/Testserver im Labor, das Framework selbst (Selbsttest).

### 3.2 Nicht im Umfang (verbindlicher Ausschlusskatalog)
Der Ausschlusskatalog **AK-01 bis AK-09** (`security-testing-requirements.md` 1.3) gilt unverändert. Insbesondere werden **keine** Funktionen zur Umgehung von Prüfungssoftware, zur Nachahmung fremder Clients oder Prüfoberflächen, zur Erfassung von Prüfungskonfigurationen oder zur automatisierten Erfassung und Beantwortung realer Prüfungsfragen entwickelt, wiederhergestellt, verbessert oder vorbereitet. Tests gegen reale Prüfungsplattformen finden nicht statt.

Der Arbeitsstrang „Prüfungsintegrität" (Erkennung der Nutzung solcher Werkzeuge) ist **kein** Teil dieses Projekts. Er benötigt einen gesonderten Auftrag und Rahmen (ED-5).

## 4. Verbindliche Rahmenbedingungen aus den Vorphasen

| Nr. | Rahmenbedingung | Quelle |
|---|---|---|
| R-1 | Ausschlusskatalog AK-01 bis AK-09 | Phase 3 |
| R-2 | Datenschutz-Voraussetzungen P-1 bis P-7 (u. a. keine Fremdübertragung, keine Speicherung von Seiteninhalten/Geheimnissen, Lösch- und Aufbewahrungsfunktion, Lizenz-/Markenklärung, Datenschutzfreigabe) | `security-privacy-audit.md` 10 |
| R-3 | Sicherheitsbefunde SEC-01 bis SEC-15 und NF-01 bis NF-06 sind durch Anforderungen zu ersetzen; ein Wiederauftreten gleichartiger Mängel ist ein Abnahmehindernis | `security-privacy-audit.md` 2, 9 |
| R-4 | Ein Bestandteil der OEB-Grundlage gilt nur dann als wiederverwendbar, wenn seine Sicherheits- **und** Lizenzgrundlage ausreichend geklärt ist. **Stand heute: nicht erfüllt** (Lizenz OFFEN, siehe `transformation-decision-matrix.md` 3) | Auftrag Phase 4 |
| R-5 | Synthetische Testdaten, isolierte Browserprofile, minimale Berechtigungen, validierte interne Nachrichten, sichere Zugangsdatenhaltung, datenschutzgerechtes Logging, reproduzierbare Ergebnisse | Auftrag Phase 4; `security-testing-requirements.md` |
| R-6 | Nur Dokumentation in dieser Phase: keine Änderung von Quell-, Konfigurations- oder Build-Dateien | Auftrag Phase 4 |

## 5. Produktidentität

- Das Produkt erhält einen **eigenen Namen** ohne Bezug zu „OEB", „Open Exam Browser" oder „SEB". Der Arbeitstitel in diesen Dokumenten ist „Browser-Security-Testing-Framework". **Der Produktname ist OFFEN.**
- Namenskriterien: keine Verwechslungsgefahr mit bestehenden Marken, keine Anspielung auf Prüfungsumgehung, Prüfung auf Namens- und Markenkollision durch die Rechtsabteilung (NICHT GEPRÜFT).
- Bezeichner, Kennungen und Speicherschlüssel des neuen Projekts tragen keine Namen der Altbasis (Anforderung für Phase 5).

## 6. Organisation und Rollen

Personen sind in diesem Entwurf nicht benannt; die Zuordnung obliegt der Projektleitung (**OFFEN**, ED-1).

| Rolle | Aufgabe |
|---|---|
| Auftraggeber / Projektleitung | Entscheidungen an den Gates, Freigabe von Charta und Roadmap |
| Verantwortliche Stelle für Prüfgenehmigungen | Prüfaufträge, Scope-Dateien, Freigabe von Testläufen (GOV-01 bis GOV-04) |
| Technische Leitung | Architektur, technische Spezifikation, Qualitätssicherung |
| Entwicklung | Umsetzung freigegebener Komponenten |
| Datenschutz | Freigaben und Begleitung (P-7), Aufbewahrungskonzept |
| Rechtsabteilung | Lizenz-, Marken- und Herkunftsbewertung, Namensprüfung, Rechtsfragen zu Beteiligungspflichten |
| Betriebsrat / Personalvertretung | Beteiligung, soweit Test- oder Auswertungsverfahren sie berühren (NICHT GEPRÜFT, ob erforderlich) |
| Informationssicherheit | Review, Ablage und Zugriffsschutz des Altmaterials |

## 7. Governance und Entscheidungen

### 7.1 Gates
Die Gates G0 bis G5 aus Phase 3 werden übernommen und um G6 (Release) ergänzt. Details und Abnahmekriterien: `project-roadmap.md`.

| Gate | Inhalt | Entscheider |
|---|---|---|
| G0 | Entscheidung zur Ausgangsbasis (Optionen A/B/C, `transformation-decision-matrix.md`) | Projektleitung |
| G1 | Governance und Recht: verantwortliche Stelle, Lizenz-/Herkunftsklärung, Datenschutz-/Rechtsfreigabe | Projektleitung mit Rechtsabteilung und Datenschutz |
| G2 | Anforderungen freigegeben, ED-1 bis ED-7 beantwortet | Projektleitung |
| G3 | Architektur- und Bedrohungsmodell | Technische Leitung, Informationssicherheit |
| G4 | Labor-Prototyp, Negativtests grün | Technische Leitung |
| G5 | Selbsttest bestanden, erster Prüfbericht freigegeben | Projektleitung, Datenschutz |
| G6 | Dokumentation vollständig, Release-Freigabe | Projektleitung |

### 7.2 Stop-Kriterien
Das Projekt wird angehalten und der Projektleitung vorgelegt, wenn
- eine Fähigkeit aus dem Ausschlusskatalog (AK-01 bis AK-09) entdeckt, eingebracht oder angefordert wird,
- eine Lizenz- oder Herkunftsklärung negativ ausfällt, die für ein bereits gewähltes Vorgehen Voraussetzung war,
- ein Test außerhalb der Scope-Allowlist ausgeführt wurde oder werden soll,
- reale personenbezogene Daten oder Zugangsdaten in Testdaten, Protokollen oder Berichten auftauchen.

### 7.3 Änderungsmanagement und Dokumentenpflege
- Alle Dokumente tragen Stand, Revision und Änderungsprotokoll; Befund-IDs und das Kennzeichnungsschema (BESTÄTIGT / ABGELEITET / OFFEN / NICHT GEPRÜFT) bleiben einheitlich.
- Änderungen an Anforderungen und am Ausschlusskatalog erfordern eine Freigabe der Projektleitung.
- Nach jeder Revision erfolgt ein Querverweis- und Konsistenzabgleich der Dokumente (Erfahrung aus Phase 2: Dokumentationsdrift trat bereits auf, M-15).

## 8. Annahmen und Randbedingungen

| Nr. | Annahme / Randbedingung | Status |
|---|---|---|
| A-1 | Das Projekt arbeitet mit synthetischen Daten in isolierten Laborumgebungen | Anforderung (LAB-01, LAB-03) |
| A-2 | Zielbrowser und -versionen werden benannt | OFFEN (ED-2); heute NICHT GEPRÜFT |
| A-3 | Die Architekturrichtung (Erweiterung, externer Prüfstand, Kombination) wird erst in Phase 5 entschieden | OFFEN (ED-3) |
| A-4 | Personal-, Budget- und Terminrahmen werden von der Projektleitung vorgegeben | OFFEN (diese Dokumente enthalten keine Aufwands- oder Terminschätzung) |

## 9. Hauptrisiken (Auszug aus `migration-risk-assessment.md`)

| ID | Risiko | Status |
|---|---|---|
| M-01 | Lizenz- und Herkunftsrisiko der Altbasis | Fakten BESTÄTIGT, Rechtslage NICHT GEPRÜFT |
| M-02 | Funktionsdrift in Richtung ausgeschlossener Funktionen | ABGELEITET |
| M-03 | Reputations- und Compliance-Risiko bei Anknüpfung an die Altbasis | ABGELEITET |
| M-08 | Fehlende Governance (Prüfaufträge, Scope, Freigaben) | BESTÄTIGT (nicht definiert) |
| M-09 | Unklare Zielumgebung | NICHT GEPRÜFT |
| M-10 | Scope-Verletzung im Betrieb | ABGELEITET |

## 10. Berichtswesen

Statusbericht an die Projektleitung am Ende jeder Phase und an jedem Gate: erreichter Stand, Abweichungen, offene Punkte (mit Kennzeichnung), Entscheidungsbedarf. **Eine Phase wird erst dann als abgeschlossen bezeichnet, wenn alle für sie geforderten Ergebnisse vorliegen und die Abnahmekriterien erfüllt sind** (`project-roadmap.md`, `phase-4-status.md`).

## 11. Freigabe

| Rolle | Name | Datum | Unterschrift |
|---|---|---|---|
| Projektleitung | *(offen)* | *(offen)* | *(offen)* |
| Datenschutz | *(offen)* | *(offen)* | *(offen)* |
| Rechtsabteilung | *(offen)* | *(offen)* | *(offen)* |

Diese Charta ist **nicht freigegeben**. Sie ist ein Entwurf zur Entscheidung der Projektleitung.
