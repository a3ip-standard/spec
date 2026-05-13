# A3IP Specification v1.0
**AI Infrastructure Installation Package**

> A platform-agnostic format for packaging and sharing complete AI workspace workflows — a superset of the [SKILL.md](https://agentskills.io) open standard and [Microsoft APM](https://microsoft.github.io/apm/).

---

## Overview

An A3IP package bundles everything needed to reproduce a working AI workflow into a single transferable unit: skills, protocols, UI artifacts, prompt templates, and dependency declarations. Any AI agent on any platform can receive an A3IP package and install it into their workspace.

A3IP extends the existing ecosystem rather than replacing it:

| Layer | Existing standard adopted | A3IP addition |
|---|---|---|
| Skills | SKILL.md (Anthropic open standard) | — |
| Dependencies | APM `apm.yml` model | Artifact + protocol component types |
| Tooling | APM CLI compatible | AI-readable `INSTALL.md` |
| Configuration | — | Installation wizard via `CONFIGURE.md` |
| Audience | Developer / coding agents | + Knowledge worker / non-developer workspaces |

---

## Package Structure

An A3IP package exists in one of three distribution forms:

| Form | Extension | When to use |
|---|---|---|
| **Directory** | `my-workflow.a3ip/` | Local development and editing |
| **Zip archive** | `my-workflow.a3ip.zip` | Human-to-human file transfer (email, cloud drive) |
| **Bundle** | `my-workflow.a3ip.bundle` | AI-to-AI transfer — a single plain-text file any AI can read directly |

The bundle format is the **primary AI consumption format**. It embeds all package files inline with text delimiters, requiring no shell access, no unzip, and no filesystem writes. See [Bundle Format](#bundle-format) for details.

All three forms contain the same content and are interchangeable. Package authors should distribute the bundle alongside the zip for maximum compatibility.

The directory layout is:

```
my-workflow.a3ip/
├── manifest.yaml          # Required. Package metadata and dependency declarations.
├── INSTALL.md             # Required. AI-readable installation guide.
├── CONFIGURE.md           # Required if config exists. AI-readable wizard definition.
├── README.md              # Recommended. Human-readable overview.
│
├── components/
│   ├── skills/            # SKILL.md-compatible skill folders (one per skill)
│   │   └── <skill-name>/
│   │       ├── SKILL.md   # Required per skill (follows SKILL.md open standard)
│   │       └── ...        # Optional supporting scripts, templates, resources
│   │
│   ├── artifacts/         # UI artifacts — persistent views the AI manages
│   │   └── <artifact-name>/
│   │       ├── artifact.md    # Required: artifact metadata and description
│   │       └── artifact.html  # Optional: HTML implementation (platform-specific)
│   │
│   ├── protocols/         # Named command/workflow definitions
│   │   └── <protocol-name>.md
│   │
│   └── prompts/           # Reusable prompt templates
│       └── <prompt-name>.md
│
└── adapters/              # Optional. AI-platform and OS-specific overrides.
    ├── windows/           # OS adapter — e.g. PowerShell script implementations
    │   └── scripts/
    ├── macos-linux/       # OS adapter — e.g. bash/sh script implementations
    │   └── scripts/
    ├── claude/            # AI-platform adapter
    ├── codex/
    └── generic/
```

The `adapters/` directory serves two distinct purposes:

- **OS adapters** (`adapters/windows/`, `adapters/macos-linux/`, etc.) — provide OS-specific implementations of scripts when a cross-platform default exists in `scripts/`. The installer detects the OS and picks the best available implementation.
- **AI-platform adapters** (`adapters/claude/`, `adapters/codex/`, etc.) — provide platform-specific hints for registering skills, protocols, and artifacts.

---

## Bundle Format

A `.a3ip.bundle` file is a single plain-text document that embeds every file in the package using a simple delimiter syntax. It requires no special tooling — an AI reads it directly, extracts each embedded file's path and content, and installs from that in-memory representation.

### Why bundles exist

Zip archives are binary. An AI on a web interface, a mobile device, or a sandboxed environment cannot unzip a file without shell access. A bundle solves this: the user pastes it into chat, attaches it as a text file, or sends it via any channel that carries text — and the receiving AI has the full package.

### Delimiter syntax

```
---
a3ip-bundle: "1.0"
package: <name>
version: <semver>
generated: <ISO-8601 timestamp>
files: <total file count>
---

=== FILE: <relative/path/to/file> ===
<raw file content — preserves all whitespace and newlines exactly>
=== END FILE ===

=== FILE: <next/file> ===
...
=== END FILE ===
```

Rules:
- The opening frontmatter is mandatory and must come first.
- Each `=== FILE: <path> ===` line marks the start of an embedded file. The path is relative to the package root.
- Each `=== END FILE ===` line marks the end. Content between these delimiters is taken verbatim — no escaping.
- Files are embedded in any order; the receiver reconstructs the directory tree from paths.
- Binary files (images, compiled assets) should be Base64-encoded and noted with a comment `# encoding: base64` on the line immediately after the `FILE:` marker.

### Generating a bundle

Any AI with filesystem access can generate a bundle from a package directory:

```
For each file in the package (recursively, sorted by path):
  Write: === FILE: <relative path> ===
  Write: <file contents verbatim>
  Write: === END FILE ===
```

### Consuming a bundle

When an AI receives a `.a3ip.bundle` file or its contents:

1. Parse the frontmatter to identify the package name and version.
2. Split on `=== FILE:` / `=== END FILE ===` delimiters to extract each embedded file.
3. Hold the files as an in-memory map of `{ path → content }`.
4. Follow `INSTALL.md` exactly as if the files were on disk — substituting all file reads with lookups from the in-memory map.
5. Write files to disk only when installation explicitly requires it (e.g. config.json, scripts).

### Example (excerpt)

```
---
a3ip-bundle: "1.0"
package: hello-world
version: "1.0.0"
generated: 2026-05-11T10:00:00Z
files: 3
---

=== FILE: manifest.yaml ===
a3ip: "1.0"
name: hello-world
version: "1.0.0"
=== END FILE ===

=== FILE: INSTALL.md ===
---
format: a3ip-install
spec: "1.0"
---
# Installation Guide: Hello World
...
=== END FILE ===
```

---

## manifest.yaml

The manifest is the single source of truth for the package. It declares what the package contains and what it needs.

```yaml
# ─────────────────────────────────────────
# A3IP Manifest
# ─────────────────────────────────────────

a3ip: "1.0"                          # Spec version. Required.
name: "code-review-flow"             # Unique package name. Required.
version: "1.2.0"                     # SemVer. Required.
description: "Complete code review workflow with inbox artifact, GitLab + Jira integration."
author: "Alex Morgan <alex.morgan@example.com>"
license: "MIT"                       # SPDX identifier or "proprietary"
homepage: "https://..."              # Optional. Docs or repo URL.

# ─────────────────────────────────────────
# Components included in this package
# Each entry is a path relative to package root.
# ─────────────────────────────────────────

components:
  skills:
    - path: components/skills/code-review
      description: "Guides the AI through the code review protocol step by step."

  artifacts:
    - path: components/artifacts/review-inbox
      description: "Persistent inbox view showing all open review requests."

  protocols:
    - path: components/protocols/move-to-review.md
      description: "The 'move to code review' command — initiates the review flow."

  prompts:
    - path: components/prompts/review-summary.md
      description: "Template for generating a review summary comment."

  scripts:
    # Scripts follow a primary-plus-adapters pattern.
    # The primary implementation should be cross-platform (e.g. Python).
    # OS-specific alternatives live in adapters/<os>/scripts/ and are preferred
    # when the installer detects the matching OS.
    - key: create_mr
      description: "Creates a GitLab MR via REST API and returns its URL."
      parameters: [ProjectPath, SourceBranch, TargetBranch, Title, Description, AssigneeId, ReviewerIds]
      implementations:
        - file: scripts/create_mr.py
          platform: any              # cross-platform default
        - file: adapters/windows/scripts/create_mr.ps1
          platform: windows          # preferred on Windows

    - key: post_teams_message
      description: "Posts a review request notification to the Teams channel."
      parameters: [MrUrl, JiraId, JiraTitle, JiraUrl]
      implementations:
        - file: scripts/post_teams_review_request.py
          platform: any
        - file: adapters/windows/scripts/post_teams_review_request.ps1
          platform: windows

# ─────────────────────────────────────────
# External dependencies
# Things the package needs that are NOT bundled inside it.
# ─────────────────────────────────────────

dependencies:
  mcp:
    - name: gitlab
      required: true
      purpose: "Read merge requests, post review comments, check pipeline status."
      registry: "mcp-registry/gitlab"        # hint for auto-install if supported
      fallback: "Manual: paste MR diff into chat when GitLab MCP is unavailable."

    - name: jira
      required: false                        # false = graceful degradation
      purpose: "Link review to Jira issue, update ticket status."
      registry: "mcp-registry/jira"
      fallback: "Skip Jira linking step if MCP is unavailable."

  skills:
    # Skills from other A3IP packages or the skills.sh registry
    - name: "git-basics"
      source: "skills.sh/git-basics@^2.0"
      required: false

# ─────────────────────────────────────────
# Configuration schema
# Defines what the AI must collect from the user before/during installation.
# Collected values are referenced throughout components as {{config.<key>}}.
# ─────────────────────────────────────────

configuration:
  - key: api_token
    label: "Service API Token"
    description: "Personal Access Token for authenticating API calls."
    type: string
    required: true
    sensitive: true                 # AI must not echo or log this value — confirm receipt only
    placeholder: "token-xxxxxxxxxxxx"
    when: before

  - key: reviewers
    label: "GitLab Reviewers"
    description: "GitLab usernames or numeric IDs of people to assign as reviewers on MRs."
    type: list<string>              # string | list<string> | boolean | number | select | multi-select
    required: true
    example: ["@john.doe", "@jane.smith"]
    placeholder: "GitLab username, e.g. @john.doe"
    when: before                    # before | during | after (default: before)

  - key: assignee
    label: "Default MR Assignee"
    description: "GitLab username to auto-assign MRs to. Leave blank to skip."
    type: string
    required: false
    default: ""
    placeholder: "@username"
    when: before

  - key: jira_project_key
    label: "Jira Project Key"
    description: "The Jira project key where tickets will be linked (e.g. PROJ, ACME)."
    type: string
    required: false                 # Optional because jira MCP dependency is also optional
    default: ""
    placeholder: "PROJ"
    when: before
    condition: "jira MCP is available"   # Only ask if this condition is met

  - key: components
    label: "Components to install"
    description: "Choose which parts of the package to install."
    type: multi-select
    required: true
    when: before
    options:
      - value: artifacts
        label: "Review Inbox artifact"
        default: true
      - value: jira_integration
        label: "Jira integration"
        default: true
      - value: gitlab_comments
        label: "Auto-post GitLab review comments"
        default: true

# ─────────────────────────────────────────
# Platform compatibility
# Informational — the package aims to be platform-agnostic,
# but authors may note where it has been tested.
# ─────────────────────────────────────────

platforms:
  tested:
    - cowork          # Anthropic Cowork (knowledge worker desktop)
    - claude-code     # Anthropic Claude Code (CLI)
    - codex           # OpenAI Codex
    - cursor          # Cursor IDE
  untested:
    - copilot
    - gemini-cli
```

---

## CONFIGURE.md

`CONFIGURE.md` defines the installation wizard. It instructs the AI on how to collect configuration values from the user — what to ask, in what order, how to present options, and what to do with the answers. Think of it as the script for an installation wizard dialog, but executed by an AI in natural conversation.

### Configuration types

| Type | Description | Example value |
|---|---|---|
| `string` | Single text value | `"@john.doe"` |
| `list<string>` | Multiple text values, one per line or comma-separated | `["@john.doe", "@jane.smith"]` |
| `boolean` | Yes / No | `true` |
| `number` | Numeric value | `5` |
| `select` | One choice from a fixed list | `"main"` |
| `multi-select` | Multiple choices from a fixed list | `["artifacts", "jira_integration"]` |

### Timing (`when` field)

| Value | Meaning |
|---|---|
| `before` | Collected upfront before any installation step begins (default) |
| `during` | Collected inline at the relevant installation step |
| `after` | Collected at the end for optional fine-tuning |

### Sensitive fields (`sensitive: true`)

Any configuration field may be marked `sensitive: true`. This signals the installing AI to:

- **Not echo** the value back to the user at any point (not in prompts, not in the confirmation summary, not in logs)
- **Confirm receipt** only — e.g. "✅ API token received and will be stored securely."
- **Not substitute** `{{config.<key>}}` into any human-visible output; only write it to config files or use it in script arguments
- **Store securely** — write to the config file on disk; do not keep in conversation memory

Typical use cases: API tokens, Personal Access Tokens, passwords, OAuth secrets.

```yaml
- key: gitlab_pat
  label: "GitLab Personal Access Token"
  type: string
  required: true
  sensitive: true
  placeholder: "glpat-xxxxxxxxxxxxxxxxxxxx"
  when: before
```

The `sensitive` flag is purely a behavioural instruction to the AI — the value is still stored in plain text in config.json. For higher security requirements, package authors should note this and instruct users to use a secrets manager instead.

### Variable reference syntax

Collected values are referenced anywhere in the package using `{{config.<key>}}`. The installing AI substitutes real values before using any component. For lists, `{{config.reviewers}}` expands to a comma-separated string by default; use `{{config.reviewers[0]}}` for a specific index.

### CONFIGURE.md example

```markdown
---
format: a3ip-configure
spec: "1.0"
package: code-review-flow
---

# Configuration Wizard: Code Review Flow

You are setting up a Code Review workflow. Before installing, I need a few details
about your team. Ask the user each question below in order. Be conversational —
you do not need to present this as a form, but make sure you collect every required
value before moving to installation.

## Required — ask before installation

### 1. API Token (sensitive)
Ask: "I need an API token to authenticate. Please create one in your account
settings and paste it here."

- Key: `api_token`
- Type: string
- Sensitive: true — do NOT echo the value back. Once received, confirm only:
  "✅ Token received and will be stored in config.json."
- Do NOT include `{{config.api_token}}` in the confirmation summary.

### 2. GitLab Reviewers
Ask: "Who should be assigned as reviewers on merge requests? Please provide their
GitLab usernames (e.g. @john.doe) or numeric user IDs. You can list multiple."

- Key: `reviewers`
- Type: list of strings
- Validation: each entry must start with @ or be a positive integer
- Error: "I need at least one reviewer to proceed. Please provide a GitLab username."

### 2. Default MR Assignee
Ask: "Should MRs be auto-assigned to a specific person, or leave assignee blank?
If yes, provide their GitLab username."

- Key: `assignee`
- Type: string (optional — blank is valid)
- If blank: store as empty string, skip assignee step during protocol execution

## Conditional — ask only if Jira MCP is available

### 3. Jira Project Key
Ask: "What is your Jira project key? This is the short prefix on your ticket numbers
(e.g. if your tickets look like ACME-123, the key is ACME)."

- Key: `jira_project_key`
- Type: string
- Validation: uppercase letters only, 2–10 characters
- Condition: only ask if the Jira MCP dependency is available

## Optional — ask before installation

### 4. Components to install
Ask: "Which components would you like to install? I'll enable all by default —
let me know if you want to skip any."

- Key: `components`
- Type: multi-select
- Options:
  - `artifacts` — Review Inbox artifact (recommended)
  - `jira_integration` — Jira ticket linking
  - `gitlab_comments` — Auto-post review comments to GitLab
- Default: all enabled

## After collecting all values

Summarize what you collected and ask the user to confirm before proceeding:

"Here's what I'll set up:
- Reviewers: {{config.reviewers}}
- Assignee: {{config.assignee || 'none'}}
- Jira project: {{config.jira_project_key || 'not configured'}}
- Components: {{config.components}}

Shall I proceed with installation?"

Wait for confirmation before continuing to INSTALL.md.
```

---

## INSTALL.md

`INSTALL.md` is written **for an AI agent**, not a human. It is the key innovation of A3IP over APM and SKILL.md. It provides the receiving AI with enough context to adapt and install the package on whatever platform it runs on — including graceful degradation when features are unsupported.

### Format

`INSTALL.md` uses YAML frontmatter for machine-readable metadata and a Markdown body for natural-language instructions an AI can follow.

```markdown
---
format: a3ip-install
spec: "1.0"
package: code-review-flow
---

# Installation Guide: Code Review Flow

## 1. What This Package Does

This package installs a complete code review workflow. After installation, a team
member can say "move to code review" to trigger a protocol that:
- Creates a review entry in the Review Inbox artifact
- Posts a structured summary to the linked GitLab MR
- Updates the associated Jira ticket status

## 2. Dependency Check

Before installing, verify the following. Tell the user what's missing.

**Required:**
- [ ] GitLab MCP — check if you can call a GitLab tool. If not, ask the user
      to install it. Without this, the core workflow cannot function.

**Optional:**
- [ ] Jira MCP — check if you can call a Jira tool. If unavailable, note that
      Jira linking will be skipped and install the rest normally.

## 3. Run Configuration Wizard

Before installing any components, read and follow `CONFIGURE.md` to collect
user-specific values. Do not proceed to step 4 until all required config values
are confirmed by the user.

Config values collected here will be referenced as {{config.<key>}} in subsequent
steps. Substitute real values wherever you see this syntax.

## 4. Install Skills

Copy each folder under `components/skills/` into your platform's skills directory.
If your platform does not have a skills directory concept, load the SKILL.md
content as a persistent instruction set in your system context.

## 5. Set Up Artifacts

The Review Inbox (`components/artifacts/review-inbox/`) is a persistent UI view.

- **If your platform supports HTML artifacts** (e.g., Cowork, Claude.ai):
  Create a new artifact using `artifact.html` as the source.

- **If your platform does not support HTML artifacts** (e.g., Codex, CLI tools):
  Use the description in `artifact.md` to maintain an equivalent in-memory
  or file-based structure (e.g., a markdown table in a tracked file).

## 6. Register Protocols

Read each file in `components/protocols/`. Each protocol defines a named command
and its trigger phrase, steps, and expected outputs. Substitute all `{{config.*}}`
references with the values collected in step 3 before registering.

Register these as:
- A slash command or skill (if your platform supports it)
- A remembered instruction ("when the user says X, follow these steps")
- A workflow node (if your platform has visual workflow builders)

## 7. Load Prompts

Files in `components/prompts/` are reusable templates. Substitute all `{{config.*}}`
references with collected values. Load them as:
- Named prompt templates (if supported)
- Inline context the AI remembers and applies when appropriate

## 8. Confirm Installation

After completing the steps above, summarize to the user:
- What was successfully installed
- Configuration values in effect (reviewers, assignee, etc.)
- What dependencies were missing and how the workflow is degraded as a result
- How to trigger the main workflow (the primary trigger phrase)

## Platform-Specific Notes

See `adapters/` for hints tailored to specific platforms. If your platform
is not listed, follow the generic instructions above — the package is designed
to degrade gracefully.
```

---

## Component Formats

### Skill (`components/skills/<name>/SKILL.md`)

Follows the [SKILL.md open standard](https://agentskills.io) exactly. A3IP adds no extra requirements — any valid SKILL.md is a valid A3IP skill component.

```markdown
---
name: code-review
description: "Step-by-step protocol for conducting and logging a code review."
version: "1.0.0"
---

## When to use this skill
Use when the user asks to start a code review, move an MR to review, or conduct a peer review.

## Steps
[full skill instructions here]
```

### Artifact (`components/artifacts/<name>/artifact.md`)

```markdown
---
name: review-inbox
description: "Persistent inbox listing all open code review requests with status and links."
type: ui-view                    # ui-view | data-store | dashboard
refresh: on-demand               # on-demand | scheduled | realtime
---

## Purpose
Displays all open MRs awaiting review in a structured, scannable view.

## Data Sources
- GitLab MCP: open merge requests assigned to team
- Jira MCP: linked ticket status

## Fields
| Field | Source | Description |
|---|---|---|
| MR Title | GitLab | Title of the merge request |
| Author | GitLab | Who opened the MR |
| Jira Ticket | Jira | Linked issue key and status |
| Waiting Since | GitLab | Created date |
| Review Status | local | One of: Pending, In Review, Approved, Changes Requested |

## Fallback (no HTML artifact support)
Maintain a markdown table in `review-inbox.md` with the same columns.
Update it each time the review protocol runs.
```

### Protocol (`components/protocols/<name>.md`)

```markdown
---
name: move-to-code-review
trigger: "move to code review"     # phrase(s) that activate this protocol
aliases:
  - "start review"
  - "request review"
---

## What this protocol does
Moves a development task from "in progress" to "code review" state,
notifying relevant parties and updating all connected systems.

## Steps

1. **Identify the MR** — Ask the user for the MR URL or number if not provided.
2. **Validate** — Check via GitLab MCP that the MR is in a reviewable state (not draft, CI passing or acknowledged).
3. **Assign reviewers** — Assign {{config.reviewers}} as reviewers on the MR via GitLab MCP.
   If {{config.assignee}} is set, assign it as the MR assignee.
4. **Update GitLab** — Post a structured review-request comment using the `review-summary` prompt template.
5. **Update Jira** — Transition the linked ticket in project {{config.jira_project_key}} to "In Review" status.
   Skip this step if Jira MCP is unavailable or `jira_project_key` is not configured.
6. **Update inbox** — Add or update the entry in the Review Inbox artifact.
7. **Confirm** — Tell the user what was done and who was assigned as reviewer.

## Outputs
- GitLab reviewers assigned: {{config.reviewers}}
- GitLab comment posted
- Jira ticket transitioned (if configured)
- Review Inbox updated
```

### Prompt Template (`components/prompts/<name>.md`)

```markdown
---
name: review-summary
description: "Structured comment posted to GitLab when a review is requested."
variables:
  - mr_title        # runtime variable: provided when protocol runs
  - author          # runtime variable
  - jira_key        # runtime variable
  - ready_checklist # runtime variable
# Note: {{config.*}} values are substituted at install time.
# Runtime {{variables}} are substituted each time the protocol executes.
---

## Template

🔍 **Code Review Requested**

| | |
|---|---|
| MR | {{mr_title}} |
| Author | {{author}} |
| Jira | {{jira_key}} |
| Reviewers | {{config.reviewers}} |

**Ready checklist:**
{{ready_checklist}}

Please review at your earliest convenience. Tag me with questions.
```

---

## Versioning

A3IP follows [SemVer](https://semver.org):

- **Major**: breaking changes to the manifest or INSTALL.md format
- **Minor**: new component types or manifest fields (backward compatible)
- **Patch**: clarifications, fixes to existing definitions

Package versions follow SemVer independently of the spec version.

---

## Relationship to Existing Standards

| Concept | SKILL.md | APM | A3IP |
|---|---|---|---|
| Portable skills | ✅ Core format | ✅ Supports | ✅ Adopts SKILL.md exactly |
| Dependency declarations | ❌ | ✅ apm.yml | ✅ Extends APM model |
| MCP server configs | ❌ | ✅ | ✅ Inherited from APM |
| UI Artifacts | ❌ | ❌ | ✅ First-class component |
| Named protocols / commands | ❌ | ❌ | ✅ First-class component |
| AI-readable install guide | ❌ | ❌ | ✅ INSTALL.md |
| Installation wizard | ❌ | ❌ | ✅ CONFIGURE.md with typed schema |
| User config substitution | ❌ | ❌ | ✅ `{{config.*}}` syntax |
| Knowledge-worker platforms | ❌ | ❌ | ✅ Explicit design goal |
| CLI / developer platforms | ✅ | ✅ | ✅ |
| Graceful degradation | ❌ | ❌ | ✅ Per-dependency fallback |

---

## Design Principles

1. **Platform agnostic by default.** Components are described at intent level. AI-platform-specific implementations go in `adapters/<platform>/`.
2. **OS agnostic by default.** Scripts must provide a cross-platform implementation (e.g. Python) as the primary. OS-specific alternatives (e.g. PowerShell for Windows, bash for macOS/Linux) live in `adapters/<os>/scripts/` and are used when the installer detects a matching OS. The installer always falls back to the cross-platform default.
3. **AI-first installation.** The receiving AI reads `INSTALL.md` and performs setup. No human-operated CLI required.
4. **User-configurable.** `CONFIGURE.md` turns installation into a conversation. The AI collects what it needs from the user, validates answers, and substitutes values throughout the package via `{{config.*}}`.
4. **Two-variable model.** `{{config.*}}` values are set once at install time. `{{variable}}` values are filled at runtime each time a protocol runs. These two namespaces are distinct and never mixed.
6. **Graceful degradation.** Every dependency declares a fallback. A package installs as much as possible even on platforms with missing features. Optional config values that depend on unavailable MCPs are skipped automatically.
7. **Composable.** A3IP packages can declare other A3IP packages or skills.sh skills as dependencies.
8. **Plain text end-to-end.** Every component file is plain text. The `.a3ip.bundle` format embeds the entire package as a single readable text file — no binary formats, no zip required. A human can open and understand any file. An AI on any platform can consume a bundle with no tooling.
9. **Superset, not fork.** Any valid SKILL.md skill and any APM-compatible manifest block is valid inside A3IP.

---

*A3IP Specification v1.0 — Draft*
