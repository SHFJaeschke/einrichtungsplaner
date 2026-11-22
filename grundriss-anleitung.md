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
- `roof` (optional, nur für Dachwohnungen): Dachschrägen, siehe Abschnitt "Dachwohnung" unten. Liegt kein Raum
  unter einer Schräge, lässt du `roof` ganz weg.
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

## Dachwohnung: Dachschrägen mit `roof`

Nur nötig, wenn ein Teil der Wohnung unter einer geraden Dachschräge liegt. Die Decke ist dann dort
`Kniestock + Abstand von der Trauflinie × tan(Dachneigung)`, höchstens aber die Raumhöhe `height`; überall sonst
gilt die Raumhöhe. Der Planer kürzt Wände und Einbauten auf diese Höhe, schliesst in der Rendering-Ansicht die
schräge Decke und färbt den Boden dort dunkler, wo man weniger als 2,0 m und weniger als 1,5 m Raumhöhe hat.

```json
"roof": {
  "knee": 1.1,
  "pitch": 35,
  "slopes": [
    { "eave": [[0, 5.6], [8.8, 5.6]], "area": [[-0.2, 2.8], [9.0, 2.8], [9.0, 5.8], [-0.2, 5.8]] }
  ],
  "windows": [ [[2.0, 4.0], [2.8, 4.0], [2.8, 5.0], [2.0, 5.0]] ]
}
```

- `knee`: Kniestock in Metern (0 bis 3), also die Raumhöhe an der niedrigen Wand, bei der die Schräge beginnt.
  Üblich sind 0,8 bis 1,4 m; beginnt die Schräge am Boden, ist es 0.
- `pitch`: Dachneigung in Grad (5 bis 75), üblich 30 bis 45. Beide Werte gelten für alle Schrägen der Wohnung und
  lassen sich später im Planer unter "Wohnung" nachstellen. Stehen sie nicht auf dem Plan, schätze sie und sage
  das im Feld `note`. Zeigt der Plan eine gestrichelte Höhenlinie (zum Beispiel "1.50 m"), rechne zurück:
  `knee = 1.5 − Abstand der Linie von der Traufwand × tan(pitch)`.
- `slopes`: eine bis vier Dachschrägen. Jede hat
  - `eave`, die Trauflinie: zwei Punkte auf der **Innenkante der niedrigen Aussenwand**, bei der diese Schräge
    beginnt (sie läuft parallel zum First). Die Linie darf kürzer oder länger sein als die Wand, es zählt nur ihre
    Lage und Richtung.
  - `area`, den Bereich, in dem diese Schräge gilt: ein Vieleck (3 bis 40 Ecken) von der Traufwand bis zum First,
    seitlich bis zu den Giebelwänden. Zeichne es an den Aussenseiten etwa 20 cm über die Wände hinaus, damit auch
    die Aussenwände gekürzt werden. Bereiche mit voller Raumhöhe (Gauben, Lukarnen, der Teil hinter dem First)
    gehören nicht hinein: spare sie aus.
- Satteldach mit First über der Wohnung: zwei Schrägen, je eine Trauflinie an den beiden niedrigen Wänden, die
  beiden Bereiche stossen am First aneinander. Die Bereiche sollen sich nicht überlappen (wo sie es doch tun, gilt
  die tiefere Decke).
- `windows` (optional, höchstens 8): Dachfenster als Vielecke in der Draufsicht. Dort bleibt die schräge Decke in
  der Rendering-Ansicht offen, damit Tageslicht hereinfällt.
- So erkennst du Schrägen auf einem Plan: der Vermerk "DG" oder "Dachgeschoss", gestrichelte Linien parallel zu
  einer Aussenwand (Höhenlinien oder der First), Dachfenster als Rechtecke mitten im Raum, Flächenangaben mit
  Zusatz wie "ab 1.50 m".

## Was die Datei nicht kann

Treppen, mehrere Geschosse, gewölbte Decken und Schrägen mit unterschiedlicher Neigung oder unterschiedlichem
Kniestock in derselben Wohnung sind in diesem Format nicht vorgesehen.
