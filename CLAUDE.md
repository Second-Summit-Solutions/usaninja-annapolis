# usaninja-annapolis — AI Context

**Purpose:** Ninja Annapolis

This file tells any agent landing in this repo how to work here. It overrides default behavior.

---

<!-- per-repo:start -->
<!-- Per-repo sections: project overview, conventions, decisions, current focus.
     Edit these placeholders when customizing for a specific repo.
     Surrounding merge-block markers let future rollouts overwrite structural
     sections without touching your per-repo edits. -->
## Project

_What this repo does, who its customers are, what systems it touches._

## Conventions

_Code style, naming, patterns, anti-patterns specific to this repo._

## Decisions

_Architecture and design decisions that constrain current work._

## Current focus

_What we're actively shipping, what's blocked, what's next._

<!-- per-repo:end -->

---

## Role architecture

Claude acts as **PM/QA**. OpenCode is the **code worker**. This is non-negotiable.

- Writing code -> delegate to `ss-opencode-agent[bot]` via a spec'd issue.
- Planning, spec authoring, review, merging -> Claude owns.
- Exceptions: single-line fixes (<10 lines), live API calls, or analysis-only tasks.

## Two-identity review model (worker != approver)

**The branch ruleset requires 1 approving review + `require_last_push_approval: true`, with Admin bypass reserved for emergencies / doc-only fallback-worker PRs.** Dan does not click approve himself -- Claude, authenticated via `github.pat`, is the approver identity.

The two identities:

- **`ss-opencode-agent[bot]` (GitHub App 3555338)** -- primary worker. Pushes branches, opens PRs.
- **Claude (github.pat)** -- reviews, approves, merges. Also acts as fallback worker when OpenCode is down or unsuitable (noted in PR body with fallback-checkbox + reason).

What this means for Claude:

- **Claude is the reviewer of record.** Runs the full QA checklist (see below). Accountable for catching defects.
- **Dan is consulted in chat, not on the PR.** When ambiguous, Claude asks Dan in conversation, records the answer in the PR body under `## Decisions from chat`, and proceeds.
- **CI is the hard gate.** Red CI blocks merge -- GitHub enforces this, not a human.
- **Never push code under `github.pat` when OpenCode should have done it.** That collapses the worker/approver split. The only exception: explicit fallback-worker mode on doc-only PRs, documented in the PR body.
- **Admin bypass is not a default tool.** It's for emergencies, fallback-worker PRs, and repo setup. Document use in the PR body.

## Vault-first workflow

Before starting any non-trivial task, read `vault/Memory.md` plus relevant `/vault` subfolders (decisions/, errors/, patterns/, sessions/). Use vault-tools CLI for grep/read/write or vault-mcp tools for MCP-aware reads. Gotchas and decisions accumulate in vault, not in a cloud API.

## Spec-Required Rule

**Every** coding issue tagged `claude-ready` MUST carry a valid `ss-spec` comment before any worker picks it up. The spec is a GitHub issue comment with the following sentinel format and seven required headings:

```
<!-- ss-spec v=1 generated_at=YYYY-MM-DDThh:mm:ssZ generated_by=<agent-name> -->

## TASK
## FILES
## CONTEXT
## REQUIREMENTS
## CONSTRAINTS
## EXECUTION
## VERIFICATION
```

The validate-issue-spec workflow checks every `claude-ready` issue for:
- The sentinel line (`<!-- ss-spec v=1 ... -->`)
- All seven headings, each with non-placeholder content
- Applies `needs-spec` or `has-spec` labels accordingly

The `require-spec-pr-check.yml` workflow is a **required status check** -- PRs whose linked issue lacks `has-spec` cannot merge. Waive with `Skip-spec: <reason>` line and `skip-spec` label.

Canonical template: `github-repo-creator/templates/common/.github/SPEC_TEMPLATE.md`. Per-repo edits forbidden.

## Spec Template

See `.github/SPEC_TEMPLATE.md` in the repo. The canonical format lives at `github-repo-creator/reference/cursor_spec_template.md` (legacy name; tool-agnostic content). When authoring specs for OpenCode, use the EXACT same format -- the template is identity-agnostic by design. All headings required, sentinel mandatory.

## Post-OpenCode QA Checklist

Before Claude merges any OpenCode-authored PR:

1. **Output scan** -- OpenCode stdout clean, no silent errors?
2. **File existence** -- all claimed files present on the branch?
3. **Syntax/build** -- language-appropriate compile/parse step green?
4. **Import check** -- module imports without error / build succeeds?
5. **Tests added/updated** -- new behavior has coverage?
6. **Vault gotcha cross-ref** -- any known gotcha in vault/errors/ violated?
7. **Pattern conformance** -- follows repo conventions in this CLAUDE.md?
8. **Secrets/schema/prod check** -- touches anything sensitive? If yes, escalate in chat.
9. **Dry-run** -- for scripts with side effects, run limited scope first.
10. **CI green** -- all required checks must be green. Non-negotiable.

Any failed check -> fix directly if trivial, otherwise refine spec + re-run OpenCode. Never merge with a failed check.

## Skill-First Approach

Use skills for domain-specific tasks before writing ad-hoc code. Skills encode known gotchas, API quirks, and context. If a relevant skill exists, load it and follow its instructions.

## Global Engineering Standards

Derived from Second Summit Engineering Doctrine v0.2. Applies to every repo.

### TDD where it makes sense

Test-first for pure logic, parsers, transformations. Integration tests for external API code. E2E for browser-automation flows. Skip tests where they don't pay rent. The `tdd-guard` hook checks PR diffs against non-trivial logic and verifies corresponding tests exist. High-value repos opt in by default.

### Conventional Commits

Every commit on `main` follows `feat:|fix:|chore:|docs:|refactor:|test:|build:|ci:|perf:` prefix (Conventional Commits). Squash-merge is the only merge method -- branch-internal commits are squashed away; the PR title IS the commit message. Scope SHOULD reference module: `feat(ghl-engine): ...`.

### Atomic commits

One logical change per commit (and per PR). If a change touches two concerns, split into two PRs sequenced back-to-back (not stacked). Never stack PRs -- it breaks AI reviewer verdict semantics.

### Linear history

Rebase before merge. Squash-merge only. No merge commits on `main`. Enforced by branch ruleset.

### No force-push to main

Enforced via branch protection ruleset (scaffolded by `github-repo-creator/scripts/apply_rulesets.sh`).

### E2E where possible

Every change includes verifiable proof it works. See Verification section below.

### "Show it works" gate

Every PR has a `## Verification` section in the body with concrete evidence: CI logs, test names and output, dry-run results, before/after API diffs. "Looks fine" / "should work" / "tests pass" without specifics is a vibe, not verification.

Enforcement via `verify-pr.yml` (scans PR body for >=30 non-template words under `## Verification`). AI reviewer rates quality: GOOD / WEAK / MISSING / BULLSHIT. `skip-verify` label + reason line waives.

## Vault Layout

```
vault/
  decisions/       -- Architecture and design decisions (one file per decision)
  errors/          -- Known gotchas, failure modes, anti-patterns
  patterns/        -- Reusable patterns, templates, conventions
  sessions/        -- Cowork session artifacts and reflections
  stack/           -- Stack-specific reference and decisions
  Memory.md        -- Working memory: current preferences, focus areas, known no-go's
  index.md         -- Vault index / table of contents
```

Frontmatter schema for vault entries: **TBD via C-1** (ss-ops#157).

## Local Claude Code Architecture

Claude Code reads two CLAUDE.md files at session start:
1. **`~/.claude/CLAUDE.md`** -- global canonical config. Auth rules, delegation, fallback chain, spec rules. See D-2 for the canonical version (ss-ops#TBD).
2. **This file (`CLAUDE.md`)** -- per-repo overrides. Repo-specific gotchas, patterns, conventions.

The Cowork workspace also carries a snapshot of `~/.claude/CLAUDE.md` for context. Changes to the global config should be made in the canonical D-2 location, not in workspace snapshots.

## claude-subconscious (post B-3 review)

**Review pending.** B-3 (ss-ops#TBD) will evaluate claude-subconscious for security, blast-radius, and configurability. Wiring is not yet documented. This section will be populated with the approved integration pattern once the review completes.

## GitHub-First Workflow (mandatory)

All work is tracked in GitHub issues. Every change is a PR linked to an issue. No ad-hoc branches, no work without an issue number. The loop:

1. Issue filed (or spec added to existing issue).
2. OpenCode picks up the spec'd issue, pushes a `spec/<#>-<slug>` branch.
3. OpenCode opens a PR using the PR template.
4. CI runs: lint, test, check-spec, verify-pr, ai-reviewer.
5. Claude reviews, runs QA checklist, resolves ambiguity with Dan in chat.
6. Claude squash-merges (CI green, QA green).
7. Branch auto-deleted. Issue auto-closed.

## VPS prerequisites

If usaninja-annapolis contains services that deploy to the VPS, document external CLI/runtime dependencies in `docs/VPS-PREREQUISITES.md`. Any new external dependency (system packages, global npm packages, or binaries called via subprocess) should update this file before modifying deploy/install scripts.

## When to ask Dan in chat (ambiguity escalation)

Claude merges clean PRs without asking. Claude **stops and asks Dan in chat** when:

1. **A design tradeoff is present** -- library choice, schema shape, perf vs. readability, etc.
2. **The change touches sensitive surfaces:**
   - VPS deploy paths / production cron schedules
   - DB schema migrations (add/rename/drop/nullable flip)
   - Secrets, vault config, API keys, auth logic
   - Billing, payments, money-moving code
   - Customer-facing copy, pricing, waivers
   - Destructive operations (deletes, truncates, historical data)
3. **A vault gotcha is being intentionally violated** -- confirm it's intentional.
4. **A test was weakened, skipped, or xfailed** -- always ask why.
5. **A dependency was added or major-version-bumped** -- confirm the choice.
6. **OpenCode diverged from the spec in a way that might be right** -- confirm.
7. **CI is red in a way that looks flaky** -- don't paper over; ask.
8. **You're not sure.** Ask. Cost of asking = one message. Cost of bad merge = much more.

When Dan answers, record the decision in the PR body under `## Decisions from chat` before squash-merging.

## What NOT to ask Dan

To avoid wasting his time, do NOT ask about:

- Formatting, lint rules, or style -- follow existing patterns / ruff config.
- Whether to write tests -- yes.
- Whether to write a PR body -- yes.
- Small refactors inside a PR -- request them in the review, don't ask.
- Trivial Dependabot bumps with green CI -- just merge.
- Conventional-commit prefix choices -- pick the right one.

<!-- per-repo:start -->
## Repo-specific gotchas

_Accumulate over time. Start empty, add as we learn._

- [add as they come up]

## Repo-specific patterns

_Conventions unique to this repo. Start empty._

- [add as they come up]
<!-- per-repo:end -->

## Related skills

- **opencode-agent** -- code delegation to ss-opencode-agent[bot]
- **vault-ops** -- vault-first memory and decision management
- **vps-ops** -- deploys + runtime ops (if this repo deploys to VPS)
- **github-repo-creator** -- the skill that created this repo
