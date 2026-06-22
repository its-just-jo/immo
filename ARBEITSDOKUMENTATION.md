# Arbeitsdokumentation – Immobilien-Kalkulator

> Stand: 22.06.2026 · Branch: `claude/next-steps-markdown-n5fb00`
> Diese Datei dokumentiert den Umsetzungsstand, getroffene Entscheidungen und offene Punkte,
> damit die Arbeit später nahtlos fortgesetzt werden kann.

---

## 1. Was bereits umgesetzt ist

Die komplette Anwendung liegt in **einer einzigen Datei: `index.html`** – Vanilla JS, kein Build,
keine externen Abhängigkeiten, läuft per Doppelklick im Browser (auch iPhone/Safari) und offline.
Genau wie in den Anforderungen (Kapitel 2) gefordert.

### Vollständig funktionsfähig

| Bereich | Status | Details |
|---------|--------|---------|
| **Projektverwaltung** | ✅ | Projekte anlegen, Dashboard-Übersicht (Name · Phase · Ampel · Datum), Speichern in `localStorage`, Duplizieren (Szenario-Klon), Löschen. |
| **Wizard Schritt 0 – Start** | ✅ | Begrüßung, Projektname + Notizfeld. |
| **Wizard Schritt 1 – Objekt** | ✅ | Kaufpreis, Wohnfläche, Bundesland, **Region** (für Preisvergleich), Baujahr, Energieklasse A–G, WEG ja/nein. Auto-Hinweisbox „Kaufpreis/m²" mit grün/gelb/rot je nach Regionsspanne. |
| **Wizard Schritt 2 – Kaufnebenkosten** | ✅ | Vollautomatische Berechnung (GrESt BY 3,5 % / BW 5,0 %, Notar 1,8 %, Makler 3,57 %), BW-Mehrkosten-Hinweis, EK-Hinweis. Makler an/aus + Provision anpassbar. |
| **Wizard Schritt 3 – Mieteinnahmen** | ✅ | Kaltmiete, Mietpreisbremse (ja/nein/weiß nicht) mit Hinweistext, Sofortberechnung Bruttomietrendite + Kaufpreisfaktor + Preis/m² mit Ampeln. |
| **Wizard Schritt 4 – Finanzierung** | ✅ | EK-Vorschlag (20 % auto, überschreibbar), Sollzins, Zinsbindung, Tilgung (Schieberegler 1–4 %). Darlehen, LTV (Ampel), Annuität mit Zins-/Tilgungsanteil, Restschuld, 3 Zinsänderungsszenarien, LTV-Warnung. |
| **Wizard Schritt 5 – Kosten & Steuern** | ✅ | Hausverwaltung, Instandhaltung (Peterssche Formel, auto + überschreibbar), Mietausfallwagnis, nicht umlagefähige NK, WEG-Rücklage mit Warnhinweis, Grenzsteuersatz, Gebäudeanteil (Schieberegler). Auto-Steuerrechnung (AfA, Zinsen, Werbungskosten, Steuereffekt). |
| **Wizard Schritt 6 – Auswertung** | ✅ | Monatlicher Cashflow vor/nach Steuern, Kennzahlen-Tableau (7 Kennzahlen mit Ampel + Grenzwerten), **10 Risikokarten** (aufklappbar), Veto-Signal, Anbieter-Check (6 Häkchen, Gutachter-Hinweis ab 2). |
| **Mehrjahres-Simulation** | ✅ | In Schritt 6: 20-Jahres-Projektion mit Tilgungsverlauf, Mietsteigerung und Wertentwicklung (Slider für beide Raten). Schnappschüsse Jahr 1/5/10/15/20: Miete, Cashflow n. St., Restschuld, Objektwert, Immo-Nettovermögen + kumulierter Cashflow. |
| **Objekt-Vergleich** | ✅ | Eigene Vergleichsansicht (Dashboard-Button „Objekte vergleichen"): zwei oder mehr Projekte als Kennzahlen-Tabelle nebeneinander, je Zeile wird der günstigere Wert markiert (✓). |
| **Lebenszyklus-Checkliste** | ✅ | Alle 9 Phasen + Meta-Ebene aus Kapitel 4. Jeder Punkt mit Pflicht/Optional-Markierung und 4 Status (`[ ]` offen · `[~]` Arbeit · `[x]` erledigt · `[–]` übersprungen). Status wird gespeichert. Risiko-Ampel der Checkliste kippt, wenn Pflichtpunkte übersprungen werden. |
| **Druck/Export** | ✅ | Print-Stylesheet (`@media print`), „Drucken / als PDF"-Button in Schritt 6. |

### Navigation
- Fortschrittsbalken „Schritt X von 6", Weiter-Button erst aktiv wenn Pflichtfelder gefüllt,
  Zurück jederzeit, Werte bleiben erhalten. Tabs „Kalkulation" / „Projekt-Checkliste".

---

## 2. Architektur der `index.html` (Orientierung im Code)

Der `<script>`-Block ist in klar abgegrenzte Abschnitte gegliedert (jeweils per Kommentarbanner):

1. **`CFG`** – alle Richtwerte/Konstanten an einer Stelle (Steuersätze, AfA-Sätze,
   Herstellungskosten, Regionsspannen). → Hier zukünftige Wertanpassungen vornehmen.
2. **Storage-Layer** – `loadAll/saveAll/createProject/getProject/persist/duplicateProject/deleteProject`,
   Datenmodell `blankInputs()`. Speicher-Key: `immo_projekte_v1`.
3. **Formatierung** – `fmtEUR`, `fmtPct`, `esc`.
4. **Externe Datenquellen** (optional, siehe Kapitel 6) – `loadSettings/saveSettings`,
   Cache (`cacheGet/cacheSet`), API-Clients `apiGeocode/apiOSM/apiDestatis`, Orchestrator
   `lageEnrich`, Bewertung `microScoreFromPOI/lageScore`. Komplett gekapselt, beeinflusst
   `calc` nie direkt.
5. **`calc(inp)`** – **die einzige Stelle mit Finanzformeln.** Gibt ein Objekt mit allen
   abgeleiteten Werten zurück. Jede Anzeige liest nur aus diesem Ergebnis. **Bleibt rein
   synchron und netz-unabhängig.**
6. **Dashboard-Render** (inkl. `projektAmpel` mit gebremstem Lage-Nudge).
7. **Wizard** – `STEP_RENDER[0..6]` (HTML pro Schritt) + `STEP_BIND[0..6]` (Event-Handler).
   Hilfsfunktionen `numField/choiceField/bindNums/bindChoices`. Schritt 1 enthält die
   Lage-Analyse (`runLageEnrich/renderLageBox`), Schritt 6 die Simulation (`renderSim`).
8. **`riskCards()` / `vetoCheck()`** – Risikokarten (10 fix + optional Karte 11 „Mikrolage").
9. **Vergleich** (`showCompare/renderCompare`) und **Einstellungen** (`showSettings/renderSettings`).
10. **`CHECKLIST`** – Datenstruktur aller Phasen + Render.

Datenmodell eines Projekts:
```
{ id, name, notiz, createdAt, updatedAt, step,
  inputs:{ …, adresse, lage:{…}|null },
  checklist:{ "phaseKey.index": status } }
```
Separate Storage-Keys: `immo_projekte_v1` (Projekte), `immo_settings_v1` (API-Keys/Radius),
`immo_apicache_v1` (API-Cache, TTL 30 Tage).

### Hedonische Bewertung (AVM) – `bewerteImmobilie(obj, basispreis, opt)`
Reine, seiteneffektfreie Utility-Funktion neben `calc`/`simulate` (kein eigener UI-Teil,
nur als aufrufbare, per JSDoc typisierte Funktion). Multiplikatives, semi-logarithmisches
Modell: `Gesamtpreis = (Basis-m²-Preis · Produkt aller Faktoren) · Wohnfläche + absolute Zuschläge`.
Alle Faktoren/Fixwerte liegen in `CFG.avm` (Standardobjekt 80 m² / Bj. 1990 / Zustand „Normal" /
Energie D-E = Faktor 1.0). Enthält Größendegression `(Fläche/80)^-0.1`, Alterswertminderung
0,7 %/Jahr mit hartem Cap bei 0.60, kategorische Multiplikatoren (Energie/Zustand/Ausstattung),
Dummies (Balkon, Aufzug ab 3. OG, Lärm > 65 dB) und absolute Zuschläge für Garagen/Stellplätze
(konfigurierbar via `opt`). Rundung erst am Ende, kaufmännisch (`rundeKaufmaennisch`).
Rückgabe: `{ gesamtpreis, quadratmeterpreis, zuschlaege, faktoren }`.

```js
bewerteImmobilie(
  { wohnflaeche:120, baujahr:2015, energieklasse:'B', zustand:'gut',
    ausstattung:'gehoben', balkon:true, aufzug:true, etage:4, garagen:1 },
  5000,                       // regionaler Basis-m²-Preis
  { aktuellesJahr:2026 }      // optional: garagenWert, stellplatzWert, aktuellesJahr
); // -> { gesamtpreis: 759652, quadratmeterpreis: 6205.44, zuschlaege: 15000, faktoren: {...} }
```

---

## 3. Bewusst getroffene Entscheidungen (bitte prüfen)

Die Beispielzahlen in der Anforderungs-MD sind **untereinander nicht konsistent** (verschiedene
Schritte rechnen mit leicht unterschiedlichen Mieten/Basen). Ich habe durchgängig **fachlich
korrekte Formeln** implementiert statt einzelne Beispielzahlen nachzubauen. Validierung mit
dem Beispiel (400.000 € / 80 m² / Bayern): Kaufnebenkosten stimmen exakt (GrESt 14.000,
Notar 7.200, Makler 14.280, gesamt 35.480, Gesamtinvestition 435.480, Darlehen 348.384).

Abweichungen vom Wortlaut der MD – jeweils mit Begründung:

1. **Steuerersparnis** – Die MD rechnet `Grenzsteuersatz × Werbungskosten` (= 8.190 €).
   Das ist steuerlich **zu hoch**, weil die Mieteinnahmen nicht gegengerechnet werden.
   Implementiert ist der korrekte Weg: `steuerlicher Verlust = Jahreskaltmiete − Werbungskosten`,
   Ersparnis = `|Verlust| × Grenzsteuersatz`. Bei Gewinn entsteht entsprechend Steuerlast.
   Die eigene Erklärbox der MD („entsteht, weil du anfangs Verluste machst") stützt diesen Weg.
   → **Entscheidung getroffen (22.06.2026): Die fachlich korrekte Variante bleibt.** Sie wird
   auch in der neuen Mehrjahres-Simulation verwendet (Steuervorteil schmilzt mit sinkenden Zinsen).

2. **Beleihungsauslauf (LTV)** – berechnet als `Darlehen / Kaufpreis` (fachliche Definition).
   Mit dem 20 %-EK-Vorschlag ergibt das im Beispiel ~87 % (nicht die in der MD genannten 80 %,
   die sich auf die Gesamtinvestition bezogen). Der höhere, korrekte Wert löst sinnvollerweise
   die LTV-Warnung aus – passt zur Empfehlung der MD („20 % EK **plus** Kaufnebenkosten").

3. **WEG-Erhaltungsrücklage** – in der MD in Schritt 5 als Kostenzeile gelistet, aber **nicht**
   in der Cashflow-Aufstellung von Schritt 6. Um Doppelzählung zu vermeiden, fließt sie nicht in
   den Cashflow ein, sondern nur in die Risikokarte 6 (Schwellenprüfung). Cashflow entspricht damit
   exakt der Schritt-6-Tabelle der MD.

4. **Region in Schritt 1** – Für die Ampel der Kaufpreis/m²-Box wird eine Stadt/Region benötigt
   (die MD nennt Städte nur im Hinweistext). Ich habe ein optionales Region-Dropdown ergänzt
   (Konstanz/Lindau/Friedrichshafen/München/Allgäu/Sonstige).

5. **Denkmal-AfA** – vereinfacht als 2,5 % auf den Gebäudeanteil. Die echte Denkmal-AfA
   (9 %/7 % auf **Sanierungskosten**) braucht ein separates Sanierungskosten-Feld → siehe offene Punkte.

6. **Herstellungskosten/m² für die Peterssche Formel** – geschätzte Werte je Baualter im `CFG`
   hinterlegt (1.700–2.500 €/m²). Der Eigenanteil ist editierbar; ggf. Werte verfeinern.

---

## 4. Offene Punkte / Backlog (noch nicht umgesetzt)

Diese optionalen Features aus Kapitel 4 sind noch **nicht** funktional (in der Checkliste als
Punkte vorhanden, aber ohne eigene Berechnung):

- [x] **Objekt-Vergleich** – umgesetzt (Vergleichsansicht über Dashboard, mehrere Projekte als Kennzahlen-Tabelle).
- [x] **Mehrjahres-Simulation** (20 Jahre): Tilgungsverlauf, Mietsteigerung, Wertsteigerung – umgesetzt in Schritt 6.
- [ ] **15-%-Grenze-Rechner** (Sanierungskosten 3 Jahre vs. 15 % der Gebäudekosten).
- [ ] **Spekulationsfrist-Rechner** (Kaufdatum → 10-Jahres-Frist) und **3-Objekt-Grenze**.
- [ ] **Anschlussfinanzierung als eigener Rechner** (über die Szenarien in Schritt 4 hinaus).
- [ ] **Forward-Darlehen / Sondertilgung-Szenarien.**
- [ ] **Mietspiegelwert-Eingabe** für präzisere Mietpreisbremsen-Prüfung.
- [ ] **Denkmal-AfA mit Sanierungskosten-Eingabe** (9 %/7 %).
- [ ] **Steuerliche Jahresauswertung / Werbungskosten-Export.**
- [ ] **Projekt-Zusammenfassung** als eigene kompakte Seite (Druck funktioniert bereits über Schritt 6).

### Empfohlene nächste Schritte
Die vier zuvor empfohlenen Schritte sind abgearbeitet:

1. ✅ Entscheidung zu Punkt 3.1 (Steuerersparnis-Methode) – korrekte Variante bestätigt.
2. ✅ Mehrjahres-Simulation umgesetzt (20 Jahre, Schritt 6).
3. ✅ Objekt-Vergleichsansicht umgesetzt (Dashboard → „Objekte vergleichen").
4. ✅ Regionsspannen im `CFG` überarbeitet (Stand Q2/2026, München angehoben,
   Ravensburg + Ulm ergänzt; `CFG.regionenStand` wird im Hinweis angezeigt).
   → Die Werte sind weiterhin **Richtwerte** und sollten vor einer Kaufentscheidung
   gegen den lokalen Gutachterausschuss / BORIS abgeglichen werden.

**Neue Kandidaten für die Weiterarbeit:**
1. 15-%-Grenze- und Spekulationsfrist-Rechner (Phasen 6/9 der Checkliste).
2. Anschlussfinanzierung mit abweichendem Anschlusszins in der Simulation
   (aktuell wird vereinfachend der gleiche Zins über die ganze Laufzeit angenommen).
3. Denkmal-AfA mit separatem Sanierungskosten-Feld.

---

## 5. Externe Datenquellen (optional, Graceful Degradation)

Das Tool bleibt **ohne API-Keys voll funktionsfähig** und offline nutzbar. APIs reichern nur
optional den Lage-Score und den Makro-Trend an. Konfiguration unter **⚙︎ Einstellungen** (Dashboard).

### Daten-Flow (`lageEnrich`)
Ausgelöst per Button **„📍 Lage online prüfen"** in Schritt 1. Jeder Schritt ist optional und
mit `try/catch` + Timeout (`AbortController`, 12 s) gekapselt; Fehler landen als Hinweis in
`inputs.lage.hinweise`, ohne den Rest zu stoppen.

| Schritt | Quelle | Key | Ergebnis | Fallback |
|---------|--------|-----|----------|----------|
| **A** | OpenCage Geocoding | ja | lat/lon, Landkreis, ggf. Regionalschlüssel (AGS) | übersprungen → nur manuelle Eingaben |
| **B** | Overpass / OpenStreetMap | nein | POI-Zählung (Supermärkte, ÖPNV, Schulen/Kitas) → **Mikrolage-Score 0–100** | kein Score |
| **C** | Destatis GENESIS *(experimentell)* | ja | Bevölkerungstrend → **Makro-Modifier** (max. ±0,5 %) | kein Trend (oft CORS-bedingt) |

### Einfluss auf die Bewertung (bewusst „gebremst")
- **Mikrolage-Score** → eigene KPI/Risikokarte 11 und **nudged die Gesamtampel um höchstens
  eine Stufe** (`projektAmpel`).
- **Makro-Trend** → **Vorschlag** für die Wertsteigerung in der Simulation (per „übernehmen"-Link),
  greift nie automatisch in `calc` ein.

### Caching
`immo_apicache_v1` in `localStorage`, **TTL 30 Tage**. Vor jedem API-Call wird zuerst der Cache
geprüft (Schlüssel: normalisierte Adresse / `lat,lon,radius` / AGS) → spart Credits, auch
projektübergreifend. „Cache leeren" in den Einstellungen.

### Sicherheit & Einschränkungen (bewusst so)
- **API-Keys liegen unverschlüsselt in `localStorage`** und sind auf dem Gerät einsehbar – nur für
  ein privates, lokales Tool gedacht. Hinweis ist im UI sichtbar.
- **Apple-Schlüsselbund-Kompatibilität:** Je Anbieter ein eigenes `<form>` mit `username`-Feld
  (Dienstname, `autocomplete="username"`) + Key-Feld `type="password"`
  `autocomplete="current-password"` und `name="api-key"`/`id="api-key-<dienst>"`. `current-password`
  (statt `new-password`) verhindert den Passwortgenerator und triggert Autofill.
- **CORS:** OpenCage und Overpass senden `Access-Control-Allow-Origin: *` → laufen direkt im
  Browser. **Destatis** unterstützt CORS nicht zuverlässig → Schritt C kann im reinen Browser-
  Kontext scheitern (Trend bleibt leer). Parsing in `parseDestatisTrend` ist defensiv und
  müsste für eine produktive Nutzung an die konkrete GENESIS-Tabelle angepasst werden.

---

## 6. Testen / Ausführen

- **Lokal:** `index.html` doppelklicken – fertig. Daten liegen im `localStorage` des Browsers.
- **JS-Syntax geprüft:** `node --check` über den extrahierten Script-Block – OK.
- **Rechenkern verifiziert:** Beispiel 400.000 € / 80 m² / Bayern / 1.200 € → Kaufnebenkosten
  exakt wie in der MD.
- **API-Schicht (gemockt) verifiziert:** Cache-TTL, Mikrolage-Score (61 bzw. 100 bei Sättigung),
  Enrich-Flow (geo+osm), Graceful Degradation ohne Keys und der gebremste Ampel-Nudge wurden
  mit Stub-`fetch`/`localStorage` getestet.

Kein npm, kein Server, kein Build nötig.
