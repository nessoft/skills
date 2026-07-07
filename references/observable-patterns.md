# Observable Patterns — Source Analysis

This document records where the skill's guidance comes from. It separates patterns backed by
user-provided material from patterns inferred from general agent best practice. Nothing here is
derived from hidden reasoning traces, proprietary system prompts, or model internals — only
observable, user-supplied documentation of working style.

## Evidence base

| Source | Type | Status |
|---|---|---|
| `references/FABLE5-METHOD.md` (author-maintained methodology distillation, bundled) | Written description of observed Fable-5-style working discipline | **Primary evidence** |
| Repository search for transcripts, traces, eval logs, agent-run records | `fable*`, `traces/`, `transcripts/`, `evals/`, `samples/`, `conversations/` | **Nothing found** — no raw transcripts were available in this workspace |

Because no raw transcripts were provided, "evidence-backed" below means "explicitly stated in
FABLE5-METHOD.md". Patterns not stated there are marked **inferred** and should be treated as
best-practice assumptions, not observations.

The source document is bundled at `references/FABLE5-METHOD.md`, so every section citation
(§1–§10) below is independently verifiable. SKILL.md remains self-contained and never requires
the source file at runtime.

## Evidence-backed patterns (from FABLE5-METHOD.md)

### 1. Task intake
- Restate the task as the underlying need; ask "what would the user do with my output?" (§1).
- Missing information: gather it yourself first; ask the user only what only the user knows (§7).
- When the user describes a problem without requesting a fix, deliver an assessment and stop (§6).
- Decompose into individually verifiable sub-questions, each with a concrete "done" test (§4).

### 2. Planning style
- Riskiest / most uncertain part first (§4).
- Stop decomposing when a sub-task is one obvious action (§4).
- Fan out parallelizable investigation; never fake parallelism between dependent steps (§4).

### 3. Context handling
- Search wide, read narrow: grep/glob to locate, read only relevant spans (§3).
- Don't read whole large files; it "burns context and buries the signal" (§3).

### 4. Tool usage
- Read before write; match idiom, naming, comment density of the file being edited (§3).
- Never invent API surface — look up unseen signatures before calling (§2).
- Test hypotheses with the cheapest decisive experiment (§1).

### 5. Coding methodology
- Trace, don't skim: findings are exact file:line divergence points, not "somewhere in auth" (§3).
- Minimize the reproduction before theorizing (§3).
- Follow data, not names — distrust identifiers, trust call graphs and values (§3).
- Verify the diagnosis against this instance, not the general pattern it resembles (§2, §10).
- Check blast radius after a fix: callers, serializers, parallel API versions, side effects (§5).

### 6. Reasoning methodology (visible steps only)
- Explicit hypothesis format: "I believe X because Y; if right, Z should be observable" (§1).
- Adversarial self-check before asserting: one explicit attempt to refute your own finding (§2).
- When two explanations fit, say so, then run the discriminating experiment (§2).

### 7. Self-verification
- "Done" = ran it, saw the output, output matched. Typecheck passing ≠ feature working (§5).
- Exercise the changed path end-to-end (§5).
- Quote test failures exactly; never soften or claim partial success as success (§5).
- End-of-turn checklist: no dangling plans/promises, no unverified claims, answer in final message (§9).

### 8. Collaboration style
- Lead with the outcome; reasoning second (§8).
- Complete sentences; compress by selectivity, not fragments (§8).
- Quantify ("3 of 14 callers") and never force the reader to cross-reference (§8).
- Fix what was asked; flag what you noticed in one sentence; don't re-litigate settled decisions (§6).
- Confirm irreversible/outward-facing actions; proceed on reversible ones (§6).

### 9. Failure handling
- Read the entire error message; the answer is usually in it (§7).
- One failure → new hypothesis before retry; never retry verbatim (§7).
- Three failures on one approach → question the framing itself (§7).
- Confidence labels CONFIRMED / PLAUSIBLE / GUESS; most errors are GUESS presented as CONFIRMED (§2).

## Inferred patterns (assumptions — not in the source material)

Marked clearly per the honesty constraint. These come from general published agent-design practice,
not from any Fable-specific evidence:

- **Running state note for long tasks** (done / in-progress / blocked / next) to survive
  interruption or context compaction. *Inferred from long-horizon agent best practice.*
- **Batching clarifying questions** instead of dripping them one at a time. *Inferred.*
- **Delegating broad searches to subagents** to keep the main context lean. *Inferred from
  Claude Code's documented subagent facilities; FABLE5-METHOD.md only says "fan out".*
- **Double-checking empty search results from a second angle** before concluding absence.
  *Inferred generalization of the source's "verify tool results" spirit.*
- **The final response template** (Outcome / Verified / Not verified / Blast radius / Open items).
  *Format is inferred; every field maps to an evidence-backed requirement.*
- **Progress-update cadence** (one sentence before first tool call; updates only on direction
  changes). *Inferred from Claude Code's public harness conventions.*
- **Regression-test mandate** (add a test that fails before the fix and passes after; run the
  full relevant suite). *Inferred extension of §5's end-to-end verification — the source requires
  exercising the changed path but never mandates a failing-then-passing test.*
- **Explicit intake extraction of success criteria and constraints.** *Extension of §1, which
  covers only the underlying need and deliverable shape.*
- **Primary-before-secondary source ordering** and **separating observation from
  interpretation** in research. *Plausible derivations of §2's confidence discipline; not stated
  in the source.*
- **"Don't re-derive facts already established this session."** *Companion to §6's
  "don't re-litigate settled decisions"; the facts half is not in the source.*
- **Preferring dedicated file/search tools over shell equivalents.** *Claude Code harness
  convention, not in the source.*
- **Persisting the long-task state note outside the conversation** (task tracker or scratch
  file). *Inferred — required for the note to survive context compaction; the source has no
  state-note concept at all.*

## What could not be inferred

Stated per the honesty constraint:

- Anything about internal reasoning content, thinking-token allocation, or private
  chain-of-thought — out of scope by design and unobservable.
- Quantitative behavior (how often Fable-style sessions verify, average plan depth, tool-call
  budgets) — no transcripts were available to measure.
- Model-capability differences (single-pass insight depth, recall, reasoning-chain length) —
  the source itself states these cannot be transferred by prompt; this skill raises the floor,
  not the ceiling.
