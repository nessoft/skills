[![skills.sh](https://skills.sh/b/anthropics/skills)](https://skills.sh/nessoft/fable-like-agent)

# fable-like-agent

A portable Claude Code skill that guides capable models (Opus, Sonnet, or others) to work in a
**Fable-like observable operating style**: long-horizon, autonomous, evidence-driven,
self-verifying, and disciplined about scope, context, and communication.

## What this is — and is not

**It is** a distillation of *observable workflow behavior*: planning loops, verification
checklists, tool-use rules, uncertainty labeling, and reporting conventions, sourced from
user-provided methodology notes. The source document is bundled at
`references/FABLE5-METHOD.md`; see `references/observable-patterns.md` for the
evidence-vs-assumption breakdown.

**It is not:**
- A capability upgrade. It cannot give a model deeper insight, longer reasoning chains, or
  better recall. It raises the floor (fewer process failures), not the ceiling.
- A clone of any model. It emulates workflow style only.
- Derived from hidden reasoning, private chain-of-thought, proprietary system prompts, or model
  internals. Only visible behavior descriptions were used, and the skill instructs models to
  expose *steps and evidence*, never internal reasoning traces.

## Install

**Per-user (all projects):**

```bash
git clone https://github.com/nessoft/fable-like-agent.git ~/.claude/skills/fable-like-agent
```

**Per-project (recommended for teams):** Claude Code discovers project skills in `.claude/skills/`:

```bash
mkdir -p .claude/skills
git clone https://github.com/nessoft/fable-like-agent.git .claude/skills/fable-like-agent
```

To vendor it into a repo without git history, download and extract instead:

```bash
curl -L https://github.com/nessoft/fable-like-agent/archive/refs/heads/main.tar.gz | tar xz
mv fable-like-agent-main .claude/skills/fable-like-agent
```

Restart the Claude Code session (or start a new one) so the skill list refreshes.

## Use

- Explicit: type `/fable-like-agent` or say "use the fable-like-agent skill".
- By description: the skill triggers on requests for thorough/autonomous/long-horizon work
  ("fable-like", "careful mode", "work this end-to-end and verify").
- With other models: the SKILL.md is model-agnostic prose. For non-Claude-Code harnesses, paste
  `SKILL.md` (minus frontmatter) into the system prompt or a pinned instruction.

Works best on: multi-step coding tasks, debugging with unclear root cause, refactors that span
many files, research questions that need cited evidence and honest confidence labels.

Low value on: one-line answers, trivial edits, pure conversation — the overhead isn't justified
there, and the skill itself says to stop decomposing when a task is one obvious action.

## Layout

```
fable-like-agent/
├── SKILL.md                          # the skill — operating philosophy, workflows, checklists
├── README.md                         # this file
├── references/
│   ├── observable-patterns.md        # evidence-backed vs inferred patterns, and what's unknowable
│   └── FABLE5-METHOD.md              # bundled source methodology document (the evidence base)
└── evals/
    └── fable-like-eval.md            # 12 test prompts + 7-dimension scoring rubric
```

## Testing

Run the prompts in `evals/fable-like-eval.md` with the skill loaded and unloaded; score both runs
on the rubric (autonomy, correctness, context gathering, verification, concise communication,
safe uncertainty handling, tool discipline). The pass criteria — an absolute per-condition bar and
a comparative skill-effectiveness bar (≥ 3 runs per prompt, median-scored, ≥ 15% relative
improvement on **verification** and **safe uncertainty handling** with no other dimension
regressing) — are defined in the eval file; that file is authoritative.

## Honest limitations

- Built from a written methodology document, not raw transcripts — no session logs were available
  when it was authored. Patterns marked *inferred* in `references/observable-patterns.md` are
  best-practice assumptions, clearly separated from evidence-backed ones.
- Smaller models may follow the checklists yet still miss deep bugs; process discipline reduces
  unforced errors, it does not create insight.
- The skill adds turn overhead (recon, verification, blast-radius checks). That's the point on
  complex tasks and a waste on trivial ones.
