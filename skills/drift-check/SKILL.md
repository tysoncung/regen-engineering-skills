---
name: drift-check
description: Use to detect divergence between knowledge and implementation in a Regen Engineering repo, especially when reviewing a pull request or investigating why knowledge looks out of date. Triggers on "drift", "is the knowledge current", "did anyone change the code without updating knowledge", or during code review.
---

# Drift check

Drift is divergence between knowledge and implementation. It has a direction, and the two directions mean opposite things.

**Knowledge ahead of code** is normal. Someone decided something and the implementation has not caught up. That is a backlog item, not a problem.

**Code ahead of knowledge** is a defect. Something is true of the running system that the source of truth does not know. This is how documentation has always rotted, and it is what this check exists to prevent.

## The mechanical rule

Detection does not require comparing prose to code. It is structural:

> If a change touches a module's generated paths and contains no corresponding change to that module's knowledge, it is code-ahead drift.

No semantic understanding is needed, only the observation that a build artifact changed while its source did not.

## Steps

1. **Get the changed paths.** For a pull request:
   ```bash
   git diff --name-only main...HEAD
   ```

2. **Partition them** into knowledge paths (under any `knowledge/` directory) and implementation paths (everything else in a module).

3. **For each module with implementation changes**, check whether the same change touches that module's knowledge. If not, that module has code-ahead drift.

4. **Confirm it is behavioural.** This is where your judgment is needed, because the mechanical rule over-reports. A pure refactor with no observable behaviour change is not drift. Ask: could a contract distinguish before from after? If no, it is a refactor; note it and move on. If yes, or if you are unsure, it is drift.

5. **Cross-check the lock files.** `node <schema>/tools/debt.mjs .` reports declared drift and, in a git repository, compares each lock's `knowledge_version` against the last commit touching that module's knowledge.

## What this does not catch

The rule fires when implementation files change and knowledge files do not. Two consequences worth knowing before you trust a clean result:

- **On a branch that changes both, it always passes.** It cannot tell whether the knowledge that moved had anything to do with the code that moved. A large change with a cosmetic knowledge edit reads identically to a careful one.
- **It is a merge-time check, not a review-time one.** Its value is catching the hotfix that lands with no knowledge at all, which is what it was built for. Do not read a clean result on a feature branch as evidence the knowledge is right.

Judging whether the knowledge actually describes the change is human work, and no tool here does it.

## What to do about it

**Blocking finding.** Report code-ahead drift as blocking. The author has three legitimate options, and you should say which you recommend:

1. **Reconcile.** Write the knowledge delta that describes what the code now does. This is the right answer almost always, and the `knowledge-delta` skill does it.
2. **Revert.** If the change was not meant to alter behaviour.
3. **Accept as drift debt.** For genuine emergencies only. Record it in the module's lock:
   ```yaml
   drift: code-ahead
   drift_debt:
     since: 2026-07-30
     reason: hotfix for incident 4412, payment retry loop
     reconciliation_task: ENG-991
   ```
   Drift debt is the acute form of knowledge debt: known cause, known fix, and it should be measured in days. An emergency hatch that becomes routine is just rot with paperwork.

What is never acceptable is silence.

## Report

For each module: drift direction, the evidence (which paths changed without which knowledge), whether you judged it behavioural, and the recommended action. If there is no drift, say that plainly and show what you checked.
