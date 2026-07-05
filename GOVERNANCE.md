# A3IP Governance

How the A3IP specification, reference toolchain, and reference package
registry are maintained. Read this before opening an issue, proposing a change,
or asking who decides what.

This document is honest about the current stage of the project: A3IP
is **single-maintainer** as of this writing. The governance described
below is the model the project commits to as it grows; today most
roles are filled by one person. Where the future-state model differs
from the current-state model, both are stated explicitly.

---

## Who maintains this

**Maksym Prydorozhko** ([@maksymprydorozhko](https://github.com/maksymprydorozhko)) — author of the specification, maintainer of the reference toolchain (`a3ip` CLI, `a3ip-creator`), and current sole gatekeeper of the project's reference package registry.

Contact: open a GitHub issue in the relevant repo (see [Repositories](#repositories) below). Direct email is intentionally not the channel — issue tracking keeps the project's record public.

A long-term goal is to transition maintenance to a Dutch Stichting (foundation) once the project has demonstrable adoption and at least three independent implementations of the spec. Until then, treat the maintainer's voice as the project's voice.

---

## How changes are proposed

Different change types use different paths. The discriminator is **whether the change affects how compliant tools must behave**.

### Normative spec changes (binding on implementers)

Anything that changes what a compliant A3IP implementation MUST or MUST NOT do — adding required manifest fields, changing INSTALL.md semantics, renaming components, modifying the bundle format, etc.

**Process:**

1. Open a **GitHub issue** in [`a3ip-standard/spec`](https://github.com/a3ip-standard/spec) titled `[Proposal] <short description>`. State the problem, the proposed change, and the motivating use case.
2. Two-week **public comment period**. The issue stays open for comment from anyone. The maintainer participates in the discussion and steers toward consensus or clarifies tradeoffs.
3. Maintainer drafts a spec patch as a pull request. The PR cites the originating issue.
4. The PR sits open for at least **one additional week** to give late commenters a chance to flag objections.
5. Maintainer merges if no blocking objection surfaces, or returns to step 1 if the proposal needs material revision.

### Non-normative spec clarifications

Wording fixes, expanded examples, fixed-up references, typo corrections, additional rationale paragraphs.

**Process:** open a PR directly. No comment period. Maintainer merges if the change is genuinely non-normative.

### Reference toolchain changes (`a3ip` CLI, `a3ip-creator`)

The reference toolchain is **versioned independently** of the spec. CLI behavior may evolve faster than the spec; non-breaking CLI improvements don't require spec changes.

**Process:** open a PR in the relevant repo ([`a3ip-standard/cli`](https://github.com/a3ip-standard/cli) or [`a3ip-standard/creator`](https://github.com/a3ip-standard/creator)). Maintainer reviews. If the change implies a spec change too, the spec PR happens first.

### New packages in the reference registry

A3IP is **registry-neutral**: the standard defines packages and how they install, not how they are distributed. [`a3ip-standard/packages`](https://github.com/a3ip-standard/packages) is the project's **reference registry** — one registry among possibly many. Anyone, including AI-platform vendors, can host their own A3IP registry; publishing here is never required for a package to be valid, it simply lists the package in the maintained reference gallery.

Anyone can submit a package to the reference registry.

**Process:** see [CONTRIBUTING.md](https://github.com/a3ip-standard/packages/blob/main/CONTRIBUTING.md) in that repo. Acceptance criteria: passes `a3ip validate` with zero errors, has a README explaining what the package does, is not company-specific (no internal URLs, no private API endpoints, no hardcoded team names).

---

## How versions are released

A3IP follows **semantic versioning** with explicit semantics for each version dimension.

### Spec versions (e.g. v1.10)

- **MAJOR** (v2.0, v3.0): breaking changes — implementations need code updates to remain compliant. Bundles built against an older major version may not install on a runtime targeting the newer major. Decided deliberately, after a feedback period.
- **MINOR** (v1.10, v1.11): new features that don't break existing implementations. New optional manifest fields, new INSTALL.md template steps, new spec clarifications. Bundles built against an older minor still install on newer-minor runtimes.
- **PATCH**: not used at the spec level. Spec patches are folded into the next minor.

The CHANGELOG at the top of each spec version document names the breaking changes (if any) and the migration path.

### CLI versions (e.g. v1.5.2)

Standard semver. Major = breaking CLI behavior; minor = new features; patch = bug fixes. The CLI declares which spec version it targets via `__spec_version__`; spec compatibility is independent of CLI version.

### Creator versions (e.g. v3.0.0)

Same standard semver. Each Creator release pins its minimum CLI version via `dependencies.tools.a3ip` in its manifest.

### Sample packages

Versioned independently. Sample A (`ai-code-review-flow`), Sample B (`ai-research-workspace`), and Sample C (`ai-standup-assistant`) each have their own version trajectory.

---

## Who merges what

**Current state (single-maintainer):** Maksym Prydorozhko merges everything across all repos in the `a3ip-standard` organization. There is no separate review committee, no rotating quorum, no maintainer team.

**Future state (when 3+ independent implementations exist):** the project will form a steering committee with representation from at least 3 distinct implementers. The committee will set merge policy, with explicit roles for spec stewardship, toolchain maintenance, and registry curation. Until that threshold is reached, the future-state structure is aspirational — don't cite it as current process.

**Conflict of interest disclosure:** the maintainer is also the primary author of the reference toolchain and the most-used packages in the registry. This is the normal state for an early open standard; it's named here so participants can factor it into their engagement.

---

## Trademark and intellectual property

**Specification text:** Licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE-SPEC.md). You may read, implement, redistribute, and build on the spec freely with attribution. The spec is intentionally permissive: implementations are encouraged, alternative tooling is welcomed, and competing distributions of the same spec are allowed.

**Reference toolchain (CLI, Creator, official adapters):** Licensed under [Apache 2.0](LICENSE). Includes an explicit patent grant. You may use the code in commercial products freely.

**Sample packages** (in [`a3ip-standard/packages`](https://github.com/a3ip-standard/packages)): each package declares its own license in its manifest. The maintainer-authored samples are Apache 2.0.

**Trademark "A3IP":** the name "A3IP" is in the process of trademark registration (EUIPO; USPTO filing planned post-launch). The pending trademark covers the project identity — implementations may describe themselves as "A3IP-compatible" without restriction, but should not use "A3IP" in a way that implies official endorsement or affiliation. See [COMPATIBILITY.md](COMPATIBILITY.md) for the trademark acknowledgements covering other standards we reference.

**Logo:** the A3IP suitcase mark, released under the same trademark scope as the name.

---

## Conflict resolution

In the current single-maintainer model, the maintainer is the final arbiter on every decision. This is documented honestly rather than disguised behind a process. Disagreements have these paths:

1. **Open dialogue in the issue / PR.** Most disagreements resolve once both sides understand the constraints.
2. **Public dissent.** If a contributor disagrees with a maintainer decision, they may document it in a follow-up issue. The disagreement becomes part of the public record even if the maintainer's decision stands.
3. **Fork.** A3IP's spec license (CC BY 4.0) and tooling license (Apache 2.0) explicitly permit forking. A maintainer who consistently makes poor decisions deserves to be forked away from. The project's goal is to be useful enough that forking is rare; if it isn't, that's signal worth listening to.

There is no escalation path beyond the maintainer until the future-state steering committee exists. Acknowledged limitation.

---

## Repositories

| Repo | What's in it | Maintainer |
|---|---|---|
| [`a3ip-standard/spec`](https://github.com/a3ip-standard/spec) | The spec text, JSON Schemas, COMPATIBILITY, GOVERNANCE, README | Maksym Prydorozhko |
| [`a3ip-standard/cli`](https://github.com/a3ip-standard/cli) | The `a3ip` Python CLI (PyPI: `pip install a3ip`) | Maksym Prydorozhko |
| [`a3ip-standard/creator`](https://github.com/a3ip-standard/creator) | The reference authoring tool (`a3ip-creator` skill) | Maksym Prydorozhko |
| [`a3ip-standard/packages`](https://github.com/a3ip-standard/packages) | Reference package registry: bundles + `registry.yaml` | Maksym Prydorozhko |

---

*A3IP Specification v1.10 · © 2026 Maksym Prydorozhko · [a3ip.dev](https://a3ip.dev)*
