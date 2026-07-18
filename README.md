# A3IP — AI Infrastructure Installation Package
*Pronounced "ay-trip"*

> **A3IP is a permission-aware package format for portable AI agent workflows** — an AI-readable install protocol that lets one AI hand a complete, permission-declaring workflow to another AI to set up.

[![Spec](https://img.shields.io/badge/spec-v1.12-blue)](docs/A3IP-SPEC-v1.12.md)
[![License: CC BY 4.0](https://img.shields.io/badge/spec-CC%20BY%204.0-lightgrey)](LICENSE)
[![CLI](https://img.shields.io/badge/pip%20install-a3ip-green)](https://pypi.org/project/a3ip)

> **Project status — concluded experiment (July 2026).** A3IP explored an AI-mediated install protocol for agent workflows: hand one AI a package and it sets the whole thing up conversationally, confirming permissions first. The design works, but the space it aimed at is now covered more maturely by **[Microsoft APM](https://github.com/microsoft/apm)**, an actively maintained dependency manager and installer for agent context. For real use, prefer APM. This repository remains as an honest, open record of the experiment; it is not being developed further.

---

## The problem

You built a great AI workflow on Claude Code. You want to share it with a teammate on Cursor. You send the files — and it breaks. Wrong paths. Missing MCP server setup. Config keys that mean nothing without context. No explanation of what to run first. You end up on a call explaining it step by step.

Microsoft's APM covers much of this for developers today — a CLI that resolves, pins, and installs agent context across many coding harnesses. A3IP took a different angle: the receiving AI reads the package and does the install itself, asking the setup questions and showing what it will touch before it acts.

**A3IP is a format for that AI-mediated install.**

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

The `permissions:` block is declared upfront. A3IP does not sandbox or enforce it; the receiving AI is trusted to present the plan and honor the contract before it acts. The declaration is what makes the workflow auditable, not any runtime containment.

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
- A **runtime-agnostic install protocol** — the same `.a3ip.bundle` is meant to install consistently across Cowork, Claude Code, Codex, and Cursor. The outcome depends on the receiving AI following the protocol; per-platform mechanics (skill registration, artifact creation, scheduled tasks) live in adapter files inside the package. Cowork, Codex, and Claude Code have shipped adapters; other runtimes are less tested.
- An **open standard** — spec text is CC BY 4.0, reference tooling is Apache 2.0. Anyone can implement, fork, or extend it.

**A3IP IS NOT:**

- **Not a competitor to Cowork Plugins.** Cowork Plugins are Anthropic's first-party packaging surface for Cowork specifically — they bundle skills, connectors, slash commands, and sub-agents into an installable unit governed by Anthropic, native to Cowork's runtime. A3IP is the cross-platform sibling: same packaging discipline, no platform lock-in. The two are complementary — an A3IP package can target Cowork (and does so first-class via the cowork runtime adapter); a future `a3ip export --format cowork-plugin` adapter is planned.
- **Not a competitor to MCP.** MCP is the wire protocol an AI uses to call tools at runtime. A3IP sits above MCP: it declares which MCPs a workflow needs, installs and configures them as part of the user-confirmed install plan, and then the workflow runs against them.
- **Not a competitor to SKILL.md.** A3IP packages SKILL.md as a native component type. Every A3IP skill is already SKILL.md-compatible. A3IP adds the install protocol around it.
- **Overlaps heavily with Microsoft APM.** APM is a mature dependency manager for agent context: a CLI that resolves, pins, and security-scans dependencies, enforces install-time policy, and configures seven coding harnesses (Copilot, Claude Code, Cursor, Codex, Gemini, Windsurf, OpenCode). It already covers cross-harness portability and installation. A3IP's distinct angle is narrower: a conversational install the receiving AI runs itself, with a setup wizard (`CONFIGURE.md`) and a per-install consent step. Where APM enforces policy deterministically, A3IP declares intent for an AI to present.
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

A3IP adopts [SKILL.md](https://agentskills.io) as its skill format — any A3IP skill is a plain SKILL.md file, compatible with every SKILL.md-supporting platform.

---

## How does this relate to Cowork Plugins, SKILL.md, MCP, and APM?

| Concept | Cowork Plugins | SKILL.md | MCP | APM | A3IP |
|---|---|---|---|---|---|
| Portable skills | ✅ (Cowork only) | ✅ | — | ✅ | ✅ adopts SKILL.md exactly |
| Cross-platform portability | ❌ (Cowork-native) | ✅ | ✅ | ✅ (7 harnesses) | ✅ |
| MCP dependency declarations | ✅ | ❌ | self | ✅ | ✅ |
| AI-readable install guide | partial | ❌ | — | ❌ | ✅ `INSTALL.md` |
| Installation wizard | ❌ | ❌ | — | ❌ | ✅ `CONFIGURE.md` |
| Permission / policy controls | partial | ❌ | — | ✅ policy engine | ✅ declared `permissions:` block |
| Delta upgrades | ✅ | ❌ | — | ✅ lockfile + pinning | ✅ `CHANGELOG.md` |
| Plain-text bundle format | ❌ (.zip) | ❌ | — | ❌ | ✅ `.a3ip.bundle` |
| Uninstall protocol | ✅ (UI) | ❌ | — | ❌ | ✅ spec v1.10 |

**Cowork Plugins** are Anthropic's first-party packaging surface for Cowork — they bundle skills, connectors, slash commands, sub-agents, and MCP integrations into an installable unit governed by Anthropic. Cowork Plugins are Cowork-native: they install into Cowork's runtime, are loaded by Cowork's plugin system, and depend on Anthropic-owned distribution channels. A3IP is the cross-platform sibling — same packaging discipline, no platform lock-in. An A3IP package can target Cowork (and does so as a first-class runtime via the bundled Cowork adapter), and a future `a3ip export --format cowork-plugin` adapter will let A3IP packages publish to Cowork's plugin marketplace.

**MCP** connects an AI to tools — file systems, databases, APIs. A3IP sits above that: it packages a complete workflow that *uses* those tools, with an explicit list of which MCP servers it requires declared in the manifest. Installing an A3IP package can include configuring MCP connections.

**Microsoft APM** is a mature dependency manager for agent context: `apm install` resolves and pins dependencies with a content-hash lockfile, scans them for tampering, enforces install-time policy (`apm-policy.yml`), and configures seven coding harnesses. It already covers cross-harness portability, installation, and governance for developers. A3IP's narrower distinct angle is the *conversational* install: the receiving AI reads `INSTALL.md`, walks the user through `CONFIGURE.md`, and presents a per-install consent step. The overlap is large, and for developer use APM is the more complete, better-supported option. The proposed "APM block inside an A3IP package" and `export --format apm` integrations were never built.

**SKILL.md** is the open standard for describing a single skill's invocation contract. A3IP adopts it as a native component type: every A3IP skill is already SKILL.md-compatible without modification. A3IP adds packaging + install + uninstall around it.

A3IP adopts SKILL.md as its skill format and composes with MCP and Cowork Plugins. Existing SKILL.md skills remain valid inside A3IP unmodified. (The claimed APM-manifest compatibility was aspirational and never implemented.) See [COMPATIBILITY.md](COMPATIBILITY.md) for the full detail.

---

## Documentation

| | |
|---|---|
| [Spec v1.12](docs/A3IP-SPEC-v1.12.md) | The canonical format reference |
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

*A3IP Specification v1.12 · © 2026 Maksym Prydorozhko · [a3ip.dev](https://a3ip.dev)*
