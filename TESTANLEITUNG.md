# Test auf einem anderen Rechner

Ziel: zeigen, dass die App MDD4All.DME („Synorvia") auf einem fremden Windows-Rechner
läuft — **zuerst nur aus den NuGet-Paketen, danach aus dem Quellcode**. Vorher wird
installiert, was fehlt, hinterher wird alles wieder entfernt.

Für eine KI-Sitzung: die Schritte der Reihe nach ausführen und nach Test 1 auf das
Go warten, bevor Test 2 beginnt.

## Die zwei Repos

| Test | Repo | Was drin ist |
|---|---|---|
| 1 — mit NuGet | https://github.com/synorvia-editor/MDD4All.DME | Nur die App. Alle Bibliotheken kommen als Pakete von nuget.org. |
| 2 — ohne NuGet | https://github.com/synorvia-editor/MDD4All.DME-dev | Die App und alle 18 Bibliotheken als Quellcode, als Submodule. |

In beiden Repos: Projektmappe `src\MDD4All.DME.sln`, gestartet wird
`MDD4All.DME.App.Wpf`.

## Vorher installieren

1. **Git for Windows** — https://git-scm.com/download/win
2. **.NET 9 SDK, x64** — https://dotnet.microsoft.com/download/dotnet/9.0
3. **WebView2-Laufzeit** — bei Windows 11 schon vorhanden, bei Windows 10 prüfen.

Visual Studio ist nicht nötig, alles geht über die Kommandozeile. Notieren, welche
der drei vorher schon da waren — die bleiben hinterher stehen.

## Test 1 — mit NuGet

In einer neuen Eingabeaufforderung:

```
mkdir C:\DMETest
cd C:\DMETest
git clone --recurse-submodules https://github.com/synorvia-editor/MDD4All.DME.git dme
dotnet build dme\src\MDD4All.DME.sln -c Release
start dme\src\MDD4All.DME.App.Wpf\src\MDD4All.DME.App.Wpf\bin\Release\net9.0-windows7.0\MDD4All.DME.App.Wpf.exe
```

Das Fenster „Synorvia" muss aufgehen. Durchklicken: eine Datei neu anlegen, etwas
ändern, speichern, wieder öffnen. App schließen.

**Erst nach dem Go weiter.**

## Test 2 — ohne NuGet

```
cd C:\DMETest
git -c core.longpaths=true clone --recurse-submodules https://github.com/synorvia-editor/MDD4All.DME-dev.git dev
dotnet build dev\src\MDD4All.DME.sln -c Release
start dev\src\MDD4All.DME.App.Wpf\src\MDD4All.DME.App.Wpf\bin\Release\net9.0-windows7.0\MDD4All.DME.App.Wpf.exe
```

`core.longpaths` und der kurze Ordner `C:\DMETest` sind nötig, weil manche Pfade in
diesem Repo rund 200 Zeichen lang sind. Wieder durchklicken, App schließen.

## Danach entfernen

1. Beide Apps schließen, dann `dotnet build-server shutdown`.
2. Diese Ordner löschen — die unter dem Benutzer aber nur, wenn es sie vor dem Test
   nicht gab:
   - `C:\DMETest`
   - `%APPDATA%\DME` — Einstellungen der App
   - `%USERPROFILE%\.nuget` — Paket-Cache
   - `%LOCALAPPDATA%\NuGet` und `%APPDATA%\NuGet`
   - `%USERPROFILE%\.dotnet`
3. Unter *Einstellungen → Apps* deinstallieren, was für den Test installiert wurde:
   - alle Einträge mit **.NET 9.0** — SDK, .NET Runtime, ASP.NET Core Runtime,
     Windows Desktop Runtime
   - **Git**
