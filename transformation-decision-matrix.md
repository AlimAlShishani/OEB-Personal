# Entscheidungsmatrix: Ausgangsbasis für das Browser-Security-Testing-Framework (Phase 4)

**An:** Projektleitung / IT-Abteilung · **Von:** technischer Analyst · **Stand:** 2026-11-12 · **Status:** Entwurf zur Entscheidung · **Klassifizierung:** intern
**Zugehörig:** `project-charter.md` · `project-roadmap.md` · `phase-4-status.md` · Phase-3-Dokumente · Vorarbeiten (Phase 1–2.1)

**Kennzeichnung:** BESTÄTIGT (durch Quellcode, Dateien oder Dokumente belegt) · ABGELEITET (Schlussfolgerung) · OFFEN (Klärung nötig; Grund angegeben) · NICHT GEPRÜFT (außerhalb des Prüfumfangs).
**Prüfstatus:** Es wurden keine Laufzeit-, Browserkompatibilitäts- oder Netzwerktests durchgeführt. Es wurden keine Quell-, Konfigurations- oder Build-Dateien verändert.

---

## 1. Fragestellung und Bewertungsumfang

**Frage:** Welche Teile der bestehenden OEB-Projektgrundlage können als **organisatorische, strukturelle oder technische Ausgangsbasis** für ein eigenständiges defensives Security-Testing-Framework dienen? Es geht ausdrücklich nicht darum, die bisherige SEB-bezogene Funktionalität zu erhalten.

**Der Bewertungsumfang ist auf sechs Bereiche beschränkt:**
1. Allgemeine Projekt- und Verzeichnisstruktur
2. Build- und Entwicklungsorganisation
3. Dokumentationsstruktur
4. Allgemeine Benutzeroberflächenkonzepte
5. Unkritische Konfigurations- und Verwaltungsmechanismen
6. Allgemeine Test- und Reporting-Konzepte

**Nicht untersucht, rekonstruiert, optimiert oder übernommen werden** die ausgeschlossenen Funktionsbereiche (Ausschlusskatalog AK-01 bis AK-09, `security-testing-requirements.md` 1.3). Dieses Dokument verweist auf sie nur als Gesamtmenge und nicht in ihrer Funktionsweise. Die dazu verwendeten Angaben stammen aus den Phase-3-Dokumenten.

**Methode:** Grundlage sind die bereits dokumentierten und gegengeprüften Befunde (Phase 1–3), ergänzt durch eine gezielte Suche nach Lizenz- und Herkunftsangaben in Dateiliste, Lockfile, Manifesten und README sowie zwei Websuchen (Abschnitt 3.5). Kein Funktionsmodul wurde erneut untersucht.

---

## 2. Bestand im Bewertungsumfang

| Bereich | Vorhandener Bestand | Fundstelle / Status | Eignung als Ausgangsbasis |
|---|---|---|---|
| **1. Projekt-/Verzeichnisstruktur** | `source/` mit den Unterordnern `seb/` und `assistant/` (nach Funktionsbereichen benannt) sowie wenigen Wurzeldateien; 55 Dateien. Bezeichner im Bestand tragen den Namen der Altbasis | BESTÄTIGT (Dateiliste; `technical-handover.md` 3) | **Gering.** Die Gliederung folgt den ausgeschlossenen Funktionsbereichen und trägt deren Namen; sie ist weder neutral noch auf die neuen Verantwortungsbereiche übertragbar. |
| **2. Build-/Entwicklungsorganisation** | `build.mjs` (95 Z.) und `package.json` (14 Z.): eine Quelle, zwei Zielpakete, Manifestprüfung, ZIP; Lockfile v3 mit 25 Dev-Paketen. **Fehlt:** Lint, Tests, CI, `engines`/Node-Vorgabe, Lizenzfeld, SBOM. Das Build-Skript importiert ein funktionsgebundenes Modul (`build.mjs:6,17`) | BESTÄTIGT (`build.mjs`, `package.json`, `package-lock.json`) | **Mittel als Vorlage.** Das Muster („eine Quelle, mehrere Zielpakete, Prüfung im Build") ist allgemein; die konkrete Datei benötigt grundlegende Überarbeitung und Lizenzklärung. |
| **3. Dokumentationsstruktur** | README (80 Z., Projektbeschreibung, Haftungsausschluss, Herkunftshinweis). **Fehlt:** Lizenz, Changelog. **Eigene Dokumentation (Phase 1–4):** Handover, Verifikationsbericht, Lückenliste, Anforderungen mit Befund-IDs, Kennzeichnungsschema, Änderungsprotokoll | BESTÄTIGT (README; Repository) | **Hoch für die eigene Dokumentationsstruktur** (eigene Arbeit, neutral); **keine** Übernahme des README-Textes. |
| **4. Allgemeine UI-Konzepte** | Popup mit zwei Reitern, Statusanzeige und Meldungen; Darstellung über Textzuweisung statt HTML-Einfügung; Zustandsaktualisierung bei Speicheränderung. `popup.css` (914 Z.) wurde nicht geprüft | BESTÄTIGT (Konzepte, `security-privacy-audit.md` 9.3); `popup.css` NICHT GEPRÜFT | **Konzeptuell nutzbar** (Reiter, Status, sichere Textdarstellung), als **Code** wegen funktionsgebundener Inhalte nicht. |
| **5. Unkritische Konfigurations-/Verwaltungsmechanismen** | Normalisierung von Einstellungen mit Standardwerten und Altformat-Behandlung (16 Z.); zentrale Weitergabe von Einstellungsänderungen. **Fehlt:** Schema-Versionierung, zentrale Speicherschicht, Löschfunktion | BESTÄTIGT (`technical-handover.md` 3.3, 7.1) | **Gering bis mittel als Muster;** die konkreten Einstellungen gehören zu ausgeschlossenen Bereichen. |
| **6. Allgemeine Test-/Reporting-Konzepte** | **Im Bestand nicht vorhanden:** keine Tests, keine CI, kein Berichtswesen. Wohl vorhanden sind testfreundliche Muster (injizierbare Abhängigkeiten) in funktionsgebundenen Modulen | BESTÄTIGT (Negativbefund); Muster `security-privacy-audit.md` 9.3 | **Keine Basis.** Konzepte stammen aus der eigenen Dokumentation (Evidenzklassen, Befund-IDs), nicht aus dem OEB-Code. |

**Mengengerüst der geeigneten Bestandteile (BESTÄTIGT):** Verwertbar als Vorlage wären höchstens Build-Dateien (109 Z.), die beiden Manifeste (171 Z.) und die Einstellungsnormalisierung (16 Z.): **296 Zeilen, rund 4 % des Gesamtbestands von 6 844 Zeilen** (6 735 Zeilen Quelltext plus 109 Zeilen Build-Dateien). Das Stylesheet `popup.css` (914 Z.) ist nicht bewertbar (NICHT GEPRÜFT). Der Ausschlussbereich umfasst nach der Phase-4-Zählung 30 Dateien mit 5 346 Zeilen (Phase 3: 5 427 Zeilen einschließlich 81 Zeilen technisch zu ersetzender Hilfsdateien; die Zahlen stimmen überein).

---

## 3. Lizenz- und Herkunftsstand

**Grundsatz:** Es wird hier keine Lizenzfreigabe behauptet. Rechtliche Bewertungen sind der Rechtsabteilung vorbehalten.

### 3.1 Identifizierte Vorlagen und Fremdbestandteile

| Nr. | Bestandteil | Wie identifiziert | Status |
|---|---|---|---|
| V-1 | Projekt `UmmItKin/SebBypass` | README nennt es als Grundlage („Based on …", `README.md:7`) | Nennung BESTÄTIGT; Lizenz **OFFEN** (3.5) |
| V-2 | Projekt `cycyrild/SebBypass` („archived" laut README) | `README.md:7` | Nennung BESTÄTIGT; Lizenz **OFFEN** (3.5) |
| V-3 | OEB selbst | Namensraum `63kitsune` in den README-Links (`README.md:18,36`); kein Rechteinhaber im Paket benannt | Nennung BESTÄTIGT; Rechteinhaber **OFFEN** |
| V-4 | Fremder Urheberrechtsvermerk als Anzeigetext in einer Oberflächendatei („Copyright © 2025 ETH Zürich, IT Services") | `source/seb/assets/startup.html:210,230` | Vorhandensein BESTÄTIGT; Bedeutung und Rechtslage **OFFEN**. Bereich gehört zum Ausschlusskatalog (AK-03) |
| V-5 | Logo-/Bilddateien der Oberfläche der Altbasis (`seb/icons/SebSplashScreen.png`, `seb_logo.png`) | Dateiliste | Herkunft nicht dokumentiert, **OFFEN**. AK-03 |
| V-6 | Projekt-Symbole (`assets/icons/*`, `icon.svg`) | Dateiliste | Herkunft nicht dokumentiert, **OFFEN** |
| V-7 | Vorschau-GIFs (`preview/*`) | Dateiliste; Verweis im README | Herkunft nicht dokumentiert, **OFFEN**; zeigen Funktionen der Altbasis (AK) |
| V-8 | Dev-Abhängigkeiten (25 Pakete: u. a. `terser`, `fflate`, `lightningcss`) | `package-lock.json` | Lockfile-Angaben: MPL-2.0 ×12, MIT ×10, Apache-2.0 ×1, BSD-3-Clause ×1, BSD-2-Clause ×1 (BESTÄTIGT als Angabe im Lockfile); unabhängige Verifikation und Rechtswirkung **NICHT GEPRÜFT** |

### 3.2 Tatsächlich vorliegende Lizenzinformationen

| Information | Befund |
|---|---|
| Lizenzdatei (`LICENSE`, `NOTICE`, `COPYING`, `AUTHORS`) im Paket | **Nicht vorhanden** (BESTÄTIGT) |
| Lizenz-/Autorenfeld in `package.json` und beiden Manifesten | **Nicht vorhanden** (BESTÄTIGT) |
| Lizenz- oder Copyright-Kopfzeilen in Quelldateien | **Nicht vorhanden**; einziger Treffer ist der Anzeigetext V-4 (BESTÄTIGT durch Suchlauf) |
| README | Enthält einen Haftungs-/Nutzungshinweis („Use only where permitted. Provided as is, without warranty; users are responsible for their use.", `README.md:78`). Das ist **ein Haftungsausschluss und keine Einräumung von Nutzungsrechten** (ABGELEITET; rechtliche Bewertung durch die Rechtsabteilung OFFEN) |
| Lizenzen der Vorlageprojekte V-1, V-2 | **Nicht ermittelt** (3.5) |
| Lizenzen der Dev-Abhängigkeiten | Nur als Lockfile-Angabe vorhanden (V-8) |

### 3.3 Ungeklärte Herkunftsfragen

| Nr. | Frage | Warum relevant | Status |
|---|---|---|---|
| H-1 | Unter welchen Bedingungen dürfen V-1 und V-2 und daraus abgeleitete Werke weiterverwendet werden? | Abgeleitetes Werk der Altbasis | OFFEN (Quelle nicht eingesehen, 3.5) |
| H-2 | Wer ist Rechteinhaber von OEB, und unter welchen Bedingungen wird es weitergegeben? | Ohne Lizenz grundsätzlich keine Nutzungsrechte (ABGELEITET; Rechtsabteilung) | OFFEN |
| H-3 | Welche Rechte bestehen an den Oberflächen-/Logo-Assets (V-4, V-5) und wie sind sie markenrechtlich einzuordnen? | Fremde Marken-/Gestaltungselemente | OFFEN; Bereich ausgeschlossen |
| H-4 | Herkunft der Symbole und Vorschau-Dateien (V-6, V-7) | Unbekannte Urheber | OFFEN |
| H-5 | Sind die Lizenzangaben im Lockfile zutreffend, und welche Pflichten folgen daraus für Werkzeuge im neuen Projekt? | Compliance im Build | NICHT GEPRÜFT |
| H-6 | Ist die Weiterverwendung eines Projekts, dessen Zweck die Umgehung von Prüfungssoftware ist, für die Organisation unabhängig von der Lizenz zulässig und vertretbar? | Compliance-/Reputationsfrage (M-03) | OFFEN (Entscheidung der Projektleitung mit Rechtsabteilung) |

### 3.4 Bis zur Klärung nicht wiederzuverwenden (Sperrliste)

| Bestandteil | Sperre | Begründung |
|---|---|---|
| Alle Quelldateien der Altbasis (JS, HTML, CSS, Manifeste) einschließlich Build-Skript und `package.json` | **Gesperrt** | Keine Lizenz; abgeleitetes Werk (V-1 bis V-3) |
| README-Text, Vorschau-GIFs, Symbole, Logo-/Oberflächen-Assets | **Gesperrt** | Herkunft ungeklärt (V-4 bis V-7); teilweise Ausschlussbereich |
| `package-lock.json` | Nicht übernehmen | Wird neu erzeugt; Lizenzprüfung der neuen Abhängigkeiten ist ein eigener Schritt (H-5) |
| Ausschlussbereich (AK-01 bis AK-09) | **Dauerhaft gesperrt**, unabhängig von jeder Lizenzklärung | Ausschlusskatalog |
| Eigene Dokumentation (Phase 1–4) | Frei innerhalb der Organisation verwendbar, **aber zugriffsbeschränkt** für Dokumente, die die Altbasis beschreiben | Eigenes Werk; Informationsschutz (M-07) |
| Allgemeine Konzepte und Muster (z. B. Reiter-Oberfläche, einheitliches Antwortformat, Allowlist-Gedanke, Dokumentationsschema) | Nach Einschätzung des Analysten unkritisch, wenn sie **neu implementiert** werden und kein Code oder Text kopiert wird; rechtliche Bestätigung **OFFEN** | Ideen und allgemeine Muster werden anders behandelt als Code (ABGELEITET, nicht rechtlich geprüft) |

### 3.5 Recherchen und ihre Grenzen
- Ausgewertet: Dateiliste, `package.json`, `package-lock.json`, beide Manifeste, `README.md`, Suchlauf nach Lizenz-/Copyright-Begriffen im Quelltext.
- Zwei Websuchen zu den Lizenzen von V-1 und V-2 lieferten **keine belastbare Information** (weder Lizenz noch Archivstatus). Eine Einsicht in die Originalrepositories durch eine berechtigte Person ist erforderlich (OFFEN). Aus der Websuche werden keine Tatsachen übernommen.

---

## 4. Optionen

**Zuordnung zu den Optionsbezeichnungen aus Phase 3** (zur Vermeidung von Verwechslungen):

| Phase 4 (hier) | Entspricht ungefähr (Phase 3) |
|---|---|
| **A** – Bestehende OEB-Projektstruktur als organisatorische Grundlage verwenden | A (Umbenennen/Fork) und B (selektive Übernahme) |
| **B** – Unkritische Bestandteile nach unabhängiger Prüfung neu aufbauen | C (Neuentwicklung mit Wissensübernahme) |
| **C** – Vollständige Neuentwicklung | C, ohne dokumentierte Wissensübernahme aus der Altbasis |

### 4.1 Beschreibung

| | **A – Bestehende Struktur als Grundlage** | **B – Unkritisches nach Prüfung neu aufbauen** | **C – Vollständige Neuentwicklung** |
|---|---|---|---|
| Kern | Das neue Projekt entsteht auf der vorhandenen Projektstruktur (Ordner, Build, Manifeste, Bezeichner); Ausschlussbereiche werden zurückgebaut, der Rest umgebaut | Die sechs Bereiche aus Abschnitt 2 werden **unabhängig geprüft**; geeignete Muster werden als Anforderung beschrieben und **neu implementiert** (kein Kopieren) | Eigenes Repository, eigene Struktur und Konzepte aus den Anforderungen; keine Bezugnahme auf die Altbasis außer als Abgrenzung |
| Code aus der Altbasis | Ja, zunächst vollständig | Nein (nur Ideen/Muster) | Nein |
| Neues Repository | Ja, Import des gesamten Altbestands nötig, weil das heutige Repository nur Dokumentation enthält | Ja | Ja |

### 4.2 Bewertung

Skala: **günstig** / **mittel** / **ungünstig** / **OFFEN**. Die Bewertungen sind fachliche Einschätzungen des Analysten (ABGELEITET); belastbare Aufwandszahlen liegen nicht vor.

| Kriterium | **A** | **B** | **C** |
|---|---|---|---|
| **Aufwand** | **Ungünstig.** Das Funktionale (Prüfobjekte, Checks, Runner, Evidenz, Berichte) existiert in der Altbasis nicht und ist in jeder Option neu zu entwickeln. Hinzu kommen bei A: Import und Quarantäne, Rückbau von 30 Dateien (5 346 Zeilen), Nachweis der Entfernung, Bereinigung von Bezeichnern, Überarbeitung der verbleibenden Gerüste. Der mögliche Gewinn ist auf rund 296 Zeilen Gerüst begrenzt (Abschnitt 2). Absolute Zahlen: **OFFEN** (keine Architektur, keine Aufwandsbasis) | **Mittel.** Neuaufbau plus kleine Prüfung der sechs Bereiche; die Vorarbeit (Phase 1–3) ist weitgehend erledigt | **Mittel bis günstig.** Geringster Zusatzaufwand; Verzicht auf die Prüfung der Altbasis |
| **Risiken** | **Hoch.** Restbestände ausgeschlossener Fähigkeiten, Historienbelastung, verdeckte Kopplungen, Anknüpfung an den ursprünglichen Zweck (M-02, M-03, SEC-14) | **Niedrig bis mittel.** Risiko eines versehentlichen Übernehmens bleibt gering, wenn das Verbot des Kopierens in der Governance verankert ist | **Niedrig.** Verbleibendes Risiko liegt in neuen Eigenfehlern, kontrollierbar durch Qualitätssicherung |
| **Wartbarkeit** | **Ungünstig.** Altstruktur: zentraler Hub, verstreute Schlüssel, keine Tests, Manifest V2 (SEC-02, SEC-10) | **Günstig.** Architektur und Tests von Anfang an | **Günstig.** wie B |
| **Lizenzlage** | **Ungünstig.** Keine Lizenz, zwei Vorlageprojekte, Fremdassets, Fremd-Copyright-Text; Voraussetzung wäre positive Klärung von H-1 bis H-4 (alles OFFEN) | **Günstig bis mittel.** Keine Codeübernahme; offen ist nur die rechtliche Bestätigung, dass allgemeine Muster frei sind (3.4) | **Günstig.** Keine Anknüpfung |
| **Sicherheitsniveau** | **Ungünstig bis mittel.** Alle Befunde SEC-01 bis SEC-15 sind zunächst vorhanden; Beseitigung und Nachweis aufwändig | **Günstig.** Vorgaben aus den Anforderungen von Beginn an umsetzbar | **Günstig.** wie B |
| **Voraussetzungen** | Positive Lizenz-/Herkunftsklärung (H-1 bis H-4); Compliance-Freigabe (H-6); Quarantäne-Import in gesondertes Repository; Rückbau- und Nachweisverfahren; Entscheidung über die Plattform (MV2→?) | Rechtliche Bestätigung für Muster-Übernahme (optional); Freigabe der Prüfung der sechs Bereiche; Governance (G1) | Governance (G1); Anforderungsfreigabe (G2); Zielbrowser (ED-2) |
| **Eignung für das Projektziel** | Fraglich | Gegeben, sofern Voraussetzungen erfüllt | Gegeben, sofern Voraussetzungen erfüllt |

### 4.3 Zusätzliche Dimensionen (zur Information)

| Dimension | A | B | C |
|---|---|---|---|
| Entwicklungsdauer | **OFFEN.** Beginn von der Lizenzklärung abhängig, die außerhalb des Projekts liegt; zusätzlich Rückbauphase vor dem Aufbau | **OFFEN.** Beginn nach G1/G2; keine Abhängigkeit von Fremdrechten | **OFFEN.** wie B; geringster Vorlauf |
| Langfristige Erweiterbarkeit | Ungünstig (Strukturen folgen dem ausgeschlossenen Zweck) | Günstig | Günstig |
| Produktidentität | Schwierig: Bezeichner und Struktur tragen den Namen der Altbasis | Eigenständig | Eigenständig |
| Nachweisbarkeit der Ausschlusskonformität | Aufwändig (Abwesenheit nachweisen) | Einfach (Negativtests auf neuem Code) | Einfach |

---

## 5. Empfehlung des Analysten

Die Entscheidung liegt bei der Projektleitung (Gate G0). Die Empfehlung des Analysten lautet:

1. **Option B**, hilfsweise **Option C**. Beide Optionen führen zu einem neuen Repository ohne Code der Altbasis. B ergänzt C um die dokumentierte, unabhängige Prüfung der sechs Bereiche, die in Phase 1–4 bereits weitgehend geleistet ist. Die Wissensübernahme besteht aus Anforderungen, nicht aus Code.
2. **Option A wird nicht empfohlen.** Ihr Vorteil ist auf rund 4 % des Bestands (Gerüst) begrenzt, die Voraussetzungen (Lizenz, Compliance, Quarantäne, Rückbau, Nachweis) sind teuer und teilweise nicht in der Hand des Projekts, und die Produktidentität bleibt belastet. Falls die Projektleitung A dennoch verfolgen möchte, ist sie erst nach positiver Klärung von H-1 bis H-4 und H-6 zu bewerten (dann wäre diese Matrix zu aktualisieren).
3. **Kein Gerüst und keine Datei der Altbasis wird als wiederverwendbar eingestuft,** solange die Lizenz- und Herkunftsfragen offen sind (Sperrliste 3.4).

### Was die Empfehlung ändern würde
- Eine positive, schriftlich belegte Lizenz- und Herkunftsklärung für V-1 bis V-3 **und** eine Compliance-Freigabe (H-6) würden Teile der Kategorie „Gerüst" (Build- und Manifest-Struktur) einer Übernahme zugänglich machen. Der Vorteil bliebe auf die genannten rund 296 Zeilen begrenzt.
- Eine belastbare Aufwandsabschätzung, die einen deutlichen Vorteil von A zeigt, wäre nach heutiger Kenntnis nicht zu erwarten. Sie ist erst nach dem Architekturentwurf möglich (**OFFEN**).

---

## 6. Offene Punkte dieser Matrix

| Nr. | Punkt | Grund | Status |
|---|---|---|---|
| OP-1 | Lizenzen von V-1, V-2 | Originalrepositories nicht eingesehen; Websuche ohne Ergebnis | OFFEN |
| OP-2 | Rechteinhaber und Weitergabebedingungen von OEB | Im Paket nicht angegeben | OFFEN |
| OP-3 | Herkunft von Symbolen, Vorschauen, Logos | Nicht dokumentiert | OFFEN |
| OP-4 | Rechtliche Bestätigung zur Übernahme allgemeiner Muster | Aufgabe der Rechtsabteilung | OFFEN |
| OP-5 | Lizenzprüfung der Build-Abhängigkeiten | Nur Lockfile-Angaben vorhanden | NICHT GEPRÜFT |
| OP-6 | Aufwands- und Terminschätzung | Keine Architektur, keine Basis | OFFEN |
| OP-7 | Bewertung von `popup.css` (914 Z.) | Nicht geprüft; für B/C ohnehin ohne Bedeutung | NICHT GEPRÜFT |
| OP-8 | Compliance-Bewertung (H-6) | Entscheidung von Projektleitung und Rechtsabteilung | OFFEN |
