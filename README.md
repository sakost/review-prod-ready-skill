# review-prod-ready

A [Claude Code](https://code.claude.com) skill that dispatches **7 specialized parallel reviewers** to perform a comprehensive production-readiness code review. Each reviewer focuses on one dimension deeply — catching issues a single broad reviewer misses.

## Why

A generic "review this code" prompt tends to produce shallow output: one reviewer tries to cover everything and ends up skimming across dimensions. This skill splits the review into 7 parallel tracks, each handled by a subagent primed with a nitpicking, skeptical mindset.

Observed gaps in single-reviewer setups that this skill addresses:

- Useless tests (tests that would still pass if the implementation were broken)
- `type: ignore`, `noqa`, `eslint-disable` directives hiding real bugs
- DRY violations left unflagged
- Partially-implemented plan items slipping through
- Overly complex code accumulating bugs
- Architecture-level issues missed while focused on line-level nits

## Install

Personal install (available in all your projects):

```bash
git clone git@github.com:sakost/review-prod-ready-skill.git \
  ~/.claude/skills/review-prod-ready
```

Project-scoped install (only in the target repo):

```bash
git clone git@github.com:sakost/review-prod-ready-skill.git \
  .claude/skills/review-prod-ready
```

## Usage

From inside Claude Code:

```
/review-prod-ready
```

Optionally pass a description or plan reference:

```
/review-prod-ready auth refactor per docs/auth-plan.md
```

The skill auto-detects the git range (merge-base with `main` or `master`) and injects the list of changed files into the prompt before dispatching the reviewers. Each reviewer reads the changed files in full, focuses on its dimension, and reports findings as:

- **Severity:** Critical / Important / Minor
- **File:line** reference
- **What's wrong**
- **Why it matters**
- **Suggested fix**

The orchestrator aggregates findings into a single report with a final verdict (Yes / No / With fixes).

## The 7 reviewers

| # | Reviewer                         | Focus                                                                     |
| - | -------------------------------- | ------------------------------------------------------------------------- |
| 1 | Test Quality & Coverage          | Meaningful assertions, edge cases, mock abuse, useless tests              |
| 2 | Plan Completeness                | Every requirement implemented, no stubs or TODOs slipping through         |
| 3 | Production Readiness             | Secrets, error handling, timeouts, resource leaks, logging                |
| 4 | Architecture                     | Coupling, boundaries, scalability, dependency direction                   |
| 5 | Code Smells & Suppressed Warnings| `type: ignore`, `noqa`, `eslint-disable` — justified or hiding bugs?      |
| 6 | Code Complexity                  | Cyclomatic/cognitive complexity, nesting depth, function size             |
| 7 | DRY Violations & Dead Code       | Near-duplicate blocks, unused exports, commented-out code                 |

## Design notes

- **Adversarial mindset.** Every reviewer is prefixed with a "grumpy, skeptical, nitpicking senior engineer" prompt. Observed to produce noticeably more findings than neutral prompts — LLMs default to "summarize what works"; this flips that to "find what's broken."
- **Anti-softening aggregation.** The orchestrator is explicitly forbidden from downgrading severities or rewriting findings to sound polite. A Critical stays Critical.
- **Dynamic context injection.** The git range and changed files are pre-computed via `!`shell injection before the skill prompt is sent — no tool call burned on basic context gathering.
- **Skip irrelevant reviewers.** Small diffs or missing context (e.g. no plan document) let reviewers be skipped to avoid noise.

## Files

```
review-prod-ready/
├── SKILL.md         # Orchestration: dispatch process + aggregation rules
├── reviewers.md     # The 7 specialized reviewer prompts
├── LICENSE-MIT
├── LICENSE-APACHE
├── .gitignore
└── README.md        # You are here
```

## License

Dual-licensed under your choice of:

- [MIT License](./LICENSE-MIT)
- [Apache License 2.0](./LICENSE-APACHE)
