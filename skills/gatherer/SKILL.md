---
name: gatherer
description: Use on a schedule to find changes that implied knowledge nobody wrote down. Triggers on "gatherer", "what did we learn and not record", "catch up the knowledge", after an incident, or a scheduled sweep of recent commits. Distinct from mine, which bootstraps a codebase with no knowledge at all.
---

# Gatherer

Watches merged changes and asks one question of each: **does this imply something true about the system that the knowledge does not contain?**

Distinct from `mine`, which is a one-shot bootstrap for a codebase with no knowledge at all. This is the steady state, and the catch is smaller and harder: the commit whose message contains a business rule, the incident fix that reveals a constraint, the review comment where someone explains why something must be a certain way.

Those are the moments when knowledge exists in a person's head and is briefly written down somewhere that is not the knowledge tree. They are also the moments it is cheapest to capture and easiest to lose.

## Steps

1. **Collect the candidates:**

   ```bash
   npx -p regen-engineering-schema regen-gather .
   ```

   The filter is the whole idea: commits that changed an implementation and changed no knowledge. A commit that touched both has already recorded itself, whatever else it did. What remains is the set of changes that had something to say and no place to say it.

   If it reports **CANNOT TELL**, stop and fix that first. It means nothing in the range touched anything recognised as implementation, which almost never means a quiet history and almost always means `implementation_paths` is missing from the lock. A tool searching the wrong directory finds nothing, and finding nothing looks exactly like good news.

2. **Read the diffs and the messages.** `--bundle` emits the packet, `--read` sends it to a model.

3. **For each candidate, sort what you find:**
   - **Behaviour a caller could observe** becomes a business rule
   - **A choice between real alternatives** becomes a decision, with the alternatives that lost
   - **Something believed but unverified**, very common after incidents, becomes an assumption and not a rule
   - **Incidental implementation detail** belongs in none of them, and must not be promoted

4. **Most commits imply nothing.** Say so and move on. A refactor with no observable behaviour change has nothing to gather, and inventing knowledge for it is worse than skipping it, because it dilutes the tree with items nobody needs and everybody has to read.

5. **Write drafts,** `status: draft`, evidence quoted, confidence stated. Never merged.

## The failure mode to hunt for in your own draft

Reread every draft looking for **justification you invented**. This is not hypothetical: exercising the reconciliation loop on the reference demo produced a draft asserting a database-shaped cause in a system with no database, and it read as the most competent sentence in the file. Fluency is what makes it dangerous.

Check every causal claim against the diff and the message. If the commit does not say why, your draft must not either. Write "the reason was not recorded" and leave it, because a gap invites a question and a confident wrong sentence closes it.

## Hard rules

- **Never merge a draft.** A person decides whether what the code does is what the system *should* do. Those are different questions, and an incident hotfix is exactly where they diverge.
- **Never launder an emergency into a design.** "The limit is three" and "the limit is three because someone picked it at 2am and it stopped the bleeding" are different knowledge, and only the second is honest.
- **Never normalise a change that looks wrong.** If the code now does something indefensible, say so. Gathering records reality; it does not bless it.

## Report

How many commits were examined, how many implied nothing, the drafts with evidence quoted, and what you could not determine.

The count of commits that implied nothing is not padding. It is what makes the ones that did credible.
