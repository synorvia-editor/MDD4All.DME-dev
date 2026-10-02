# Synorvia Editor – Development Repository

This repository is the **development workspace** of the Synorvia Editor (MDD4All.DME), an object graph editor for Windows built with WPF and Blazor.

It contains no application code of its own. Instead, every library and the application host are included as **Git submodules** under [`src/`](src) and tied together by one Visual Studio solution, [`src/MDD4All.DME.sln`](src/MDD4All.DME.sln). This lets each component live in its own repository, with its own version history and NuGet package, while you still work on the whole product in a single solution with debugging across project boundaries.

## Repository layout

```
SynorviaEditor-dev/
├── src/
│   ├── MDD4All.DME.sln                  <- solution referencing all -dev projects
│   ├── MDD4All.DME.App.Wpf/             <- submodule: WPF desktop host (startup project)
│   ├── MDD4All.DME.DataModels/          <- submodule
│   ├── MDD4All.DME.ViewModels/          <- submodule
│   ├── MDD4All.DME.Views/               <- submodule
│   └── ...                              <- further submodules (see .gitmodules)
├── .github/workflows/                   <- CI: integration build and snapshot build
└── .gitmodules
```

Each submodule follows the same structure:

```
<Submodule>/
└── src/
    └── <Submodule>/
        ├── <Submodule>-dev.csproj   <- development project (project references)
        └── <Submodule>.csproj       <- NuGet package project (package references only)
```

### Submodules

| Group | Submodules |
|---|---|
| Application | `MDD4All.DME.App.Wpf` |
| Editor | `MDD4All.DME.Views`, `MDD4All.DME.ViewModels`, `MDD4All.DME.DataModels`, `MDD4All.DME.DataAccess`, `MDD4All.DME.AssemblyTree`, `MDD4All.DME.AssemblyLoading`, `MDD4All.DME.Proxies` |
| UI | `MDD4All.UI.Blazor`, `Synorvia.UI.BlazorComponents`, `Synorvia.UI.DataModels` |
| File access | `Synorvia.FileAccess.Contracts`, `Synorvia.FileAccess.WPF` |
| Infrastructure | `MDD4All.AssemblyLoading.Contracts`, `MDD4All.Configuration`, `MDD4All.Localization`, `MDD4All.Reflection.TypeAnalyzer`, `MDD4All.Person.DataModels` |

## Prerequisites

- Windows 10 or 11 (WPF)
- [Git for Windows](https://git-scm.com/download/win)
- [.NET 9 SDK (x64)](https://dotnet.microsoft.com/download/dotnet/9.0)
- [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) (preinstalled on Windows 11)
- Visual Studio 2022 (17.12 or later) with the *.NET desktop development* workload is recommended, but the command line is sufficient.

## Getting started

Some paths in the submodules are long, so enable long paths and clone into a short directory such as `C:\work`:

```
git -c core.longpaths=true clone --recurse-submodules https://github.com/synorvia-editor/MDD4All.DME-dev.git SynorviaEditor-dev
```

If you already cloned without submodules:

```
git submodule update --init --recursive
```

Build and run:

```
dotnet build src\MDD4All.DME.sln -c Release
```

In Visual Studio, open `src\MDD4All.DME.sln` and start `MDD4All.DME.App.Wpf-dev` (the startup project).

## Working with the submodules

Each submodule is a full Git repository. Changes are committed and pushed **inside the submodule first**, then the updated submodule pointer is committed in this repository.

```
cd src\MDD4All.DME.Views
git checkout dev                 # submodules are checked out detached; switch to a branch first
# ... edit, commit ...
git push

cd ..\..
git add src/MDD4All.DME.Views    # record the new submodule commit
git commit -m "Update Views submodule"
git push
```

Pull the latest state of everything:

```
git pull
git submodule update --init --recursive
```

Update all submodules to the head of their tracked branch (`dev` where configured in `.gitmodules`):

```
git submodule update --remote
```

> Always push the submodule commit before pushing the pointer in this repository. Otherwise others cannot check out the referenced commit.

## Project files: `-dev` vs. NuGet

Every project has two project files that must be kept in sync:

| File | Purpose | References between our projects |
|---|---|---|
| `<Name>-dev.csproj` | Development inside this workspace and the solution | **`ProjectReference`** to the other `-dev` projects |
| `<Name>.csproj` | Building the NuGet package in the component's own repository | **`PackageReference`** only. `ProjectReference` is **not allowed** |

Rules:

- The solution contains only `-dev` projects.
- When you add a dependency, add a `ProjectReference` in the `-dev` project and the matching `PackageReference` (with version) in the non-dev project.
- When you add a new source file, make sure it is included by both project files (for SDK-style projects this is automatic unless files are listed explicitly).
- Never commit a `ProjectReference` into a non-dev project, since the NuGet build of a single repository has no access to the sibling submodules.

## Adding a new library

1. Create the library in its own repository, with `src/<Name>/<Name>-dev.csproj` and `src/<Name>/<Name>.csproj`.
2. Add it here as a submodule:
   ```
   git submodule add -b dev https://github.com/synorvia-editor/<Name>.git src/<Name>
   ```
3. Add `src\<Name>\src\<Name>\<Name>-dev.csproj` to `src\MDD4All.DME.sln`.
4. Add the `ProjectReference` in the `-dev` projects that use it.

## Continuous integration

Workflows in [`.github/workflows/`](.github/workflows):

- **Development Integration Build** (`integration-dev-build.yml`): runs on pushes to `main` and `dev` and on pull requests to `main`. It checks out all submodules, restores and builds the solution with .NET 9 on Windows. The version is generated as `yyyy.M.d.<run number>`.
- **Build frontend snapshot** (`frontend-publish-snapshot.yml`): started manually. It publishes `MDD4All.DME.App.Wpf-dev` with the `FolderProfile` and uploads the zipped application as a snapshot build.

