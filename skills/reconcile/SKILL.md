---
name: reconcile
description: Use when code-ahead drift is found, to turn an unplanned code change into a drafted knowledge delta for human review. Triggers on "reconcile", "drift debt", "the code changed but the knowledge did not", after drift-check reports a finding, or when clearing a hotfix that shipped without a knowledge change.
---

# Reconcile

Drift-check finds that an implementation changed while its knowledge did not. This turns that finding into a **proposed knowledge change**, so a human reviews a draft instead of facing a blank page.

That inversion is the point. Documentation has always failed because keeping it true is work nobody is paid to do. Nobody writes it from nothing; everybody will correct a wrong sentence about their own domain.

## When this applies

After `drift-check` reports code-ahead drift, or when clearing drift debt recorded during an incident.

If the change was a pure refactor with no observable behaviour change, there is nothing to reconcile. Record that judgment and stop.

## Steps

1. **Read the actual change**, not the commit message. `git diff` the drifted paths against the last commit where the lock matched.

2. **Read the module's existing knowledge first.** A reconciliation that contradicts a rule already written is not a reconciliation, it is a second source of truth. If the code now contradicts an existing rule, that is the finding: say so loudly rather than quietly adding a rule that disagrees with its neighbour.

3. **Work out what is now true** that the knowledge does not know. Separate carefully:
   - **Behaviour** a user or caller could observe: this becomes a business rule
   - **A choice between real alternatives**: this becomes a decision, with the alternatives that lost
   - **Something believed but unverified**, very common after incidents: this becomes an assumption, not a rule
   - **Incidental implementation detail**: this belongs in neither; do not promote it

4. **Write the delta** following the schema, marked `status: draft`. State the evidence in the body, including the commit and the reason if known:

   > Reconciled from the hotfix in a1b2c3d during incident 4412. The retry limit was
   > lowered to 3. Whether 3 is correct or merely what stopped the bleeding at 2am has
   > not been established.

   That last sentence is the kind of honesty that makes reconciled knowledge trustworthy. Do not launder an emergency decision into a considered one.

5. **Write or update a contract** if the new behaviour is checkable. Unverified reconciled knowledge is barely better than a comment, because nothing stops the next regeneration from losing it.

6. **Validate**: `node <schema>/tools/validate.mjs .`

7. **Clear the drift debt** in the module's lock once the delta is merged: set `drift: none` and remove the `drift_debt` block.

8. **Report and stop.** Show the human the proposed delta, the evidence, what you could not determine, and anything the change contradicts.

## Hard rules

- **Never merge a reconciliation without human review.** The whole point is that a person decides whether what the code does is what the system *should* do. Those are different questions, and an incident hotfix is exactly where they diverge.
- **Never silently normalise a bad change.** If the code now does something that looks wrong, say so. Reconciliation records reality; it does not bless it.
- **Never delete a contradicting rule to make things tidy.** Surface the contradiction and let a human resolve it.

## Report

The proposed delta, the evidence behind each item, confidence on anything inferred, what remains unknown, and any contradiction with existing knowledge.
