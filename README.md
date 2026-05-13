# A3IP — AI Infrastructure Installation Package
*Pronounced "ay-trip"*

> **A3IP is a permission-aware package format for portable AI agent workflows** — install complete workflows into Claude Code, Codex, Cursor, or Copilot the way you'd install a Docker container.

[![Spec](https://img.shields.io/badge/spec-v1.5-blue)](docs/A3IP-SPEC-v1.5.md)
[![License: CC BY 4.0](https://img.shields.io/badge/spec-CC%20BY%204.0-lightgrey)](LICENSE-SPEC.md)
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
$schema: https://a3ip.dev/schema/v1.5/manifest.json

name: ai-code-review-flow
version: "1.0.0"
description: "AI-powered code review workflow for GitHub and GitLab."
min_a3ip_spec: "1.2"
trust_level: standard

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

## Get started

**Install a package**

Drop a `.a3ip.bundle` file into a conversation with any A3IP-compatible AI and ask it to install. The AI reads `INSTALL.md`, walks you through `CONFIGURE.md`, and confirms every step before executing.

```
→ Browse the registry: https://a3ip.dev/packages
→ Download a bundle → paste into your AI conversation → follow the install plan
```

**Create a package**
```bash
pip install a3ip
a3ip new my-workflow      # scaffold a valid package in seconds
a3ip validate my-workflow/
a3ip bundle my-workflow/
```

→ [Browse the package gallery](https://a3ip.dev/packages)
→ [Read the authoring guide](docs/AUTHORING.md)

---

## Platform compatibility

| Platform | Status |
|---|---|
| Claude Code | ✅ Supported |
| GitHub Copilot / Codex | ✅ Supported |
| Cursor | 🟡 Community support |
| Windsurf / Codeium | 🟡 Community support |
| Claude.ai (web) | 🟡 Session install (no persistent state required) |

A3IP skills are a superset of [SKILL.md](https://agentskills.io) — any A3IP skill is already compatible with every SKILL.md-supporting platform.

---

## How does this relate to MCP, APM, and SKILL.md?

| Concept | SKILL.md | APM | A3IP |
|---|---|---|---|
| Portable skills | ✅ | ✅ | ✅ adopts SKILL.md exactly |
| MCP dependency declarations | ❌ | ✅ | ✅ |
| AI-readable install guide | ❌ | ❌ | ✅ `INSTALL.md` |
| Installation wizard | ❌ | ❌ | ✅ `CONFIGURE.md` |
| Permission declarations | ❌ | ❌ | ✅ `permissions:` block |
| Delta upgrades | ❌ | ❌ | ✅ `CHANGELOG.md` |
| Plain-text bundle format | ❌ | ❌ | ✅ `.a3ip.bundle` |

## How does this relate to MCP and APM?

**MCP** connects an AI to tools — file systems, databases, APIs. A3IP sits above that: it packages a complete workflow that *uses* those tools, with an explicit list of which MCP servers it requires declared in the manifest. Installing an A3IP package can include configuring MCP connections.

**Microsoft APM** manages agent context dependencies. A3IP adds what APM doesn't have: a configuration wizard, a user-confirmed install plan, and explicit permission declarations before anything runs. The two are complementary — an APM manifest can live inside an A3IP package as a dependency, and an A3IP package can reference APM context blocks.

A3IP is a **superset** of both SKILL.md and APM-compatible manifest blocks. Existing skills and APM packages remain valid inside A3IP without modification.

---

## Documentation

| | |
|---|---|
| [Spec v1.5](docs/A3IP-SPEC-v1.5.md) | The canonical format reference |
| [JSON Schema](https://a3ip.dev/schema/v1.5/manifest.json) | For IDE validation and tooling |
| [Package gallery](https://a3ip.dev/packages) | Browse installable workflows |
| [CLI reference](https://github.com/a3ip-standard/cli) | `pip install a3ip` |
| [Creator tool](https://github.com/a3ip-standard/creator) | Scaffold and publish packages |
| [COMPATIBILITY.md](COMPATIBILITY.md) | Full MCP / APM / SKILL.md compatibility notes |
| [GOVERNANCE.md](GOVERNANCE.md) | How the spec evolves and who decides |

---

## License

**Specification text:** [Creative Commons Attribution 4.0](LICENSE-SPEC.md) — read, implement, redistribute, and build on freely, with attribution.

**Tooling and CLI:** [Apache 2.0](LICENSE) — use freely in commercial and open source projects. Includes explicit patent grant.

---

*A3IP Specification v1.5 · © 2026 Maksym Prydorozhko · [a3ip.dev](https://a3ip.dev)*
