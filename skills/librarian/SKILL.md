---
name: librarian
description: Use on a schedule to read a whole knowledge tree at once and find what only a reader can see. Triggers on "librarian", "review the knowledge", "audit the tree", "are these rules consistent", "what contradicts what", or a scheduled corpus review. Unlike every other skill here, this one is not invoked because something is already suspected.
---

# Librarian

Every other skill in this methodology looks at a change: a delta, a diff, a regeneration, a drift finding. This one looks at the **corpus**, on a schedule, and hunts for the failures that only appear when items are read against each other.

It exists because of a specific failure. The reference demo carried a rule capping a customer's address book at twenty alongside another describing an address list response of fifty. Both files were individually well formed, so validation reported no problems. No contract asked, so the suite stayed green. Everything needed to catch it had already been built, and nothing was watching. A person found it days later by reading the two files side by side.

## The thing that makes this different

You are reading **everything at once**. That is not a stylistic preference, it is the entire job. A contradiction has no location: it is not in either file, it is between them, and any process that looks at one item at a time is structurally incapable of finding it.

So do not skim, and do not sample. If the tree is too large to hold at once, partition it by module and read each partition whole, then do a final pass over the cross-module knowledge and the module overviews together, because that is where cross-module contradictions live.

## Steps

1. **Run the mechanical pass first**, and treat it as a list of places to look rather than a list of findings:

   ```bash
   npx -p regen-engineering-schema regen-librarian .
   ```

   It compares numbers, counts references, and checks dates. It cannot read. An empty report means nothing was structurally odd, which is not the same as nothing being wrong, and the demo's contradiction is the proof: it sat in a tree that validated cleanly.

2. **Get the corpus in front of you.** `regen-librarian . --bundle` emits every item with its frontmatter and body, plus the mechanical candidates. Read it end to end before forming any conclusion.

3. **Hunt, in roughly this order of value:**

   - **Contradictions.** Two items that cannot both be true. The valuable ones are never adjacent: a rule and an assumption, a decision and the NFR it quietly violates, a contract asserting behaviour a newer rule forbids. Watch particularly for the same word used in two senses in one module, which is how contradictions hide in plain sight.
   - **Unstated dependencies.** Knowledge stated in one place and silently assumed in another. These are not visible as errors and they break regeneration, because the module being regenerated does not carry the fact it depends on.
   - **Confidence that should have moved.** A low-confidence mined item that three later changes have implicitly confirmed, or a high-confidence item since contradicted by an incident. Both directions matter, and the second is more dangerous.
   - **Staleness.** Drafts never promoted or retired, assumptions never revisited, decisions whose rejected alternatives have since been adopted anyway.
   - **Orphans.** Rules nothing references, contracts verifying nothing. Sometimes dead knowledge to retire; sometimes something depends on it and failed to say so, which silently shrinks the regeneration scope.
   - **Duplication.** The same fact in two places is drift waiting for one copy to change.

4. **Rank ruthlessly before reporting.** This skill fails on attention, not accuracy. Twenty findings nobody reads is worse than three that get fixed, because it trains people to ignore the channel where the real one will arrive. Order by what would break if it stayed wrong.

5. **Propose, do not merge.** Output is a report, or a delta marked `status: draft`, or an issue. Never a merged change.

## The failure mode to hunt for in your own findings

Before reporting, reread every finding looking for **a contradiction you constructed rather than found**. Two rules that use a word differently may be a genuine ambiguity or may be ordinary English. A rule and an assumption that seem to disagree may be describing different states.

The test: quote both items verbatim and check the conflict survives the quotation. If stating the finding requires you to paraphrase either side, you probably built the contradiction in the paraphrase.

This is the same failure the `reconcile` skill warns about, in a different costume. Fluent, confident, wrong findings cost more review attention than missed ones, and a Librarian that cries wolf gets switched off, after which it catches nothing at all.

## Hard rules

- **Never edit knowledge to resolve a contradiction you found.** Deciding which of two rules is right is a domain judgement and belongs to a human. Surface both, say which you believe is wrong and why, and stop.
- **Never report a finding without quoting the evidence.** A finding that cannot be checked in ten seconds will not be checked.
- **Say unsure when unsure.** A ranked list where the bottom half is marked uncertain is far more useful than one where everything is asserted flatly.
- **Report finding nothing plainly.** A quiet week is a real result. Padding it with weak findings is how the channel dies.

## Report

What you read, in items and modules. The findings, ranked, each with the items involved and both sides quoted. What you were unsure about. And explicitly: what you checked and found clean, because that is what makes a short report trustworthy rather than lazy.
