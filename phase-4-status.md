# Phase 4 – Bearbeitungsstand

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 (Revision 2) · **Klassifizierung:** intern
**Zugehörig:** `project-charter.md` · `oeb-transformation-concept.md` · `target-architecture.md` · `transformation-decision-matrix.md` · `project-roadmap.md` · Phase-3-Dokumente

**Kennzeichnung:** BESTÄTIGT · ABGELEITET · OFFEN · NICHT GEPRÜFT (wie in den Vorphasen).

---

## 1. Gesamtstatus

> **Dokumentationsumfang von Phase 4: vollständig.** Alle fünf vorgesehenen Dokumente (`project-charter.md`, `oeb-transformation-concept.md`, `target-architecture.md`, `transformation-decision-matrix.md`, `project-roadmap.md`) sind vorhanden und wurden auf Konsistenz geprüft (Abschnitt 4). Mit dem vorliegenden Statusbericht gilt die von der Projektleitung gesetzte Bedingung für die Bezeichnung „abgeschlossen" als erfüllt.
>
> **Aber:** Alle Dokumente sind **Entwürfe und nicht freigegeben.** Die Abnahmekriterien der Roadmap für Phase 4 (Optionsentscheidung G0, benannte verantwortliche Stelle, Rechtsbewertung, Freigabe von Charta und Roadmap) sind **nicht erfüllt**. Phase 4 ist damit **dokumentarisch abgeschlossen; die Abnahme durch die Projektleitung steht aus.** Phase 5 darf erst nach den Entscheidungen in Abschnitt 6 beginnen.

## 2. Verlauf

| Schritt | Ergebnis |
|---|---|
| Ursprünglicher Phase-4-Auftrag (fünf Dokumente) | Wurde während der Erstellung des Umbaukonzepts abgebrochen; das unvollständige Dokument wurde gelöscht. |
| Fortsetzung 1 („neutrale Umbau- und Entscheidungsvorlage") | Umfang auf sechs allgemeine Bereiche beschränkt; erstellt wurden Charta, Entscheidungsmatrix, Roadmap und Statusbericht. |
| Fortsetzung 2 („fehlende Konzeptdokumente fertigstellen") | `oeb-transformation-concept.md` und `target-architecture.md` wurden neu erstellt; Roadmap, Charta, Matrix und Statusbericht wurden abgeglichen. |
| Ausgeschlossene Funktionsbereiche | Zu keinem Zeitpunkt untersucht, rekonstruiert oder bewertet über die in Phase 1–3 dokumentierten Befunde hinaus. Das Umbaukonzept behandelt sie nur als Gesamtmenge. |

## 3. Die fünf Dokumente

| Nr. | Dokument | Inhalt | Zustand |
|---|---|---|---|
| 1 | `project-charter.md` | Vision, Ziele, Abgrenzung, verbindliche Rahmenbedingungen, Rollen, Gates G0–G6, Stop-Kriterien, Freigabeblock | Entwurf, nicht freigegeben, Freigabeblock leer |
| 2 | `oeb-transformation-concept.md` | Trennung Dokumentation/Konzepte/Code (K-D, K-K, K-C); Erkenntnisse und Anforderungen (EK-1 bis EK-12); neue Projektstruktur; entfallende Bereiche; Konzepte zum Neuaufbau (KO-1 bis KO-10); Lizenz-/Compliance-Ablauf (LC-1 bis LC-9, Freigabeverfahren CF-1 bis CF-6); Vor- und Nachteile; Übergangsplan | Entwurf, nicht freigegeben |
| 3 | `target-architecture.md` | Prinzipien, Vertrauenszonen, Trennung der Verantwortungsbereiche, acht Module mit Aufgabe, Ein-/Ausgaben, Abhängigkeiten, Sicherheitsanforderungen und Abgrenzung, Querschnittsbausteine, Schnittstellen I-01 bis I-13, Datenobjekte, Laufmodell, Bedrohungsliste TH-01 bis TH-12, Entwurfsentscheidungen DD-1 bis DD-9, Unabhängigkeitsnachweis | Entwurf, nicht freigegeben |
| 4 | `transformation-decision-matrix.md` | Bewertungsumfang, Bestand, Lizenz-/Herkunftsstand mit Sperrliste, Optionen A/B/C, Empfehlung | Entwurf, nicht freigegeben |
| 5 | `project-roadmap.md` | Phasen 4–8 mit Ergebnissen und Abnahmekriterien, Gates, Zusatzschritte bei Option A, Risiken, offene Punkte | Entwurf, nicht freigegeben |
| – | `phase-4-status.md` | dieses Dokument | liegt vor |

## 4. Konsistenzprüfung (durchgeführt am 2026-11-12)

| Prüfung | Ergebnis |
|---|---|
| Alle Dokumente vorhanden; alle in Dokumenten genannten Dateiverweise lösen auf | Bestanden |
| Alle verwendeten Kennungen (ED, G, AK, NT, GOV, LAB, FR-, SR-NF, LOG, REP, SEC, NF, DS, TR, M, OP, H, V, P, Q, LC, CF, EK, KO, US, OF, DD, AP, TH, MOD, QB, PL, I-, E4–E8, Z-A) haben eine Definition | Bestanden (automatischer Querverweis-Abgleich; ein Format-Artefakt bei E4–E8, QB, PL ist keine fehlende Definition) |
| Optionsbezeichnungen A/B/C einheitlich (Phase-4-Bedeutung; Zuordnung zu Phase 3 in Matrix 4) | Bestanden |
| Zahlen einheitlich (6 735 Zeilen in 37 Dateien; Ausschlussbereich 30 Dateien/5 346 Zeilen; ersetzbare Gerüste 4 Dateien/304 Zeilen; Gerüste ca. 296 Zeilen/rund 4 %; Phase 3: 5 427 Zeilen einschließlich 81 Zeilen Hilfsdateien) | Bestanden |
| Nummerierung der offenen Punkte `OP-n` zwischen Matrix, Konzept und Statusbericht | Zu Beginn **abweichend** (Compliance als OP-5 bzw. OP-8); bereinigt, Matrix ist maßgeblich (Abschnitt 5) |
| Statusangaben in Roadmap und Statusbericht abgestimmt | Bestanden nach Aktualisierung (E4-5, E4-6 und Phase-4-Zeile in der Roadmap) |
| Ausschlusskatalog AK-01 bis AK-09 und Datenschutz-Voraussetzungen P-1 bis P-7 unverändert | Bestanden |
| Keine Quell-, Konfigurations- oder Build-Dateien verändert; keine Implementierung | Bestanden (Repository enthält ausschließlich Markdown) |
| Keine Lizenzfreigabe behauptet | Bestanden (K-C gesperrt, `oeb-transformation-concept.md` 2, 7) |

**Grenze der Prüfung:** Die Prüfung ist eine Dokumentenprüfung (Querverweise, Zahlen, Begriffe). Sie ersetzt weder die fachliche Freigabe noch Tests (NICHT GEPRÜFT).

## 5. Offene Punkte (maßgebliche Nummerierung gemäß Matrix)

| Nr. | Punkt | Grund | Status |
|---|---|---|---|
| OP-1 | Lizenzen der Vorlageprojekte (UmmItKin/SebBypass, cycyrild/SebBypass) | Originalrepositories nicht eingesehen; Websuchen ohne belastbares Ergebnis | OFFEN |
| OP-2 | Rechteinhaber und Weitergabebedingungen von OEB | Im Paket nicht angegeben | OFFEN |
| OP-3 | Herkunft von Symbolen, Vorschau-Dateien, Logo-/Oberflächen-Assets; Bedeutung eines Fremd-Urheberrechtsvermerks im Anzeigetext (`seb/assets/startup.html:210,230`) | Nicht dokumentiert | OFFEN |
| OP-4 | Rechtliche Bestätigung zur Neuumsetzung allgemeiner Muster (K-K) | Aufgabe der Rechtsabteilung | OFFEN |
| OP-5 | Lizenzprüfung der Build-Abhängigkeiten | Nur Lockfile-Angaben vorhanden | NICHT GEPRÜFT |
| OP-6 | Aufwand, Dauer, Personal, Budget | Keine Architekturentscheidung, keine Vorgabe | OFFEN |
| OP-7 | Bewertung von `popup.css` (914 Z.) | Nicht geprüft; für Option B/C ohne Bedeutung | NICHT GEPRÜFT |
| OP-8 | Compliance-Bewertung der Weiterverwendung eines Projekts, dessen Zweck die Umgehung von Prüfungssoftware ist (H-6) | Entscheidung von Projektleitung und Rechtsabteilung | OFFEN |
| OP-9 | Zielbrowser/-versionen, Plattformentscheidung | Nicht benannt | OFFEN |
| OP-10 | Laufzeit-, Browserkompatibilitäts- und Netzwerktests | **Nicht durchgeführt** | NICHT GEPRÜFT |

## 6. Offene Entscheidungen, erforderliche Freigaben, Voraussetzungen für Phase 5

### 6.1 Offene Entscheidungen der Projektleitung

| Nr. | Entscheidung | Bezug |
|---|---|---|
| D-1 | **Optionsentscheidung** (A/B/C) – Gate G0 | Matrix 4, 5 |
| D-2 | Benennung der verantwortlichen Stelle für Prüfgenehmigungen und Freigaben (ED-1) | Charta 6, 7 |
| D-3 | Zielbrowser und -versionen (ED-2) | Roadmap RP-3 |
| D-4 | Architekturrichtung (ED-3; Entscheidung in Phase 5, Optionen in `target-architecture.md` 11) | Architektur DD-1, DD-2 |
| D-5 | Aufbewahrungsfristen (ED-4) | Anforderungen LOG-05, SR-NF-12 |
| D-6 | Umgang mit dem Arbeitsstrang „Prüfungsintegrität" und mit `OEB-Analyse-IT.md` (ED-5) | Charta 3.2 |
| D-7 | Beteiligung von Datenschutz, Rechtsabteilung und ggf. Personalvertretung (ED-6) | Charta 6 |
| D-8 | Zielwert der Testabdeckung (ED-7) | SR-NF-10 |
| D-9 | Produktname und Lizenz des neuen Projekts | Charta 5; Konzept 4, 11 (OF-4) |
| D-10 | Compliance-Entscheidung (OP-8) und Umgang mit dem Altmaterial (Isolierung, Ablage, Aufbewahrung) | Matrix 3.3; Migrationsbewertung 5 |
| D-11 | Aufwands-, Zeit- und Ressourcenrahmen (OP-6) | Roadmap RP-1 |

### 6.2 Erforderliche Freigaben

| Freigabe | Gegenstand | Wer |
|---|---|---|
| F-1 | Charta, Roadmap, Entscheidungsmatrix, Transformationskonzept, Zielarchitektur (Entwürfe) | Projektleitung |
| F-2 | Anforderungen (`security-testing-requirements.md`) – Gate G2 | Projektleitung |
| F-3 | Lizenz-, Marken- und Herkunftsbewertung (OP-1 bis OP-4, OP-8) – Teil von G1 | Rechtsabteilung |
| F-4 | Datenschutzfreigabe-Plan, Aufbewahrungskonzept (P-7) | Datenschutz |
| F-5 | Zugriffsschutz und Ablage des Altmaterials | Informationssicherheit |
| F-6 | Beteiligung der Personalvertretung, soweit erforderlich (NICHT GEPRÜFT, ob erforderlich) | Projektleitung, Rechtsabteilung |
| F-7 | Für eine etwaige Codeübernahme (nicht eingeplant): schriftliche Lizenz- **und** Sicherheitsfreigabe je Datei (CF-1 bis CF-6) | Rechtsabteilung, Informationssicherheit, Projektleitung |

### 6.3 Voraussetzungen für Phase 5 (gemäß Roadmap)

1. Phase 4 ist durch die Projektleitung abgenommen: Entwürfe geprüft, Charta und Roadmap freigegeben (F-1).
2. Optionsentscheidung dokumentiert – **G0** (D-1).
3. **G1** angestoßen bzw. erfüllt: verantwortliche Stelle benannt (D-2), Rechtsabteilung beauftragt (LC-1), Datenschutz-/Rechtsfreigabe-Plan, Beteiligungspflichten geklärt (D-7).
4. Zielbrowser benannt (D-3), damit die Browser-Matrix (LAB-04) geplant werden kann.
5. Ressourcen- und Zeitrahmen vorgegeben (D-11), da keine Schätzung vorliegt.
6. Altmaterial isoliert (F-5).

Hinweis: Das Gate G2 (Anforderungen freigegeben) und G3 (Architektur) sind Abnahmekriterien **der Phase 5 selbst**, nicht Voraussetzungen ihres Beginns.

## 7. Bericht an die Projektleitung

**Stand:** Alle fünf Dokumente liegen vor und sind untereinander abgestimmt; es sind Entwürfe. Es wurde nichts implementiert, nichts getestet (weder Laufzeit noch Browser noch Netzwerk), und es wurden ausschließlich Markdown-Dokumente erstellt bzw. angepasst.

**Wesentliche Aussagen**
1. Die Altbasis bietet nach heutiger Kenntnis keinen belastbaren Code-Vorteil: Rund 4 % des Bestands (Gerüste) sind überhaupt als Vorlage geeignet; der Rest gehört zum Ausschlussbereich (92 % des geprüften Codes ohne das ungeprüfte Stylesheet) oder ist technisch zu ersetzen.
2. Der Mehrwert liegt in **Erkenntnissen und Anforderungen** (EK-1 bis EK-12) und in **allgemeinen Konzepten**, die neu umgesetzt werden (KO-1 bis KO-10). Die Wiederverwendung von Quellcode (K-C) bleibt **gesperrt**, bis eine schriftliche Lizenz- **und** Sicherheitsfreigabe je Datei vorliegt. Eine solche Freigabe liegt nicht vor; ihr Ergebnis wird nicht vorausgesetzt.
3. Die Zielarchitektur ist unabhängig von SEB-Umgehung und Prüfungsautomatisierung (`target-architecture.md` 12) und trennt Teststeuerung, Testausführung, Ergebnisverwaltung, Berichtswesen und Benutzeroberfläche.

**Empfehlung des Analysten (Entscheidung bei der Projektleitung):**
- **Option B** (unkritische Bestandteile nach unabhängiger Prüfung neu aufbauen), hilfsweise **Option C** (vollständige Neuentwicklung). Beide führen zu einem neuen Repository ohne Altcode.
- **Option A** (bestehende Struktur als Grundlage) nicht verfolgen; sie wäre erst nach positiver, schriftlicher Lizenz-/Herkunftsklärung und Compliance-Freigabe erneut zu bewerten und würde Zusatzschritte Z-A1 bis Z-A6 auslösen.
- Als nächste Schritte: Abnahme der Entwürfe (F-1), Optionsentscheidung (D-1), Benennung der verantwortlichen Stelle (D-2), Beauftragung der Rechtsabteilung (LC-1) und Festlegung der Zielbrowser (D-3).
