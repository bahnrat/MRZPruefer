# MRZ-Prüfer

Kleine Android-App, die die maschinenlesbare Zone (MRZ) von Ausweisen, Pässen und
Visa nach ICAO 9303 per Kamera liest und prüft. Komplett offline: Das
Texterkennungsmodell (Google ML Kit) ist in der App enthalten, die
Internet-Berechtigung ist im Manifest ausdrücklich entfernt.

## Formate

| Format | Zeilen | Typische Dokumente |
|---|---|---|
| TD1 | 3 × 30 | Personalausweise, ID-Karten, Aufenthaltstitel |
| TD2 | 2 × 36 | ältere Ausweise (z. B. deutscher Ausweis bis 2010), Visum Typ B |
| TD3 | 2 × 44 | Reisepässe, Visum Typ A |

Das Format wird beim Scannen und bei der Prüfung automatisch erkannt.

## Was geprüft wird

Für alle Dokumente:
- alle Prüfziffern: Dokumentnummer, Geburtsdatum, Ablaufdatum, Gesamtprüfziffer,
  beim Pass die persönliche Nummer (Visa haben keine Gesamtprüfziffer)
- verlängerte Dokumentnummern (mehr als 9 Zeichen, Rest im optionalen Feld)
- Dokumentart passend zum Format (TD1: A/C/I, TD2: A/C/I/P/V, TD3: P/V)
- Länder-Codes, Geschlechtsfeld, Geburts- und Ablaufdatum, Aufbau des Namensfelds

Zusätzlich bei deutschen Ausweisen und Pässen (Aussteller D):
- Dokumentnummer nur aus 0–9, C F G H J K L M N P R T V W X Y Z
- Staatsangehörigkeit D, höchstens 10 Jahre gültig
- Personalausweis: leere Füllbereiche bzw. Dokumentversion (z. B. 2108)

## Bedienung

1. MRZ in den Rahmen halten. Sobald zwei Kamerabilder hintereinander dieselbe
   stimmige MRZ liefern, öffnet sich das Ergebnis.
2. „Übernehmen“ holt die zuletzt gelesene (fehlerhafte) MRZ zum Korrigieren,
   „Eintippen“ öffnet die leere Eingabe.
3. Bei der Eingabe Dokumentart wählen. Ergänzt wird beim Verlassen eines Feldes:
   - Deutscher Personalausweis: nur Nummer, Ziffern und Name tippen
   - Kopf-/Namenszeilen: „P D Mustermann, Erika“ → `P<D<<MUSTERMANN<<ERIKA<<<…`
   - Datenzeilen: abtippen, die lange `<`-Kette darf ein einzelnes Leerzeichen sein

## Bauen

Voraussetzung: Android Studio (Ladybug oder neuer) bzw. JDK 17 + Android SDK 34.

- **Android Studio:** Ordner öffnen, Gradle-Sync abwarten, „Run“ aufs angeschlossene Handy.
- **Kommandozeile:** `./gradlew assembleDebug` → `app/build/outputs/apk/debug/app-debug.apk`
- **Tests der Prüflogik:** `./gradlew test`
-  Alternativ APK erstellen (Nutze dafür möglichst Deinen eigenen Signaturschlüssel), aufs Smartphone kopieren und dort installieren (bei der Installation wird die App durch "Google Play Protect auf Schadcode geprüft).
- - Tipp: Mindestens 1 mal im Jahr die App aktualisieren (Projekt in Android-Studio öffnen, App-Version erhöhen, Bibliotheken aktualisieren. Dann neu kompilieren, auf Smartphone kopieren und installieren/aktualisieren).
- Mindestens für Android 8.0 (API 26) einstellen. Keine niedere Version wählen.

## Aufbau

- `Mrz.java` – die gesamte Prüflogik (TD1/TD2/TD3), reines Java ohne Android, mit Unit-Tests
- `MainActivity.java` – Kamera (CameraX) und Texterkennung (ML Kit)
- `ResultActivity.java` – Ergebnis, Abgleichdaten und manuelle Eingabe

## Datenschutz

Die App ist Datenschutzkonform. Sie benötigt lediglich Zugriff auf die Kamera.

- Kein Internetzugriff
- Kein Zugriff auf WLAN und Bluetooth
- Bildschirmfotografie (Screenshoot) und Bildschirmaufnahme gesperrt (Gilt auch bei Verbindung mit PC, z.B. ++Samsung LinkToWindows**, Das Bild sollte dort schwarz erscheinen)
- Datensicherung (Android Backup) gesperrt
- Keine Datenspeicherung (Eingaben werden temporär gespeichert und beim Beenden verworfen)
- Es werden keine Daten gesendet
- - Hinweis: Android kann Daten zur App-Nutzung (z.B. Anzahl der Aufrufe oder Absturzdaten) sammeln und an Google senden. Um dies zu verhindern, kann man über die Einstellung "**Daten und Nutzung**" (oder ähnlich) deaktivieren. **App-Inhalte werden jedoch nicht gespeichert und gesendet**

## Grenzen

Die Prüfung sagt nur, dass die MRZ in sich stimmig und plausibel ist. Eine
Fälschung mit korrekt berechneten Prüfziffern erkennt sie nicht. Die App
speichert keine Daten.
