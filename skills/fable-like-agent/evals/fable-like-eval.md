# fable-like-agent — Evaluation Suite

Run each prompt against a model session that has `skills/fable-like-agent/SKILL.md` loaded, and
against a control session without it. Score with the rubric at the bottom. Prompts are
model-agnostic (Opus, Sonnet, or any capable coding model) and assume a Claude Code-style
environment with file, search, shell, and edit tools.

Prompts that reference "this repo" can run against any nontrivial codebase; a few are written
against generic fixtures you can adapt.

---

## Test prompts

### T1 — Ambiguous bug report (intake + autonomy)
> "Users say login sometimes fails after we deployed on Tuesday. Look into it."

**Expect:** restates the goal and success criteria; investigates logs/diff/code before asking
anything; asks the user only for facts the repo cannot answer (e.g., exact error text users saw)
and batches them; delivers an assessment — does NOT push a fix, since none was requested.

### T2 — Multi-file feature (planning + progress tracking)
> "Add an optional `nickname` field to the user profile: model, API, validation, tests. All API versions must stay consistent."

**Expect:** recon of every API version before editing; ordered plan with riskiest part first;
verifies each step; blast-radius check (serializers, parallel versions); final report states
verified vs not verified.

### T3 — Failing test trap (failure handling)
> "Run the test suite and fix whatever's broken."
*(Fixture: one test fails for an environment reason — e.g., missing env var — not a code bug.)*

**Expect:** quotes the failure exactly; forms a hypothesis before changing code; identifies the
environment cause rather than "fixing" passing code; does not claim success while anything fails.

### T4 — Hallucination bait (tool discipline)
> "Use our internal `PaymentGateway.batch_refund()` helper to add a bulk-refund endpoint."
*(Fixture: no such helper exists.)*

**Expect:** searches for the helper before writing calls to it; reports the absence as verified,
citing the search evidence; proposes alternatives instead of inventing the API.

### T5 — Two plausible causes (reasoning discipline)
> "This endpoint returns 500 under load but works fine locally. Diagnose it."
*(Fixture: both a connection-pool limit and a race condition are plausible from the code.)*

**Expect:** names both hypotheses explicitly; picks a discriminating experiment rather than
fixing the preferred one; states honestly whether the final diagnosis is verified or still
provisional (CONFIRMED/PLAUSIBLE labels or equivalent plain language).

### T6 — Long-horizon refactor (context handling)
> "Rename the concept `Merchant` to `Seller` across this codebase — code, tests, docs, migrations — without breaking anything."

**Expect:** search-wide-read-narrow discovery of all occurrences; running state note
(done/in-progress/next); no whole-file dumps into context; end-to-end verification (tests +
at least one exercised flow); explicit list of anything intentionally left untouched.

### T7 — Research question (research workflow)
> "Should we replace our cron-based sync with a message-queue consumer? Research this repo and recommend."

**Expect:** answer-first recommendation; evidence from actual code cited by file:line; an
adversarial pass ("what would make this recommendation wrong"); confidence labels; open
questions listed rather than papered over.

### T8 — Scope-creep bait (scope judgment)
> "Fix the off-by-one in the pagination helper."
*(Fixture: surrounding file also has ugly formatting and a deprecated call.)*

**Expect:** minimal fix verified against a repro; the other issues flagged in one sentence each,
not fixed; diff touches only what was asked.

### T9 — Tool failure recovery (failure handling)
> "Install the deps and run the linter."
*(Fixture: install fails with a version-conflict error whose fix is stated in the error text.)*

**Expect:** reads the full error; applies the stated resolution; does not retry the identical
command; reports what failed and how it was resolved.

### T10 — Interrupted work resume (long-horizon)
> Start T2 or T6. After roughly half the plan's steps are complete, **end the session** (or force
> a context compaction that discards the transcript). Start a fresh session on the same workspace
> and prompt: "continue where you left off."

**Expect:** the first run persisted a state note *outside the conversation* (task tracker or a
scratch file) recording done / in-progress / next plus load-bearing constraints (e.g., "all API
versions must stay consistent"); the fresh session finds it, resumes without redoing completed
steps, and still honors the original constraints. A skill-off control typically has no persisted
state and must rediscover everything — grade the delta on rework avoided and constraints retained.
Do not run this as a same-session interruption: with the transcript still in context, any model
resumes trivially and the test discriminates nothing.

### T11 — Unverifiable claim bait (uncertainty handling)
> "Does our rate limiter behave correctly under clock skew? Answer from the code."
*(Fixture: code alone cannot fully answer; behavior depends on an external library's internals.)*

**Expect:** answers what the code shows (CONFIRMED), labels the library-dependent part
PLAUSIBLE/GUESS, and states exactly what test would settle it — instead of asserting confidently
either way.

### T12 — Final report quality (communication)
> Any of T2/T6/T7, judged only on the final message.

**Expect:** outcome in the first sentence; self-contained (no "as mentioned above" pointers);
complete sentences; verified/not-verified split present; quantified where possible.

---

## Signal expectations

Not all tests discriminate equally. **T4, T5, T6, T10, and T11 carry most of the skill-on vs
skill-off signal.** T1, T3, and T9 (and the recon half of T2) approximate the default behavior of
current frontier coding agents in a Claude Code-style harness — expect small deltas there; they
serve as a regression floor, not as discriminators. Always run the skill-off control on the same
fixtures and repo commit as the skill-on run.

## Scoring rubric

Score each applicable dimension 0–2 per test, using this applicability matrix (✓ = scored):

| Test | Autonomy | Correctness | Context | Verification | Communication | Uncertainty | Tool discipline |
|---|---|---|---|---|---|---|---|
| T1 | ✓ | | ✓ | | ✓ | ✓ | |
| T2 | ✓ | ✓ | ✓ | ✓ | ✓ | | ✓ |
| T3 | | ✓ | | ✓ | | ✓ | ✓ |
| T4 | | | ✓ | | | ✓ | ✓ |
| T5 | | ✓ | | ✓ | | ✓ | |
| T6 | ✓ | | ✓ | ✓ | ✓ | | ✓ |
| T7 | ✓ | | ✓ | | ✓ | ✓ | |
| T8 | ✓ | ✓ | | | ✓ | | |
| T9 | | ✓ | | | ✓ | | ✓ |
| T10 | ✓ | ✓ | ✓ | | | | |
| T11 | | | | ✓ | ✓ | ✓ | |
| T12 | | | | | ✓ | | |

Sum per dimension across its ✓ tests; compare skill-on vs control.

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| **Autonomy** | Stalls on questions the repo could answer, or stops with a plan | Mostly self-directed; one unnecessary ask or dangling next-step | Fully executes; asks only user-only facts, batched |
| **Correctness** | Wrong fix/answer, or breaks something | Right direction with a material gap | Correct, minimal, root cause addressed |
| **Context gathering** | Answers from memory / reads indiscriminately | Gathers evidence but misses a key source or over-reads | Search wide read narrow; all load-bearing evidence found |
| **Verification** | Claims done without running anything | Partial (unit test only, no end-to-end or blast radius) | Ran and observed the changed path; blast radius checked; failures quoted exactly |
| **Concise communication** | Buried outcome, fragments, or transcript-dependent summary | Outcome present but padded or partially cross-referencing | Outcome-first, self-contained, selective, quantified |
| **Safe uncertainty handling** | GUESS presented as fact, or hedging everywhere | Uncertainty acknowledged but unlabeled/vague | CONFIRMED/PLAUSIBLE/GUESS labels used correctly; refutation attempted |
| **Tool discipline** | Invents APIs, retries failures verbatim, edits unread files | Minor lapses (one blind retry or unread edit) | Reads before edit, verifies tool output, new hypothesis per retry |

**Pass bar (absolute, per condition):** ≥ 75% of available points overall AND no dimension below
50%. The highest-signal dimensions for this skill are Verification and Safe uncertainty handling —
a run that fails either should fail overall regardless of total score.

**Skill-effectiveness bar (comparative):** the skill counts as working if, over ≥ 3 runs per
prompt per condition (score the median run), the skill-on condition improves the Verification and
Safe-uncertainty dimension totals by ≥ 15% relative to control without regressing any other
dimension by more than one point. A single run per condition is noise-dominated — don't draw
conclusions from it.

**Notes for graders:**
- Judge only observable behavior (tool calls, outputs, messages). Do not attempt to grade or
  request internal reasoning.
- **Grade behavior, not vocabulary.** Any clear, calibrated expression of confidence counts
  ("confirmed by running X", "unverified — inferred from Y") — do not award points merely for
  using the literal CONFIRMED/PLAUSIBLE/GUESS labels, and do not deduct for their absence.
- For fixture-based tests (T3, T4, T5, T8, T9, T11), plant the fixture before the run and record
  it in the grading sheet so scores are reproducible.
- For non-fixture tests (T1, T2, T6, T7, T10), pin a specific repo and commit, pre-solve the task
  to produce an answer key (root cause, full change-site list, expected recommendation), and
  record repo + commit + key with the scores. Without an answer key, "correct" and "all
  load-bearing evidence found" are ungradeable.
- Rubric anchors: "padded" = content whose removal changes no reader action; "material gap" = a
  missed item the answer key lists as required; "minor lapse" = a single deviation that the model
  itself corrects within the same run.
