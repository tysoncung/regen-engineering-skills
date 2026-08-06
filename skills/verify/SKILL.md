---
name: verify
description: Use to check a Regen Engineering repo is sound: schema validation, contract results, and the knowledge debt report. Triggers on "verify", "check the knowledge", "is this healthy", "knowledge debt", or before merging and releasing.
---

# Verify

Three separate questions, asked in order. Do not conflate them.

## 1. Is the knowledge well-formed?

```bash
npx -p regen-engineering-schema regen-validate .
```

Errors mean the tree is broken: bad frontmatter, dangling references, duplicate IDs, unknown modules, malformed lock files. Fix all of them; nothing downstream is meaningful until this passes.

Warnings are knowledge debt, most often an active rule no contract verifies. They do not block, but they should be shrinking over time rather than growing.

## 2. Do the contracts pass?

Run the project's contract suite. Report results **per contract ID**, not as a single number, because "14 tests passed" tells a reviewer nothing about which rules are actually verified. A reviewer needs to see that CT-002 passed, because that is what ties back to BR-002.

If the suite cannot be run, say so plainly. Never infer that contracts pass from code that looks correct.

## 3. What is the knowledge debt?

```bash
npx -p regen-engineering-schema regen-debt .
```

Four metrics, and they mean different things:

- **Coverage**: modules with a structurally complete knowledge package. A floor, not a quality measure. A package can be complete and still wrong.
- **Freshness**: modules built from current knowledge. Anything stale is a regeneration backlog, which is normal and healthy in small amounts.
- **Integrity**: modules with code-ahead drift. **Target zero.** This is the rot metric, and a non-zero value means the system knows things its source of truth does not.
- **Traceability**: active rules with both a verifying contract and an implementing module.

**Traceability at 100% is weaker than it sounds.** It counts rules that have a contract pointing at them. It cannot tell whether that contract would fail if the rule were violated. Observed on the reference demo: three of five new scenarios passed against an implementation that ignored the new rule entirely, while the metric read fully verified.

### 4. Do the contracts assert anything?

Automate the check rather than remembering to do it. Run the whole suite against a **straw implementation**: a server that answers every request with a plausible shape and no behaviour at all, typically an empty JSON object with a 200.

Every scenario should fail. Any that passes is asserting nothing, and the rule it claims to verify is unverified no matter what traceability says.

The reference demo does this in `contracts/vacuity.mjs`, in CI on every run. It found a vacuous scenario on its first execution: a step asserting "authentication succeeds" checked only for a 200 status, which the straw returns for everything. The fix was to assert that the response identifies the customer who registered.

Two habits follow from it. Assertions should name a value, not a status, wherever the prose names one. And a `Given` step is a precondition: if it fails, the scenario never reached its subject, so it should stop loudly rather than continue into assertions that now mean nothing.

A contract nobody has seen fail is a contract nobody has tested.

The report also flags possible under-linking: an item whose prose cites another module's items without declaring that module in `affects`. Take these seriously. A missing link silently shrinks the regeneration scope, which produces confident, incomplete work, and that is the worst failure mode this methodology has.

## Report

Give the human all three answers separately, with the actual command output rather than your summary of it. If anything failed, say what failed and what it means. Never report "everything looks good" without having run all three.
