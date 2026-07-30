---
name: regeneration-test
description: Use to test whether a module's knowledge is actually sufficient, by regenerating it from knowledge alone in an independent context and scoring against pre-existing contracts. Triggers on "regeneration test", "is our knowledge complete", "can we rebuild this", "knowledge debt audit", or on a schedule as a health check. Not for producing shippable code; use `regenerate` for that.
---

# Regeneration Test

This does not produce software. It produces **information about how much you failed to write down**.

> Could you delete a module's implementation today and regenerate a functionally equivalent one from the knowledge alone, with the pre-existing contract suite passing?

Whatever makes the answer no is knowledge debt, itemised.

## The rule that makes it worth anything

**The regenerating agent must not see the implementation it is replacing, and must not be the agent that wrote it.**

This is not fastidiousness, it is the whole experiment. An agent with the existing code in context, or with the domain already in its head from having built it, will reproduce what it remembers and tell you the knowledge was fine. That is marking your own homework, and it yields exactly zero information.

The reference demo learned this the hard way: both implementations were originally written by one agent in one session and proved nothing. Run independently, the same knowledge failed a fifth of its contracts within minutes.

## Steps

1. **Move the implementation out of reach.** Not `git rm`, which leaves it in history the agent can read. Move it outside the repository:
   ```bash
   mv impl/<stack> /tmp/regen-test-backup/
   ```

2. **Dispatch an independent agent** with a fresh context. Give it an explicit allowlist of knowledge files, and an explicit denylist. The denylist matters more:
   - Forbid: any other implementation, the contract runner, the test harness, the README, anything under `.git`
   - Allow: the module's `knowledge/` package, global `knowledge/`, and the interface contract
   - One attempt. No running the contract suite first. Someone else scores it.

3. **Require a guess list before scoring.** This is the highest-value step and the one people skip. Ask the agent to report, before it learns its score:

   > List every question the knowledge did not answer, where you had to guess or infer. Be specific. Also list anything ambiguous or self-contradictory.

   Those guesses are knowledge debt **whether or not the contracts happen to catch them**. In the reference run, eighteen guesses were reported and only three cost contracts; the other fifteen were real gaps nothing had exercised yet.

4. **Score it** against the unchanged contract suite.

5. **Diagnose before blaming the knowledge.** A failure means one of three things and they need different responses:
   - The knowledge was incomplete: real debt, fix the knowledge
   - The model was not capable enough: note the model, do not book it as debt
   - The harness was flaky: fix the harness

   The test question: would a careful human engineer, handed only this knowledge package, have been able to build it? If no, the debt is real.

6. **Fix the knowledge, never the code, and never the contracts.**
   - Editing the implementation to pass leaves the next regeneration failing identically
   - Editing a contract to go green is the most destructive act available in this methodology
   - If a contract is genuinely wrong, stop and escalate to a human

7. **Re-run** with the corrected knowledge to confirm the loop closes.

8. **Record the result** in the module's lock or a findings note: date, model, score, and the guess list.

## Running it continuously

Once it passes, schedule it. Continuous integration proves your code still works; this proves you still understand it. A module nobody has regenerated in six months is not known to be regenerable, only believed to be.

Never block a merge on it. It is slow, it costs money, and it is non-deterministic. Report it as a health signal, the way a dependency audit is reported.

## Report

- Score, per contract, not as a single number
- The guess list, in full, as the primary output
- Which failures were knowledge debt and which were model or harness, with reasoning
- What was added to the knowledge as a result
- Cost: model, tokens if available, wall clock

An honest failure is a better outcome than a pass. A pass tells you the contracts held; a failure tells you exactly what you did not know you were missing.
