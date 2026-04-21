# CLAUDE.md

## **MANDATORY**: Session start — do this FIRST, before responding

Read `AGENTS.md` from your working directory and every ancestor directory up to (and including) the repo root (run `git rev-parse --show-toplevel` if unsure of the root). Call Read directly for each path in parallel — a missing file just errors, which is fine, so don't check existence first. Do NOT read `AGENTS.md` in subdirectories — those load on demand when you work there.

Skipping this step is a bug. If you catch yourself answering without having done it, stop and do it now.

## Universal Tool Rules

- Never use `find` or `grep` in Bash — use the dedicated Glob and Grep tools instead