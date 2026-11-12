# GitHub work-start convention

This convention applies to `sansa/verdeflow`. `main` is the trunk branch. It is for contributors and any coding agent that can read repository files; the workflow does not depend on a particular AI tool or GitHub integration.

Codex and Cursor can discover the [repository skill](../../.agents/skills/github-start-work/SKILL.md). Claude Code has a [skill entrypoint](../../.claude/skills/github-start-work/SKILL.md). Other agents can read this file directly. The shared rules here take precedence over the short agent entrypoints.

Use it when someone asks to **start tracked work**, create or find an issue, file a bug or feature, or create a branch. Ordinary edits do not require an issue or branch merely because an agent is doing them. Follow a more specific request from the contributor when one is given.

## Decide what needs tracking

1. Identify the outcome, affected repository, and whether files will change in that repository. Documentation, tests, and tooling count as file changes.
2. Search open issues for the same outcome. Reuse a matching issue and add missing context instead of creating a duplicate.
3. Use one issue in `sansa/verdeflow` for work contained here. If a contributor names another repository that must change, track that repository's work there and cross-link the issues. Do not assume a second repository is involved.
4. Create a branch only when files in this repository will change. An issue for a discussion, access task, lab operation, or other work with no repository diff needs no branch.

If GitHub is unavailable, draft the issue body and report that it was not created. Wait for a real issue number before using the numbered branch format, unless the contributor explicitly chooses a temporary offline branch.

## Issue labels

Use labels that **already exist** in the target repository. Apply every relevant label, without creating a new label silently. If a useful label is missing, record the category in the issue body and propose the label separately.

| Work | Existing label |
| --- | --- |
| Broken behavior or incorrect result | `bug` |
| New capability or improvement | `enhancement` |
| Documentation or contributor guidance | `documentation` |
| Clarification or discussion | `question` |
| Suitable starter task | `good first issue` |
| Extra contributor attention wanted | `help wanted` |

Use `good first issue` and `help wanted` only when the issue is genuinely ready for outside contributors. Avoid `invalid`, `duplicate`, and `wontfix` on new work. An issue can have more than one applicable label, such as `enhancement` and `documentation`.

Record the affected area in the body, such as measurement, collectors, agents, control plane, UI, CLI, or documentation. Mark **measurement validity** when work may change energy calculations, units, time windows, clock alignment, raw observations, calibration, provenance, or quality flags. Mark **security/privacy** for authentication, authorization, secrets, or personal data. These are risk descriptions until matching labels are deliberately created.

## Issue title and body

Use an imperative, specific title that describes the outcome, for example `Preserve raw meter samples when a run fails`. Avoid titles such as `Update code`.

Fill the relevant fields in this template. Acceptance criteria should be observable for the actual work, not a generic checklist pasted into every issue.

```markdown
## Summary
<User or researcher outcome in one paragraph>

## Context
- Area:
- Repository files expected to change: yes / no
- Related issue, PR, experiment, or design document:
- Other repositories affected, if any:

## Risk
- Measurement validity / security/privacy / standard:
- What could go wrong:
- Labels selected from this repository:

## Proposed approach
- <Short steps or design direction; omit when discovery is the task>

## Out of scope
- <Boundary that prevents accidental expansion>

## Acceptance criteria
- [ ] <Observable outcome>
- [ ] <Relevant verification or evidence>
```

For measurement-validity changes, include criteria for the relevant units, timing boundary, raw data and provenance, failure/quality flags, and comparison against a known reference or fixture. For security/privacy changes, include authorization and data-handling checks. For documentation-only work, check accuracy and links. Do not promise physical hardware validation when the hardware is unavailable; state the remaining validation explicitly.

## Branch naming

When repository files will change, start from the latest `origin/main` without discarding uncommitted work. Use a separate worktree or pause if the current checkout is dirty. Do not create an empty branch for issue-only or operational work.

Name the branch:

```text
<prefix>/<issue-number>-<initials>-<3-or-4-word-slug>
```

| Prefix | Use |
| --- | --- |
| `feature` | New user-facing capability |
| `fix` | Defect correction |
| `hotfix` | Urgent production defect with tightly limited scope |
| `chore` | Documentation, tooling, maintenance |
| `refactor` | Restructure without intended behavior change |

Use the **human issue creator's** initials, not the agent's. Prefer their GitHub profile name or local Git author name. Lowercase the first letter of the first and last name; for a single-token name, use its first two letters. Ask if initials cannot be determined reliably. The slug has exactly three or four lowercase hyphenated words describing the issue. Do not put secrets or full usernames in it.

Example: issue `#1`, creator name `mbanyick`, and a tooling change: `chore/1-mb-add-agent-workflow-skill`.

Fetch before branching; verify the base is current. Once the branch is created, publish it with an upstream if the contributor asked to create a branch on GitHub. Never push to or force-update `main`, merge, close an issue, or delete remote refs as part of starting work unless explicitly requested.

## Report the result

Give the issue URL and number, applied labels, branch name or the reason no branch was made, risk classification, and the next work step. If a GitHub write failed, say what was drafted or completed locally and what still needs to be done.
