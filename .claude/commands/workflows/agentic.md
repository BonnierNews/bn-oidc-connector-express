---
name: Agentic Pipeline
description: Activate the full 4-agent pipeline (Coder → Tester → Reviewer → Analyzer) for implementation tasks
category: Workflow
tags: [workflow, pipeline, agents]
---

# Agentic Workflow

**The orchestrator never modifies source code directly.** No Edit, Write, or Bash tool calls to change code — not even for trivial fixes. Always delegate to the appropriate agent. Agents are defined in `.claude/agents/`.

## Pipeline

Every task runs through all four agents in order:

1. **Coder** — Implements the change following TDD. Writes the failing test first, then the implementation.
2. **Tester** — Validates that tests are comprehensive, passing, and cover the full e2e scenario. Runs the project's test suite.
3. **Reviewer** — Reviews the code for quality, readability, security, and adherence to project conventions.
4. **Analyzer** — Evaluates whether the work aligns with the project's strategic goals as defined in AGENTS.md. **The orchestrator must include the original task description and plan in the analyzer's prompt.**

## Execution rules

- **Dependencies:** Coder must finish before Tester. Tester must pass before Reviewer and Analyzer.
- **Parallelism:** Run independent work concurrently — multiple Coder agents for independent modules, Reviewer + Analyzer together. Always prefer parallel execution.
- **Isolation:** Coder and Tester run in git worktrees (`isolation: worktree` in their agent definitions) so they get isolated file copies and branches. Reviewer and Analyzer are read-only and don't need isolation.
- **Verdicts:** Every agent ends its report with `Verdict: pass` or `Verdict: fail`. The orchestrator uses this to decide whether to proceed or loop back.
- **Feedback loop:** When any agent reports `Verdict: fail`, spawn a new Coder to fix the issues, then restart from Tester. Repeat until all four agents report `Verdict: pass`. No agent may be skipped.
- **Verify, don't trust:** The Coder must run `git diff` to confirm changes are in the working tree before reporting. The Tester must read source files to verify claimed fixes exist — tests passing alone is not proof. The Reviewer must search for cascading dead code when any function/import is removed.

## Usage

Invoke with `/workflows:agentic` followed by the task description:

```
/workflows:agentic refactor the token validation middleware to use the new OIDC provider
```
