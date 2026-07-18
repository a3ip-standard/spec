# A3IP Compatibility

How A3IP relates to neighboring standards in the AI-agent ecosystem.

A3IP is a **package format and install protocol** for portable AI agent
workflows. Several adjacent standards address pieces of the same problem
space — skill description, dependency resolution, runtime tool calls,
plugin distribution. This document explains how A3IP relates to each,
what's complementary versus competitive, and where the integration
points are.

The framing throughout this document: A3IP **composes with** these
standards, it does not replace them.

---

## Anthropic Cowork Plugins

Cowork Plugins are Anthropic's first-party packaging surface for the
Cowork desktop product. A plugin bundles **skills, connectors, slash
commands, sub-agents, and MCP integrations** into an installable
unit. Plugins are loaded by Cowork's plugin system, distributed
through Anthropic-governed channels, and run inside Cowork's runtime.

**Where A3IP and Cowork Plugins overlap.** Both formats let an author
package a multi-component AI workflow as a single installable unit,
both pre-declare dependencies, both support skill-shaped contents,
both let an install AI (or installer UI) walk the user through setup.
The packaging *discipline* is the same: bundle the workflow so a
recipient can install it cleanly without manual file-shuffling.

**Where A3IP differs.** A3IP is **cross-platform by design**. The same
`.a3ip.bundle` aims to install consistently across Cowork, Claude
Code, Codex, and Cursor — the outcome depends on the receiving AI
following the protocol; runtime-specific mechanics live in adapter
files inside the package. A3IP also adds **explicit permission
declarations** (every filesystem path, network domain, MCP server,
and shell command pre-declared in the manifest for the install AI to
present as a plan — A3IP declares, it does not enforce) and a
**normative uninstall protocol** with per-key data-preservation
policy. Cowork
Plugins handle these implicitly via the Cowork UI and Anthropic's
trust model; A3IP makes them explicit so the same package can install
safely on any compliant AI runtime.

**Where A3IP and Cowork Plugins compose.** An A3IP package targets
Cowork as a first-class runtime — the `adapters/runtime/cowork/`
folder in every A3IP package contains the platform-knowledge an
install AI needs to install correctly into Cowork (Personal Skills UI,
`mcp__cowork__create_artifact`, the `.skill` zip flow). A planned
`a3ip export --format cowork-plugin` adapter will allow an A3IP
package to be repackaged for distribution through Cowork's plugin
marketplace, retaining the A3IP manifest as the source of truth and
generating the Cowork plugin metadata as a derived artifact. The
direction is intentionally one-way: A3IP is the broader contract,
Cowork Plugins are a derived target.

**Adapter status:** authored. The Cowork runtime adapter is included
in every A3IP package as part of Creator v3.0's bundled templates.

---

## Microsoft APM (Agent Package Manager)

APM is a dependency manager for agent context — the "npm for agents."
You declare the skills, prompts, instructions, plugins, and MCP
servers your project needs in one `apm.yml`; `apm install` resolves
and pins them with a content-hash lockfile, scans them for tampering,
enforces install-time policy (`apm-policy.yml`), and configures seven
coding harnesses (Copilot, Claude Code, Cursor, Codex, Gemini,
Windsurf, OpenCode).

**Where A3IP and APM overlap.** Both declare external dependencies
(APM context blocks, A3IP MCPs / tools / other skills), both support
a manifest-based declaration of what's needed, both can be consumed
by an AI runtime to compose a working agent.

**Where A3IP differs.** APM already does dependency resolution *and*
installation — `apm install` is a real installer with a lockfile,
security scanning, and policy enforcement. So the honest difference
is narrow: APM's install is a deterministic CLI governed by policy;
A3IP's is *conversational*, run by the receiving AI, which walks the
user through CONFIGURE.md, follows INSTALL.md step by step, and
presents a permissions plan for the user to confirm. A3IP declares
permissions for an AI to present; APM enforces policy at install. For
developer workflows, APM's model is more complete and better
supported.

**Where A3IP and APM compose.** They do not compose today. Two
integrations were proposed and never built: a `dependencies.apm:`
manifest block and an `export --format apm` adapter. Both were on the
roadmap when the project was active; neither shipped.

**Adapter status:** not built.

---

## SKILL.md (agentskills.io open standard)

SKILL.md is the open standard for describing a single skill's
**invocation contract**: a markdown file with frontmatter declaring
the skill's name, description, trigger phrases, and tool
requirements, followed by the procedure the AI follows when the
trigger fires.

**Where A3IP and SKILL.md overlap.** Every A3IP skill is a SKILL.md
file. The standards are compatible by design — A3IP adopts SKILL.md
exactly as the skill component format. An existing SKILL.md file can
be dropped into an A3IP package's `components/skills/<name>/SKILL.md`
slot and shipped without modification.

**Where A3IP differs.** SKILL.md describes a single skill; A3IP
packages **a complete workflow** that may include multiple skills,
artifacts, protocols, prompts, and scripts. SKILL.md says nothing
about how the skill gets onto a user's machine in the first place;
A3IP is the install protocol that delivers it. SKILL.md doesn't
declare external dependencies, configuration keys, or permissions;
A3IP does.

**Where A3IP and SKILL.md compose.** A3IP builds directly on
SKILL.md — it adopts the format unchanged and wraps packaging and
install around it. The platforms that support SKILL.md (Claude Code, Codex,
Cursor, GitHub Copilot, Windsurf) automatically support the skill
component inside any A3IP package — the runtime adapter just copies
the SKILL.md into the platform's skills directory and the existing
SKILL.md infrastructure does the rest.

**Adapter status:** native. SKILL.md is A3IP's primary skill format;
no adapter needed.

---

## Anthropic MCP (Model Context Protocol)

MCP is the wire protocol an AI uses to **call external tools at
runtime**: filesystems, databases, APIs, third-party services. MCP
defines a JSON-RPC contract between an AI client and an MCP server;
servers expose tools (functions the AI can call); the client invokes
them during a session.

**Where A3IP and MCP overlap.** Both involve external integrations
the AI needs to talk to. Both declare those integrations in a
machine-readable form.

**Where A3IP differs.** MCP is a **runtime wire protocol**. A3IP is
a **packaging format**. They operate at completely different layers:
A3IP packages declare *which MCPs the workflow requires* in
`manifest.yaml` under `dependencies.mcp:`, but A3IP itself does not
implement MCP. The install AI uses A3IP's manifest to verify
the required MCPs are connected, configure them if missing
(prompting the user during INSTALL.md), and then the workflow runs
against those MCPs at runtime via MCP's own protocol.

**Where A3IP and MCP compose.** An A3IP install plan typically
includes "verify MCP X is connected, prompt user to install if
missing." A3IP adds the missing layer above MCP: the user-confirmed
install of the workflow that uses MCPs, with permissions
pre-declared. The two are complementary — A3IP makes MCP installs
auditable; MCP makes A3IP workflows useful.

**Adapter status:** native. MCP dependencies are declared in the
manifest's `dependencies.mcp:` block; no separate adapter needed.

---

## Summary table

| Standard | What it does | A3IP relation | Adapter status |
|---|---|---|---|
| **Cowork Plugins** | Packaging + install for Cowork-native plugins | Cross-platform sibling; A3IP packages target Cowork first-class | Authored (cowork runtime adapter) |
| **Microsoft APM** | Dependency manager + installer for agent context (resolve, pin, scan, policy, 7 harnesses) | Large overlap; A3IP's only distinct angle is a conversational AI-run install | Not built |
| **SKILL.md** | Single-skill invocation contract | Adopts SKILL.md unchanged as its skill component format | Native |
| **MCP** | Runtime wire protocol for AI-tool calls | Composes — A3IP declares MCP dependencies; MCP delivers them at runtime | Native |

---

## What A3IP is NOT trying to replace

For clarity, A3IP does NOT aim to compete with or replace:

- **The AI runtimes themselves** (Cowork, Claude Code, Codex, Cursor, Copilot, Windsurf). A3IP doesn't execute workflows; the host runtime does.
- **Anthropic's, OpenAI's, or Microsoft's commercial agent products.** A3IP is a format and protocol, not a product.
- **MCP's tool-call protocol.** A3IP and MCP solve different layers.
- **The SKILL.md format.** A3IP adopts SKILL.md verbatim as its skill component.
- **APM.** APM already covers dependency resolution and installation for developers; A3IP only explored a conversational install variant.
- **Cowork Plugins as a distribution channel.** A3IP does not distribute through Cowork's marketplace — the export adapter was proposed, not built.

A3IP's contribution is the **install protocol layer** — manifest with
explicit permissions, CONFIGURE.md wizard, INSTALL.md plan, user
confirmation gate, normative uninstall flow, plain-text bundle
format, cross-runtime portability via adapter files. Existing
standards remain valid inside A3IP without modification.

---

## Trademark acknowledgements

- **MCP** (Model Context Protocol) is a protocol developed by Anthropic.
- **SKILL.md** is an open standard maintained by the Agent Skills community ([agentskills.io](https://agentskills.io)).
- **Microsoft APM** (Agent Package Manager) is a project of Microsoft.
- **Cowork** and **Cowork Plugins** are products of Anthropic.
- **Claude Code**, **Claude**, and **Anthropic** are trademarks of Anthropic, PBC.
- **Codex** and **GitHub Copilot** are trademarks of OpenAI / GitHub respectively.
- **Cursor** is a trademark of Anysphere Inc.

A3IP is an independent open standard and claims no affiliation with or
endorsement by any of these organizations. Compatibility with these
standards is technical, not contractual.

---

*A3IP Specification v1.11 · © 2026 Maksym Prydorozhko · [a3ip.dev](https://a3ip.dev)*
