# Projekt: Immobilien-Kalkulator

Single-File-Web-App in **`index.html`** (Vanilla JS, kein Build, keine Abhängigkeiten,
offline lauffähig). Wird über **GitHub Pages** vom `main`-Branch (Root) ausgeliefert.
Begleitdokumentation: `ARBEITSDOKUMENTATION.md`.

## VERBINDLICHE REGEL — „Letzte Änderung"-Stempel (bei jeder Code-Änderung)

Bei **jeder** Änderung am Code (insbesondere an `index.html`) **muss** der Build-/
Versionsstempel aktualisiert werden:

- Konstante **`BUILD_STAMP`** im `<script>`-Kopf von `index.html` auf das **aktuelle
  Datum + Uhrzeit** setzen (Format `TT.MM.JJJJ, HH:MM Uhr`, lokale Zeit — vorher per
  `date "+%d.%m.%Y, %H:%M Uhr"` ermitteln).
- Sie wird im **Footer** als _„Letzte Änderung: …"_ angezeigt (Element `#buildStamp`,
  CSS-Klasse `.buildstamp`: dezent, kursiv, sehr kleine Schrift).
- Zweck: Auf der GitHub-Pages-Seite ist sofort erkennbar, **welcher Stand live ist**
  (hilft, veralteten Browser-Cache zu erkennen → Hard-Reload ⌘/Strg+⇧+R).

Diese Aktualisierung gilt als Teil der Änderung und ist **nicht optional**.

## Weitere Konventionen

- Alle Finanzformeln leben ausschließlich in `calc(inp)`. Externe/asynchrone Daten
  niemals direkt in `calc` einspeisen — Ergebnisse nach `inputs.lage` schreiben.
- Richtwerte/Konstanten zentral in `CFG`.
- Nach jeder Änderung an `index.html` die JS-Syntax prüfen:
  `node -e "..."` extrahiert den `<script>`-Block, dann `node --check`.
- Funktionalität bleibt **ohne API-Keys** voll erhalten (Graceful Degradation).
- Standard-Referenzbeispiel zur Regressionskontrolle: 400.000 € / 80 m² / Bayern /
  1.200 € → Kaufnebenkosten 35.480 €, Darlehen 348.384 €.
