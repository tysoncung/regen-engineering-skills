---
name: mine
description: Use to derive a draft knowledge package from an existing codebase that has no knowledge tree yet. The brownfield entry path for Regen Engineering. Triggers on "mine knowledge", "adopt regen engineering", "document this legacy module", "extract the business rules", or when starting on a repo with no knowledge/ directory.
---

# Knowledge mining

Almost nobody starts from nothing. This is how an existing system enters the methodology.

The insight that makes it work: **nobody writes documentation from a blank page, but everybody will correct a wrong sentence about their own domain.** Your job is to produce something specific enough to be worth correcting. A vague draft gets ignored; a draft that is confidently wrong in an interesting way gets fixed immediately.

## Steps

1. **Pick one module.** Never mine a whole system at once. Choose something with clear boundaries and, ideally, existing tests.

2. **Read widely before writing anything:**
   - The implementation, especially conditionals and validation, since business rules hide in `if` statements
   - The tests, which are often the most honest statement of intended behaviour that exists
   - Commit history and messages, which explain why odd things are the way they are
   - Issue trackers and code comments, particularly any comment explaining a workaround

3. **Draft the knowledge package** following the schema: `overview.md`, then `rules/`, `decisions/`, `assumptions/`, and `contracts/`.

4. **Mark confidence explicitly on every item.** This is the most important part of mining, and the part most likely to be skipped. Use `status: draft` for everything you mined, and state the evidence in the body:

   ```markdown
   Confidence: high. Enforced at src/customer/validate.ts:40 and covered by three tests.
   ```
   ```markdown
   Confidence: low. Inferred from a single conditional with no test and no comment.
   Nobody has confirmed whether this is deliberate or a bug that became load-bearing.
   ```

   A reviewer's attention is finite. Confidence markers point it at what matters.

5. **Separate rules from accidents.** Not everything the code does is a business rule. A rule is behaviour someone decided on. An accident is behaviour that emerged from an implementation choice. When you cannot tell, say so and record it as an assumption rather than promoting it to a rule.

6. **Write contracts for what you can verify.** Where tests exist, they are close to contracts already; restate them at the interface boundary. Where none exist, write the contract you believe should hold and mark it draft.

7. **Validate.** `npx -p regen-engineering-schema regen-validate .`

8. **Hand it to a domain expert.** State clearly what you are asking for: not approval, but correction. Point at the low-confidence items first. Their corrections are the highest-value knowledge capture the organisation will ever do, because that is exactly the knowledge that existed only in heads.

## Then: validate by regeneration

Mining produces a hypothesis. Regeneration tests it.

Once experts have corrected the draft, regenerate the module from the corrected knowledge and run both implementations against the same contract suite. Where behaviour matches, the knowledge was sufficient. Where it differs, you have found a real rule nobody wrote down: mine it, add it, repeat.

> Regeneration is not only how you produce software. It is how you audit whether you understand it.

Note the caveat honestly: a failed regeneration might mean incomplete knowledge, but it might equally mean the model was not up to the task. Diagnose before booking it as knowledge debt. If a careful human engineer, handed only this knowledge package, would also have been unable to build the thing, the debt is real.

## Report

Give the human: what you mined, the confidence distribution, the specific items you most want an expert to look at, and anything in the code you could not explain at all. That last list is often the most valuable output of the entire exercise.
