# A3IP: AI Infrastructure Installation Package
### A standard for sharing, distributing, and installing AI workflows

*by Maksym Prydorozhko*

---

## The Problem

Anyone who has spent serious time building AI-assisted workflows knows how quickly they accumulate invisible complexity.

Take a code review workflow. On the surface, it sounds simple: when a branch is ready, move it to review. But in practice, that single phrase — "move to code review" — might trigger a sequence that verifies the build, pushes the branch, creates a merge request with specific reviewers and an assignee, transitions a Jira ticket, posts a notification to a Teams channel, and logs everything to a persistent inbox artifact. Each step connects to a different external system. Each system requires its own authentication. The AI that executes all of this knows the steps because they were taught to it, encoded in instructions files, skill definitions, and protocol documents that live in a specific directory on a specific developer's machine.

Now try to share that workflow with a colleague.

You could send them the instruction files. But which files? In what order do they go? What are the external dependencies? Which config values need to be filled in for their environment — their GitLab user ID, their Jira project, their Teams channel? And if they're on a different AI platform, does any of it even apply?

There is no standard answer to these questions. AI workflows today are fragile, siloed, and non-transferable. They exist as tribal knowledge encoded into platform-specific configurations that only work for the person who built them.

**A3IP is an attempt to solve this.**

---

## The Idea

A3IP (AI Infrastructure Installation Package) is a format for packaging complete AI workflows into a single transferable unit that any AI agent on any platform can receive, configure, and install.

The analogy is a software installer. When you install an application, a wizard walks you through the steps: choose your install directory, pick which components you want, enter your license key. At the end, everything is in the right place and the application works. You didn't need to understand the internals — the installer handled the complexity.

A3IP packages work the same way, except the installer is an AI. You hand the AI a package, it reads the instructions, asks you the questions it needs answered, and sets everything up in your workspace — adapted to your environment and your platform.

The key insight is that the installation instructions are written *for an AI*, not a human. That changes what's possible. The AI can reason about what's available, gracefully skip steps that don't apply to its platform, ask only the questions that are relevant given the dependencies it finds, and confirm what was set up at the end.

---

## A Spec for an AI, Not a Parser

A3IP's specification sits in an unusual position. Most software specs are written for deterministic parsers — HTML for browsers, JSON Schema for validators, the C language for compilers. In those worlds, the parser cannot reason. The spec is strict precisely because the parser cannot make judgment calls; it can only check rules. Every ambiguity becomes a parsing error.

A3IP packages, by contrast, are read and acted upon by general-purpose reasoning AIs. That changes the design tradeoff at the foundation.

A strict spec applied to a reasoning agent doesn't just remove ambiguity — it removes the agent's room to apply context. A spec that enumerates every file an installer should copy, every command it should run, every fallback it should consider, prevents the AI from doing what it does best: reasoning about the specific situation in front of it. The strict spec becomes a straitjacket, not a safety rail.

But a spec that's too vague — too abstract, too hand-waving — fails in the opposite direction. The AI doesn't have enough mental model to make consistent decisions. It falls back to guessing, and its guesses are inconsistent across runs, across installs, across packages.

The right balance is a layered approach. Different parts of the A3IP spec sit at different points on the strict-vs-loose continuum, deliberately:

**Tier 1 — Mechanical structure.** File layouts, schemas, format conventions. These are strict. They describe shapes that have no context-dependent right answer: `manifest.yaml` lives at the root of every package; `components/skills/<name>/SKILL.md` is where skills go; the `.a3ip.bundle` delimiter syntax is fixed. The validator checks these.

**Tier 2 — Required outcomes.** Behaviors that must happen during install or upgrade, where the WHAT is mandated but the HOW is left to the AI. Step 1 of every package's INSTALL.md must check for an existing installation; the AI chooses which file-reading tool to use given its host environment. These are semi-strict — precise in their outcome, loose in their implementation.

**Tier 3 — Semantics.** Descriptions of what each piece of a package IS — what role it plays at runtime, what it's for. The `docs/` directory contains reference material the skill may read at runtime. The `scripts/` directory contains executable code the skill invokes. The `adapters/` directory contains platform-specific install guidance. These are deliberately loose. The spec describes the meaning; the AI reasons about how that meaning applies to whatever situation it's installing into. The validator does NOT check these — there's no oracle for "did the AI make the right judgment call." That's the AI's job.

This layered structure is the conscious answer to "how strict should the spec be?" The answer is: as strict as it can be without preventing the AI from doing its job. Tier 1 should be machine-checkable. Tier 2 should describe outcomes, not procedures. Tier 3 should give the AI a mental model and trust it to apply that model to the specific case in front of it.

The discipline that follows from this is uncomfortable for spec authors. The temptation, when ambiguity surfaces, is to add a rule. *"What if the AI doesn't include the `docs/` directory in the install? Let's add a manifest field that lists exactly what to install."* That's the wrong reflex. The right reflex is: *"Did we describe what `docs/` IS clearly enough that the AI can reason about whether to include it?"* If yes, the spec is doing its job. If no, fix the semantic description — don't add a mechanical rule.

This also means A3IP packages are not fully self-describing in the npm or Docker sense. A reader cannot tell from `manifest.yaml` exactly what gets installed at runtime. They have to read the skill's instructions and reason about what those instructions reference. That feels strange from a traditional-spec perspective. It's the consistent application of the same trust we already place in the AI for every other Tier 3 question.

The spec is a teaching artifact for a reasoning agent, not an instruction set for a deterministic parser. Designing it well means choosing, deliberately, which tier each question belongs to — and resisting the urge to over-formalize.

---

## What a Package Contains

An A3IP package is a directory (or zip archive) with a predictable structure:

```
my-workflow.a3ip/
├── manifest.yaml      — what's in the package and what it needs
├── CONFIGURE.md       — the installation wizard (AI-readable)
├── INSTALL.md         — step-by-step installation guide (AI-readable)
├── CHANGELOG.md       — per-version upgrade guide (AI-readable)
├── README.md          — human-readable overview
├── components/
│   ├── skills/        — procedural instructions the AI follows
│   ├── protocols/     — named commands with trigger phrases and steps
│   ├── artifacts/     — persistent UI views (like dashboards or inboxes)
│   └── prompts/       — reusable prompt templates
└── scripts/           — supporting code (Python, PowerShell, etc.)
```

Every file is plain text. A human can open any file in the package and understand it. There are no binary formats, no compiled artifacts, no platform-specific encoding.

### Four component types

**Skills** are procedural instructions — they tell the AI *how* to do something. A skill for code review might say: when the user asks to move a branch to review, check the build first, then create the MR, then update Jira. Skills follow the [SKILL.md open standard](https://agentskills.io), meaning any A3IP skill is already compatible with Claude Code, Codex, Cursor, GitHub Copilot, and dozens of other AI platforms.

**Protocols** are named commands. They define a trigger phrase ("move to code review"), a list of ordered steps, and what the expected outputs are. Where skills describe general capability, protocols describe a specific workflow with a specific shape.

**Artifacts** are persistent views — things like a review inbox that shows pending items, or a dashboard that displays project status. Artifacts have a description of what they do and what data they show, plus optionally an HTML implementation for platforms that support rendered views. On platforms that don't support HTML artifacts, the installer adapts the artifact to a simpler format.

**Prompts** are reusable templates. A prompt might define the structure for a GitLab MR description, or the format for a code review comment. They support both static text and variable substitution.

---

## The Installation Wizard

The most distinctive feature of A3IP over existing packaging standards is `CONFIGURE.md`: a structured wizard that the installing AI runs before setting anything up.

The concept comes from software installers. Before an installer touches your filesystem, it asks: where do you want to install? Which components do you want? What's your license key? A3IP packages do the same thing for AI workflows: before registering any skills or writing any config files, the AI collects the information it needs from the user.

For a code review workflow, the wizard might ask:

> "What's your GitLab user ID? I'll use this to assign MRs to you."
> "Who should be the reviewers? Please give me their names and GitLab IDs."
> "What Teams channel should I monitor for incoming review requests?"
> "What's your Jira project key?"

The answers are stored as configuration values and substituted throughout the package using `{{config.key}}` syntax — similar to template variables, but resolved at install time rather than at runtime. Every protocol, prompt, and script that needs to know your reviewer IDs gets them baked in during installation.

`CONFIGURE.md` is written as instructions to the AI, not as a rigid form schema. This means the AI can conduct the wizard conversationally, ask follow-up questions, validate answers, resolve IDs from available tools (for example, looking up a Teams channel ID from the channel name using the Teams MCP), and only ask questions that are relevant given what dependencies are available.

Sensitive values — API tokens, Personal Access Tokens — are handled carefully. The wizard is instructed not to echo them back or include them in the confirmation summary, only to confirm they were received.

---

## Platform Agnosticism

The central design goal of A3IP is that a package should work regardless of which AI platform installs it.

This is harder than it sounds, because platforms differ significantly. Claude Code runs in a terminal. Cowork runs as a desktop application with support for rendered artifacts. Codex operates in a code editor context. GitHub Copilot lives inside VS Code. Each has different notions of what a "skill" is, how commands are registered, and whether persistent UI views are possible.

A3IP handles this through two mechanisms.

First, components are described at the *intent level*. A protocol says what should happen and in what order, not how to implement it on a specific platform. The installing AI translates the protocol into whatever form its platform supports — a slash command, a remembered instruction, a workflow node.

Second, `INSTALL.md` contains explicit fallback instructions for every component that might not be supported. If a platform doesn't support HTML artifacts, the installer is told: maintain a markdown file with the same schema instead. If a dependency MCP isn't available, the installer is told: skip this step and note that the workflow will be degraded.

This is a practical form of graceful degradation. A package never fails completely just because one feature isn't supported. It installs what it can, skips what it can't, and tells the user what's missing.

---

## Relationship to Existing Standards

A3IP is designed as a superset of two existing standards, not a competitor to them.

[SKILL.md](https://agentskills.io) is an open standard, originally developed by Anthropic, for packaging AI agent skills as portable markdown files. It's widely supported across AI platforms. A3IP adopts SKILL.md for its skill components without modification — any valid SKILL.md skill works inside an A3IP package unchanged.

[Microsoft APM](https://microsoft.github.io/apm/) (Agent Package Manager) is the closest existing tool to A3IP. Its `apm.yml` manifest declares skills, instructions, prompts, MCP servers, and other agent dependencies. A3IP extends APM's dependency model and is designed to be compatible with it.

What A3IP adds that neither standard covers:

- **Artifacts** as a first-class component type, including both an HTML implementation and a platform-agnostic fallback
- **Protocols** as a distinct component — named commands with trigger phrases, ordered steps, and outputs
- **The installation wizard** (`CONFIGURE.md`) — AI-readable instructions for collecting user-specific configuration before setup
- **`{{config.*}}` substitution** — values collected during the wizard are baked into all components at install time
- **Sensitive field handling** — the `sensitive: true` flag on config fields instructs the AI not to echo or log the value
- **Delta upgrades** (`CHANGELOG.md` + `installed.json`) — the AI knows what version is installed and can apply precisely the changes needed, not a full reinstall
- **Stale config refresh** — config values that go stale over time (OAuth tokens, sprint dates) can be marked with a `refresh:` flag so the AI knows when to prompt for renewal
- **Non-developer platforms** — explicit support for knowledge-worker contexts like Cowork, not just developer tools

---

## A Real Example: Acme Code Review Flow

The first A3IP package produced as a proof of concept packages the code review workflow from a real software team's AI workspace.

The workflow has two main commands. "Move to code review" is triggered by a developer when a feature branch is ready: the AI verifies the build, commits and pushes, creates a GitLab MR with configured reviewers, transitions the Jira ticket to Code Review status, and posts a notification to the team's review channel in Microsoft Teams. "Check review requests" polls that same Teams channel for incoming review requests from teammates, fetches the diffs from GitLab, generates structured AI code reviews with severity-labelled inline comments (🔴 BLOCKER / 🟡 SHOULD FIX / 🟢 NIT), and surfaces them in a Review Inbox artifact for approval before posting.

Packaging this workflow required:

- 1 skill (the AI's understanding of when and how to run the flow)
- 2 protocols (one for each command)
- 1 artifact (the Review Inbox)
- 2 prompt templates (MR description format, review comment format)
- 9 scripts (PowerShell and Python, for GitLab API calls, Teams integration, diff parsing, state management)
- 12 configuration values collected at install time — including GitLab user IDs for the assignee and reviewers, Teams channel and team IDs, Jira project key and transition ID, a GitLab Personal Access Token (marked sensitive), sprint dates, and local directory paths

The package was tested against Codex, which flagged several inconsistencies in the first version: missing scripts, config values collected in the manifest but not in the wizard, parameters declared in a protocol but not defined in the script. These were fixed, and the exercise produced a real finding for the spec: the `sensitive: true` flag on configuration fields, which hadn't been designed upfront but was obviously needed once a PAT was being collected.

This is the value of packaging a real workflow: the spec becomes concrete, and gaps become visible.

---

## What This Enables

If A3IP works as intended, a few things become possible that aren't today.

**Workflow sharing across teams.** A developer who has built a well-functioning code review flow, a ticket triage workflow, or a documentation generation process can share it with colleagues as a single file. The recipient's AI installs it, adapted to their environment, without the originator needing to explain every step.

**Workflow libraries.** Teams or communities could maintain registries of A3IP packages for common workflows — the way npm has packages for common JavaScript patterns. A new team starting with GitLab and Jira could install a code review package in minutes rather than spending weeks building one from scratch.

**AI platform independence.** A workflow built for Claude Code today could be installed on Codex or Cursor tomorrow. The package format is the portability layer. Teams are not locked into a single AI platform because their workflows are encoded in something portable.

**Knowledge capture.** Much of what makes an experienced developer productive is implicit — the protocols they follow, the patterns they apply, the checklists they run through. A3IP gives a format for making that knowledge explicit and transferable, not just to humans but to AI agents.

---

## Versioning and Upgrades

Software packages get updated. Workflows change. A script that worked against last month's GitLab API may need updating. A new step gets added to the protocol. An artifact gets a better UI. When that happens with a conventional piece of software, there's a clear path: download the update, run the installer, done. With AI workflows, there was no equivalent — until now.

A3IP v1.1 introduces a first-class versioning model.

When a package author releases a new version, they include a `CHANGELOG.md` alongside the updated bundle. Each version entry in that file has two parts: a human-readable summary of what changed, and an AI-readable set of upgrade steps — concise instructions, written exactly like an `INSTALL.md` section, covering only what the AI needs to do differently for this version.

At install time, the AI writes an `installed.json` file to the user's config directory. It records the package name, the installed version, the timestamp, and the platform. This is a small but critical piece of state: it's what allows the AI to answer the question "what's already here?" the next time a bundle arrives.

When a new bundle is handed to the AI and `installed.json` already exists, the AI goes to the `## Upgrading` section of `INSTALL.md` rather than running the full 14-step install from scratch. It reads the changelog entries for every version between what's installed and what's in the bundle, applies them in order, and updates `installed.json` at the end.

This distinction between install and upgrade matters in practice. When a new version fixes the Review Inbox artifact, the upgrade path is: replace `artifact.html`, re-register the artifact. It does not involve re-running the configuration wizard, re-copying all the scripts, or re-registering the protocols. The AI does exactly the minimum required — no more.

The versioning model also distinguishes between breaking and non-breaking changes. A new optional config key (with a default value) is non-breaking: existing installations keep working. A renamed or removed config key is breaking: the old value in `config.json` is now wrong, and the AI must re-ask for the affected fields. A new required MCP dependency is breaking: the workflow won't work without it. The `CHANGELOG.md` format requires these to be declared explicitly under a `### Breaking changes` section, so there's no ambiguity.

Packages that go through multiple contributors follow the same discipline as any open-source project's `CHANGELOG.md`: whoever merges the change writes the entry. The A3IP Creator tool generates a template for each new version entry so the format stays consistent.

## Current Status

A3IP is at the concept and prototype stage. The v1.1 specification has been written, and work is proceeding on two tracks simultaneously.

**Spec track:** v1.0 established the core format — manifest, CONFIGURE.md, INSTALL.md, bundle, four component types, `{{config.*}}` substitution, dependency declarations. v1.1 adds the versioning model — CHANGELOG.md, installed.json, the Upgrading section in INSTALL.md, `refresh:` on config keys, and a formal breaking-change taxonomy.

**Tooling track:** The A3IP Creator — a Cowork skill plus Python scripts — automates the authoring process. It conducts a structured intake conversation, generates the full package directory (manifest, wizard, install guide, component stubs, script stubs), runs completeness validation, and produces the distributable bundle. The Creator exists precisely because hand-authoring the first Acme package revealed nine distinct friction points: config keys falling out of sync with the wizard, scripts declared but never written, OS-specific scripts missing their cross-platform fallbacks. The Creator makes all nine impossible to hit by accident.

What comes next is wider validation: more packages, more platforms, more installs — and exercising the upgrade path in practice, with a real v1.0.0 → v1.1.0 upgrade applied to the Acme package.

---

## Design Principles

**Three-tier specification.** Mechanical structure (file layouts, schemas, formats) is strict and validator-checkable. Required outcomes (install detection, upgrade flow) are mandated in WHAT, loose in HOW. Semantics (what each piece IS) are described, not enumerated — the AI reasons from the description. Resist the urge to formalize Tier 3 questions into Tier 1 rules; that strips the AI of the judgment it's there to apply.

**Platform agnostic by default.** Components describe intent, not implementation. Platform-specific adaptations go in adapters or fallback instructions.

**AI-first installation.** The receiving AI reads the package and installs it through conversation. No CLI, no human-operated toolchain required.

**User-configurable.** Packages are not one-size-fits-all. The wizard collects what it needs to adapt the package to the user's environment before anything is installed.

**Two-variable model.** `{{config.*}}` values are set once at install time and baked into all components. `{{variable}}` values are dynamic, filled each time a protocol runs. These namespaces don't overlap.

**Graceful degradation.** Every dependency declares a fallback. A missing MCP or unsupported platform feature degrades specific steps, not the whole package.

**Human-readable throughout.** Every file is plain text. A person can audit, edit, or fork any part of a package without special tooling.

**Delta upgrades, not re-installs.** A new bundle version gives the AI a precise, scoped upgrade path. The AI applies only what changed — replaces only the files that changed, re-runs only the steps that changed, re-asks only the config questions that changed.

**Installation state is local.** `installed.json` lives with the user's config, not inside the package. The package is a pure distributable artifact. It carries no knowledge of where it has been installed or who is using it.

**Superset, not fork.** SKILL.md skills and APM-compatible manifest blocks are valid inside A3IP without modification. The format extends the existing ecosystem.

---

*A3IP is an open concept. Feedback, collaboration, and criticism are welcome.*
