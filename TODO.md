# Offene Punkte

Stand 2026-09-23. Sortiert nach dem, was am ehesten weh tut — nicht nach Aufwand.

---

## 1 — Bekannte Fehler

### Rohdatenansicht folgt den Einstellungen nicht

Ändert man in den Optionen etwas, das die Ausgabe betrifft — `SaveTypeInformation`,
`WriteComplexDictionaryKeys` — bleibt die JSON/XML-Ansicht auf dem alten Stand.
`JsonString` und `XmlString` in `DataManagerFileViewModel` werden bei jedem Lesen
neu erzeugt, aber niemand meldet, dass sie neu zu lesen wären.

Derselbe Bau wie beim Sprachwechsel und beim Statusbalken: der Wert ist frisch, es
fehlt der Anstoß. Beide Male hat es gereicht, die Ansicht bei ihrem eigenen
ViewModel anzumelden — siehe `StatusBarView`.

### `.xml` öffnen wirft

Seit längerem notiert, nie untersucht. Vor dem Angehen einmal nachstellen — es ist
nicht sicher, dass es noch auftritt.

### Doppelter Eintrag bei zuletzt geöffneten Dateien

Ebenfalls länger notiert. `AddNewRecentDataFile` entfernt einen vorhandenen
Eintrag vor dem Einfügen, also entweder ein anderer Pfad oder ein zweiter Aufrufer.

---

## 2 — Datenannotationen, letzter Rest

Der Editor wertet inzwischen aus: `[Display]` (übersetzt), `[DataType]` für
Passwort, Datum, Uhrzeit und Mehrzeiler, sowie `[Range]`, `[StringLength]`,
`[MinLength]`, `[MaxLength]`, `[EmailAddress]`, `[Url]` und `[RegularExpression]`
als Prüfung mit rotem Rahmen und übersetzter Meldung.

Offen bleibt:

- **`[Required]` ist ein Zustand, kein abgelehnter Wert.** Die Meldung gibt es, die
  Durchsetzung nicht. Ein leeres Feld wird übernommen.
- **Enums:** `[Display]` auf den einzelnen Werten wird nicht gelesen, angezeigt wird
  der Bezeichner aus dem Code. `[Flags]` wird nicht erkannt, mehrere Werte
  gleichzeitig sind nicht wählbar.
- **Eine geladene Datei wird nie geprüft.** Die Regeln greifen nur beim Tippen. Eine
  Datei voller ungültiger Werte öffnet sich ohne ein Wort.

---

## 3 — NuGet-Pakete

**Seit 2026-09-28 liegen alle vierzehn Bibliotheken auf nuget.org.** Das Repo
`MDD4All.DME` (ohne `-dev`) baut die App nur aus diesen Paketen — frisch geklont,
gebaut und gestartet, sie läuft.

So geht ein Release, von unten nach oben:

- In der Projektmappe steht nur `<Name>-dev.csproj` mit Projektverweisen. Daneben ist
  `<Name>.csproj` das Paketprojekt und verweist nur auf Pakete von nuget.org.
- `dev` nach `main` pushen: `build.yml` baut und legt eine Probe in GitHub Packages
  ab, Version aus `src/version.txt` plus Laufnummer.
- Auf GitHub „Publish to nuget.org" mit genau dieser Nummer — nur von Hand.
- Hängt ein Paket an einem geänderten anderen, kommt das untere zuerst raus, dann wird
  oben die Version angehoben.
- `main` trägt den veröffentlichten Stand; nachziehen ohne Release nur, wenn sich in
  `src/` nichts ändert.

Offen:

- **DataAccess scheitert ohne vorher geladenes Proxies**: `ToJson` wirft, `ToXml`
  gibt leer zurück, `LoadFromJson` meldet Fehlschlag. Braucht eine Code-Änderung.
- **Die Versionsprüfung in `publish-nuget.yml` ist zu großzügig** — `2.0.0.1.` ging
  durch und scheiterte erst beim Bauen. In allen Repos auf einen genauen Ausdruck
  umstellen.
- **GitHub baut mit dem neuesten SDK**, zurzeit .NET 10, weil kein `global.json` das
  festlegt.
- **Beide Projektdateien in einem Ordner teilen sich `obj/`.** Wer im Hauptprojekt das
  Paketprojekt baut, überschreibt die `project.assets.json` der `-dev`-Fassung. Im
  Repo `MDD4All.DME` stört das nicht, dort gibt es nur das Paketprojekt.
- **`Synorvia.UI.BlazorComponents` steht nur auf `net9.0`.** Bei Bedarf
  `<TargetFrameworks>net8.0;net9.0</TargetFrameworks>`.

---

## 4 — .NET 10

Alles steht auf `net9.0` oder `netstandard2.0`, nichts mehr auf `net6.0`. Aber
`net9.0` ist selbst schon aus der Pflege (Mai 2026) — wir stehen dort, weil die
Apps dort standen und es nichts gekostet hat.

Ruhe bringt erst .NET 10 (LTS bis November 2028). Vorher zu klären: auf dem Rechner
liegt nur SDK 9.0.316, und Visual Studio ist 2022 in 17.14. Ob das Gespann .NET 10
treibt oder ein neueres Visual Studio nötig ist, muss nachgesehen werden, bevor
etwas installiert wird.

`MDD4All.FileAccess.WPF` ist seit dem 23.09. ein fork und heißt seit dem 28.09.
`Synorvia.FileAccess.WPF`. Den Rahmensprung bekäme oalt nur noch von Hand, weil der
Namensraum abweicht.

---

## 5 — Einstellungsdialog

- **Ein Abbrechen-Knopf fehlt.** Es gibt nur OK, und jede Umschaltung wirkt
  ohnehin sofort und wird sofort gespeichert (`SetAndStore`). Ein Abbrechen
  müsste also entweder den Stand beim Öffnen merken und zurückrollen, oder die
  Schalter dürfen erst bei OK wirken. **Das ist die eigentliche Frage** — der
  Knopf ist nur die Folge.
- **Auf Werkseinstellungen zurücksetzen.** Die Vorgaben stehen bereits in den
  Konfigurationsklassen als Feldinitialisierer, es genügt also ein frisches
  Objekt und einmal speichern.
- **Besser sortieren.** Der Editor-Reiter hat inzwischen sieben Schalter ohne
  erkennbare Ordnung.

---

## 6 — Übersetzung, letzter Rest

`DataManagerFileViewModel` setzt rund fünfzehn Meldungen als festen englischen
Text (`LoadErrorMessage`, `SaveWarningMessage`, `DescribeLoadFailure`), dazu die
Statuszeile (`Filename:`, `Data Model:`) und die Titel der Dateidialoge. Alles
andere in den Views läuft inzwischen über `AppTexts.resx`.

Zusammen damit lohnt sich die Frage aus der Serialisierungsrunde: eine Property
ohne Gleichheitsprüfung ist ein verkleidetes Ereignis. `LoadErrorMessage` und
`SaveWarningMessage` sollten echte Ereignisse werden, dann stellt sich die Frage
nach der Sprache dort gar nicht mehr — die Ansicht formuliert.

---

## 7 — Aufräumen im Repository

- `MDD4All.Configuration` hat nur einen `main` und keinen `dev`. Es ist oalts Repo,
  dort lässt sich keiner anlegen — bei Bedarf fragen oder abspalten.

---

## 8 — Was im Editor noch fehlt

Aus dem Durchklicken gesammelt, nichts davon dringend.

### Funktionen

- Knopf "Alle zuklappen".
- Ein `null`-Element in Liste oder Array an Ort und Stelle anlegen.
- **"Nur den Wert löschen, Schlüssel behalten"** bei Dictionary-Einträgen. Die
  Logik ist vorhanden — `ReferenceEditorViewModel.ExecuteDeleteItem`, der Zweig
  ab Zeile 141. Es fehlt der Knopf: die Wertkarte wird mit `ShowOwnHeader="false"`
  gezeichnet, und der Löschknopf sitzt im Kopf.
- Eine Editor-Einstellung, ob Dictionaries mit komplexem Schlüssel überhaupt
  angezeigt werden. **Vorher neu fragen, was das heute heißen soll** — der Wunsch
  ist älter als der Editor für komplexe Schlüssel, den es inzwischen gibt.
- **Der Randfall beim Speichern:** hat ein Dokument ungespeicherte Änderungen *und*
  komplexe Wörterbuchschlüssel, kommt nach "Speichern" die zweite Warnung, und der
  Wechsel fällt aus. Der Nutzer muss noch einmal auf Öffnen klicken. Bewusst so —
  zwei Fragen übereinander helfen niemandem. Wenn es im Gebrauch stört, neu
  ansehen.

### Aussehen

- Editor/Rohdaten-Knöpfe in `EditorMainToolbar.razor` sind fest auf
  `btn-outline-dark`, ignorieren das Farbschema.
- Oberste Karte füllt die Höhe ihres Bereichs nicht.
- Ohne Explorer-Symbole klebt die Stufennummer am Titel — dort fehlt die Lücke,
  wo vorher das Symbol war.
- Leerzeile zwischen Titel und Indexnummer gewünscht.
- Bei kleinem Fenster und aufgeklappter Listenkarte liegt der Löschknopf hinter
  einem waagerechten Rollbalken.
- Die JSON/XML-Umschaltung in der Rohdatenansicht "braucht nochmal einen Blick" —
  was genau, ist nie gesagt worden.
- Die Werkzeugleiste und der Tiefenschalter sind am 22.09. umgebaut worden. Was
  damals an ihnen notiert war, ist vermutlich überholt und müsste neu angesehen
  werden.

### Nie geprüft

- Vererbung in Datenmodellen. Kein Modell im Repository benutzt sie, der ganze
  `$type`-Weg ist mit einer echten Unterklasse nie gelaufen.
- Array-Bearbeitung. Seit dem 22.09. gibt es mit `PersonRepository.Archived`
  wenigstens ein Array aus Objekten zum Anfassen.

### Vielleicht

- `EditorState` echte Änderungsmeldungen senden lassen, statt sich auf Blazors
  Zeichenlauf zu verlassen. Lohnt nur, wenn je etwas außerhalb des Zeichnens
  darauf reagieren muss.

### Nicht anfassen

Der Tiefenschalter läuft im Ring: beide Knöpfe gehen rundherum,
`2 3 … Limit All 2 …`. Das ist **so gewollt**. Eine "Korrektur" war schon
geschrieben und wurde auf Ansage wieder zurückgenommen; der Kommentar in
`EditorDepthStepperView.razor` hält es fest.

---

## Erledigt und deshalb gestrichen

**Am 23.09.:**

- **Repo-Benennung.** Das `-dev` gehört ans Projekt, nicht ans Repository. Sieben
  eigene Repos sind umbenannt, `.gitmodules` und alle Arbeitskopien nachgezogen.
- **Zielrahmen.** Nichts steht mehr auf `net6.0` oder `net7.0`. Die 6.0er Pakete
  sind nicht angehoben, sondern verschwunden — `FrameworkReference` hat keine
  Fassung, die man pflegen muss.
- **Kein Weg zurück vom Startbildschirm.** `ViewState` kehrte nur zurück, wenn ein
  neues Wurzelobjekt entstand. `CanShowEditor` und `ShowEditor` sind die fehlende
  Hälfte von `ShowStartPage`.
- **Ungespeicherte Änderungen.** Merker am Dokument, Stern in Statusleiste und
  Fenstertitel, und dieselbe Frage mit drei Antworten beim Öffnen, Neuanlegen und
  Schließen des Fensters.

**Am 22.09.:**

- **Listen und Arrays umsortieren.** Ein Modus im Kartenkopf, Pfeile an jedem
  Eintrag. Liste und Array brauchten kein eigenes Verfahren — beide antworten als
  `IList`.
- **Blinken beim Sprachwechsel.** Gelöst über `LocalizedComponent`: nur die
  Bauteile zeichnen neu, die wirklich Text zeigen.
- **Sprachwahl nur noch in den Optionen**, nicht mehr in der Schnellleiste.
