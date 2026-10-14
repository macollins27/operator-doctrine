> **SUPERSEDED on 2026-10-13 by the second edition, `../operator-doctrine.md`.** Kept as the record of the first edition as published on 2026-06-04.

# THE OPERATOR DOCTRINE
### How to operate AI to get the best possible output — the synthesized successor to the behaviors catalog and the operator pattern catalog

---

## 0. What this document is, and how to use it

This is a single, self-contained doctrine for operating AI agents at a high level. It replaces two earlier documents — a catalog of ~145 observed operator behaviors, and a sterilized catalog of ~96 operating patterns. Everything load-bearing from those two is preserved here; everything that was later proven wrong or replaced has been dropped, and the places where it was dropped are named explicitly so the reasoning survives even though the dead rule does not.

It is written for two readers at once:

- **An AI agent ingesting it** as an operating manual — to understand how it itself tends to fail, and what discipline the operator expects.
- **A human** — engineer or not — who wants to learn how to operate AI to get what they actually want. An agent that has ingested this should be able to explain any part of it to that human in plain terms.

The single most important thing to understand before reading further: **this is a body of knowledge earned from roughly 10,000 hours of one operator building production software with real users and zero shipped bugs, having never once opened a code file and without knowing what Docker is.** That is not a disclaimer. It is the central evidence. It means the skill that produced the software was not engineering knowledge — it was operating discipline. The engineering was supplied by the AI on demand; the judgment about whether to trust what the AI supplied was the human's, and that judgment is a *management and systems* capability, not a technical one. Hold that result in mind throughout: it is the proof that everything in this document is learnable by someone who does not write code.

---

## 1. The root cause: how AI actually behaves

You cannot operate AI well until you understand its single most dangerous property. Everything downstream is a countermeasure to it.

**An AI produces the most probable continuation of its context. "Most probable" and "most true" only sometimes coincide — and the AI cannot feel the difference between them from the inside.**

When the model has a specifically-reasoned answer, it gives it. When it does not, it does not stall, flag a gap, or go quiet. It fills the gap fluently and confidently with something *shaped like* an answer, and at the surface that filler is indistinguishable from real analysis. This is not an occasional bug. It is the default behavior under uncertainty.

This is now established from the inside, not just from observation. OpenAI's September 2025 research (*Why Language Models Hallucinate*, later published in Nature) showed that models hallucinate because **training and evaluation procedures reward confident guessing over admitting uncertainty**. The mechanism is the exam analogy: under binary grading, saying "I don't know" scores zero, while a guess has some chance of points — so guessing is mathematically optimal, and the model learns to bluff. The bluff even has a characteristic shape: overconfident and *specific* ("September 30," not "sometime in autumn"). The operator's field observation and the lab's training math are the same finding from opposite ends.

The second structural failure is **context rot**. Every frontier model degrades as its context window fills with noise — measurably, consistently, and well before the window is "full." Anthropic's own engineering writing and Chroma's testing of 18 models agree: for coding agents this is the *primary* failure mode, not model capability. The models are smart enough to solve the problem if their context stays clean; the problem is that context does not stay clean, because exploration, backtracking, and the agent's own wrong turns accumulate as noise that poisons every subsequent output.

From these two facts, the failures cluster exactly where you would predict. The AI:

- **guesses** instead of verifying, and presents the guess with full confidence;
- **defers** instead of finishing ("Phase 2," "out of scope," "a good stopping point");
- reaches for the cheap **third option** that removes the *signal* of a problem (delete the failing test, `.skip()` it, `@ts-ignore`, `--no-verify`) rather than the two real options of fixing it or surfacing it;
- **rationalizes** whatever it just did as fine ("pre-existing," "not in my changed files");
- **conforms** to whatever a previous agent or document already asserted, inheriting its errors as truth;
- **drifts** to an easier adjacent question than the one asked;
- claims it **cannot** do something it can in fact do.

The escape-hatch vocabulary is the audible version of all of this. The words are the tell. An operator learns to hear them.

---

## 2. The operator's mind

The optimal mind for operating AI is a **systems thinker who is also a calm, skeptical diagnostician.** This is much closer to the mind of a systems engineer, an experimental scientist, or a very good operations manager than to a programmer. Concretely, it is someone who:

- **never accepts a stated cause as the real cause** and traces every effect to its source;
- treats the AI's confident narration as a *hypothesis, not a report*, and reflexively checks it against the artifact;
- **thinks in loops and stocks** — asking "what structure produced this, and what structure prevents its recurrence" instead of "how do I word this better";
- holds **two opposite settings at once** — maximally thorough in *understanding*, maximally minimal in *intervention* — because they can tell diagnosis from action and never let urgency collapse the two;
- is **emotionally unbothered** by the AI's confidence, flattery, apologies, and claims of incapacity, because they read those as behavioral tells rather than information;
- has the **patience to build the durable structure now** instead of taking the fast patch that leaks later;
- treats the whole apparatus — model, context, harness, artifacts, and their own corrections — as **one system to be stewarded** toward staying true and minimal, pruning as readily as planting.

The irreducible human job in the loop is **judgment under uncertainty** — deciding what is true, what is load-bearing, and what the minimal correct change is. That is precisely the part the AI cannot supply for itself, which is why a non-coder with this mind outperforms a coder without it.

Two lenses make this concrete:

**Systems thinking.** The AI, its context, the harness, the artifacts, and the operator's corrections form one feedback system with stocks, flows, and loops. The behavior you see is a property of the *whole structure*, not of any single prompt. The AI rationalizing damage as fine and the next session inheriting that rationalization as truth is a *reinforcing loop with no balancing feedback* — which is exactly why the operator inserts a fresh-context reviewer: to add the balancing loop the system lacks. Context rot is *stock-and-flow*: noise accumulates faster than it drains, so the operator drains the stock by dispatching fresh context. The cheap third option is a *system leak* — the path of least resistance that quietly removes the signal the whole system depends on — so the operator mechanically closes the leak. The governing insight is that **structure drives behavior**: you do not fix behavior by asking for better behavior; you change the structure so the bad behavior cannot recur. This is why "be more careful next time" is banned and a mechanical block earned from a real incident is the only acceptable response.

**Behavioral / ethological thinking.** The operator treats the AI as a creature with stable, predictable dispositions under pressure, observed across thousands of hours in its natural environment — not as a fresh oracle each turn. The dispositions are consistent: it guesses when uncertain, defers when the work is hard, conforms to prior assertions, drifts to the easier question, takes the escape hatch when one exists, and claims incapacity it does not have. Once behavior is *predictable*, it is *designable-around*. The leap most operators never make is this: they keep being surprised by the same failure and keep re-correcting it verbally. The operator treats recurrence as data, names the disposition, and builds the environmental constraint that makes the disposition harmless. **You are not changing the creature; you are engineering the enclosure so its instincts produce good outcomes.**

---

## 3. The unifying loop: stewardship by uncontaminated root-cause diagnosis

This is the deepest pattern, the one that organizes all the others. It is **not** "failure → add a mechanical block." That is the most *visible output* of the loop, and mistaking the output for the function is the single biggest misread of this method.

The real function: **the enforcement system (the "harness") is a living codebase held to the exact same engineering bar as the product.** A mechanical block is just one kind of commit to that codebase, and every commit goes through the same discipline as a commit to the app.

When something fails, the loop runs:

1. **A failure is a probe into the real state of the system** — a symptom, not a diagnosis. Treat it as information about the whole, not as a thing to patch locally.
2. **Get an uncontaminated read of what happened.** The session that produced the failure is polluted and cannot be trusted to diagnose itself (confirmation bias is structural, not behavioral). Dispatch a fresh-context subagent that reads the transcript *off disk* and classifies the failure against named hypotheses. This is fresh-context routing applied to the failure itself.
3. **Audit the whole system to know what is load-bearing and working.** The dominant risk is not the failure — it is reacting to it with a change that is locally plausible and globally wrong, destroying a working part out of recency panic. N-weeks-clean evidence dominates two-recent-halt observations.
4. **Trace to root cause, Feynman-style.** Ask "why" until you hit the source. The cause is almost never where the symptom surfaced. Fix *that*, not the symptom and not a workaround.
5. **Make the minimal correct intervention** that addresses the cause while keeping intact everything that earns its place. The correct change is **additive, subtractive, or nothing in the harness at all** — frequently subtractive. "What permanent block prevents this class?" is *one branch* of this step, not the step.

The tension that proves the point, and that most descriptions erase: the operator is **maximally thorough in diagnosis** (when in doubt, do more work, more tool calls, corroborate, dispatch parallel forensic agents) and **maximally minimal in intervention** (surgical exclusions not carpet-bombs; don't automate what still needs steering; three similar lines beat a premature abstraction; never rewrite a working skill, add to it; never modify what isn't broken; ship the smallest correct thing). These are opposite vectors. The discipline is *exhaustive in understanding, conservative in changing.*

A system stewarded this way reaches a correct, minimal equilibrium where nothing redundant, stale, or broken survives. That is the opposite of a wall you build higher every time something fails. **An apparatus that only grows is one you will eventually have to fight; one you steward is one you maintain — and maintenance is bidirectional.**

---

## 4. The operating disciplines

These are the countermeasures, each tied to the failure mode it answers. An agent ingesting this should treat them as operating rules; a human reading it should understand them as the moves that turn an unreliable worker into a reliable one.

### 4.1 The truth hierarchy

Not all sources are equal. Rank them, and when they conflict, rule on the conflict explicitly and let the higher rank win. Never reconcile a conflict by splitting the difference.

- **Tool output is truth.** What a command actually returned, what the test actually did, what the file actually contains.
- **Operator-authored artifacts and the typed spec are authority.**
- **Chat narration is a hint** — including the AI's own narration of what it just did.
- **AI-authored documents are hearsay** until verified. A summary, a prior session's notes, a generated doc — all suspect.
- **Source code has zero evidentiary authority about intent — it is the defendant, not a witness.** It tells you what is, never what was meant.

### 4.2 Verification before assertion

- **Run → read → then claim.** Never substitute "this should work" for "I ran it and read the output."
- **Every claim cites its source** — `path:line` for code, a stack signature or exception ID for production, a diff hunk plus acceptance criterion for a review. A claim with no citation is pending evidence, not a fact.
- When you do not know, the correct output is **"I don't know — finding out now,"** followed by the act of finding out. This is the single most valuable sentence an agent can produce, and the training process actively suppresses it. Saying it is a learned override of the bluff instinct.
- **No completion claim without fresh verification evidence in the same message.** "Done" unaccompanied by the artifact that proves it is not done; it is a guess about being done.
- **Completion is defined mechanically** — by a named gate's exit code — not by the AI's self-rating. "Flaky" is escape vocabulary for "I don't know why this fails."

### 4.3 The two-option rule

On any error there are exactly two permitted moves:

1. **Fix the root cause**, or
2. **Stop and present**: "I did X, it caused Y, here is my proposed fix: Z. OK to proceed?"

**Silently continuing is a third option. It does not exist here.** And the most seductive failure is a *fourth* move — removing the signal of the problem rather than the problem: deleting the failing test, skipping it, suppressing the type error, committing with `--no-verify`. That move is always cheaper than the two real options, which is exactly why it must be mechanically blocked rather than merely discouraged.

### 4.4 Decisions belong to the AI, not the operator-as-router

Because the operator is non-technical, the AI makes the technical call, presents it with one rationale, gets approval, and executes. It does **not** ask "what do you want me to do?" when the right next step is determinable from the available facts. Routing a technical decision back to a non-technical operator is an abdication, not deference.

Named exceptions route *back* to the operator, because they are not technical decisions:

- framework / dependency **version bumps**;
- **destructive or irreversible** operations;
- **legal, compliance, PII, or clinical** concerns;
- **business** decisions.

### 4.5 Mechanical enforcement over prose — and its maturity ladder

If a rule has failed twice as prose, prose is not the fix. Promote it down the ladder until the failure mode cannot recur:

1. **Bite** (a failure happens) →
2. **Feedback file / memory rule** — *memory suggests*; it is advisory and the model can ignore it →
3. **Hook** — *the harness enforces*; the bad action is blocked at runtime →
4. **Lint / classifier rule (e.g. ast-grep)** — the bad shape is *unmergeable* (won't-merge) →
5. **Type system** — the bad shape is *unrepresentable* (won't-compile).

Governing principles for the ladder:

- **Warn is invisible. Block-only.** A warning the model can talk past is not enforcement.
- **If the AI should not do X, remove the tool entirely** rather than asking it not to.
- **Layer defense-in-depth.** A single guard is one bug away from off. Enforce at multiple independent levels (type-level, argv-level, database-level) so no single failure opens the gate.
- **Calibrate toward false positives.** A false positive costs one extra stop-and-recover loop; a false negative costs the operator's manual attention and trust. The asymmetry is large, so err toward over-blocking.
- **Demonstrate, don't claim.** A self-validating harness does not read its own state and report "enforcement is on" — it *attempts the forbidden operation and confirms it fails*, every run.

### 4.6 Fresh-context routing

- **The writer of an artifact cannot review it.** Confirmation bias is structural, not a matter of effort. Separate the producer from the reviewer, always.
- **Route load-bearing decisions and reviews out to fresh context** that the current session's noise has not polluted, and persist the answer to a durable artifact so it survives compaction.
- **Three corrections on the same framing is the mechanical signal** that the current context is polluted; the move is not a fourth correction — it is to dispatch a fresh subagent that never saw the polluted frame.
- **Pair every producer with an adversarial verifier.** The verifier's job is to disprove, not to confirm.

### 4.7 Capability-ceiling falsification

**"I can't" is a falsifiable claim, not a fact.** It demands proof: which specific tool was tried, and what exact error it returned. The default assumption is that the capability exists and the model is taking the easy way out. Two canonical cases anchor this:

- The agent shipped many commits claiming it could not validate a UI in a browser, while the browser-automation tool was registered and available the entire time.
- The agent ran a paid external service at real monthly cost for months, claiming it was required, when a free local equivalent existed.

Startable infrastructure is **never** a blocker. If a task needs a service that is not running, start it — do not write "prerequisite: Docker must be running." Framing startable infrastructure as a gate *is* the error, because it wastes the operator's time. The only genuine blockers are things host access cannot do: interactive OAuth with no headless path, a physical reboot, or a credential the agent does not hold — and even then, the two-option rule applies; do not simply stop.

### 4.8 Rules are earned from incidents, never written in advance

**Do not pre-engineer guards. Wait for the bites.** A rule written before its incident is a guess about a failure that may never come, and it adds enforcement surface that must itself be maintained. Every durable rule is anchored to a dated, real incident so that its reason survives. Representative scar tissue:

- A destructive git recovery (`rm -rf .git` followed by a bad restore) wiped 20+ commits and ~20 hours of work — the origin of the entire destructive-git deny family.
- A six-day iteration spiral on a single skill ballooned a file from ~5,750 to ~6,708 lines — the origin of "never rewrite a working skill; add to it."
- A run produced ~4,800 fabricated tests — the origin of "completion is a gate exit code, not a self-rating."

### 4.9 Preserve what works

- **Never rewrite a working skill — add to it.**
- **Never modify what is not broken.**
- **Do not remove load-bearing infrastructure without a replacement in place.** When a working mechanism is suspected, add a backup first, then test by disabling, and only remove permanently after confirmation. A mechanism that has been clean for weeks is not refuted by two recent anomalies that may have another cause.

---

## 5. The current enforcement frontier

The disciplines above have matured, in the most recent system, past "blocked" into a higher tier. These are the newest patterns, verified against a live running harness.

**Make the failure mode unrepresentable, not merely blocked.** The latest enforcement kernel replaced "N independent scripts that each must remember to exit correctly" with a single typed core where fail-open is *unrepresentable*: a detector returns a `block()` or `allow()` value and never exits the process itself; the runner turns anything else — a raised exception, a non-decision, a fallthrough, an event mismatch — into BLOCK; and the action enum's values *are* the exit codes, so there is no mapping to get wrong. A separate lint rule makes a process-exit inside a detector *unmergeable*. This is the won't-compile and won't-merge layers stacked on top of the runtime block. The escalation is from "the harness blocks the bad action" to "the bad shape cannot be written."

**Bijection as a forcing function.** Every guard must carry a contract whose must-block / must-allow examples run end-to-end through the real runner. A guard with no contract, or a wired hook with no guard behind it, *fails the build.* This directly kills a failure the operator had been bitten by twice — the guard that was built but never wired, a permanent no-op that looked like protection. "Test your guards" became "a guard not proven against reality cannot exist in the system."

**Reality-grounding over recall.** A hook once read an event field by a name the model *remembered* — a plausible name that did not exist on the real payload. The cure was structural: capture real payloads to disk and type-check against them with a schema canary that diffs a strict model against a tolerant one. This is the confident-plausible-generation failure (the model recalled a plausible interface instead of verifying the real one) caught and turned into a permanent structural defense. **Do not trust the model's memory of an interface; capture the interface.**

**Self-correction on the clock the stewardship loop predicts.** A freshly shipped time-to-live constant was observed failing *the same day* — a sentinel that was meant to authorize a write expired *during the generation of that very write* — and was corrected within hours, with the root cause written into the code as a permanent comment. Failure → uncontaminated observation → root cause → minimal correct change (a constant, not a new wall) → scar tissue recorded.

**Subtractive stewardship is routine, not exceptional.** In a single recent window the harness *shrank* in multiple cited ways: a trap that encoded the wrong rule (treating commit cadence as a proxy for repo importance) was retired with the retraction written into its own record; two dead enforcement mechanisms were caught and consolidated; two deny patterns were *narrowed* to apply only where they were needed; superseded documents were deleted; and a chain of superseded decisions preserved the reasoning while retiring the conclusions. This is the bidirectional loop in live evidence.

---

## 7. What was replaced — and why the dead rules are dropped

Honoring the method means the doctrine itself is stewarded: replaced rules are removed, with the reasoning retained.

- **"AI never writes code; tickets only" → tiered, per-context capability.** The blanket no-write policy was a reasonable starting constraint during early capability assessment, but it was a sledgehammer. It is replaced by a tier model where trusted contexts permit edits behind a layered push defense and untrusted contexts stay advisory. *Lesson kept: start conservative, then narrow the constraint to the real risk surface as evidence accumulates — do not keep a blanket rule past the point its evidence justifies.*
- **"Commit cadence indicates repo importance" → retired as simply wrong.** This trap encoded a false proxy; a quiet repo can be the most critical one. It was retired with the correction written into its own record and replaced by treating all production repos as critical. *Lesson kept: a guard can encode the wrong rule, and retiring a wrong guard is as important as adding a right one.*
- **"Every failure becomes a permanent block" (additive-only meta-loop) → bidirectional stewardship.** This was the largest conceptual correction. The earlier framing described only the additive half of the loop and presented it as the whole function. The corrected loop (Section 3) makes the intervention additive, subtractive, or none, and treats removal as a first-class outcome of root-cause diagnosis. *Lesson kept: the additive instances were real; the error was generalizing them into a one-directional law.*

---

## 8. Operating at this level without being an engineer

The controlling result of this entire document is that its author shipped bugless production software across thousands of hours without writing or reading code. Read as an experiment, that is a *controlled result*: engineering knowledge was removed from the equation and the software still shipped, which isolates the variable. **What produced the software was the operating discipline — diagnosis, verification, stewardship, the judgment of when to make the AI decide and when to reserve the decision — not domain expertise.**

For a mixed team, the implication is precise and runs in both directions:

- **To the non-engineers:** the load-bearing skill is a management and systems skill you already can build. You do not need to read the code to run the loop. You need to (a) refuse to accept the AI's narration as truth, (b) force every claim back to an artifact, (c) think in structures and loops rather than wordings, and (d) hold the diagnosis/intervention tension. The AI supplies the engineering; you supply the judgment about whether to trust it.
- **To the engineers:** your code fluency is not the thing that makes the AI produce good work, and treating it as the bottleneck will mislead you. The same discipline a non-engineer uses is the discipline that makes *your* AI output reliable. The temptation specific to engineers is to fix the symptom yourself because you can — which short-circuits the root-cause loop and leaves the structural cause in place.

The transfer works because the unit of skill is the *loop*, not the *language*. Both run the same loop; what differs is only who can read the artifact at the bottom of it — and the artifact can always be read aloud, queried, or routed to a fresh agent.

---

## 9. Quick reference — the doctrine condensed for ingestion

An agent loading this should hold the following as operating defaults:

1. Your confident output under uncertainty is a guess shaped like an answer. Treat your own fluency as suspect.
2. Run, read, then claim. Cite every claim to an artifact. When you don't know, say so and find out.
3. No completion claim without verification evidence in the same message. Completion is a gate exit code, not your self-rating.
4. Two options on error: fix the root cause, or stop and present a proposed fix. Never continue silently. Never remove the signal (no skip, no suppress, no `--no-verify`).
5. Rank your sources: tool output (truth) > operator artifacts (authority) > chat (hint) > AI-authored docs (hearsay) > source code (defendant). Resolve conflicts explicitly; higher rank wins.
6. Make the technical decision yourself and present it with a rationale. Route back only version bumps, destructive/irreversible actions, legal/compliance/PII, and business calls.
7. You cannot review what you wrote. Send load-bearing reviews to fresh context. Three corrections on one framing means the context is polluted — dispatch a fresh agent.
8. "I can't" requires proof of the exact tool tried and error returned. Start startable infrastructure yourself; never frame it as a blocker.
9. A failure is a probe into the system, not a thing to patch. Diagnose exhaustively from uncontaminated context; intervene minimally. The right fix is additive, subtractive, or nothing.
10. Don't pre-engineer guards; earn them from incidents. Don't rewrite what works; add to it. Don't remove load-bearing infrastructure without a replacement.
11. When a rule must be enforced, promote it down the ladder until the failure can't recur: memory → hook → lint (won't-merge) → type (won't-compile). Warn is invisible; block. Layer enforcement; demonstrate it, don't claim it.

---

## 10. The meta-principle

Everything here reduces to one stance: **understand exhaustively, intervene minimally, and steward the whole system — model, context, harness, artifacts, people, and your own corrections — toward staying true and minimal over time.**

Mastering AI is not accumulating tricks or building walls ever higher. It is becoming the kind of operator who understands the failure modes well enough to build the structures that make them visible and recoverable, who can tell what is load-bearing from what is noise, who reads behavior rather than words, and who prunes as readily as they plant. The AI supplies capability without judgment. The operator supplies judgment under uncertainty. The output you actually want lives at the meeting point of the two — and it is reached not by asking the AI for better behavior, but by building the system in which good behavior is the only behavior available.
