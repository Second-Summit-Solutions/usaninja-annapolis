# PR

## Summary

_What this PR does in 1-3 sentences._

## Linked issue

Closes #___

## Changes

- [ ] Change 1
- [ ] Change 2

## Post-OpenCode QA checklist (Claude fills this in)

- [ ] OpenCode output scan clean (no errors in stdout)
- [ ] All claimed files exist at stated paths
- [ ] Syntax / build check passes
- [ ] Import check passes (`python -c "import ..."` or equivalent)
- [ ] Tests added or updated for new behavior
- [ ] Tests pass locally AND in CI
- [ ] No known memory gotchas violated
- [ ] Follows existing repo patterns (see `CLAUDE.md`)
- [ ] Dry-run performed for scripts with side effects
- [ ] Required CI checks are all green

## Review model reminder

Second Summit uses a two-identity model: worker bot (`ss-opencode-agent[bot]`) pushes code; Claude (under `github.pat`) reviews, approves, and merges. The ruleset requires **1 approving review + `require_last_push_approval`** — that's enforced, not theatre, because the approver identity is distinct from the pushing identity.

- Claude is the reviewer of record — and the approving identity.
- If any of the ambiguity conditions below apply, Claude asks Dan **in chat**, records the decision under `## Decisions from chat`, and then approves + merges.
- If none apply and CI is green → Claude approves and squash-merges.
- Admin bypass is used only for emergencies, doc-only fallback-worker PRs (noted below), or repo setup. Document any use here.

**Was this PR authored as a Claude fallback-worker (OpenCode down/out-of-quota)?**
- [ ] No — normal worker-then-approver flow.
- [ ] Yes — Claude authored + will admin-bypass merge. Reason: _______________________

## Ambiguity / escalation check (ask Dan in chat if any are true)

- [ ] Touches prod paths (VPS deploy, cron, live systemd units)
- [ ] Touches secrets, vault config, `.env*`, or SOPS files
- [ ] Touches DB schema (migrations, renames, nullable flips, drops)
- [ ] Touches auth, permissions, webhook signatures, or rate-limiting
- [ ] Touches billing, payments, waivers, or customer-facing copy
- [ ] Introduces a new dependency or a major-version bump
- [ ] Weakens, skips, or xfails an existing test
- [ ] OpenCode diverged from the spec in a material way
- [ ] A known memory gotcha is being intentionally violated
- [ ] CI looks flaky in a suspicious way
- [ ] There's a genuine design tradeoff Claude can't call alone

**If any box above is checked → ask Dan in chat before merging. Record his answer below.**

## Decisions from chat

_If Dan was consulted, paste the question + his answer here so it's auditable later._

- [ ] N/A — no ambiguity, self-merged.

## Notes for future readers

_Anything a future agent or human should know when they come back to this PR._

## Bypasses

Four automated PR quality gates may block merge (see `pr-quality-checks.yml`).
Each can be bypassed with a **label** + a **bare reason line at column 0**
in the PR body (no backticks, headings, or indentation):

| Gate | Block reason | Bypass label | Reason line |
|------|-------------|--------------|-------------|
| `check-size` | PR >500 additions or >15 files | `size:large-justified` | `Big-PR-justified: <reason>` |
| `check-tests` | Code changed without tests in a tested repo | `no-tests-justified` | `Tests-skipped: <reason>` |
| `check-atomicity` | PR closes >1 issue | `multi-issue-justified` | `Multi-issue-justified: <reason>` |
| `check-risky-paths` | Risky files touched without review sign-off | `risk:reviewed` | (label-only — no reason line required) |

Claude (PAT) is the universal applier — add these labels + lines during review.
Dependabot PRs auto-pass all four gates.
