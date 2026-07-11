# A3IP — AI Infrastructure Installation Package
*Pronounced "ay-trip"*

> **A3IP is a permission-aware package format for portable AI agent workflows** — install complete workflows into Claude Code, Codex, Cursor, or Copilot the way you'd install a Docker container.

[![Spec](https://img.shields.io/badge/spec-v1.11-blue)](docs/A3IP-SPEC-v1.11.md)
[![License: CC BY 4.0](https://img.shields.io/badge/spec-CC%20BY%204.0-lightgrey)](LICENSE)
[![CLI](https://img.shields.io/badge/pip%20install-a3ip-green)](https://pypi.org/project/a3ip)

---

## The problem

You built a great AI workflow on Claude Code. You want to share it with a teammate on Cursor. You send the files — and it breaks. Wrong paths. Missing MCP server setup. Config keys that mean nothing without context. No explanation of what to run first. You end up on a call explaining it step by step.

There is no standard way to package an AI workflow so that someone else's AI can install it from scratch, ask the right setup questions, and confirm what permissions it needs before touching anything.

**A3IP is that standard.**

---

## How it works

```
  workflow.a3ip.bundle
          │
          ▼
  AI reads INSTALL.md          ← what steps to perform
          │
          ▼
  AI reads CONFIGURE.md        ← asks you questions, fills in {{config.*}} values
          │
          ▼
  AI presents install plan     ← shows every step and permission upfront
          │
          ▼
  You confirm → workflow installed
```

One bundle file. Any A3IP-compatible AI. No shell scripts, no manual setup, no guesswork.

---

## See it in 30 seconds

A `manifest.yaml` for a code review workflow:

```yaml
$schema: https://a3ip.dev/schema/v1.10/manifest.schema.json

name: ai-code-review-flow
version: "1.0.0"
description: "AI-powered code review workflow for GitHub and GitLab."
min_a3ip_spec: "1.10"
trust_level: network

permissions:
  filesystem:
    - path: "./reviews/"
      access: write
      reason: "Stores generated review reports."
  network:
    - domain: "api.github.com"
      reason: "Read pull requests, post review comments."
    - domain: "gitlab.com"
      reason: "Read merge requests, post review comments."
  mcp:
    - name: github-mcp
      reason: "Read and comment on pull requests."
    - name: gitlab-mcp
      reason: "Read and comment on merge requests."

components:
  skills:
    - path: components/skills/code-review.md
  artifacts:
    - path: components/artifacts/review-report.md
```

The `permissions:` block is declared upfront. Your AI presents it to you **before** executing a single step — no silent side effects.

A step from the matching `INSTALL.md`:

```markdown
## Step 2: Configure repository access

- [ ] Confirm your {{config.platform}} token is set in your MCP server config
- [ ] Set your default branch name: {{config.default_branch}}
```

`{{config.platform}}` was filled in during setup — the AI asked which platform you use at install time via `CONFIGURE.md`.

---

## What A3IP IS / IS NOT

**A3IP IS:**

- A **package format** for AI agent workflows — bundle, share, install across platforms.
- A **permission contract** — every filesystem path, network domain, MCP server, and shell command a package will touch is declared upfront in the manifest. The install AI presents the full plan; the user confirms before anything runs.
- A **runtime-agnostic install protocol** — the same `.a3ip.bundle` installs identically on Cowork, Claude Code, Codex, or Cursor. Per-platform mechanics (skill registration, artifact creation, scheduled tasks) live in adapter files inside the package; the protocol stays the same.
- An **open standard** — spec text is CC BY 4.0, reference tooling is Apache 2.0. Anyone can implement, fork, or extend it.

**A3IP IS NOT:**

- **Not a competitor to Cowork Plugins.** Cowork Plugins are Anthropic's first-party packaging surface for Cowork specifically — they bundle skills, connectors, slash commands, and sub-agents into an installable unit governed by Anthropic, native to Cowork's runtime. A3IP is the cross-platform sibling: same packaging discipline, no platform lock-in. The two are complementary — an A3IP package can target Cowork (and does so first-class via the cowork runtime adapter); a future `a3ip export --format cowork-plugin` adapter is planned.
- **Not a competitor to MCP.** MCP is the wire protocol an AI uses to call tools at runtime. A3IP sits above MCP: it declares which MCPs a workflow needs, installs and configures them as part of the user-confirmed install plan, and then the workflow runs against them.
- **Not a competitor to SKILL.md.** A3IP packages SKILL.md as a native component type. Every A3IP skill is already SKILL.md-compatible. A3IP adds the install protocol around it.
- **Not a competitor to Microsoft APM.** APM specifies what agent context to load. A3IP adds the install layer around it — wizard, permissions, plan, confirmation. The two compose.
- **Not a runtime.** A3IP doesn't execute the workflow; the host AI runtime does. A3IP is the package format and the install contract — nothing more.

---

## Get started

**Install a package**

Drop a `.a3ip.bundle` file into a conversation with any A3IP-compatible AI and ask it to install. The AI reads `INSTALL.md`, walks you through `CONFIGURE.md`, and confirms every step before executing.

```
→ Browse the registry: https://github.com/a3ip-standard/packages
→ Download a bundle → paste into your AI conversation → follow the install plan
```

**Create a package**
```bash
pip install a3ip
a3ip new my-workflow      # scaffold a valid package in seconds
a3ip validate my-workflow/
a3ip bundle my-workflow/
```

→ [Browse the package gallery](https://github.com/a3ip-standard/packages)
→ [Create a package with the a3ip-creator skill](https://github.com/a3ip-standard/creator)

---

## Platform compatibility

| Platform | Status | Notes |
|---|---|---|
| Cowork | ✅ Supported | First-class runtime adapter; `.skill` zip install via Personal Skills UI |
| Codex | ✅ Supported | First-class runtime adapter; AGENTS.md marker registration |
| Claude Code | ✅ Supported | First-class runtime adapter; auto-discovery from `~/.claude/skills/` |
| Cursor | 🟡 Community support | Tested platforms list; runtime adapter TBD |
| Windsurf / Codeium | 🟡 Community support | No adapter shipped yet |
| Claude.ai (web) | 🟡 Session install | No persistent state required |

A3IP skills are a superset of [SKILL.md](https://agentskills.io) — any A3IP skill is already compatible with every SKILL.md-supporting platform.

---

## How does this relate to Cowork Plugins, SKILL.md, MCP, and APM?

| Concept | Cowork Plugins | SKILL.md | MCP | APM | A3IP |
|---|---|---|---|---|---|
| Portable skills | ✅ (Cowork only) | ✅ | — | ✅ | ✅ adopts SKILL.md exactly |
| Cross-platform portability | ❌ (Cowork-native) | ✅ | ✅ | partial | ✅ |
| MCP dependency declarations | ✅ | ❌ | self | ✅ | ✅ |
| AI-readable install guide | partial | ❌ | — | ❌ | ✅ `INSTALL.md` |
| Installation wizard | ❌ | ❌ | — | ❌ | ✅ `CONFIGURE.md` |
| Permission declarations | partial | ❌ | — | ❌ | ✅ `permissions:` block |
| Delta upgrades | ✅ | ❌ | — | ❌ | ✅ `CHANGELOG.md` |
| Plain-text bundle format | ❌ (.zip) | ❌ | — | ❌ | ✅ `.a3ip.bundle` |
| Uninstall protocol | ✅ (UI) | ❌ | — | ❌ | ✅ spec v1.10 |

**Cowork Plugins** are Anthropic's first-party packaging surface for Cowork — they bundle skills, connectors, slash commands, sub-agents, and MCP integrations into an installable unit governed by Anthropic. Cowork Plugins are Cowork-native: they install into Cowork's runtime, are loaded by Cowork's plugin system, and depend on Anthropic-owned distribution channels. A3IP is the cross-platform sibling — same packaging discipline, no platform lock-in. An A3IP package can target Cowork (and does so as a first-class runtime via the bundled Cowork adapter), and a future `a3ip export --format cowork-plugin` adapter will let A3IP packages publish to Cowork's plugin marketplace.

**MCP** connects an AI to tools — file systems, databases, APIs. A3IP sits above that: it packages a complete workflow that *uses* those tools, with an explicit list of which MCP servers it requires declared in the manifest. Installing an A3IP package can include configuring MCP connections.

**Microsoft APM** manages agent context dependencies. A3IP adds what APM doesn't have: a configuration wizard, a user-confirmed install plan, and explicit permission declarations before anything runs. The two are complementary — an APM manifest can live inside an A3IP package as a dependency, and an A3IP package can reference APM context blocks.

**SKILL.md** is the open standard for describing a single skill's invocation contract. A3IP adopts it as a native component type: every A3IP skill is already SKILL.md-compatible without modification. A3IP adds packaging + install + uninstall around it.

A3IP is a **superset** of SKILL.md and APM-compatible manifest blocks, and a **complement** to Cowork Plugins and MCP. Existing skills, APM packages, and Cowork Plugin internals remain valid inside A3IP. See [COMPATIBILITY.md](COMPATIBILITY.md) for the full detail.

---

## Documentation

| | |
|---|---|
| [Spec v1.11](docs/A3IP-SPEC-v1.11.md) | The canonical format reference |
| [JSON Schemas](https://a3ip.dev/schema/v1.10/manifest.schema.json) | manifest + installed schemas — for IDE validation and tooling |
| [Package gallery](https://github.com/a3ip-standard/packages) | Browse installable workflows |
| [CLI reference](https://github.com/a3ip-standard/cli) | `pip install a3ip` |
| [Creator tool](https://github.com/a3ip-standard/creator) | Scaffold and publish packages |
| [COMPATIBILITY.md](COMPATIBILITY.md) | Full MCP / APM / SKILL.md compatibility notes |
| [GOVERNANCE.md](GOVERNANCE.md) | How the spec evolves and who decides |

---

## License

**Specification text:** [Creative Commons Attribution 4.0](LICENSE) — read, implement, redistribute, and build on freely, with attribution.

**Tooling and CLI:** Apache 2.0 — see the [CLI repo LICENSE](https://github.com/a3ip-standard/cli/blob/main/LICENSE) and [Creator repo LICENSE](https://github.com/a3ip-standard/creator/blob/main/LICENSE). Includes explicit patent grant; use freely in commercial and open source projects.

---

*A3IP Specification v1.11 · © 2026 Maksym Prydorozhko · [a3ip.dev](https://a3ip.dev)*
