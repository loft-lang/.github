<!--
Copyright (c) 2026 Jurjen Stellingwerff
SPDX-License-Identifier: LGPL-3.0-or-later
-->

# loft-lang org defaults

Shared, **defined-once** infrastructure for every `loft-lang` repo — so we never
copy workflows/templates/labels into N repos that then drift. (Drift is the
re-derivation anti-pattern; see the loft repo's `doc/claude/GOALS.md` §
"the method mirrors the goals".)

## Reusable workflows — the `fixed-pending-merge` lifecycle

The label lifecycle runs off the one `Fixes #NNN` commit trailer, with the logic
held here and each repo opting in via a tiny **stub** (the stub supplies the
event trigger; this repo holds the behaviour):

| Reusable workflow | What it does | Stub trigger |
|---|---|---|
| `.github/workflows/apply-fixed-pending-merge.yml` | on a push to a non-`main` branch, label every issue a commit says it `Fixes/Closes/Resolves`; warn on bare `(#N)` mentions | `push: branches-ignore: [main]` |
| `.github/workflows/strip-fixed-pending-merge.yml` | on issue close, remove the label (a closed issue is never "pending merge") | `issues: types: [closed]` |

Together: **write `Fixes #N`** → labeled on branch push → GitHub auto-closes on
merge to `main` → label stripped on close. No hand-labeling; multiple actors fix
bugs on their own branches concurrently with the tracker staying correct.

### Opt a repo in — add two stubs

`.github/workflows/fpm-apply.yml`:

```yaml
name: fixed-pending-merge (apply)
on:
  push:
    branches-ignore: [main]
permissions:
  issues: write
jobs:
  apply:
    uses: loft-lang/.github/.github/workflows/apply-fixed-pending-merge.yml@main
```

`.github/workflows/fpm-strip.yml`:

```yaml
name: fixed-pending-merge (strip)
on:
  issues:
    types: [closed]
permissions:
  issues: write
jobs:
  strip:
    uses: loft-lang/.github/.github/workflows/strip-fixed-pending-merge.yml@main
```

Proven 2026-06-07: the apply logic fires and labels in the loft-lang org context
(org Actions default permissions already allow workflow-level `issues: write`).

## Issue templates + labels (org defaults)

`.github/ISSUE_TEMPLATE/` here serves as the default for any org repo without its
own. `LABELS.md` is the shared label glossary (`sev:*` / `wa:*` / `area:*` /
`fixed-pending-merge` / …).

## Conventions

- **Where a bug's issue lives** — in the repo whose source fixes it (closing
  keywords auto-close only same-repo). Compiler/runtime → the loft repo;
  extracted-library code → that library's repo.
- **Canonical library repo name** — **`loft-lang/loft-libs-<chunk>`** (chunk
  repos group packages: `loft-libs-graphics` holds graphics / shapes / imaging /
  gridmesh). Not `loft-<pkg>`, not the read-only `tests/fixtures/libs/` mirror in
  the loft repo.

Full rationale lives in the loft repo: `doc/claude/ISSUE_TRACKING.md`,
`LIBRARY_AUTHORING.md`, `MOVING.md`.
