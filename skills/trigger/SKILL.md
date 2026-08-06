---
name: trigger
description: Use to decide whether regenerating a module is worth what it costs, and to refuse when regenerating would be harmful. Triggers on "should we regenerate", "trigger", "is it worth rebuilding this", or a scheduled review of decay signals.
---

# Trigger

Regeneration costs money and attention, so *always* and *never* are both wrong. This weighs the signals and proposes, with reasoning.

```bash
REGEN_LLM_MODEL=<what you would build with today> \
  npx -p regen-engineering-schema regen-trigger .
```

## The most useful thing this does is refuse

Two states make regeneration actively harmful rather than merely wasteful, and both are easy to walk into while looking at a dashboard saying a module is unhealthy.

**Code-ahead drift.** The implementation contains behaviour the knowledge does not describe. Regenerating from knowledge deletes it, silently, and the module looks *healthier* afterwards because the evidence is gone. Run `reconcile` first. This is the one that matters most, because every decay signal points at the module at exactly the moment regenerating it is most destructive.

**A failing Regeneration Test.** The knowledge is already known to be insufficient. Spending again to re-prove that buys nothing. Improve the knowledge, then retest.

Neither refusal is a judgement call, which is why the tool makes both mechanically.

## What to weigh, when nothing is blocking

- **Knowledge-ahead**, the strongest signal: the knowledge moved and the implementation has not caught up. This is the ordinary case the loop is built for.
- **Never verified**: no Regeneration Test has run, so regenerability is believed rather than known. Consider running the test rather than regenerating; it answers the question more cheaply.
- **Verification stale**: it passed, but long enough ago that the tree has moved underneath it.
- **Model moved**: built by one model, you would build with another today. This is the manifesto's "regenerate when the model improves" case, and it is worth saying plainly that this claim has never actually been exercised, and that a different model is not automatically a better one. Weak on its own.

**A pass with many guesses is a warning, not a success.** If the last regeneration passed while the agent guessed at nineteen questions, the module got lucky, and luck is not a property you can rely on twice. Answering those questions in the knowledge is cheaper than regenerating and far more likely to help. Prefer it.

## Judgement the tool cannot make

The script knows the signals and a rough cost. It does not know:

- **Whether anyone is about to change this module anyway.** Regenerating a week before a rewrite is waste.
- **Whether the module matters.** A decayed module nobody depends on can stay decayed.
- **What the cost figure actually means here.** It is an order of magnitude, and it says pennies or pounds and no more. Review attention is the scarcer resource and does not appear in it at all.
- **Whether the team can absorb the result.** A regenerated module needs reviewing by someone who understands it, and a queue of unreviewed regenerations is worse than a decayed module.

## Hard rules

- **Never regenerate past a blocker**, whatever the decay score says.
- **Never regenerate to improve a metric.** The number is a proxy; regenerating to move it is optimising the proxy.
- **Propose, do not act.** Every output is a recommendation with reasoning, for a person to accept or decline.
- **Prefer the cheaper answer.** Running the Regeneration Test, or answering the guesses in the knowledge, usually costs less than a regeneration and often makes it unnecessary.

## Report

Per module: the verdict, the signals with their weights, any blocker stated first and plainly, the estimated cost with its basis, and the cheaper alternative if one exists.
