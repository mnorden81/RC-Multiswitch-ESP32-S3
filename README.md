# RC-Multiswitch-ESP32-S3

RC-Multiswitch mit 8 Schaltkanaelen auf einem Waveshare ESP32-S3-Zero.

Frueher `CRSF-Multiswitch-ESP32-S3`. Der neue Name ist neutral, weil der Multiswitch neben CRSF auch SBUS (UART A) als Eingang versteht.

Gleiche Funktion wie die C3-Variante, aber mit dem ESP32-S3-Zero und einem dritten UART. Das PCB wurde vom C3-Layout uebernommen, die Leitungen am ESP32 sind neu verlegt (DRC steht noch aus).

Dokumente:
- [docs/Projektbeschreibung.docx](docs/Projektbeschreibung.docx): Projektbeschreibung
- [docs/Hardware-Beschreibung_C3_S3.docx](docs/Hardware-Beschreibung_C3_S3.docx): Hardware-Beschreibung C3 und S3 (Stecker, Kanalzuordnung, Stueckliste, Dimensionierung)
- [docs/Abnahmepruefung.md](docs/Abnahmepruefung.md): Abnahmepruefung (Pruefliste fuer ein fertig gebautes Modul)
- [docs/Firmware-Spezifikation.md](docs/Firmware-Spezifikation.md) (auch als DOCX): Firmware-Spezifikation (Basis V3, Router fuer Modulketten, BOOT-Taster, Status-LED)

## Merkmale
- 8 Schaltkanaele ueber TBD62083APG (negativ) oder TBD62783APG (positiv), Kontroll-LED je Kanal
- UART A (Eingang), UART B und UART C mit waehlbarer Rolle: Modulkette (CRSF), S.Port-LiPo-Sensoren, GPS oder Raw, je mit eigener Baudrate
- Versorgung des ESP32 aus +5 V des Empfaengers, Last getrennt mit 5-12 V
- Eigener THT-Footprint fuer den S3-Zero (nur 2x9 Header)

## Hardware (v0.4 (PCB vom C3 uebertragen))
Board: Waveshare ESP32-S3-Zero

- Lastspannung 5-12 V (VLOAD) an CN2 (JST PH 2), TVS-Diode P6KE18A als Schutz.
- Treiber TBD62083APG (DMOS, Nachfolger des ULN2803A, gleiche Pinbelegung; ein ULN2803A passt weiterhin in den Sockel), 8 Kanaele, max. 500 mA je Kanal, Summe aller Kanaele ca. 1,5 A. Spannungsabfall am Ausgang nur ca. 0,2-0,4 V statt ca. 1 V beim ULN2803A. Positiv schaltende Variante mit TBD62783APG siehe [hardware/README-hardware.md](hardware/README-hardware.md).
- R9-R16 (THT 1/4 W) zwischen Treiber-Ausgang und Stiftleiste J1: kein Wert vorgegeben, der Anwender legt sie passend zu Verbraucher und Lastspannung aus.
- R1-R8 und LED1-LED8 (3 mm) als Kontroll-LED je Kanal, vom Steuersignal Sn gegen GND. Werte legt der Anwender fest, eine Empfehlung ist auf der Platine aufgedruckt.
- Keine Pulldowns und keine Widerstaende in den UART-Leitungen. Der ESP32 wird aus +5 V von CN1 (BEC/Empfaenger) versorgt.

### Belegung
| Funktion | GPIO |
|---|---|
| S0..S7 | IO11, IO4, IO5, IO6, IO10, IO9, IO8, IO7 |
| UART A (CN1, Eingang) | TX IO43, RX IO44 (UART0) |
| UART B (J5) | TX IO13, RX IO12 |
| UART C (J6) | TX IO2, RX IO1 |
| Frei | IO3 (Strapping), IO14-IO16 (Loetpads) |

### Anschluesse
- CN1, J5, J6: UART A/B/C als JST-GH 4 (Footprint BM04B-GHS-TBT)
- CN2 Last 5-12 V (JST PH 2, max. ca. 2 A je Kontakt)
- J1 Ausgaenge S0..S7

### Hinweise
- UART A liegt auf UART0: beim Booten gibt das ROM ein Log auf IO43 aus.
- Empfehlung LED-Vorwiderstaende R1-R8 bei 3,3 V (ca. 5 mA, auch auf der Platine aufgedruckt): Rot/Gelb 270 R, Gruen 220 R, Blau/Weiss 56 R.
- Footprint ESP32-S3-Zero_THT: 2x9, 2,54 mm, Reihenabstand 15,24 mm. Vor dem Layout 1:1 am echten Board pruefen.

## Firmware (v4.08)
- Arduino-Sketch `firmware/RC_Multiswitch`, gleicher Quellcode fuer C3 und S3 (Board wird automatisch erkannt). Basis: MultiSwitch_ESP32_V3 v3.12.
- CRSF auf UART A (Werkseinstellung ab v4.07), wahlweise SBUS. UART B und UART C per Lua oder Web-UI auf Aus, CRSF (Modulkette), S.Port, GPS oder Raw einstellbar, mit eigener Baudrate je UART. Beim GPS sucht die Baudrate „Auto“ (Vorgabe) die Rate selbst und stellt das GPS auf 115200 Baud und 10 Hz um, wie bei Betaflight. In der Modulkette ist jedes Modul mit eigener Adresse ueber Lua einzeln einstellbar.
- Gemeinsame Basis aller ESP32-Module seit v4.05: `firmware/RC_Multiswitch/src/RC_Common/` (siehe deren README).
- BOOT-Taster 3 s: WLAN-AP ein/aus. Status-LED nur beim S3.
- Details und Bauanleitung: [firmware/README.md](firmware/README.md).

## Firmware flashen

Es gibt drei Wege. Ein neues Modul wird einmal per USB geflasht (Weg 1 oder Weg 2), danach geht jedes Update bequem per WLAN (Weg 3).

| Weg | Braucht | Einstellungen im Modul |
|---|---|---|
| 1. Fertige Datei aus GitHub + Web-Flasher | nur Chrome oder Edge | werden **gelöscht** |
| 2. Arduino IDE | Arduino IDE 2.x | bleiben erhalten |
| 3. Update per WLAN | Handy oder PC mit WLAN | bleiben erhalten |

Der Sketch ist für C3 und S3 derselbe, das Board wird automatisch erkannt. Für dieses Repo (Waveshare ESP32-S3-Zero) gilt das Artefakt `RC_Multiswitch-S3` und die Einstellungen unten.

### Weg 1: Fertige Firmware aus GitHub (ohne Arduino IDE)

GitHub baut die Firmware bei jeder Änderung automatisch (Workflow „Firmware bauen“).

1. Auf GitHub angemeldet sein (sonst gibt es keinen Download-Knopf).
2. In diesem Repo oben auf **Actions** klicken, links den Workflow **Firmware bauen** wählen.
3. Den obersten Lauf mit grünem Haken öffnen (Branch `main`).
4. Ganz unten unter **Artifacts** **RC_Multiswitch-S3** herunterladen und die ZIP-Datei entpacken. Darin liegen u. a. `RC_Multiswitch.ino.merged.bin` (Komplett-Image für USB) und `RC_Multiswitch.ino.bin` (nur für Weg 3).
   Steht dort nichts (Artefakte werden nach 90 Tagen gelöscht): rechts oben **Run workflow** → **Run workflow** klicken, nach ca. 3 Minuten ist ein neuer Lauf fertig.
5. ESP32-S3-Zero mit einem USB-C-**Datenkabel** an den PC stecken (kein reines Ladekabel). Ein Treiber ist unter Windows 10/11 nicht nötig.
6. In **Chrome** oder **Edge** die Seite https://espressif.github.io/esptool-js/ öffnen (Espressif-Web-Flasher, Firefox geht nicht).
7. **Connect** klicken und den COM-Port des Moduls wählen. Klappt das nicht: Baudrate oben auf `115200` stellen und das Modul mit der BOOT-Taste in den Flash-Modus bringen (siehe Weg 2, „Wenn das Hochladen nicht klappt“).
8. Unter **Flash Address** `0x0` eintragen, mit **Datei auswählen** die `.merged.bin` wählen, dann **Program** klicken.
9. Wenn unten „Leaving…“ steht, das USB-Kabel kurz abziehen und wieder einstecken. Fertig.

> **Achtung:** Das Komplett-Image überschreibt den ganzen Flash (4 MB). Alle Einstellungen und die Sicherungen im Flash sind danach weg. Vorher in der Weboberfläche die Einstellungen als Datei sichern (Karte **Konfiguration sichern/wiederherstellen** → „Als Datei exportieren“) und hinterher dort wieder importieren. Wer die Einstellungen behalten will, nimmt Weg 2 oder 3.

### Weg 2: Arduino IDE

**Einmalig einrichten**

1. Arduino IDE 2.x installieren (https://www.arduino.cc/en/software).
2. **Datei → Einstellungen → Zusätzliche Boardverwalter-URLs**: `https://espressif.github.io/arduino-esp32/package_esp32_index.json` eintragen.
3. **Werkzeuge → Board → Boardverwalter**: „esp32“ suchen, **esp32 von Espressif Systems** in Version **3.0.7** installieren.
4. **Werkzeuge → Bibliotheken verwalten**: **Bolder Flight Systems SBUS 8.1.4**, **TinyGPSPlus 1.0.3** suchen und in genau dieser Version installieren.
   Alles andere (z. B. `src/RC_Common/`) liegt schon im Sketch-Ordner.

**Flashen**

1. Repo herunterladen: oben auf GitHub **Code → Download ZIP**, entpacken (oder den vorhandenen Ordner nehmen).
2. In der Arduino IDE **Datei → Öffnen** und `firmware/RC_Multiswitch/RC_Multiswitch.ino` wählen.
3. Unter **Werkzeuge** einstellen:

   | Einstellung | Wert |
   |---|---|
   | Board | **ESP32S3 Dev Module** |
   | USB CDC On Boot | **Enabled** (Pflicht, sonst Abbruch mit Meldung) |
   | Flash Size | 4MB (32Mb) |
   | Partition Scheme | **Default 4MB with spiffs (1.2MB APP/1.5MB SPIFFS)** |
   | PSRAM | Disabled |
   | Erase All Flash Before Sketch Upload | **Disabled** (sonst sind die Einstellungen weg) |

4. ESP32-S3-Zero mit einem USB-C-**Datenkabel** an den PC stecken (kein reines Ladekabel). Ein Treiber ist unter Windows 10/11 nicht nötig. Dann unter **Werkzeuge → Port** den neuen COM-Port wählen.
5. Auf den Pfeil **Hochladen** klicken. Am Ende steht „Hard resetting via RTS pin…“.
6. Das Modul startet von selbst neu. Wenn nicht: USB-Kabel kurz abziehen und wieder einstecken.

**Wenn das Hochladen nicht klappt** („Failed to connect“ oder „No serial data received“): USB abziehen, die Taste **BOOT** gedrückt halten, USB wieder einstecken, BOOT loslassen. Das Modul ist jetzt im Flash-Modus. Unter **Werkzeuge → Port** ggf. den neuen COM-Port wählen und noch einmal **Hochladen**. Danach das USB-Kabel einmal abziehen und wieder einstecken.

### Weg 3: Update per WLAN (OTA)

1. Die Datei `RC_Multiswitch.ino.bin` besorgen: aus GitHub (Weg 1, Schritte 1–4) oder in der Arduino IDE mit **Sketch → Kompilierte Binärdatei exportieren** (liegt dann im Sketch-Ordner unter `build/`). Nicht die `.merged.bin` nehmen.
2. WLAN des Moduls einschalten: BOOT-Taste 3 s halten (oder Lua „WLAN“ bzw. automatisch nach 60 s ohne RC-Signal). Name **MultiSwitch**, Passwort `123456789` (falls nicht geändert).
3. Mit Handy oder PC in dieses WLAN gehen und im Browser http://192.168.1.1 öffnen.
4. Reiter **WiFi**, Karte **Firmware-Update (OTA)**: die `.bin`-Datei auswählen und hochladen. Das Modul startet danach neu, die Einstellungen bleiben erhalten.

Schlägt das Update fehl, läuft die alte Firmware einfach weiter. Das Modul beim Update am besten über den Empfänger/BEC versorgen, nicht nur über einen schwachen USB-Anschluss.

## Ordner
- `hardware/`: KiCad-Projekt (Schaltplan, PCB, PDF, eigene Footprints, fp-lib-table). Entwurfsnotizen in [hardware/README-hardware.md](hardware/README-hardware.md). 3D-druckbares Gehaeuse in [hardware/gehaeuse/](hardware/gehaeuse/README.md).
- `firmware/`: Firmware v4.08 (Arduino-Sketch).
- `docs/`: Projektbeschreibung, Hardware-Beschreibung und Firmware-Spezifikation.

## Stand
- Hardware: Schaltplan fertig, PCB vom C3 uebernommen und an U1 neu verlegt. DRC in KiCad steht noch aus, J6 nur vorlaeufig platziert.
- Firmware: v4.08 kompiliert fuer C3 und S3, noch nicht auf Hardware getestet.

## Offene Punkte
- DRC in KiCad ausfuehren (an U1 sind laut PCB-Datei alle Anschluesse verlegt)
- Platzierung von J6 (UART C) festlegen
- Footprint am echten Board pruefen
- Firmware auf Hardware testen (Einzelmodul und Kette mit 2-3 Modulen)
