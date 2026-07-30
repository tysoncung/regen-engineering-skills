---
name: knowledge-delta
description: Use at the START of any change to a Regen Engineering codebase, before writing implementation code. Turns a task, bug report, or feature request into a reviewable change to the knowledge tree (business rules, decisions, contracts, assumptions). Triggers on "add a feature", "change this rule", "fix this behaviour", "implement X" in a repo containing knowledge/ directories.
---

# Knowledge delta

A change begins by changing knowledge, not code. This skill produces the delta and stops. It does not implement anything.

The point is not automation. It is that **review happens on the knowledge, before the implementation exists**. Arguing about a business rule as one sentence is cheaper than discovering the disagreement in a four hundred line pull request.

## When this applies

Any task that changes what the system does. If the task changes only how it does it, with no observable behaviour change, this skill does not apply; that is a refactor, and refactors do not need knowledge deltas.

## Steps

1. **Read the affected knowledge first.** Find the modules the task touches and read their `knowledge/` packages: `overview.md`, existing rules, decisions, assumptions, contracts. Never draft a delta without reading what is already there, or you will contradict a rule that already exists.

2. **Classify the change.** Decide what kind of knowledge is actually changing:
   - New or changed behaviour: a business rule (`BR-`)
   - A choice between technical options, with consequences: a decision (`ADR-`)
   - Something believed but unverified: an assumption (`ASM-`)
   - A measurable quality target: a non-functional requirement (`NFR-`)
   - Any of the above needs at least one contract (`CT-`) to be verifiable

3. **Write the delta.** Create or edit files under the owning module's `knowledge/`. Follow the schema exactly:
   - Frontmatter carries `id`, `type`, `title`, `status`, and the relation fields. Unknown fields are rejected by validation, so do not invent any.
   - IDs are unique repository-wide. Check existing IDs before choosing one; never reuse a retired ID.
   - Set `affects` to **every** module whose behaviour depends on this, not just the obvious one. This list is the regeneration blast radius, and a missing entry silently shrinks it. That is the most dangerous defect you can introduce here.
   - Changing existing behaviour means marking the old item `superseded` and pointing the new one at it with `supersedes`, rather than editing history away.

4. **Write or update contracts.** A rule with no contract is unverifiable, so regeneration cannot be trusted for it. Write contracts in given/when/then form at an **interface boundary**: HTTP calls, command-line behaviour, data formats, observable effects. Never reference internal class or function names, because a contract that reaches inside an implementation cannot survive that implementation being regenerated in another language.

5. **Validate.** Run the schema validator. Fix everything it reports before continuing:
   ```bash
   npx regen-validate .
   ```

6. **Report the delta and stop.** Show the human:
   - Which files were added or changed, and why
   - The regeneration scope (run the `impact` skill, or `npx regen-impact <ID>`)
   - Any assumption you had to make, recorded as an `ASM-` item rather than left in your head
   - Any question you could not answer from the existing knowledge

**Do not implement.** The next step is human review of the delta. Implementation happens afterwards, through the `regenerate` skill.

## What good looks like

A delta is ready for review when a colleague who has not seen the task could read only the changed knowledge files and correctly predict how the system will behave afterwards.

## Common mistakes

- Writing the rule and the contract to say the same thing in different words. The contract must state something checkable, with concrete values and edge cases.
- Listing only the module you are working in under `affects`. Ask what else reads this behaviour.
- Burying a decision inside a rule. If a choice was made between real alternatives, it is an ADR and the alternatives belong in it.
- Silently resolving an ambiguity. Record it as an assumption and say so in the report.
