# Phase 4 – Bearbeitungsstand

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Klassifizierung:** intern
**Zugehörig:** `project-charter.md` · `transformation-decision-matrix.md` · `project-roadmap.md` · Phase-3-Dokumente

**Kennzeichnung:** BESTÄTIGT · ABGELEITET · OFFEN · NICHT GEPRÜFT (wie in den Vorphasen).

---

## 1. Gesamtstatus

> **Phase 4 ist NICHT abgeschlossen.** Vier der in der Fortsetzung verlangten Dokumente liegen als **Entwurf** vor. Zwei Ergebnisse aus dem ursprünglichen Phase-4-Auftrag (Umbaukonzept, Zielarchitektur) fehlen. Keines der Dokumente ist freigegeben, und das Gate G0 (Optionsentscheidung) ist nicht erreicht.

## 2. Verlauf

| Schritt | Ergebnis |
|---|---|
| Ursprünglicher Phase-4-Auftrag (fünf Dokumente) | Bearbeitung wurde während der Erstellung des Umbaukonzepts abgebrochen. Das begonnene, unvollständige Dokument `oeb-transformation-concept.md` wurde wieder **gelöscht**; es existiert keine Teilfassung. `target-architecture.md` wurde nicht begonnen. |
| Fortsetzung („neutrale Umbau- und Entscheidungsvorlage") | Der Umfang wurde auf sechs allgemeine Bereiche beschränkt (Struktur, Build, Dokumentation, allgemeine UI-Konzepte, unkritische Konfigurationsmechanismen, Test-/Reporting-Konzepte). Vier Dokumente wurden erstellt. |
| Ausgeschlossene Funktionsbereiche | **Nicht untersucht, nicht rekonstruiert, nicht bewertet** über die bereits in Phase 1–3 dokumentierten Befunde hinaus. |

## 3. Ergebnisübersicht

### 3.1 Dokumente der Fortsetzung

| Dokument | Zustand | Anmerkung |
|---|---|---|
| `project-charter.md` | Entwurf, **nicht freigegeben** | Freigabeblock leer; Rollen ohne Namen (OFFEN) |
| `transformation-decision-matrix.md` | Entwurf, **nicht freigegeben** | Enthält Lizenz-/Herkunftsstand; mehrere Punkte OFFEN (Abschnitt 5) |
| `project-roadmap.md` | Entwurf, **nicht freigegeben** | Ohne Dauer-/Aufwandsschätzung (OFFEN) |
| `phase-4-status.md` | liegt vor | dieses Dokument |

### 3.2 Ergebnisse aus dem ursprünglichen Auftrag, die fehlen

| Ergebnis | Zustand | Begründung |
|---|---|---|
| `oeb-transformation-concept.md` (Detailkonzept zur Umbauoption mit Komponentenbewertung in vier Kategorien) | **OFFEN – nicht vorhanden** | Abbruch; in der Fortsetzung nicht gefordert. Die tiefe Komponentenbewertung würde eine Untersuchung der ausgeschlossenen Funktionsbereiche erfordern und entfällt daher; die Bewertung der sechs zulässigen Bereiche steht in der Matrix (Abschnitt 2). |
| `target-architecture.md` (Zielarchitektur mit acht Verantwortungsbereichen und Trennung Steuerung/Ausführung/Ergebnisverwaltung/Oberfläche) | **OFFEN – nicht vorhanden** | In der Fortsetzung nicht gefordert. Die Roadmap führt die Architektur als Ergebnis E5-1 (Phase 5) und als offenes Phase-4-Ergebnis E4-6 (Entscheidung der Projektleitung RP-9). |
| Entscheidungsmatrix mit den Kriterien des ersten Auftrags (Umbau gegenüber Neuentwicklung) | **Teilweise** | Die Fortsetzung verlangte andere Optionen (A/B/C) und Kriterien; die ursprünglichen Kriterien Wartbarkeit, Lizenzrisiko, Entwicklungsdauer und Erweiterbarkeit sind enthalten (Matrix 4.2, 4.3), die Dauer ist OFFEN. |
| Abschlussbericht (sechs Fragen des ersten Auftrags) | Im Chat beantwortet (Abschnitt 7); nicht als Datei | – |

## 4. Abnahmekriterien Phase 4 (`project-roadmap.md` 2) – Zielerfüllung

| Nr. | Kriterium | Stand |
|---|---|---|
| 1 | Alle Ergebnisse E4-1 bis E4-6 liegen vor oder sind per Entscheidung der Projektleitung verschoben/entfallen | **Nicht erfüllt** (E4-5, E4-6 fehlen, keine Entscheidung) |
| 2 | Optionsentscheidung dokumentiert (G0) | **Nicht erfüllt** |
| 3 | Verantwortliche Stelle für Prüfgenehmigungen benannt (ED-1) | **Nicht erfüllt** |
| 4 | Lizenz-/Herkunftsstand durch die Rechtsabteilung bewertet | **Nicht erfüllt** |
| 5 | Charta und Roadmap freigegeben | **Nicht erfüllt** (Entwürfe) |

## 5. Offene Punkte, die die Entscheidung betreffen

| Nr. | Punkt | Grund | Status |
|---|---|---|---|
| OP-1 | Lizenzen der beiden Vorlageprojekte (UmmItKin/SebBypass, cycyrild/SebBypass) | Originalrepositories nicht eingesehen; zwei Websuchen ohne belastbares Ergebnis | OFFEN |
| OP-2 | Rechteinhaber und Weitergabebedingungen von OEB | Im Paket nicht angegeben (keine Lizenzdatei, keine Lizenz-/Autorenfelder) | OFFEN |
| OP-3 | Herkunft von Symbolen, Vorschau-Dateien, Oberflächen-/Logo-Assets; Bedeutung eines Fremd-Urheberrechtsvermerks im Anzeigetext einer Oberflächendatei (`seb/assets/startup.html:210,230`) | Nicht dokumentiert | OFFEN |
| OP-4 | Rechtliche Bestätigung zur Übernahme allgemeiner Muster | Aufgabe der Rechtsabteilung | OFFEN |
| OP-5 | Compliance-Bewertung der Weiterverwendung eines Projekts, dessen Zweck die Umgehung von Prüfungssoftware ist | Entscheidung von Projektleitung und Rechtsabteilung | OFFEN |
| OP-6 | Aufwand, Dauer, Personal, Budget | Keine Architektur und keine Vorgabe | OFFEN |
| OP-7 | Zielbrowser/-versionen, Plattformentscheidung | Nicht benannt | OFFEN |
| OP-8 | Laufzeit-, Browserkompatibilitäts- und Netzwerktests | **Nicht durchgeführt** | NICHT GEPRÜFT |
| OP-9 | Lizenzprüfung der Build-Abhängigkeiten | Nur Lockfile-Angaben vorhanden | NICHT GEPRÜFT |
| OP-10 | `popup.css` (914 Z.), Bilddateien | Nicht geprüft | NICHT GEPRÜFT |

## 6. Konsistenz mit den Phasen 1 bis 3

- **Unverändert gültig:** Ausschlusskatalog AK-01 bis AK-09, Datenschutz-Voraussetzungen P-1 bis P-7, Sicherheitsbefunde SEC-01 bis SEC-15 und NF-01 bis NF-06, Anforderungen (`security-testing-requirements.md`), Gates G0 bis G5 (ergänzt um G6).
- **Optionsbezeichnungen:** Die Optionen A/B/C dieser Phase unterscheiden sich von denen in Phase 3. Die Zuordnung steht in `transformation-decision-matrix.md` 4. Die Empfehlung ist **inhaltlich dieselbe** wie in Phase 3 (neues Repository ohne Code der Altbasis, Wissensübernahme in Form von Anforderungen); sie wurde im Licht der jetzigen Beschränkung auf sechs Bereiche erneut geprüft und nicht verändert.
- **Zahlen:** Die Angaben zum Bestand (6 735 Zeilen Quelltext in 37 Dateien; Ausschlussbereich 5 346 Zeilen in 30 Dateien; Phase 3: 5 427 Zeilen einschließlich 81 Zeilen technisch zu ersetzender Hilfsdateien) sind miteinander vereinbar.
- **Dateien:** Es wurden ausschließlich Markdown-Dokumente erstellt. Es wurden keine Quell-, Konfigurations- oder Build-Dateien verändert; das Repository enthält weiterhin keinen Quellcode der Altbasis.

## 7. Bericht an die Projektleitung

**Tatsächlicher Stand:** Phase 4 ist teilweise bearbeitet, nicht abgeschlossen. Entscheidungsgrundlage (Matrix), Charta und Roadmap liegen als Entwurf vor. Umbaukonzept und Zielarchitektur fehlen. Es gibt weder eine Optionsentscheidung noch eine Freigabe.

**Zu den Fragen des ersten Auftrags**
1. **Ist ein kontrollierter Umbau grundsätzlich denkbar?** Theoretisch ja (Option A), aber nur nach positiver Lizenz-/Herkunftsklärung und Compliance-Freigabe und mit hohem Zusatzaufwand; nicht empfohlen.
2. **Welche Teile könnten sinnvoll sein?** Nur Muster und Strukturideen: Build-Organisation („eine Quelle, mehrere Zielpakete"), Manifest-Gliederung, Einstellungsnormalisierung, sichere Textdarstellung, die eigene Dokumentationsstruktur. Zusammen rund 4 % des Bestands (296 Zeilen); als Code derzeit gesperrt, solange die Lizenz offen ist.
3. **Was ist zu ersetzen oder auszuschließen?** Funktionsgebundene Struktur, Bezeichner und Oberfläche sind zu ersetzen; der Ausschlussbereich (30 Dateien) ist dauerhaft auszuschließen.
4. **Was blockiert die Entscheidung?** Lizenzen der Vorlageprojekte (OP-1), Rechteinhaber (OP-2), Herkunft der Assets (OP-3), Compliance-Bewertung (OP-5), fehlende Governance (ED-1).
5. **Voraussetzungen vor der Architekturplanung:** Gates G0 bis G2 (Optionsentscheidung, benannte verantwortliche Stelle, Lizenz-/Datenschutz-/Rechtsbewertung, freigegebene Anforderungen, Zielbrowser).
6. **Umbau gegenüber Neuentwicklung:** Das Funktionale (Prüfobjekte, Checks, Runner, Evidenz, Berichte) ist in beiden Fällen neu zu entwickeln. Der Umbau spart höchstens Gerüstarbeit und erfordert zusätzlich Import, Rückbau, Nachweise und Lizenzklärung; sein Risiko ist deutlich höher. Absolute Aufwandszahlen sind OFFEN.

**Empfehlung:** Die Projektleitung entscheidet an Gate G0. Der Analyst empfiehlt **Option B** (unkritische Bestandteile nach unabhängiger Prüfung neu aufbauen), hilfsweise **Option C**. **Option A** sollte nur nach positiver, schriftlicher Lizenz-/Herkunftsklärung und Compliance-Freigabe erneut bewertet werden. Als nächste Schritte sind sinnvoll: Benennung der verantwortlichen Stelle (ED-1), Beauftragung der Rechtsabteilung für OP-1 bis OP-5, Entscheidung über das Nachholen von Zielarchitektur und Umbaukonzept (RP-9) sowie Festlegung der Zielbrowser (ED-2).
