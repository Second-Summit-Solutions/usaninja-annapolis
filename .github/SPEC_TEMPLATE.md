<!--
CANONICAL Cursor / LLM-workbench spec template.

When filing a `claude-ready` issue, Claude posts a comment on the issue in
EXACTLY this format. The top sentinel line is REQUIRED — the
validate-issue-spec workflow uses it to identify the spec comment and
schema version. The seven headings below are REQUIRED — the workflow
greps for all of them and fails the issue (adds `needs-spec` label) if
any are missing.

DO NOT edit the sentinel line by hand. If the schema needs to evolve,
bump the `v=` number and update workflow validation to accept both
versions during the migration window.
-->

<!-- ss-spec v=1 generated_at=YYYY-MM-DDThh:mm:ssZ generated_by=<agent-name> -->

## TASK

One-line summary of what to build or change. Present tense, imperative.

## FILES

- Create: `path/to/new_file.py`
- Modify: `path/to/existing_file.py`

Be exact. Full paths relative to repo root.

## CONTEXT

- Codebase patterns this should follow (reference specific files).
- Relevant memory entries: `mem_abc123…` — "short description of what it says."
- Libraries / tooling / venv paths available.
- Any prior PR or issue that set precedent.

## REQUIREMENTS

1. Specific thing this must do, with logic detail.
2. Edge cases to handle.
3. Error handling expectations.
4. Config / env inputs and where they come from (vault, .env, etc.).

## CONSTRAINTS

- API quirks / gotchas (paste the relevant lines from memory queries).
- Libraries NOT to use.
- Patterns NOT to violate (e.g., "don't spawn subprocesses inside the
  request handler — use the async worker pattern in `lib/jobs.py`").
- Security / secrets handling rules that apply here.

## EXECUTION

Execute this end-to-end without asking for confirmation. Do not spawn
subagents. Commit changes, push to a feature branch, open a PR against
`Second-Summit-marketing/<repo>` with `Closes #<N>` in the body. Bot
identity commits (GIT_AUTHOR_* / GIT_COMMITTER_* are pre-wired by the
runtime). Do not self-approve or self-merge — Claude approves and merges.

## VERIFICATION

Commands that prove this works. The agent runs these before opening the PR.

- Syntax: `python3 -m py_compile path/to/file.py`
- Import: `cd /opt/ss-cron-system && python3 -c "import scripts.path.to.module"`
- Unit tests: `pytest tests/unit/test_<module>.py -v`
- Dry-run: `python3 path/to/file.py --dry-run` (if script has side effects)
- Smoke: Any quick end-to-end check that the happy path works.
