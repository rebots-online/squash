# squash-showcase (working name)

A native Unreal Engine 5 squash game for an RTX 3090-class desktop. Windows is
the first shipped target; Linux is the development environment (dual boot).
Not a web application. See `DOCS/PRD.md` for what it is and why.

## Current state (2026-11-09)

Inception. Only `DOCS/PRD.md` exists. There is no Unreal project, no
architecture document and no checklist yet. The engine is not installed on
this machine. Do not write product code: the order is PRD (operator
approval) -> feasibility spikes -> `DOCS/ARCHITECTURE.md` -> `CHECKLIST.md`
-> code.

## Rules

- Global source of truth: `/home/robin/Admin-Manual` (Architecture Law in its
  `CLAUDE.md`; conventions in `DOCS/TOOLING_CONVENTIONS/`).
- Continuity lives in KStore.
- Scratch files go in `.tmp/`, never `/tmp`.

## Donor repository (read-only)

`/home/robin/CascadeProjects/squash`, pinned at commit
`180ec040da06c7c444f2ace3e10730b2bcdbe173`. A three.js browser game. Never
modify it and never copy its source. It is used for tuned numeric values and
as a reference oracle: its simulation (`src/squash/*.js`) runs headless in
Node (`node --test tests/*.test.mjs` runs its existing tests).

## Roles

Claude orchestrates and reviews. GLM-5.3 with 1M context codes, dispatched
through `zclaude --acp` with `acpx`; confirm the model from returned metadata
on every run. Codex is the fallback orchestrator.

## Build

No build exists yet. Planned commands are in `DOCS/PRD.md` section 6 and are
unverified until the feasibility spikes run.
