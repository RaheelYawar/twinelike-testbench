# Twinelike Testbench

## Purpose
Sandbox for prototyping and exercising the **Twinelike** library — a C# Harlowe parser/runtime at `E:\Git\twinelike\Twinelike.sln`. Use this repo to write throwaway host programs that load stories, drive `StorySession`, and try out `IRenderOutput` adapter ideas against the real library — without polluting the main solution's test project or its commit history.

This is **not** a production project. Code here is meant to be quick, disposable, and focused on poking at one thing at a time. No CI, no published artifact, no API contract to preserve.

## Parent project at a glance
- **Solution:** `E:\Git\twinelike\Twinelike.sln`
- **Library DLL:** `E:\Git\twinelike\bin\Release\netstandard2.0\Twinelike.dll` (self-contained; HtmlAgilityPack ILRepack'd in)
- **Target framework:** `netstandard2.0` (consumer-side: Unity 2018.1+, Godot 3/4, .NET Framework 4.6.1+, .NET 5+, Mono)
- **Namespace:** `Harlowe` (+ `Harlowe.Runtime`, `Harlowe.Twee`)
- **Main entry points:**
  - `new Harlowe(htmlText)` — load a Twine 2 HTML export
  - `new TweeReader().Read(tweeText)` — load Twee 3 source
  - `new StorySession(story, output)` — runtime; drive via `Render()`, `Goto(name)`, `DispatchEvent(regionId)`, `Undo()`
  - `IRenderOutput` — the six-channel adapter contract (Text / Html / Link / Error / PushStyle+PopStyle / BeginInteractive+EndInteractive)
- **Built-in adapters:** `BufferedRenderOutput`, `HtmlRenderOutput`
- **Full details:** see `E:\Git\twinelike\CLAUDE.md` (file layout, descriptor-patch changer model) and `E:\Git\twinelike\ARCHITECTURE.md` (integrator + contributor guide).

## How to reference Twinelike from this testbench
When wiring up a test project here, prefer a **project reference** over a DLL reference so changes to the library surface immediately and the debugger steps into source:

```xml
<ItemGroup>
  <ProjectReference Include="..\twinelike\Twinelike.csproj" />
</ItemGroup>
```

Falling back to a DLL reference is fine for verifying a packaged build:

```xml
<ItemGroup>
  <Reference Include="Twinelike">
    <HintPath>..\twinelike\bin\Release\netstandard2.0\Twinelike.dll</HintPath>
  </Reference>
</ItemGroup>
```

For testing the NuGet package as a downstream consumer would see it:

```sh
dotnet pack ../twinelike/Twinelike.csproj -c Release -o ../twinelike/dist
dotnet add package Twinelike --source ../twinelike/dist
```

## Typical testbench workflows
- **Smoke-test a new story file** — drop the `.html` or `.tw` into this repo, write a 20-line `Program.cs` that loads it, calls `Render()`, walks every reachable passage, and prints the `RenderResult.Text`.
- **Prototype a new `IRenderOutput` adapter** — e.g. an ANSI terminal output, a Spectre.Console adapter, a JSON event-stream output. Iterate freely here, then once it stabilizes, decide whether it belongs in the library (`HarloweParser/Runtime/`) or stays consumer-side.
- **Reproduce a parser/runtime bug** — write the minimal failing input, confirm here, then port to a proper xUnit test inside `HarloweParser.Tests` before fixing.
- **Try a candidate next slice** end-to-end before committing to the API shape in the main repo (event/live system, `(link:)` family, transitions, etc. — see the `## Candidate next slices` section of the parent `CLAUDE.md`).

## Working agreements for this repo
- **Throwaway is fine.** Don't carry forward dead `Program.cs` experiments — overwrite or delete. Each commit should leave the testbench in a runnable state, but the *content* is allowed to churn.
- **Don't add abstractions.** No DI containers, no plugin loaders, no test framework unless a specific experiment genuinely needs one. A `static void Main` and a few `Console.WriteLine`s is the default shape.
- **Findings live in the main repo, not here.** If an experiment proves something out, port the conclusion (a fix, an xUnit test, an `ARCHITECTURE.md` note) into `E:\Git\twinelike\`. This repo's history is not authoritative.
- **No secrets, no large story corpora.** If a test needs a big story file, keep it local or `.gitignore` it. The repo should stay light.

## Repo layout
```
twinelike-testbench/
├── .gitignore         # standard C# + JetBrains ignores
├── CLAUDE.md          # this file
└── README.md          # placeholder
```
Add `*.csproj`, `Program.cs`, story fixtures, etc. as experiments demand.

## When in doubt
Read the parent docs — they're the source of truth for the library's shape:
- `E:\Git\twinelike\CLAUDE.md` — file-level architecture, conventions, current status, candidate next slices
- `E:\Git\twinelike\ARCHITECTURE.md` — Part 1 for integrator usage, Part 2 for internals, glossary at the end
- `E:\Git\twinelike\README.md` — quick-start, supported-feature matrix, engine-mapping table
