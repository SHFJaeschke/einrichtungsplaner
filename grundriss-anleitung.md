# Grundriss-Datei für den Einrichtungsplaner

Diese Anleitung ist für dein eigenes Claude (oder einen anderen Assistenten). Lade deinen Grundriss als Bild oder PDF
hoch, füge diese Anleitung dazu und bitte um eine "Grundriss-Datei". Das Ergebnis fügst du im Planer unter
"Wohnung hinzufügen" ein.

## Auftrag an den Assistenten

Erzeuge aus dem Grundriss ein einziges JSON-Objekt nach dem Schema unten, ohne Kommentare und ohne Text darum herum.

- Einheit ist Meter. `x` läuft auf dem Plan nach rechts, `z` nach unten. Lege den Ursprung in eine innere Raumecke.
- Eiche den Massstab an gedruckten Massen oder an den Flächenangaben der Räume und prüfe, dass die Raumflächen
  im Modell auf etwa 2 % stimmen. Nenne im Feld `note` ehrlich, was gemessen und was angenommen ist
  (Raumhöhe, Fensterhöhen).
- `rooms`: jeder Raum als Vieleck seiner **Innen**ecken (`pts`), im Uhrzeigersinn. `kind`: `parquet`, `tile`
  (Bad, Küche mit Platten) oder `balcony`. `area` nur angeben, wenn sie auf dem Plan steht.
- `walls`: jede Wand als Mittellinie von `a` nach `b`, 12 cm dick. Zwischen zwei Räumen liegen also 12 cm Abstand;
  die Mittellinie einer Aussenwand liegt 6 cm ausserhalb der Raumkante. Öffnungen in `ops`, gemessen in Metern
  ab `a`: `{ "f": 1.2, "t": 2.1 }` ist eine Tür, mit `"glass": true` ein Fenster oder eine Glastür
  (`"sill": 0.9` für eine Brüstung, ohne `sill` raumhoch), mit `"door": true` die Wohnungstür.
- `doors` (optional): Türflügel als Plansymbol: Drehpunkt `hinge`, Richtung des geschlossenen Flügels `closed`,
  Richtung des offenen Flügels `open`, Breite `r`.
- `rails` (optional): Balkongeländer als Strecken.
- `fixtures` (optional): Einbauten als Quader (Küchenzeile, Hochschränke, WC, Lavabo, Dusche): Breite `w`, Höhe `h`,
  Tiefe `d`, Mittelpunkt `x`/`z`, Unterkante `y`, Drehung `rot` in Grad, `mat` (`lacquer`, `stone`, `ceramic`,
  `steel`, `wood`), `color`.
- `eyes` (optional): Standpunkte für die Rendering-Ansicht: `pos` `[x, z]`, `look` `[x, z, Höhe]`.
- Keine Möbel einzeichnen; die stellt man im Planer.

## Schema mit Beispiel

```json
{
  "title": "Musterstrasse 1",
  "sub": "2.5 Zimmer, 58 m²",
  "note": "Massstab an den Raumflächen geeicht. Raumhöhe und Fensterhöhen angenommen.",
  "height": 2.5,
  "rooms": [
    { "name": "Wohnen", "kind": "parquet", "pts": [[0, 0], [5.2, 0], [5.2, 5.6], [0, 5.6]], "area": 29.1 },
    { "name": "Zimmer", "kind": "parquet", "pts": [[5.32, 0], [8.8, 0], [8.8, 3.4], [5.32, 3.4]] }
  ],
  "walls": [
    { "a": [-0.06, -0.06], "b": [8.86, -0.06] },
    { "a": [-0.06, -0.06], "b": [-0.06, 5.66], "ops": [{ "f": 1.2, "t": 3.4, "glass": true }] },
    { "a": [5.26, -0.06], "b": [5.26, 5.66], "ops": [{ "f": 2.3, "t": 3.15 }] }
  ],
  "doors": [{ "hinge": [5.32, 2.24], "closed": [0, 1], "open": [1, 0], "r": 0.85 }],
  "rails": [],
  "fixtures": [{ "w": 3.0, "h": 0.9, "d": 0.6, "x": 3.6, "z": 0.3, "mat": "lacquer", "color": "#f6f6f3" }],
  "eyes": [{ "name": "Wohnen", "pos": [4.5, 0.9], "look": [1.4, 4.2, 0.9] }]
}
```

## Was die Datei nicht kann

Dachschrägen, Treppen und unterschiedliche Raumhöhen sind in diesem Format nicht vorgesehen.
