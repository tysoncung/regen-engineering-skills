# Regen Engineering skills

The reference implementation of [Regen Engineering](https://regen.engineering), as agent skills.

> Knowledge is the asset. Code is the byproduct.

## Why skills instead of a product

The manifesto calls the pipeline from knowledge to verified code a Knowledge Compiler. It would be reasonable to assume that means building one.

It does not. Agent CLIs already are that compiler. They read repositories, run tools, write files, and iterate against tests. What was missing was not capability but a defined loop: what to change first, what to compute rather than guess, what must never be edited to make a build pass.

That loop is what these eight skills encode. There is no binary to install and no service to depend on, which is also the honest test of the methodology's central claim: if this only worked inside one vendor's platform, it would not be the stack-independent approach the manifesto argues for.

## The eleven skills

| Skill | Does |
|---|---|
| `knowledge-delta` | Turns a task into a reviewable change to the knowledge tree. Writes no code |
| `impact` | Computes the regeneration scope: which modules, which contracts |
| `regenerate` | Rebuilds modules from knowledge, at smallest scope, against pre-existing contracts |
| `verify` | Schema validation, contract results, knowledge debt report |
| `drift-check` | Finds code that changed without its knowledge changing |
| `mine` | Derives draft knowledge from an existing codebase. The brownfield path |
| `regeneration-test` | Audits whether the knowledge is actually sufficient, in an independent context |
| `reconcile` | Turns drift into a drafted knowledge change for review, not a chore |
| `librarian` | Reads the whole tree, not a diff, for contradictions and decay |
| `gatherer` | Finds changes that implied knowledge nobody wrote down |
| `trigger` | Decides whether regenerating is worth its cost, and refuses when it would be harmful |

## The loop

```
task
  -> knowledge-delta      write the change as knowledge, stop
  -> human review         the cheapest place to disagree
  -> impact               compute the blast radius
  -> regenerate           smallest scope, contracts first
  -> verify               validation, contracts, debt
  -> pull request
       drift-check runs here, and blocks code-ahead drift
         -> reconcile     drift becomes a drafted knowledge change, for review
```

Brownfield repositories start at `mine` instead of `knowledge-delta`.

Running alongside all of it, on a schedule rather than on demand:

```
regeneration-test    proves the knowledge is still sufficient
librarian            proves the knowledge is still coherent
gatherer             catches what changed and was never written down
trigger              proposes regeneration, and refuses when it would destroy something
```

Continuous integration proves your code still works. Those two prove you still understand your system. Neither blocks a merge; they report a health signal, the way a dependency audit does.

Note what the first eight have in common: every one begins "use when", and waits for a person who already suspects something. That is the wrong shape for decay, which happens silently and fastest when everyone is busy, which is exactly when nobody runs a knowledge audit. `librarian` was the first skill here that is not invoked because something is already suspected. `gatherer` and `trigger` complete the set [REP-0006](https://github.com/tysoncung/regen.engineering/blob/main/reps/REP-0006-continuous-knowledge-operations.md) specifies, with the Monitor shipping as a tool rather than a skill because its output is a ranked list and ranking is arithmetic.

All four propose. None merges. That boundary matters more the more of this runs unattended, because a system that both proposes and disposes has quietly become the author of the software with nobody deciding that it should.

## What is model work and what is not

A design line runs through this pack: **the model does judgment, scripts do the graph math.**

Impact analysis and debt metrics are deterministic traversals of the knowledge links. Asking a model to eyeball a blast radius is strictly worse than computing it, so those skills shell out to [regen-engineering-schema](https://github.com/tysoncung/regen-engineering-schema) and then apply judgment to what the script cannot see, such as whether the computed scope matches what the change actually means in prose.

What genuinely needs a model: drafting knowledge from a vague task, mining rules out of legacy conditionals, writing implementations, and deciding whether a change is behavioural or a refactor.

`librarian` is where that line is drawn most carefully, because it is the easiest place to get wrong. `regen-librarian` compares numbers, counts references, and checks dates, all of which are arithmetic. Whether two rules actually contradict each other is not, and the manifesto says so plainly: structural tooling can detect that knowledge is missing, and cannot detect that it is wrong. That claim stays true of validators. It is not true of a reader.

## Install

Requires the schema tooling for the deterministic parts:

```bash
npm install -g regen-engineering-schema   # or npx, per command
```

Then copy the `skills/` directory into your agent's skills location. For Claude Code:

```bash
cp -r skills/* ~/.claude/skills/
```

Each skill is a single `SKILL.md` with YAML frontmatter, which is a portable enough format that adapting it to Cursor rules or Copilot instructions is mostly a copy and paste.

## Hard rules these skills enforce

Worth stating outside the skill files, because they are the difference between the methodology working and quietly rotting:

- **Never edit a contract to make a build pass.** Contracts are knowledge. Changing one to go green is the most destructive available action, and `regenerate` is instructed to stop and escalate instead.
- **Never regenerate a module no contract verifies.** Nothing would catch an error, so you would be replacing working code with unverified code.
- **Never regenerate outside the computed scope.** If something outside it must change, the knowledge links are wrong; fix them first.
- **Code-ahead drift blocks a merge.** Reconcile, revert, or record it as drift debt with an owner and a date. Silence is not an option.
- **Security review is not replaced.** Generated code is untrusted code from an unfamiliar author, and contract tests do not catch injection flaws, resource leaks, or a bad dependency.
- **An agent must not verify its own work,** and must not see the implementation it is regenerating. This one is not caution, it is the difference between an experiment and a formality: the reference demo originally had both implementations written by one agent in one session and proved nothing at all. Run independently, the same knowledge failed a fifth of its contracts immediately.
- **A reconciliation is never merged without human review.** Whether the code does what the system *should* do is a human question, and an incident hotfix is exactly where those two diverge.

## Status

Version 0.3.0, draft, alongside [schema 0.3.1](https://www.npmjs.com/package/regen-engineering-schema).

These changed because they were used in anger. Running the change loop and the reconciliation loop end to end on the reference demo produced eleven findings, and the ones that mattered are now in the skills themselves: `verify` warns that traceability counts contracts which exist rather than contracts which would fail, `drift-check` documents what a clean result does not prove, and `reconcile` has a section on hunting for justification you invented. Friction logs live in the demo repository under `docs/`.

## Where these are exercised

- [regen-engineering-demo](https://github.com/tysoncung/regen-engineering-demo), the stateless reference.
- [regen-engineering-stateful](https://github.com/tysoncung/regen-engineering-stateful), the reference for systems that own data.
- [simple-cmdb](https://github.com/tysoncung/simple-cmdb), the brownfield pilot, where the standing agents run weekly against a real tree.

## Licence

MIT for the skills. Documentation is CC BY-SA 4.0.
