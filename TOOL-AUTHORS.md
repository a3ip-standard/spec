# TOOL-AUTHORS.md — Implementation guidance for A3IP tooling

**Non-normative.** This document is NOT part of the A3IP specification. The spec defines package format, manifest schema, bundle structure, INSTALL.md template, validation rules, and registry format. The spec does NOT define how tools (validators, bundlers, install adapters, scaffolders) should be implemented.

This document collects concrete implementation patterns that the A3IP project has found useful during reference-implementation development. Tool authors are free to ignore this document entirely; the spec compliance contract is unaffected.

---

## Why this document exists

During Phase 1 of A3IP, multiple iterations of the reference CLI and authoring tools hit the same handful of cross-platform pitfalls (Windows console encoding, `~` expansion across sandboxed shells, file-tool selection on cross-product OS × AI-runtime pairs). Documenting them in one place prevents new tool authors from rediscovering them.

It does NOT prescribe how every A3IP tool must work. A web-UI authoring tool, a VS Code extension, or an alternative validator in another language can ignore everything here as long as its outputs are spec-compliant.

---

## Tool-selection for filesystem operations

When an install AI on **Cowork × Windows** reads or writes a path containing `~`, it has multiple file tools available and they handle `~` differently:

| Tool | `~` expansion on Cowork × Windows |
|---|---|
| `mcp__filesystem__read_file` / `mcp__filesystem__write_file` | YES — expands to the Windows user home (e.g. `C:/Users/<u>/`) |
| Windows-native Python `os.path.expanduser('~')` | YES |
| Cowork bash sandbox (`~`, `$HOME`, `os.path.expanduser` inside bash) | NO — resolves to the ephemeral Linux sandbox home, NOT Windows |
| Read tool (raw `~/...`) | NO — does not expand tildes |

**Pattern that works:** use a YES-row tool for any persistent-file operation; never trust bash sandbox `~` to refer to the Windows user home.

Equivalent matrix for other (OS × Runtime) pairs:

| OS \ Runtime | Cowork | Claude Code | Codex | Cursor |
|---|---|---|---|---|
| **Windows** | `mcp__filesystem` (Windows-native). Do NOT use Cowork bash for `~`. | Direct shell / Python with `expanduser`. | code-exec sandbox; use explicit Windows paths or `$env:USERPROFILE`. | Editor-mediated; editor APIs handle paths. |
| **macOS / Linux** | `mcp__filesystem` if available; otherwise bash with `$HOME` (here `~` is the native user home — no sandbox). | Direct shell, `expanduser` works. | code-exec, `expanduser` works. | Editor-mediated. |

For UI-bearing operations (skill registration, artifact creation), the runtime adapter governs. For filesystem operations, the OS adapter governs. INSTALL.md and CONFIGURE.md SHOULD reference both adapter directories explicitly for any step that crosses dimensions.

---

## Unicode console output on Windows

Windows's default console encoding is `cp1252`, which crashes when Python prints common Unicode characters used in CLI UX: `→`, `←`, `✓`, `✗`, `⚠`, `─`, `│`, em-dashes, smart quotes, etc. This caused a real install failure during A3IP Phase 1 dogfooding on Codex × Windows: `a3ip validate` printed `→` in its output and crashed before completing.

Recommended remedies (any one is sufficient):

1. **Use ASCII fallbacks** in console output: `->`, `<-`, `[OK]`, `[FAIL]`, `[WARN]`, `+-`. Simplest, no runtime dependency on encoding config, works on all platforms.
2. **Set `PYTHONIOENCODING=utf-8`** in the tool's entry-point environment. Simple but doesn't help tools invoked indirectly.
3. **`sys.stdout.reconfigure(encoding='utf-8', errors='replace')`** at module init (Python 3.7+). Reconfigures the tool's own streams. Keeps Unicode in output where the terminal supports UTF-8.

The reference `a3ip` CLI chose option 1 from v1.3.1 onward.

---

## Spec-tool decoupling rationale

The A3IP specification describes capabilities, not implementations. Packages declare which tools they need via `dependencies.tools` in their manifest; the spec itself does not anoint any single CLI as canonical.

The `a3ip` CLI is the reference implementation maintained by the A3IP project, but other tools — VS Code extensions, web UIs, alternative CLIs in other languages — are equally valid as long as they implement the spec's normative requirements (the 10 validation checks, bundle format, INSTALL.md template structure, etc.).

This separation is the reason the spec describes file structure and field names but never says things like "run `a3ip validate`". An alternative validator that implements the 10 checks against the same package structure is fully spec-compliant.

---

## Where to look for further patterns

- **`adapters/os/{windows,posix}/file-ops.md`** in any A3IP package — package-local guidance on file-op tool selection per OS host. This is where packages encode their authors' expectations about file tooling.
- **`adapters/runtime/<name>/install-skill.md`** in any A3IP package — package-local guidance on the runtime-specific install flow (e.g. Cowork's `.skill` UI, Codex's file-copy convention).
- **Reference CLI source** — `github.com/a3ip-standard/cli`. The CLI's `bundle.py`, `validate.py`, and `new.py` are reference implementations of those capabilities.
- **Reference Creator source** — `github.com/a3ip-standard/creator`. An AI authoring tool that orchestrates the CLI via SKILL.md.

This document will grow as new patterns surface. Contributions welcome via PR to `github.com/a3ip-standard/spec`.
