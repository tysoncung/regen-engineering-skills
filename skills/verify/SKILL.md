---
name: verify
description: Use to check a Regen Engineering repo is sound: schema validation, contract results, and the knowledge debt report. Triggers on "verify", "check the knowledge", "is this healthy", "knowledge debt", or before merging and releasing.
---

# Verify

Three separate questions, asked in order. Do not conflate them.

## 1. Is the knowledge well-formed?

```bash
node <schema>/tools/validate.mjs .
```

Errors mean the tree is broken: bad frontmatter, dangling references, duplicate IDs, unknown modules, malformed lock files. Fix all of them; nothing downstream is meaningful until this passes.

Warnings are knowledge debt, most often an active rule no contract verifies. They do not block, but they should be shrinking over time rather than growing.

## 2. Do the contracts pass?

Run the project's contract suite. Report results **per contract ID**, not as a single number, because "14 tests passed" tells a reviewer nothing about which rules are actually verified. A reviewer needs to see that CT-002 passed, because that is what ties back to BR-002.

If the suite cannot be run, say so plainly. Never infer that contracts pass from code that looks correct.

## 3. What is the knowledge debt?

```bash
node <schema>/tools/debt.mjs .
```

Four metrics, and they mean different things:

- **Coverage**: modules with a structurally complete knowledge package. A floor, not a quality measure. A package can be complete and still wrong.
- **Freshness**: modules built from current knowledge. Anything stale is a regeneration backlog, which is normal and healthy in small amounts.
- **Integrity**: modules with code-ahead drift. **Target zero.** This is the rot metric, and a non-zero value means the system knows things its source of truth does not.
- **Traceability**: active rules with both a verifying contract and an implementing module.

The report also flags possible under-linking: an item whose prose cites another module's items without declaring that module in `affects`. Take these seriously. A missing link silently shrinks the regeneration scope, which produces confident, incomplete work, and that is the worst failure mode this methodology has.

## Report

Give the human all three answers separately, with the actual command output rather than your summary of it. If anything failed, say what failed and what it means. Never report "everything looks good" without having run all three.
