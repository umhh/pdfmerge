# pdfmerge – PDF Duplex-Mischer

Browserbasiertes Tool, das zwei Scans eines doppelseitigen Dokuments zu **einem** PDF in richtiger Seitenreihenfolge zusammenführt. Gedacht für Scanner ohne Duplex-Einzug: erst alle Vorderseiten scannen, dann den Stapel umdrehen und alle Rückseiten scannen.

**Online nutzen:** https://umhh.github.io/pdfmerge/

Alles läuft lokal im Browser. Die PDFs werden **nicht hochgeladen** und verlassen den Rechner nicht.

---

## Schnellstart

1. **https://umhh.github.io/pdfmerge/** in Safari öffnen (oder `index.html` lokal per Doppelklick).
2. Vorderseiten-PDF in das Feld **„Seite 1, 3, 5, …“** ziehen.
3. Rückseiten-PDF in das Feld **„Seite …, 6, 4, 2“** ziehen.
4. **Start** drücken.
5. Das Ergebnis in der Vorschau prüfen, Dateinamen anpassen und speichern.

## Scannen

| Schritt | Was | Ergebnis |
|---|---|---|
| 1 | Stapel einlegen, alle Vorderseiten scannen | Datei A: Seite 1, 3, 5, … |
| 2 | Stapel als Ganzes umdrehen, alle Rückseiten scannen | Datei B: Seite …, 6, 4, 2 (umgekehrt) |
| 3 | A und B im Tool ablegen, Start | Seite 1, 2, 3, 4, 5, 6, … |

Durch das Umdrehen des Stapels liegt die Rückseite der **letzten** Seite oben, die Rückseiten-Datei ist also in umgekehrter Reihenfolge. Das Tool dreht sie automatisch wieder um.

## Bedienung

### Dateien ablegen
- Per **Drag & Drop** in das jeweilige Feld oder durch **Klick/Tippen** auf das Feld (Dateiauswahl).
- Nach dem Ablegen werden Dateiname und Seitenzahl angezeigt.
- **×** oben rechts im Feld entfernt die Datei wieder.
- Eine neben die Felder gezogene PDF wird ignoriert, Safari verlässt die Seite also nicht versehentlich.

### Option „Rückseiten-Datei umdrehen“
Standardmäßig aktiv. Nur abschalten, wenn der Scanner die Rückseiten bereits in der Reihenfolge 2, 4, 6, … liefert.

### Start
Mischt die Seiten nach dem Schema

```
Vorderseite 1, Rückseite 1, Vorderseite 2, Rückseite 2, …
```

Danach erscheinen sofort die **Vorschau** des Ergebnisses sowie Dateiname und Speicher-Knöpfe.

### Ungleiche Seitenzahlen
- **Gleich viele** Vorder- und Rückseiten, oder **eine Rückseite weniger** (z. B. leere letzte Rückseite nicht gescannt): normaler Fall, Meldung in Grün.
- **Andere Abweichungen:** Das Ergebnis wird trotzdem erstellt, überzählige Seiten stehen am Ende. Ein orangefarbener Hinweis warnt davor. Meist fehlt dann eine Seite oder es wurden zwei Blätter auf einmal eingezogen.

### Vorschau
Das gemischte PDF wird direkt unter den Knöpfen im integrierten PDF-Viewer des Browsers angezeigt, zum Durchblättern und Prüfen der Reihenfolge. Wird eine neue Datei abgelegt, verschwindet die alte Vorschau automatisch.

## Speichern

Der Dateiname ist mit `<Name der Vorderseiten-Datei>_gemischt.pdf` vorbelegt und kann im Feld geändert werden. Unzulässige Zeichen (`/ \ : * ? " < > |`) werden durch `_` ersetzt, `.pdf` wird bei Bedarf ergänzt.

Wie gespeichert wird, hängt vom Gerät ab:

| Gerät / Browser | „Speichern …“ | Ordner frei wählen |
|---|---|---|
| **iPad / iPhone (Safari)** | Öffnet das **Teilen-Menü** | „In Dateien sichern“ wählen, dann Ordner und Namen festlegen |
| **Mac (Safari)** | Speichert in den Download-Ordner | **„In neuem Tab öffnen“** und dort **⌘S**, oder Rechtsklick in die Vorschau › **„Mit Vorschau öffnen“** › **⌘S** |
| **Mac (Chrome, Edge)** | Öffnet direkt den **Speichern-Dialog** mit Ordner und Namen | im Dialog |

**Tipp für Safari am Mac:** Unter *Safari › Einstellungen › Allgemein › „Ort für geladene Dateien“* die Option **„Bei jedem Download fragen“** wählen. Dann zeigt auch „Speichern …“ den normalen Dialog mit Ordnerauswahl.

**Tipp für das iPad:** Den Standard-Download-Ordner legst du unter *Einstellungen › Apps › Safari › Downloads* fest.

## Originale

Das Tool kann die Ausgangsdateien **nicht löschen**. Webseiten erhalten nur eine Kopie des Dateiinhalts, keinen Zugriff auf die Datei im Finder oder in der Dateien-App. Das ist eine Sicherheitsgrenze des Browsers. Es ist ohnehin ratsam, die Rohscans erst zu löschen, nachdem das Ergebnis geprüft ist.

## Technik

- **Eine einzige Datei** (`index.html`) mit HTML, CSS und JavaScript, ohne Build-Schritt und ohne Server.
- PDF-Verarbeitung mit [pdf-lib](https://pdf-lib.js.org/) 1.17.1, geladen von cdnjs. Für den ersten Aufruf ist daher eine Internetverbindung nötig.
- Seiten werden 1:1 kopiert, ohne Neukodierung. Die Bildqualität der Scans bleibt also unverändert.
- Verschlüsselte PDFs mit Berechtigungsschutz (ohne Öffnen-Passwort) werden mit `ignoreEncryption` geladen.
- Speichern:
  - `navigator.share` mit Dateien auf iPad/iPhone
  - `showSaveFilePicker` (File System Access API), wo verfügbar (Chrome, Edge)
  - sonst Download über einen Link mit `download`-Attribut
- Hell- und Dunkelmodus folgen der Systemeinstellung.

### Mischlogik

```js
// nOdd Vorderseiten, nEven Rückseiten (in der Datei umgekehrt)
for (i = 0; i < max(nOdd, nEven); i++) {
  if (i < nOdd)  Vorderseite[i]
  if (i < nEven) Rückseite[nEven - 1 - i]   // bzw. [i], wenn Umdrehen aus
}
```

Beispiel: Vorderseiten `1, 3, 5` und Rückseiten `6, 4, 2` ergeben `1, 2, 3, 4, 5, 6`. Mit Rückseiten `4, 2` (letzte Rückseite leer, nicht gescannt) ergibt sich `1, 2, 3, 4, 5`.

## Bekannte Einschränkungen

- Kein Löschen der Originaldateien (siehe oben).
- Safari am Mac bietet keinen eigenen Speichern-Dialog für Webseiten. Dafür gibt es den Umweg über einen neuen Tab oder die Vorschau-App.
- PDFs mit Öffnen-Passwort können nicht verarbeitet werden.
- Sehr große Scans (mehrere hundert MB) können auf dem iPad an Speichergrenzen stoßen.
