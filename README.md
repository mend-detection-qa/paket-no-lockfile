# paket-no-lockfile

## Probe metadata

| Field | Value |
|---|---|
| Pattern | paket-no-lockfile |
| Target framework | net8.0 |
| NuGet source | https://api.nuget.org/v3/index.json |
| Storage mode | none |
| Lock file present | NO |
| Generated | 2026-04-22 |

## Feature exercised

This probe exercises Mend's UA fallback behavior when `paket.lock` is absent.
Only `paket.dependencies` and `paket.references` are present. Mend must attempt
to parse `paket.dependencies` directly for direct dependency information. This
is a known degraded detection scenario: without a lockfile Mend cannot resolve
transitive dependencies, so the expected tree contains only the three direct
packages with no children.

## Expected dependency tree

### Direct dependencies (Main group — from paket.dependencies only)

| Package | Constraint | Degraded resolved version | Has transitives |
|---|---|---|---|
| Newtonsoft.Json | >= 13.0.0 | unknown / constraint-only | No |
| Serilog | >= 3.0.0 | unknown / constraint-only | No |
| Microsoft.Extensions.Logging | >= 8.0.0 | unknown / constraint-only | No |

### Transitive dependencies

None expected. Without `paket.lock` Mend cannot determine transitive closure.

### Detection expectations

- Mend should detect the **3 direct packages** declared in `paket.dependencies`.
- Package versions may be reported as the lower-bound constraint value (e.g.
  `13.0.0`, `3.0.0`, `8.0.0`) or left empty, depending on Mend UA version.
- No transitive packages should appear — `children` must be `null` for all
  top-level entries.
- The `dependencyFile` field is expected to reference `paket.dependencies`, not
  `paket.lock`, because no lockfile exists.
- This probe validates graceful degradation, not full tree accuracy.

## File structure

```
paket-no-lockfile/
├── paket.dependencies       # Single Main group, 3 direct deps, NO lockfile
├── expected-tree.json       # Degraded expected tree (direct deps only, no children)
├── README.md                # This file
└── src/
    └── MyProject/
        ├── MyProject.csproj # SDK-style net8.0 project
        └── paket.references # 3 direct package references
```
