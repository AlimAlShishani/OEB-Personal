# Projekt-Roadmap: Browser-Security-Testing-Framework (Vorschlag)

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** Entwurf zur Entscheidung · **Klassifizierung:** intern
**Zugehörig:** `project-charter.md` · `oeb-transformation-concept.md` · `target-architecture.md` · `transformation-decision-matrix.md` · `phase-4-status.md` · Phase-3-Dokumente (insbesondere `security-testing-requirements.md`, `migration-risk-assessment.md`)

**Kennzeichnung:** BESTÄTIGT · ABGELEITET · OFFEN · NICHT GEPRÜFT (wie in den Vorphasen).
**Prüfstatus:** Es wurden bisher keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt.
**Zeitangaben:** Dieses Dokument enthält **keine Dauer- und Aufwandsschätzung** (OFFEN: erst nach Architekturentwurf und Vorgabe von Personal- und Terminrahmen durch die Projektleitung möglich). Es legt Reihenfolge, Ergebnisse, Abhängigkeiten und Abnahmekriterien fest.

**Geltung:** Der Plan gilt für die Optionen B und C (neues Repository ohne Code der Altbasis). Für Option A gelten zusätzliche Schritte (Abschnitt 4). Welche Option gewählt wird, entscheidet die Projektleitung an Gate G0.

---

## 1. Überblick

| Phase | Titel | Gate am Ende | Status heute |
|---|---|---|---|
| **4** | Konzept und Umbauentscheidung | G0 (Option), G1 (Governance/Recht) angestoßen | **Dokumentarisch vollständig (Entwürfe), Abnahme ausstehend** (`phase-4-status.md`) |
| **5** | Architektur und technische Spezifikation | G2, G3 | Nicht begonnen |
| **6** | Kontrollierter Aufbau der freigegebenen Komponenten | G4 | Nicht begonnen |
| **7** | Defensive Tests und Qualitätssicherung | G5 | Nicht begonnen |
| **8** | Dokumentation und Release-Freigabe | G6 | Nicht begonnen |

**Grundregeln für alle Phasen**
- Der Ausschlusskatalog AK-01 bis AK-09 und die Stop-Kriterien der Charta (7.2) gelten ohne Ausnahme.
- Eine Phase gilt erst dann als abgeschlossen, wenn **alle** geforderten Ergebnisse vorliegen **und** alle Abnahmekriterien nachweislich erfüllt sind. Ergebnisse, die fehlen, werden im Statusbericht als OFFEN geführt.
- Jede Phase endet mit einem Statusbericht an die Projektleitung (Charta 10) und einem Konsistenzabgleich der Dokumente.
- Tests laufen ausschließlich in autorisierten, isolierten Umgebungen gegen Laborziele.

---

## 2. Phasenplan

### Phase 4 – Konzept und Umbauentscheidung

**Ziel:** Belastbare Grundlage für die Entscheidung über die Ausgangsbasis (Optionen A/B/C) und Klärung der Voraussetzungen.

| Ergebnis | Inhalt | Status |
|---|---|---|
| E4-1 `project-charter.md` | Vision, Ziele, Abgrenzung, Governance, Gates | Entwurf liegt vor, nicht freigegeben |
| E4-2 `transformation-decision-matrix.md` | Lizenz-/Herkunftsstand, Bewertung A/B/C, Empfehlung | Entwurf liegt vor, nicht freigegeben |
| E4-3 `project-roadmap.md` | dieses Dokument | Entwurf liegt vor, nicht freigegeben |
| E4-4 `phase-4-status.md` | Bearbeitungsstand, offene Ergebnisse | liegt vor |
| E4-5 `oeb-transformation-concept.md` | Transformationskonzept: Erkenntnisse und Anforderungen, neue Projektstruktur, entfallende Bereiche, Lizenz-/Compliance-Ablauf, Vor- und Nachteile | Entwurf liegt vor, nicht freigegeben |
| E4-6 `target-architecture.md` | Konzeptionelle Zielarchitektur der acht Verantwortungsbereiche | Entwurf liegt vor, nicht freigegeben |

**Abnahmekriterien Phase 4**
1. Alle Ergebnisse E4-1 bis E4-6 liegen vor und sind untereinander konsistent. **Erfüllt als Entwurf** (Stand 2026-11-12, siehe `phase-4-status.md` 4); die Freigabe durch die Projektleitung steht aus (Kriterium 5).
2. Projektleitung hat die Option gewählt und dokumentiert (G0).
3. Verantwortliche Stelle für Prüfgenehmigungen ist benannt (ED-1).
4. Lizenz-/Herkunftsstand (Matrix 3) ist von der Rechtsabteilung bewertet; Sperrliste bestätigt oder angepasst.
5. Charta und Roadmap sind freigegeben oder mit Änderungsauftrag zurückgegeben.

**Abhängigkeiten:** Rechtsabteilung, Datenschutz, Projektleitung.
**Phase gilt heute als:** dokumentarisch vollständig (alle sechs Ergebnisse liegen als Entwurf vor), **Abnahme durch die Projektleitung ausstehend** (Kriterien 2 bis 5 offen).

### Phase 5 – Architektur und technische Spezifikation

**Ziel:** Prüffähiger Entwurf, aus dem sich Phase 6 umsetzen lässt.

| Ergebnis | Inhalt |
|---|---|
| E5-1 Zielarchitektur (Ausgangspunkt: Entwurf `target-architecture.md`, E4-6) | Verantwortungsbereiche (Browser Environment, Security Checks, Permission Analysis, Communication Security, Privacy Analysis, Test Runner, Evidence Manager, Reporting) mit klarer Trennung von Teststeuerung, Testausführung, Ergebnisverwaltung und Benutzeroberfläche; Schnittstellen auf hoher Ebene |
| E5-2 Bedrohungsmodell | Vertrauensgrenzen, Missbrauchsfälle (insbesondere Scope-Umgehung, Funktionsdrift), Gegenmaßnahmen |
| E5-3 Technische Spezifikation | Datenmodelle (Prüfauftrag, Scope, Testfall, Lauf, Beobachtung, Befund, Nachweis, Ereignis), Nachrichten- und Schnittstellenverträge mit Schema-Validierung, Ereignisschema, Berichtsvorlagen |
| E5-4 Plattform- und Technologieentscheidung | Architekturrichtung (ED-3), Umgang mit auslaufenden Plattformen (SR-NF-01), Zielbrowser/-versionen (ED-2, Browser-Matrix) |
| E5-5 Teststrategie | Testfallformat, Reproduzierbarkeit, Negativtests zum Ausschlusskatalog (NT-01 bis NT-06), Selbsttest |
| E5-6 Lieferketten- und Lizenzkonzept | SBOM, Abhängigkeitsprüfung, Lizenzdatei, Drittassets (keine Fremdmarken) |
| E5-7 Datenschutzkonzept | Aufbewahrung, Löschung, Protokollinhalte (ED-4), Datenschutzfreigabe-Plan |
| E5-8 Aufwands- und Terminschätzung | Erst auf Basis von E5-1 bis E5-5 (bislang OFFEN) |
| E5-9 Produktname | Namens- und Markenprüfung (Charta 5) |

**Abnahmekriterien Phase 5 (Gates G2, G3)**
1. `security-testing-requirements.md` ist freigegeben; ED-1 bis ED-7 sind beantwortet (G2).
2. Jede MUSS-Anforderung ist mindestens einer Komponente und einem geplanten Testfall zugeordnet (Rückverfolgbarkeitsmatrix ohne Lücke).
3. Jede Schnittstelle hat einen Schema-/Vertragsentwurf mit Absender-Autorisierung und Validierungsregeln; kein Schnittstellenentwurf sieht sensible Daten mit offenem Zielursprung vor.
4. Das Bedrohungsmodell ist von der Informationssicherheit reviewt; offene hohe Risiken sind keine vorhanden oder mit Gegenmaßnahme eingeplant (G3).
5. Keine Komponente und kein Schnittstellenentwurf bildet eine Fähigkeit des Ausschlusskatalogs ab (schriftliche Bestätigung nach Review).
6. Plattform- und Browser-Matrix sind beschlossen.

**Abhängigkeiten:** G0, G1; Rückmeldungen von Datenschutz und Rechtsabteilung.

### Phase 6 – Kontrollierter Aufbau der freigegebenen Komponenten

**Ziel:** Schrittweiser Aufbau nur der in Phase 5 freigegebenen Komponenten, jeweils mit Review und Tests.

| Ergebnis | Inhalt |
|---|---|
| E6-1 Projektgrundlage | Neues Repository, Build- und CI-Gerüst, Lint, Test-Rahmen, SBOM-Erzeugung, Lizenzdatei, Review-Regeln (Vier-Augen) |
| E6-2 Governance-Kern | Prüfauftrag, Scope-Allowlist mit Negativliste, Autorisierungsprüfung, Not-Aus (GOV-01 bis GOV-08) |
| E6-3 Basis-Komponenten | Ereignisprotokoll (LOG), Evidence Manager, Nachrichtenvalidierung |
| E6-4 Prüf-Komponenten | Browser Environment, Test Runner, Security Checks, Permission Analysis, Communication Security, Privacy Analysis (in Inkrementen) |
| E6-5 Reporting | Berichtsvorlagen, Befund- und Evidenzklassen, Exportformate |
| E6-6 Negativtests | NT-01 bis NT-06 als fester Bestandteil jedes Builds |

**Abnahmekriterien Phase 6 (Gate G4)**
1. Jedes Inkrement wurde reviewt (Vier-Augen) und hat grüne automatisierte Tests; Herkunft jeder Datei ist nachvollziehbar (kein Fremdcode ohne dokumentierte Lizenz).
2. Negativtests NT-01 bis NT-06 sind grün und laufen bei jedem Build.
3. Der Prototyp läuft ausschließlich in der Laborumgebung; ein Lauf ohne gültigen Prüfauftrag oder außerhalb der Scope-Allowlist wird nachweislich verweigert und protokolliert.
4. Keine Geheimnisse, Seiteninhalte oder personenbezogenen Daten in Protokollen oder Speicher (stichprobenartig geprüft mit Canary-Daten).
5. Alle MUSS-Anforderungen der Phase-6-Inkremente sind implementiert; Abweichungen sind dokumentiert und freigegeben.

**Stoppregel:** Wird in einem Inkrement eine ausgeschlossene Fähigkeit festgestellt, ist die Entwicklung zu unterbrechen und die Projektleitung zu informieren (Charta 7.2).

### Phase 7 – Defensive Tests und Qualitätssicherung

**Ziel:** Nachweis, dass das Framework die Anforderungen erfüllt und selbst den Sicherheits- und Datenschutzvorgaben genügt.

| Ergebnis | Inhalt |
|---|---|
| E7-1 Selbsttest | Anwendung der eigenen Verfahren auf das Framework (FR-ST-01) |
| E7-2 Reproduzierbarkeitsnachweis | Wiederholte Läufe auf frischem Profil mit gleichen Ergebnissen (FR-TC-02) |
| E7-3 Datenschutzprüfung | Speicherinventur, Canary-Verfahren, Löschnachweis, Egress-Inventar |
| E7-4 Browser-Matrix-Läufe | Ausführung auf den beschlossenen Browsern und Versionen (erstmalige Laufzeit-Browsertests des Projekts) |
| E7-5 Robustheit | Grenzwert-/Fuzzing-Tests der eigenen Parser, Limit-Nachweise auf allen Eingabepfaden |
| E7-6 Erster Prüfbericht | Vollständiger Bericht nach REP-01 bis REP-06 für ein Labor-Prüfobjekt |
| E7-7 Sicherheitsreview | Unabhängige Durchsicht durch die Informationssicherheit |

**Abnahmekriterien Phase 7 (Gate G5)**
1. Selbsttest ohne offene Befunde der Schwere „hoch"; mittlere Befunde sind bewertet und eingeplant.
2. Zwei unabhängige Läufe derselben Testauswahl in frischer Umgebung liefern gleiche Ergebnisse (bis auf definierte, dokumentierte Zeit-/Kennungswerte).
3. Datenschutz bestätigt die Löschfunktionen und die Protokollinhalte schriftlich (P-4, P-5, P-7).
4. Der Prüfbericht wurde von Datenschutz geprüft und freigegeben; Befunde tragen Evidenzklassen und Reproduktionsangaben.
5. Alle in Phase 4/5 als NICHT GEPRÜFT oder OFFEN geführten Annahmen, die für die Freigabe relevant sind (z. B. Verfügbarkeit des Sitzungsspeichers, Browserverhalten), sind entweder durch Tests geschlossen oder als Einschränkung dokumentiert.

### Phase 8 – Dokumentation und Release-Freigabe

**Ziel:** Übergabe in den kontrollierten Betrieb.

| Ergebnis | Inhalt |
|---|---|
| E8-1 Dokumentation | Benutzer-, Betriebs-, Sicherheits- und Entwicklerdokumentation; Zustands- und Nachrichtenmodelle; Änderungsprotokoll |
| E8-2 Governance-Handbuch | Prüfauftragsprozess, Scope-Pflege, Rollen, Not-Aus, Umgang mit Ausnahmen |
| E8-3 Release-Artefakte | Signierte Pakete, SBOM, Lizenzdatei, Release-Notes |
| E8-4 Betriebs- und Löschkonzept | Aufbewahrungsfristen, Archivierung, Entsorgung von Testdaten |
| E8-5 Schulung | Einweisung der Prüfer und Freigeber |
| E8-6 Übergabe der Altmaterialien | Archivierung des Altbestands gemäß Entscheidung (Isolierung, Zugriff) |
| E8-7 Abschlussbericht | Stand der Anforderungen, Restrisiken, Einschränkungen |

**Abnahmekriterien Phase 8 (Gate G6)**
1. Jede Anforderung ist erfüllt, begründet abgewichen oder als Einschränkung dokumentiert.
2. Dokumentation ist vollständig, konsistent und von der Projektleitung freigegeben.
3. Lizenz-, Marken- und Herkunftsnachweise für alle Bestandteile liegen vor (keine ungeklärten Drittassets).
4. Release-Artefakte sind signiert und reproduzierbar erzeugbar; SBOM und Schwachstellenprüfung der Abhängigkeiten liegen vor.
5. Restrisiken und Einschränkungen sind der Projektleitung schriftlich vorgelegt und akzeptiert.

---

## 3. Gates (Übersicht)

| Gate | Phase | Kriterium (Kurz) | Entscheider |
|---|---|---|---|
| G0 | 4 | Optionsentscheidung dokumentiert | Projektleitung |
| G1 | 4/5 | Verantwortliche Stelle benannt; Lizenz-/Herkunftsbewertung; Datenschutz-/Rechtsfreigabe-Plan; Beteiligungspflichten geklärt | Projektleitung, Rechtsabteilung, Datenschutz |
| G2 | 5 | Anforderungen freigegeben; ED-1 bis ED-7 beantwortet | Projektleitung |
| G3 | 5 | Architektur- und Bedrohungsmodell reviewt; Plattform-/Browser-Matrix beschlossen | Technische Leitung, Informationssicherheit |
| G4 | 6 | Labor-Prototyp; Negativtests grün; Governance-Kern wirksam | Technische Leitung |
| G5 | 7 | Selbsttest, Reproduzierbarkeit, Datenschutzprüfung, erster Prüfbericht | Projektleitung, Datenschutz |
| G6 | 8 | Dokumentation, Nachweise, Release-Freigabe | Projektleitung |

---

## 4. Zusätzliche Schritte bei Option A (Bestehende Struktur als Grundlage)

Nur falls die Projektleitung Option A wählt (nicht empfohlen, siehe Matrix 5):

| Zusatzschritt | Phase | Voraussetzung / Abnahmekriterium |
|---|---|---|
| Z-A1 Positive, schriftliche Lizenz-/Herkunftsklärung und Compliance-Freigabe (H-1 bis H-4, H-6) | 4 | Rechtsabteilung bestätigt; ohne Bestätigung kein Fortgang |
| Z-A2 Quarantäne-Ablage des Altbestands in gesondertem, zugriffsbeschränktem Repository | 5 | Zugriff protokolliert; kein Zugriff aus der Entwicklungsumgebung des neuen Projekts |
| Z-A3 Rückbauspezifikation: Liste aller Dateien und Bezeichner, die verbleiben dürfen, mit Begründung | 5 | Review durch Informationssicherheit; jede verbleibende Datei hat eine Lizenz- und Herkunftsklärung |
| Z-A4 Rückbau und Nachweis im Aufbau | 6 | Negativtests NT-01 bis NT-06 für den gesamten Bestand grün; Historie enthält keine Ausschlussbereiche im Zielrepository |
| Z-A5 Bereinigung von Bezeichnern und Kennungen der Altbasis | 6 | Keine Altbezeichner im Paket (Suchlauf) |
| Z-A6 Erweiterte Nachweise zur Ausschlusskonformität | 7 | Unabhängige Prüfung bestätigt Abwesenheit der Ausschlussbereiche |

---

## 5. Risiken je Phase

| Phase | Hauptrisiken | Bezug |
|---|---|---|
| 4 | Lizenz-/Herkunftsklärung verzögert sich; Entscheidung ohne ausreichende Grundlage; unvollständige Phase-4-Ergebnisse werden als abgeschlossen gewertet | M-01, M-08; `phase-4-status.md` |
| 5 | Überladene Architektur; Plattformentscheidung ohne Browser-Matrix; Aufwandsschätzung zu früh | M-05, M-09, M-14 |
| 6 | Funktionsdrift in Richtung ausgeschlossener Fähigkeiten; Fremdcode ohne Lizenznachweis; Scope-Verletzungen im Betrieb | M-02, M-10 |
| 7 | Falsche Sicherheit durch lückenhafte Tests; Datenschutz-Befunde spät erkannt | M-11, M-06 |
| 8 | Unvollständige Nachweise, Dokumentationsdrift | M-15 |

---

## 6. Offene Punkte der Roadmap

| Nr. | Punkt | Status |
|---|---|---|
| RP-1 | Dauer, Aufwand, Personal und Budget | OFFEN (keine Schätzgrundlage) |
| RP-2 | Namen der Verantwortlichen je Rolle | OFFEN (ED-1) |
| RP-3 | Zielbrowser und -versionen | OFFEN (ED-2) |
| RP-4 | Architekturrichtung | OFFEN (ED-3, Phase 5) |
| RP-5 | Aufbewahrungsfristen | OFFEN (ED-4) |
| RP-6 | Umgang mit dem Arbeitsstrang „Prüfungsintegrität" | OFFEN (ED-5) |
| RP-7 | Beteiligung von Datenschutz, Rechtsabteilung, Personalvertretung | OFFEN (ED-6) |
| RP-8 | Zielwert der Testabdeckung | OFFEN (ED-7) |
| RP-9 | Freigabe der Entwürfe E4-1 bis E4-6 | OFFEN (Projektleitung) |
