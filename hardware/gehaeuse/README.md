# Gehaeuse fuer RC-Multiswitch-ESP32-S3 (Platine v0.4)

3D-druckbares Gehaeuse aus Unterteil und Deckel. Die Masse stammen aus `hardware/RC-Multiswitch-ESP32-S3.kicad_pcb` (Platine 66,52 x 50,6 mm), die Bauteilhoehen sind Annahmen (siehe unten).

![Gehaeuse offen](gehaeuse_offen.png)

## Dateien
| Datei | Inhalt |
|---|---|
| `Gehaeuse_Unterteil.stl` | Unterteil, druckfertig (Boden aufs Bett) |
| `Gehaeuse_Deckel.stl` | Deckel, druckfertig (schon gedreht, Oberseite aufs Bett) |
| `Gehaeuse_Unterteil.step`, `Gehaeuse_Deckel.step` | zum Weiterbearbeiten in Fusion 360, FreeCAD oder KiCad (Deckel in Einbaulage) |
| `gehaeuse.py` | Quelle (CadQuery), alle Masse als Parameter oben in der Datei |
| `Gehaeuse3D_Vorschau.html` | drehbare 3D-Vorschau mit Platine (im Browser oeffnen) |

## Masse
- Aussen 71,9 x 61,6 x 19,6 mm (mit Deckel), Wand 2,2 mm, Boden und Deckel 2,0 mm.
- 3 mm Platz unter der Platine (Loetstellen), 11 mm ueber der Platine.

## Annahmen
- Der Waveshare ESP32-S3-Zero ist direkt auf einer Stiftleiste eingeloetet (Unterkante 2,5 mm ueber der Platine, bestaetigt). Danach richtet sich der USB-Ausschnitt.
- Hoechste Teile: J1 (8,5 mm) und der ULN2803A im Sockel (ca. 8 mm). JST-PH-Buchsen liegend, ca. 6 mm hoch.
- Lage von BOOT, RESET und RGB-LED auf dem S3-Zero ist nicht in der KiCad-Datei, deshalb ein Fenster ueber dem ganzen Modul.

## Oeffnungen
- Links: USB-C des S3-Zero (aussen mit Tasche fuer dicke Stecker), CN1 UART A, CN2 Last 5-12 V.
- Hinten: J5 UART B und J6 UART C (eine gemeinsame Oeffnung).
- Deckel: Ausschnitt ueber J1 (Ausgaenge 2x8, Stecker von oben), Fenster ueber dem S3-Zero, 8 Lichtloecher fuer die Kontroll-LEDs, 4 Lueftungsschlitze ueber dem ULN2803A, vertiefte Beschriftung.

## Drucken und Montieren
- PETG oder PLA, 0,2 mm Schicht, 3 Wandlinien, 20 % Fuellung, keine Stuetzen noetig.
- Platine: Sie hat keine Montageloecher. Sie liegt ringsum auf einem 1,2 mm breiten Rand und 2 Stuetzen und wird vom Deckel mit 10 kleinen Rippen am Rand festgedrueckt. Keine Schrauben durch die Platine.
- Deckel: 4 x Blechschraube 2,5 x 8 (Senkkopf) in die Dome in der vorderen und hinteren Wand.
- Einbau ins Modell: 4 Laschen am Boden mit 3,4-mm-Loch fuer Schrauben oder Kabelbinder.
- Noch nicht gedruckt und nicht am echten Aufbau geprueft.

## Aendern
Masse in `gehaeuse.py` anpassen und neu erzeugen (Python 3 mit `pip install cadquery`):
```
python gehaeuse.py export
```
Die Dateien landen im Ordner `export`. Mit `--pruefen` wird zusaetzlich die Platine als vereinfachtes Modell erzeugt und auf Ueberschneidungen mit dem Gehaeuse geprueft.
