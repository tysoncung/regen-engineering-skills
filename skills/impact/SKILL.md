---
name: impact
description: Use before regenerating anything, or when asked "what breaks if I change this?", "what does this affect?", or "what needs retesting?" in a Regen Engineering repo. Computes the regeneration scope of a knowledge change: which modules must be rebuilt and which contracts must pass.
---

# Impact analysis

Do not reason about blast radius yourself. It is deterministic graph traversal over the knowledge links, and a script does it correctly every time. Your judgment is needed for what the script cannot see.

## Steps

1. **Run the tool.** For a changed knowledge item:
   ```bash
   npx -p regen-engineering-schema regen-impact BR-002
   ```
   For everything asserted about a module:
   ```bash
   npx -p regen-engineering-schema regen-impact --module customer
   ```
   To map an actual diff back to scope:
   ```bash
   npx -p regen-engineering-schema regen-impact --changed $(git diff --name-only main...HEAD)
   ```
   Add `--json` when you need to consume the result rather than show it.

2. **Read what it returns.** Three things matter: the modules to regenerate, the contracts that must pass afterwards, and anything it warns about. Note that contracts in scope include every contract owned by an in-scope module, not just those verifying the changed item. That is deliberate: it stops a regeneration from quietly breaking a rule nobody edited.

3. **Sanity-check the scope against the prose.** This is the part the script cannot do. Read the changed item's body. If it discusses behaviour in a module that does not appear in the computed scope, the `affects` links are incomplete. Fix the knowledge, then run the tool again.

4. **Escalate if the scope is suspicious.**
   - **Scope is empty** but the change is real: the item declares no `affects`, so the knowledge is wrong. Fix it before doing anything else.
   - **Scope is the entire system** for a small change: usually an over-broad `affects` on some item, and worth correcting, because a scope that always says "everything" is the same as having no impact analysis at all.
   - **Modules in scope but zero contracts**: the tool warns about this. Regenerating is unsafe, because nothing would catch a mistake. Write contracts first.

## Report

Give the human the module list, the contract list, and your own note on whether the computed scope matches what the change actually means. Say explicitly when they disagree, and do not proceed to regeneration until they agree.
