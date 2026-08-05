---
name: regenerate
description: Use to (re)build a module's implementation from its knowledge package in a Regen Engineering repo, after a knowledge delta has been reviewed. Triggers on "regenerate", "rebuild from knowledge", "implement the delta", or after the knowledge-delta skill. Regenerates at the smallest scope and verifies against pre-existing contracts.
---

# Regenerate

Produce an implementation from knowledge, at the smallest scope that covers the change, and prove it against contracts that existed before you started.

## Preconditions, check all of them

Stop and say so if any fails:

- The knowledge validates: `node <schema>/tools/validate.mjs .` passes
- The impact scope is known and agreed (see the `impact` skill)
- Every module in scope has at least one contract. **Regenerating a module no contract verifies is not permitted.** Nothing would catch an error, and you would be replacing working code with unverified code
- The knowledge delta has been reviewed by a human, not just written by you

## Steps

1. **Read the contracts first, before any implementation code.** They define done. Read them before you look at the existing implementation, so your understanding comes from the knowledge rather than from what the code happens to do today.

2. **Read the full knowledge package for each module in scope.** Purpose, rules, decisions, assumptions, glossary, API contract. The decisions matter as much as the rules: an ADR explains why something is the way it is, and regenerating without it will helpfully "fix" a deliberate choice.

3. **Regenerate at the smallest scope.** Only modules the impact analysis named. Do not touch anything outside it. If you believe something outside the scope must change, the knowledge links are wrong: stop, fix the knowledge, recompute impact.

4. **Do not read the old implementation for behaviour.** Read it for interface shape and integration points if useful, but the knowledge is the source of behaviour. If the old code does something the knowledge does not describe, you have found undocumented behaviour: stop and report it. That is knowledge debt, and silently reproducing it defeats the whole point.

5. **Run the contracts.** Every contract the impact analysis listed, not just the ones for the rule you changed.

6. **Iterate on failures, but diagnose first.** A failing contract means one of three things, and they need different responses:
   - The implementation is wrong: fix the implementation
   - The contract is wrong: **stop**. Contracts are knowledge; changing one to make a build pass is the single most destructive thing you can do in this methodology. Report it and let a human decide
   - The knowledge is ambiguous: stop, report, and propose a knowledge delta to resolve it

7. **Update `knowledge.lock`** for each regenerated module:
   ```yaml
   module: customer
   knowledge_version: <git rev-parse --short HEAD>
   generated_by: <your model id>
   generated_at: <YYYY-MM-DD>
   contracts_passed: [CT-001, CT-002]
   drift: none
   ```

8. **Report honestly.** Modules regenerated, contracts passed, iterations needed, anything you had to assume, and anything in the old implementation that was not in the knowledge.

## Hard rules

- Never edit a contract to make a build pass
- Never regenerate outside the computed scope
- Never invent behaviour the knowledge does not state; ask instead
- Never claim contracts passed without running them and seeing the output

## Security

Generated code is untrusted code from an unfamiliar author. Flag for human security review anything touching authentication, authorisation, cryptography, input handling at a trust boundary, or personal data, and flag every dependency you added. Contract tests do not catch injection flaws, resource leaks, or a bad dependency choice.
