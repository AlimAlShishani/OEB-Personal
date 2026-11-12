# Zielarchitektur: Defensives Browser-Security-Testing-Framework (konzeptionell, Phase 4)

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** Entwurf zur Entscheidung, **nicht freigegeben** · **Klassifizierung:** intern
**Zugehörig:** `project-charter.md` · `oeb-transformation-concept.md` · `transformation-decision-matrix.md` · `project-roadmap.md` · `phase-4-status.md` · `security-testing-requirements.md` (Anforderungen, Entwurf) · `security-privacy-audit.md` (Befunde)

**Kennzeichnung:** **ANF** = ergibt sich aus den Anforderungen aus Phase 3 (`security-testing-requirements.md`, selbst noch nicht freigegeben) · **VORSCHLAG** = Entwurfsvorschlag, in Phase 5 zu bestätigen oder zu verwerfen · **OFFEN** = Entscheidung oder Information fehlt · **NICHT GEPRÜFT** = nicht untersucht.
**Prüfstatus:** Dies ist ein **konzeptioneller** Entwurf auf hoher Ebene. Es wurden keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt, und es wurde nichts implementiert. Technologie-, Produkt- und Plattformentscheidungen sind bewusst **nicht** getroffen (Abschnitt 11).

---

## 1. Zweck und Geltung

Die Architektur beschreibt Verantwortungsbereiche, Schnittstellen und Vertrauensgrenzen eines **eigenständigen** Frameworks, das in autorisierten, isolierten Laborumgebungen Browser-Sicherheitskonfigurationen, Berechtigungsmodelle, Nachrichtenkommunikation, Eingabevalidierung und Speicher-/Datenschutzverhalten überprüft, reproduzierbare Testfälle ausführt, Nachweise verwaltet und Berichte erzeugt.

**Unabhängigkeit (Abschnitt 12):** Die Architektur funktioniert vollständig ohne Bezug zu Prüfungssoftware, zur Nachahmung fremder Clients und zur automatisierten Prüfungsbearbeitung. Sie nimmt **keine** Komponente, Datei oder Struktur der Altbasis in Anspruch (`oeb-transformation-concept.md` Kategorie K-C bleibt gesperrt).

**Verbindliche Grenzen:** Ausschlusskatalog AK-01 bis AK-09 und Stop-Kriterien der Charta (7.2).

---

## 2. Architekturprinzipien

| Nr. | Prinzip | Herkunft |
|---|---|---|
| AP-1 | **Autorisierung vor Ausführung:** Nichts läuft ohne gültigen Prüfauftrag und Scope | GOV-01 bis GOV-03 (ANF) |
| AP-2 | **Doppelte Durchsetzung des Scope:** logisch (Governance) und technisch (Netzwerk-/Umgebungsgrenze) | GOV-02, LAB-01 (ANF) |
| AP-3 | **Trennung der Zuständigkeiten:** Teststeuerung, Testausführung, Ergebnisverwaltung, Berichtswesen und Benutzeroberfläche sind getrennte Verantwortungsbereiche mit definierten Schnittstellen | Auftrag Phase 4 |
| AP-4 | **Minimale Rechte:** jedes Modul erhält nur die Fähigkeiten, die es braucht; Fähigkeiten werden deklariert und geprüft | SR-NF-02 (ANF) |
| AP-5 | **Nachrichten sind Verträge:** jede Nachricht hat ein versioniertes Schema, einen authentifizierten Absender und wird validiert; keine offenen Zielursprünge für sensible Daten | SR-NF-07 (ANF), NF-01, NF-04 |
| AP-6 | **Synthetische Daten und Datenminimierung:** keine Realdaten, keine Geheimnisse, keine Rohinhalte in Protokollen und Berichten | LAB-03, LOG-02, SR-NF-12 (ANF) |
| AP-7 | **Reproduzierbarkeit:** definierter Ausgangszustand, festgelegte Versionen, deterministische Ausführung, vollständige Metadaten | FR-TC-02, FR-TC-03 (ANF) |
| AP-8 | **Nachweisbarkeit und Integrität:** Ergebnisse sind unveränderlich versiegelt und lückenlos protokolliert | LOG-03, LAB-07 (ANF) |
| AP-9 | **Fehler sind sichtbar:** kein stilles Scheitern, kein „fail-open" bei sicherheitsrelevanten Prüfungen; Teilfehler hinterlassen keinen halb aktiven Zustand | SR-NF-13 (ANF), SEC-14 |
| AP-10 | **Ausschlusskatalog ist technisch durchgesetzt,** nicht nur organisatorisch (Abschnitt 9.4) | GOV-06, NT-01 bis NT-06 (ANF) |

---

## 3. Systemkontext und Vertrauenszonen

```
  ┌────────────────────────────── Z1 Steuerungszone ───────────────────────────────┐
  │  Bediener ──► [Benutzeroberfläche]                                               │
  │                    │ I-01 (Control-Schnittstelle, authentifiziert)               │
  │                    ▼                                                             │
  │  [Governance Gate] ◄─ I-02 ─ [Test Runner (Teststeuerung)] ── I-07 ─► [Ereignisprotokoll]
  │   Prüfauftrag, Scope,           │        │                                         │
  │   Negativliste, Freigaben       │ I-03   │ I-04                                    │
  └─────────────────────────────────┼────────┼─────────────────────────────────────────┘
                                    │        │        Vertrauensgrenze B2 (nur Befehle hinein,
                                    │        │        validierte Beobachtungen heraus)
  ┌─────────────────────────────────▼────────▼──────── Z2 Ausführungszone (Labor) ────┐
  │  [Browser Environment]◄─ I-05 ─ [Security Checks] [Permission Analysis]            │
  │   isolierte Profile, Laborziele,  [Communication Security] [Privacy Analysis]      │
  │   Egress-Allowlist, Snapshots     (Prüfmodule, gemeinsamer Check-Vertrag)          │
  │   ► kein Zugang zu Produktivnetzen, keine Realdaten                                │
  └─────────────────────────────────────┬──────────────────────────────────────────────┘
                                        │ I-06 (Beobachtungen, Artefakte) – Grenze B3
  ┌─────────────────────────────────────▼──────── Z3 Nachweiszone ────────────────────┐
  │  [Evidence Manager]  append-only, Hash-Verkettung, Versiegelung, Aufbewahrung       │
  └─────────────────────────────────────┬──────────────────────────────────────────────┘
                                        │ I-09 (nur lesend) – Grenze B4
  ┌─────────────────────────────────────▼──────── Z4 Berichtszone ────────────────────┐
  │  [Reporting]  Vorlagen, Bewertung, Export, Signatur  ──► I-10/I-11 Berichte         │
  └────────────────────────────────────────────────────────────────────────────────────┘
  Grenze B5: Ausgehender Verkehr aus Z2 nur zu Laborzielen; alles andere gesperrt.
```

| Zone | Zweck | Wichtige Eigenschaft |
|---|---|---|
| Z1 Steuerungszone | Planung, Autorisierung, Steuerung, Bedienung | Kein Zugriff auf Prüfobjekt-Inhalte; hält keine Beobachtungsdaten |
| Z2 Ausführungszone (Labor) | Ausführung gegen Prüfobjekte | Isoliert, ohne Produktivzugang; gilt als **nicht vertrauenswürdig** (Prüfobjekte können fehlerhaft oder feindlich sein) |
| Z3 Nachweiszone | Ergebnisverwaltung | Unveränderlich nach Versiegelung; getrennte Zugriffsrechte |
| Z4 Berichtszone | Berichte | Nur lesender Zugriff auf Nachweise |

---

## 4. Trennung der Verantwortungsbereiche

| Verantwortungsbereich | Module / Bausteine | Verantwortet | Darf nicht |
|---|---|---|---|
| **Teststeuerung** | Test Runner, Governance Gate | Autorisierung, Planung, Ablaufsteuerung, Laufzustand, Not-Aus | Prüflogik enthalten, Ergebnisse speichern, Berichte erzeugen |
| **Testausführung** | Browser Environment, Security Checks, Permission Analysis, Communication Security, Privacy Analysis | Bereitstellung der Umgebung, Durchführung der Prüfungen, Erzeugung von Beobachtungen | Scope festlegen, Ergebnisse dauerhaft verwalten, Zugriff außerhalb der Laborgrenze |
| **Ergebnisverwaltung** | Evidence Manager, Ereignisprotokoll | Aufnahme, Integrität, Aufbewahrung, Löschung von Nachweisen und Ereignissen | Tests ausführen, Befunde bewerten, Berichtslayout bestimmen |
| **Berichtswesen** | Reporting | Befunde, Schweregrade, Berichte, Export | Nachweise verändern, Tests starten |
| **Benutzeroberfläche** | Präsentationsschicht | Anzeige, Eingabe von Aufträgen und Anfragen | Prüflogik, Autorisierungsentscheidungen, Speicherung von Nachweisen |
| **Querschnitt** | Nachrichtenschicht, Ereignisprotokoll | Schemata, Validierung, Protokollierung | Fachliche Entscheidungen |

Zwischen den Bereichen bestehen ausschließlich die Schnittstellen aus Abschnitt 6.

---

## 5. Module

Je Modul: Aufgabe und Verantwortlichkeit · Eingaben · Ausgaben · Abhängigkeiten · Sicherheitsanforderungen · Abgrenzung. Alle Angaben auf hoher Ebene (ANF/VORSCHLAG); Details folgen in Phase 5.

### 5.1 MOD-01 Browser Environment

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | Stellt eine **isolierte, reproduzierbare Browser-Laufumgebung** bereit: Browserversion und -profil (aus sauberer Vorlage), angewandte Richtlinien, Laborzieldienste mit synthetischen Daten, Netzwerkgrenze (Egress-Allowlist), Snapshot/Rücksetzung, Beobachtungsschnittstelle (nur lesend). Setzt den vom Governance Gate vorgegebenen Scope **technisch** durch (AP-2). |
| **Eingaben** | `EnvSpec` (Browser, Version, Betriebssystem, Profilvorlage, Richtliniensatz, Laborziele, Egress-Allowlist, Ressourcenlimits); `run_id`; Befehle `provision`, `apply_policy`, `reset`, `teardown`; Scope-Freigabe vom Governance Gate |
| **Ausgaben** | `EnvHandle`; Umgebungsbeschreibung (Version, Richtlinienstand, Prüfsummen) für die Lauf-Metadaten; schreibgeschützte Beobachtungsströme (z. B. Browserereignisse, Egress-Log); Bestätigung von Rücksetzung/Entsorgung |
| **Abhängigkeiten** | Governance Gate (Scope), Ereignisprotokoll, Infrastruktur für Isolation (OFFEN, DD-1), Browser-Matrix (ED-2) |
| **Sicherheitsanforderungen** | LAB-01 (Isolation, kein Produktivzugang, Egress deny-by-default), LAB-02 (sauberes Profil, festgelegte Version, Metadaten), LAB-03 (synthetische Daten), LAB-05 (Zeitsynchronisation), LAB-08 (Rücksetzung inkl. Zwischenablage); keine Geheimnisse im Profil; Ressourcenbegrenzung; Entsorgung auch bei Fehlern (AP-9) |
| **Abgrenzung** | Führt **keine** Prüflogik aus, trifft **keine** Scope-Entscheidung (setzt sie nur durch), speichert **keine** Ergebnisse (leitet Beobachtungen weiter), erzeugt **keine** Berichte |

### 5.2 MOD-02 Security Checks (Browser-Sicherheitsmechanismen)

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | Prüfmodule für **browserseitige Schutzmechanismen** gegen Laborziele: wirksame Browserrichtlinien gegen Baseline (FR-SM-01), Inhaltssicherheitsrichtlinie, Same-Origin-Verhalten, Cookie-Attribute, Frame-Einbettung, Mixed Content, Berechtigungs-Header (FR-SM-02), Bereitstellung harmloser Testerweiterungen („Canary") zur Prüfung der Richtlinienwirkung (FR-SM-03). |
| **Eingaben** | `TestCase` (Check-Referenz, Parameter, Erwartung); `EnvHandle`; Baseline; Lauf-Kontext (`run_id`, Token) |
| **Ausgaben** | `Observation`-Datensätze (Ergebnis je Prüfpunkt, minimierte Messwerte, Referenz auf Artefakte); Ereignisse |
| **Abhängigkeiten** | Browser Environment (I-05), Evidence Manager (I-06), Ereignisprotokoll, gemeinsamer Check-Vertrag (Abschnitt 6) |
| **Sicherheitsanforderungen** | Nur Laborziele (Scope-geprüft); seiteneffektarm bzw. lesend; deklarierte Fähigkeiten (AP-4); Zeit-, Größen- und Ressourcenlimits (SR-NF-04); keine Geheimnisse und keine Rohinhalte in Ausgaben; Fehler sichtbar (SR-NF-13) |
| **Abgrenzung** | Prüft Browser-/Richtlinienmechanismen. **Nicht:** Berechtigungsinventar der Prüfobjekte (MOD-03), Nachrichtenwege (MOD-04), Speicher/Datenschutz (MOD-05), Ablaufsteuerung (MOD-06) |

### 5.3 MOD-03 Permission Analysis

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | **Berechtigungsinventar** und -bewertung von Prüfobjekten (z. B. Erweiterungen, Richtlinien): Manifest-/Richtlinienauswertung, Vergleich mit Baseline der Minimalrechte, Versionsdifferenz, Hinweise auf Plattformabhängigkeiten (FR-PM-01 bis FR-PM-04). |
| **Eingaben** | Prüfobjekt-Artefakt (Datei im Labor), Baseline, optional zweite Version; Lauf-Kontext |
| **Ausgaben** | Berechtigungsinventar; Abweichungen mit Begründungsfeld; Differenzbericht; `Observation`-Datensätze |
| **Abhängigkeiten** | Evidence Manager, Ereignisprotokoll; Browser Environment nur für optionale dynamische Überprüfung (I-05) |
| **Sicherheitsanforderungen** | Überwiegend **statische** Analyse, **ohne Ausführung** von Prüfobjekt-Code; robuste Parser mit Größen-/Tiefenlimits (FR-IV-01, FR-IV-02); Prüfobjekte nur aus der Laborablage; keine Ausführung auf Produktivgeräten |
| **Abgrenzung** | Bewertet **Rechte**, nicht das Laufzeitverhalten (MOD-04, MOD-05); entscheidet nicht über Freigaben von Software |

### 5.4 MOD-04 Communication Security

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | **Prüfung der Nachrichtenkommunikation** von Prüfobjekten und des Frameworks selbst: Aufzählung der Kanäle (FR-MC-01), Tests der Absender-/Ursprungsprüfung (FR-MC-02), der Zielursprungs-Verwendung bei sensiblen Inhalten (FR-MC-03), der Schema-Validierung (FR-MC-04) und der Auslösbarkeit durch Seiteninhalte an **Labor-Testobjekten** (FR-MC-05). |
| **Eingaben** | Labor-Prüfobjekt, synthetischer Testnachrichtenkatalog (mit Canary-Werten), `EnvHandle`, Lauf-Kontext |
| **Ausgaben** | `Observation`-Datensätze (angenommen/abgelehnt/Fehler, beobachtete Reaktion), Kanalinventar, Ereignisse |
| **Abhängigkeiten** | Browser Environment, Evidence Manager, Ereignisprotokoll |
| **Sicherheitsanforderungen** | Nur gegen Laborobjekte; Testnachrichten tragen ausschließlich synthetische Werte; jede eingespeiste Nachricht wird protokolliert (LOG-04); Ratenbegrenzung; **kein** Mitschnitt oder Verändern fremden Verkehrs (AK-07); Ausgaben ohne Rohinhalte |
| **Abgrenzung** | Prüft Nachrichtenwege und deren Validierung. **Nicht:** Netzwerkverkehr manipulieren, Berechtigungen bewerten (MOD-03), Speicher analysieren (MOD-05) |

### 5.5 MOD-05 Privacy Analysis

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | **Datenschutz- und Speicheranalyse:** Speicherinventur (FR-DP-01), Canary-Verfahren (FR-DP-02), Prüfung der Löschwirkung (FR-DP-03), Egress-Inventar (FR-DP-04), Prüfung auf Klartext-Geheimnisse (FR-DP-05), Lebensdauer von Sitzungsspeichern (FR-DP-06). |
| **Eingaben** | `EnvHandle`; Canary-Spezifikation (eindeutige synthetische Kennungen); deklariertes Aktionsskript für das Prüfobjekt; Egress-Beobachtungsstrom aus der Umgebung; Lauf-Kontext |
| **Ausgaben** | Speicherinventar; Fundorte der Canary-Werte (als Ort/Prüfsumme, nicht als Inhalt); Löschnachweise; Egress-Inventar (Ziele, Auslöser, Datenkategorien); `Observation`-Datensätze |
| **Abhängigkeiten** | Browser Environment (Speicher-/Netzbeobachtung), Evidence Manager, Ereignisprotokoll |
| **Sicherheitsanforderungen** | Ausschließlich synthetische Canary-Daten (LAB-03, P-2); Mitschnitte nur im Labor, zugriffsgeschützt, mit Aufbewahrungsfrist (LAB-06); Ergebnisse minimiert (keine Rohinhalte); Löschung der Zwischendaten am Laufende (LAB-08) |
| **Abgrenzung** | Untersucht Speicher und Übertragungen. **Nicht:** Rechtsbewertung (Datenschutz/Rechtsabteilung), Berechtigungsinventar (MOD-03), Nachrichtenwege (MOD-04) |

### 5.6 MOD-06 Test Runner (Teststeuerung)

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | **Steuert** Testläufe: lädt und validiert den versionierten Testkatalog, holt die Autorisierung beim Governance Gate, orchestriert Umgebung und Prüfmodule, sorgt für **deterministische, wiederholbare** Ausführung (festgelegte Versionen, Startwerte, definierter Ausgangszustand), führt den Laufzustand (Abschnitt 7.3), nimmt Beobachtungen entgegen und übergibt sie dem Evidence Manager, besitzt den Not-Aus (GOV-08), stellt Lauf-Metadaten zusammen (FR-TC-03). |
| **Eingaben** | `RunRequest` (von der Benutzeroberfläche), `TestCase`-Katalog, Referenz auf Prüfauftrag/Scope, Autorisierungsentscheid, Beobachtungen der Prüfmodule |
| **Ausgaben** | `TestRun`-Zustand und -Metadaten, Befehle an Umgebung und Prüfmodule, Weitergabe von Beobachtungen/Artefakten, Befundkandidaten (Abweichung vom Erwartungswert), Ereignisse |
| **Abhängigkeiten** | Governance Gate, Browser Environment, Security Checks, Permission Analysis, Communication Security, Privacy Analysis, Evidence Manager, Ereignisprotokoll, Nachrichtenschicht |
| **Sicherheitsanforderungen** | GOV-01 bis GOV-08 (kein Start ohne Prüfauftrag, Scope-Prüfung, Not-Aus); laufbezogene Berechtigungsnachweise („Tokens") für alle Aufrufe; Schema-Validierung aller Nachrichten; **Serialisierung** konkurrierender Zustandsänderungen je Lauf (ein Schreiber); garantierter Abbau der Umgebung auch bei Fehlern; Katalogänderungen nur nach Vier-Augen-Freigabe (GOV-04) |
| **Abgrenzung** | Enthält **keine** Prüflogik, **keine** Ergebnisspeicherung, **keine** Berichtserzeugung und **keine** Oberfläche. Entscheidet nicht selbst über Scope (Governance Gate) |

### 5.7 MOD-07 Evidence Manager (Ergebnisverwaltung)

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | **Verwaltet Nachweise:** nimmt Beobachtungen und Artefakte entgegen, speichert sie **nur anhängend** mit Hash-Verkettung, bindet sie an `run_id`, versiegelt einen Lauf nach Abschluss, schwärzt/minimiert nach Regel, setzt Aufbewahrungsfristen und Löschung durch, stellt gefilterte lesende Sichten bereit, prüft Integrität. |
| **Eingaben** | `Observation`, Artefakte, Versiegelungsbefehl vom Test Runner, Löschaufträge nach Aufbewahrungsregel, Leseanfragen (Reporting) |
| **Ausgaben** | `evidence_id`, Integritätsnachweise (Hash, Verkettung, Siegel), gefilterte Lesesichten, Löschbestätigungen, Ereignisse |
| **Abhängigkeiten** | Speicher-Backend und Schlüsselverwaltung (OFFEN, DD-5), Ereignisprotokoll, Nachrichtenschicht |
| **Sicherheitsanforderungen** | Unveränderlichkeit nach Versiegelung (LAB-07, LOG-03); Zugriffsklassen und Rollentrennung (Prüfer können Nachweise nicht ändern, GOV-07); Verschlüsselung ruhender Daten (VORSCHLAG, DD-5); **keine** Geheimnisse und Rohinhalte (LOG-02); belegbare Löschung (FR-DP-03, SR-NF-12); Aufbewahrungsfristen (ED-4, LOG-05) |
| **Abgrenzung** | **Keine** Analyse oder Bewertung von Inhalten, **keine** Ausführung von Tests, **kein** Berichtslayout. Entscheidet nicht über Schweregrade |

### 5.8 MOD-08 Reporting

| | |
|---|---|
| **Aufgabe und Verantwortlichkeit** | **Erstellt Prüfberichte:** wendet Berichtsvorlagen an, ordnet Befunde mit eindeutigen IDs, Evidenzklasse (BESTÄTIGT/ABGELEITET/OFFEN/NICHT GEPRÜFT) und Schweregrad nach dokumentiertem Maßstab zu, fügt Reproduktionsangaben und Beleg-Verweise bei, prüft vor dem Export auf Geheimnisse und Rohinhalte, erzeugt lesbare und maschinenlesbare Ausgaben, versioniert und signiert Berichte, verteilt nach Rolle. |
| **Eingaben** | Lesende Nachweissichten (Evidence Manager), Befundkandidaten und Lauf-Metadaten, Berichtsvorlagen, Bewertungsmaßstab (Konfiguration aus Governance) |
| **Ausgaben** | Berichte (REP-01 bis REP-07), maschinenlesbare Fassung, Versions-/Freigabestatus, Ereignisse |
| **Abhängigkeiten** | Evidence Manager (nur lesend, I-09), Vorlagenspeicher, Signaturdienst (OFFEN, DD-7), Ereignisprotokoll |
| **Sicherheitsanforderungen** | Ausgabe-Bereinigung (Darstellung ausschließlich als Text, SR-NF-03); Schutz vor Geheimnissen/Rohinhalten im Export (Canary-Prüfung); Zugriff nach Rolle (GOV-07); Schwärzung sensibler Belege (REP-04); Berichte tragen Reproduktionsangaben (REP-06) |
| **Abgrenzung** | **Verändert keine** Nachweise, **startet keine** Tests, **speichert** keine Rohdaten. Bewertungsmaßstab und Vorlagen werden fachlich festgelegt, nicht vom Modul |

### 5.9 Bausteine (Querschnitt und Oberfläche)

| Baustein | Aufgabe | Eingaben → Ausgaben | Sicherheit | Abgrenzung |
|---|---|---|---|---|
| **QB-01 Governance Gate** | Vertrauensanker für Autorisierung: prüft Prüfauftrag und Scope (Signatur, Zeitfenster, Negativliste), stellt laufbezogene Berechtigungsnachweise aus, steuert die Egress-Vorgabe der Umgebung | `RunRequest`/Prüfauftrag/Scope → Entscheid (erlaubt/verweigert), Token, Egress-Vorgabe | Eigenständig vom Test Runner (Umgehung nicht durch den Runner möglich); signierte Scope-Dateien; Vier-Augen-Pflege (GOV-04); Protokollierung jedes Entscheids | Führt keine Tests aus, speichert keine Ergebnisse |
| **QB-02 Nachrichtenschicht** | Versionierte Schemata, Validierung, Absender-Autorisierung, einheitliches Antwort-/Fehlerformat | Nachrichten → validierte Nachrichten/Ablehnungen | SR-NF-07: kein Wildcard-Ursprung für sensible Daten, Typ-/Größenprüfung, Absenderkennung | Trifft keine fachlichen Entscheidungen |
| **QB-03 Ereignisprotokoll** | Strukturiertes, manipulationsgeschütztes Ereignisprotokoll | Ereignisse (I-07) → Protokolleinträge | LOG-01 bis LOG-06: Datenminimierung, Verkettung, getrennte Ablage, Fehlerereignisse | Speichert keine Nachweise (das tut MOD-07) |
| **PL-01 Benutzeroberfläche** | Präsentationsschicht: Aufträge anlegen, Läufe planen/starten/stoppen, Status und Berichte ansehen | Bedienereingaben → `RunRequest` über I-01; Statusanzeige | Authentifizierte Bediener, Rollenprüfung, nur sichere Textdarstellung (SR-NF-03), keine eigene Prüflogik, keine Speicherung von Nachweisen | Keine Autorisierungsentscheidung, kein direkter Zugriff auf Umgebung oder Nachweise |

---

## 6. Schnittstellen (hohe Ebene)

Für alle Schnittstellen gilt: versioniertes Schema, authentifizierter Absender, Validierung durch QB-02, Protokollierung durch QB-03 (AP-5).

| Nr. | Von → Nach | Zweck | Wesentliche Inhalte | Besondere Anforderungen |
|---|---|---|---|---|
| I-01 | Benutzeroberfläche → Test Runner | Steuerung | `RunRequest`, Abbruch, Statusabfrage | Rollenprüfung; keine Prüf- oder Nachweisdaten |
| I-02 | Test Runner ↔ Governance Gate | Autorisierung | Prüfauftrag-/Scope-Referenz → Entscheid, Token | Ohne positiven Entscheid keine Weiterführung |
| I-03 | Test Runner → Browser Environment | Umgebungssteuerung | `EnvSpec`, `provision`/`apply_policy`/`reset`/`teardown` | Nur mit gültigem Token; Egress-Vorgabe aus I-02 |
| I-04 | Test Runner → Prüfmodule | Ausführung | `execute(check_id, params, env_handle, run_ctx)` | Gemeinsamer Check-Vertrag; Zeitlimit |
| I-05 | Prüfmodule → Browser Environment | Beobachtung | Lesende Beobachtungs-/Probe-Zugriffe | Nur lesend; Laborziele |
| I-06 | Prüfmodule / Test Runner → Evidence Manager | Nachweisaufnahme | `Observation`, Artefakte | Nur anhängend; `run_id`-Bindung; Minimierung |
| I-07 | Alle → Ereignisprotokoll | Protokollierung | `Event` | Keine Geheimnisse/Rohinhalte |
| I-08 | Test Runner → Evidence Manager | Versiegelung | Siegelbefehl am Laufende | Danach unveränderlich |
| I-09 | Reporting → Evidence Manager | Lesen | Gefilterte Nachweissichten | Nur lesend |
| I-10 | Benutzeroberfläche → Reporting | Berichtsanforderung/-ansicht | Berichtsanfrage, Anzeige | Rollenprüfung |
| I-11 | Reporting → Ablage/Export | Bericht ausgeben | Signierter Bericht | Prüfung auf Geheimnisse vor Export |
| I-12 | Governance Gate ↔ Scope-/Auftragsablage | Laden und Verifizieren | Signierte Scope-/Auftragsdateien | Integritätsprüfung, Gültigkeitsfenster |
| I-13 | Evidence Manager ↔ Aufbewahrungsdienst | Fristen und Löschung | Löschaufträge, Bestätigungen | Belegbare Löschung |

---

## 7. Datenobjekte und Laufmodell

### 7.1 Datenobjekte (Kernfelder, VORSCHLAG)

| Objekt | Kernfelder |
|---|---|
| **Prüfauftrag** | ID, Auftraggeber, Zweck, Prüfobjekte, Zeitfenster, Verantwortliche, Scope-Referenz, Freigaben |
| **Scope** | Version, Allowlist, Negativliste, Gültigkeit, Signatur |
| **TestCase** | ID, Version, Ziel, Anforderungsbezug, Voraussetzungen, Check-Referenz, Parameter, Erwartung, Schweregrad bei Abweichung, Datenreferenz |
| **TestRun** | `run_id`, Prüfauftrag, Umgebungsbeschreibung, Katalog- und Framework-Version, Zustand, Start/Ende |
| **Observation** | ID, `run_id`, Testfall, Zeit, Subjekt, Art, minimiertes Ergebnis, Artefakt-Verweise |
| **Finding** | ID, `run_id`, Testfall, Schweregrad, Evidenzklasse, Beschreibung, Reproduktion, Empfehlung, Status |
| **EvidenceItem** | `evidence_id`, `run_id`, Typ, Hash, Vorgänger-Hash, Größe, Zeit, Schwärzungsstand, Aufbewahrung bis, Zugriffsklasse |
| **Event** | ID, Zeit, Quelle, Typ, Schweregrad, `run_id`, Testfall, Korrelations-ID, Ergebnis, Details (ohne Geheimnisse) |
| **Report** | ID, Version, Lauf-Referenzen, Vorlage, Status, Signatur |

### 7.2 Ablauf eines Testlaufs
1. Bediener legt im UI einen Lauf an (I-01); Test Runner prüft das Schema.
2. Test Runner fordert beim Governance Gate die Autorisierung (I-02). Ohne Freigabe endet der Lauf als „verweigert" (protokolliert).
3. Test Runner lädt Testfälle, wählt aus, legt Startwerte fest.
4. Browser Environment wird aus sauberer Vorlage bereitgestellt; Egress-Vorgabe wird gesetzt (I-03).
5. Prüfmodule führen die Testfälle aus (I-04/I-05); Beobachtungen gehen an den Evidence Manager (I-06); alle Schritte erzeugen Ereignisse (I-07).
6. Am Ende: Umgebung wird abgebaut und zurückgesetzt; Lauf wird versiegelt (I-08).
7. Reporting liest die Nachweise (I-09) und erzeugt den Bericht (I-11).
8. Aufbewahrung und Löschung laufen nach Regel (I-13).

### 7.3 Laufzustand (VORSCHLAG)

```
PLANNED ─► AUTHORIZED ─► PROVISIONED ─► RUNNING ─► COLLECTING ─► SEALED ─► REPORTED ─► ARCHIVED
   │            │              │            │            │
   ▼            ▼              ▼            ▼            ▼
 DENIED       ABORTED (Not-Aus)        FAILED  (jeweils mit Pflicht zum Abbau/Reset und Ereignis)
```

Regeln: ein Schreiber je Lauf (Serialisierung); Zustandswechsel nur durch den Test Runner; jeder Fehler- oder Abbruchpfad führt zu **Abbau der Umgebung und Protokollierung**, nie zu einem halb aktiven Zustand (AP-9); ein Lauf ohne Siegel gilt als unvollständig und wird als solcher gekennzeichnet.

---

## 8. Abhängigkeitsübersicht

| Modul | hängt ab von |
|---|---|
| Benutzeroberfläche | Test Runner (I-01), Reporting (I-10) |
| Test Runner | Governance Gate, Browser Environment, Prüfmodule, Evidence Manager, Ereignisprotokoll, Nachrichtenschicht |
| Browser Environment | Governance Gate (Scope), Ereignisprotokoll, Infrastruktur (OFFEN) |
| Security Checks / Permission Analysis / Communication Security / Privacy Analysis | Browser Environment, Evidence Manager, Ereignisprotokoll, Check-Vertrag |
| Evidence Manager | Ereignisprotokoll, Speicher-/Schlüsselverwaltung (OFFEN) |
| Reporting | Evidence Manager (lesend), Vorlagenspeicher, Signaturdienst (OFFEN) |
| Governance Gate | Scope-/Auftragsablage, Ereignisprotokoll |

**Keine Rückwärtsabhängigkeiten:** Evidence Manager und Reporting kennen weder Test Runner-Interna noch Prüflogik; Prüfmodule kennen weder Berichte noch Scope-Entscheidungen.

---

## 9. Sicherheitsarchitektur

### 9.1 Vertrauensgrenzen
B1 Bediener ↔ UI/Control; B2 Steuerung ↔ Ausführung (Befehle hinein, validierte Beobachtungen heraus); B3 Ausführung ↔ Nachweise (nur anhängend); B4 Nachweise ↔ Berichte (nur lesend); B5 Labor ↔ Außenwelt (gesperrt außer Laborziele).

### 9.2 Zentrale Kontrollen

| Kontrolle | Umsetzung (VORSCHLAG) | Anforderung |
|---|---|---|
| Autorisierung | Governance Gate mit signiertem Prüfauftrag/Scope, Zeitfenster, Negativliste | GOV-01 bis GOV-05 |
| Scope-Durchsetzung | Doppelt: Gate und Egress-Grenze der Umgebung | AP-2, LAB-01 |
| Minimale Rechte | Deklarierte Fähigkeiten je Prüfmodul, Ablehnung nicht deklarierter Fähigkeiten | SR-NF-02 |
| Nachrichtenschutz | Schemata, authentifizierte Absender, laufbezogene Tokens | SR-NF-07 |
| Geheimnisse | Keine Fremd-API-Schlüssel; interne Dienstzugangsdaten nicht im Klartext, kurzlebig | SR-NF-05, SR-NF-08 |
| Datenschutzgerechtes Logging | Minimierung, Redaktion, Verkettung, getrennte Ablage | LOG-02, LOG-03 |
| Reproduzierbarkeit | Festgelegte Versionen, saubere Profile, Metadaten, deterministische Startwerte | FR-TC-02, LAB-02 |
| Lieferkette | Gesperrte Abhängigkeiten, SBOM, signierte Releases | SR-NF-09 |

### 9.3 Vorläufige Bedrohungsliste (Bedrohungsmodell folgt in Phase 5, Ergebnis E5-2)

| ID | Bedrohung | Gegenmaßnahme (Auszug) |
|---|---|---|
| TH-01 | Scope-Umgehung oder Fehlkonfiguration; Ziele außerhalb des Labors | Signierte Scope-Dateien, Negativliste, Egress-Grenze, Verweigerung und Protokollierung |
| TH-02 | Funktionsdrift in Richtung ausgeschlossener Fähigkeiten | Fähigkeitsmodell, Negativtests NT-01 bis NT-06, Review, Stop-Kriterien |
| TH-03 | Feindliches oder fehlerhaftes Prüfobjekt | Isolation, Ressourcenlimits, Einbahn-Datenfluss, Ausgabe nur als Text |
| TH-04 | Manipulation von Nachweisen | Anhängender Speicher, Hash-Verkettung, Versiegelung, Rollentrennung |
| TH-05 | Offenlegung sensibler Daten in Protokollen/Berichten | Minimierung, Redaktion, Canary-Prüfung vor Export |
| TH-06 | Gefälschte interne Nachrichten | Schema, Absender-Authentisierung, Tokens, keine offenen Zielursprünge |
| TH-07 | Missbrauch durch Berechtigte (nicht autorisierte Läufe) | Rollenmodell, Vier-Augen-Prinzip, Protokollierung, Not-Aus |
| TH-08 | Manipulierte Abhängigkeiten oder Builds | SBOM, Sperrung, Signierung, reproduzierbare Builds |
| TH-09 | Falsche Zuordnung von Nachweisen zu Läufen | Bindung an `run_id`, Verkettung |
| TH-10 | Ressourcenerschöpfung durch Eingaben | Limits auf allen Eingabepfaden, Zeitlimits |
| TH-11 | Verlust der Reproduzierbarkeit | Festgelegte Umgebungen, Snapshots, Metadaten |
| TH-12 | Unbefugter Zugriff auf Berichte | Zugriffsklassen, Rollen, Verteilerlisten |

### 9.4 Durchsetzung des Ausschlusskatalogs (AP-10)
Auf **drei Ebenen**, damit kein Einzelversagen genügt:
1. **Governance:** Scope-Allowlist und Negativliste (GOV-02, GOV-03); keine Läufe gegen Produktiv- oder Prüfungsplattformen.
2. **Fähigkeitsmodell:** Prüfmodule deklarieren ihre Fähigkeiten; nicht deklarierte oder nicht zulässige Fähigkeiten (z. B. Änderung fremden Datenverkehrs, Auslesen fremder Seiteninhalte, Übertragung an externe Dienste) werden abgelehnt.
3. **Build- und Review-Prüfung:** Negativtests NT-01 bis NT-06 bei jedem Build, Herkunftsprotokoll und Vier-Augen-Review (`oeb-transformation-concept.md` 4, 6, 7).

---

## 10. Rückverfolgbarkeit: Module und Anforderungen

| Modul / Baustein | Wesentliche Anforderungen |
|---|---|
| MOD-01 Browser Environment | LAB-01 bis LAB-08, FR-TC-02 |
| MOD-02 Security Checks | FR-SM-01 bis FR-SM-03, FR-IV-01, SR-NF-04 |
| MOD-03 Permission Analysis | FR-PM-01 bis FR-PM-04, SR-NF-02 |
| MOD-04 Communication Security | FR-MC-01 bis FR-MC-05, SR-NF-07 |
| MOD-05 Privacy Analysis | FR-DP-01 bis FR-DP-06, SR-NF-12 |
| MOD-06 Test Runner | FR-TC-01 bis FR-TC-06, GOV-01 bis GOV-08 |
| MOD-07 Evidence Manager | LAB-07, LOG-02, LOG-03, LOG-05, SR-NF-12 |
| MOD-08 Reporting | REP-01 bis REP-07 |
| QB-01 Governance Gate | GOV-01 bis GOV-08 |
| QB-02 Nachrichtenschicht | SR-NF-07, FR-MC-04 |
| QB-03 Ereignisprotokoll | LOG-01 bis LOG-06, SR-NF-13 |
| PL-01 Benutzeroberfläche | SR-NF-03, GOV-07 |
| Querschnitt | SR-NF-01, SR-NF-05, SR-NF-06, SR-NF-08 bis SR-NF-11, FR-ST-01, NT-01 bis NT-06 |

Die Rückverfolgbarkeit zu den Auditbefunden steht in `security-testing-requirements.md` 6.2.

---

## 11. Offene Entwurfsentscheidungen (für Phase 5)

Es werden hier **keine** Technologien oder Produkte gewählt.

| Nr. | Entscheidung | Optionen (ohne Vorfestlegung) | Entscheidungskriterien |
|---|---|---|---|
| DD-1 | Ausführungsmodell | Externer Prüfstand mit Steuerung von außen; Instrumentierung im Browser; Kombination | Minimale Rechte im Zielbrowser, Reproduzierbarkeit, Browserabdeckung, Isolation, Risiko |
| DD-2 | Plattform (falls Browser-Komponenten) | Unterstützte, nicht auslaufende Erweiterungsplattform; keine Erweiterung | SR-NF-01, Browser-Matrix |
| DD-3 | Nachrichtentransport und Authentisierung zwischen Modulen | – | Schema-Validierung, Absenderprüfung, Isolation |
| DD-4 | Erweiterungsmodell der Prüfmodule | Festes Verzeichnis geprüfter Module; Plug-in mit Signatur und Fähigkeitsdeklaration | Ausschlusskontrolle, Wartbarkeit |
| DD-5 | Nachweisspeicher | Format, Verschlüsselung ruhender Daten, Schlüsselverwaltung | Integrität, Aufbewahrung, Löschbarkeit |
| DD-6 | Testfall- und Datenformat | – | Maschinenlesbarkeit, Versionierung, Validierbarkeit |
| DD-7 | Berichtsformate und Signaturverfahren | – | Lesbarkeit, Maschinenlesbarkeit, Nachprüfbarkeit |
| DD-8 | Art der Benutzeroberfläche | – | Rollen, Zugriff, Sicherheit |
| DD-9 | Umgang mit internen Dienstzugangsdaten | – | SR-NF-08 |

---

## 12. Unabhängigkeit von SEB-Umgehung und Prüfungsautomatisierung

| Aussage | Begründung |
|---|---|
| Kein Modul benötigt Kenntnisse über Prüfungssoftware oder Prüfungsplattformen | Prüfobjekte sind generische Browser-Konfigurationen, Erweiterungen und Webseiten im Labor |
| Keine Fähigkeit zur Identitätsnachahmung fremder Clients | Fähigkeitsmodell (9.4) und NT-01/NT-02 |
| Keine Erfassung oder Nutzung von Prüfungskonfigurationen | AK-02, NT-04 |
| Keine automatisierte Erfassung oder Beantwortung von Fragen; keine KI-Anbindung | AK-04, SR-NF-05, NT-03/NT-04 |
| Keine Nachbildung fremder Oberflächen, keine Fremdassets | AK-03, NT-05 |
| Keine Nutzung von Altbasis-Code oder -Struktur | Kategorie K-C gesperrt, `oeb-transformation-concept.md` 2, 7 |
| Funktionstest der Unabhängigkeit | Review-Checkliste je Gate und Negativtests; Verstoß = Stop-Kriterium (Charta 7.2) |

---

## 13. Offene Punkte

| Nr. | Punkt | Status |
|---|---|---|
| AO-1 | Architekturrichtung (DD-1) und Plattform (DD-2) | OFFEN (Phase 5, ED-3) |
| AO-2 | Zielbrowser und -versionen | OFFEN (ED-2); heute NICHT GEPRÜFT |
| AO-3 | Bedrohungsmodell im Detail | OFFEN (Phase 5, E5-2) |
| AO-4 | Aufbewahrungsfristen | OFFEN (ED-4) |
| AO-5 | Produktname, Lizenz des neuen Projekts | OFFEN |
| AO-6 | Freigabe der Anforderungen (`security-testing-requirements.md`) | OFFEN (G2) |
| AO-7 | Laufzeit-, Browser- und Netzwerktests | NICHT GEPRÜFT (erst ab Phase 6/7) |
