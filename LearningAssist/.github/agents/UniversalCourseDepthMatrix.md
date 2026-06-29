---
name: Course Depth Matrix (Universal)
description: "Universal reference configuration for ALL Learning Pro pipeline agents. Defines complexity tiers that auto-scale content depth, duration, MCQ count, and example count based on topic difficulty. Topic-agnostic: works for Python, SQL, C++, React, NodeJS, System Design, or any technical subject."
tools: [read]
user-invocable: false
---

# WHY THIS FILE EXISTS

Every topic in any technical course has a different cognitive load.
A CSS box-model lesson and a distributed consensus algorithm lesson are not the same.
Treating both with identical templates wastes time on simple topics and under-serves hard ones.

This matrix defines FOUR tiers. Every pipeline agent reads this file and uses the
`class_tier` field to calibrate its output automatically.

---

# THE FOUR TIERS

## TIER_1 — RECALL & QUICKEN (Light touch)

**Who needs this:** Learners who have seen this before and need syntax/concept recall.
**Cognitive load:** Low — familiarity, pattern recognition
**Design goal:** Fast re-activation. No padding. End with one "aha" gotcha they forgot.

```yaml
tier: TIER_1
label: "Recall & Quicken"
duration_minutes: 45
mcqs: 10
code_examples: 4
mistake_pairs_per_topic: 2
interview_problems: 0
visualizers: 0
playground: static-fallback-ok
tab3_depth: idioms-only
practice_problems: 1
pre_assessment: yes        # 5-question skip gate — if 80%+ score, show summary only
spaced_recall_callouts: 1
checkpoint_project: no
```

**Tone:** Brisk. "You know this. Here's what's specific/new."

---

## TIER_2 — STANDARD (Build understanding)

**Who needs this:** Topics with moderate depth — familiar concept but specific behaviour matters.
**Cognitive load:** Medium — comprehension + application
**Design goal:** Solid understanding with one real-world anchor. No handwaving.

```yaml
tier: TIER_2
label: "Standard"
duration_minutes: 75
mcqs: 15
code_examples: 6
mistake_pairs_per_topic: 3
interview_problems: 2
visualizers: 1
playground: interactive-preferred
tab3_depth: patterns-with-stdlib
practice_problems: 2
pre_assessment: yes
spaced_recall_callouts: 2
checkpoint_project: no
```

**Tone:** Explanatory. "Here is WHY it works this way — not just that it does."

---

## TIER_3 — DEEP MASTERY (Full depth)

**Who needs this:** Topics where surface understanding leads to bugs or failed interviews.
**Cognitive load:** High — multi-concept, nuanced behaviour
**Design goal:** Genuine mastery. Learner can explain it to someone else after this class.

```yaml
tier: TIER_3
label: "Deep Mastery"
duration_minutes: 90
mcqs: 20
code_examples: 8
mistake_pairs_per_topic: 4
interview_problems: 5
visualizers: 2
playground: interactive-required
tab3_depth: full-patterns-with-design
practice_problems: 3
pre_assessment: yes
spaced_recall_callouts: 3
checkpoint_project: optional
interview_section: yes
```

**Tone:** Rigorous. Every "why" answered. No concept left at definition level.

---

## TIER_4 — EXPERT & INTERVIEW-CRITICAL (Maximum depth)

**Who needs this:** Topics in technical interviews, system design, or advanced production work.
**Cognitive load:** Very high — abstract reasoning, multiple sub-problems, edge cases
**Design goal:** Interview readiness + genuine understanding of tradeoffs.

```yaml
tier: TIER_4
label: "Expert & Interview-Critical"
duration_minutes: 120
mcqs: 25
code_examples: 10
mistake_pairs_per_topic: 5
interview_problems: 7
visualizers: 3
playground: interactive-required
tab3_depth: full-patterns-plus-system-design
practice_problems: 5
pre_assessment: no           # these topics need to be learned, not skipped
spaced_recall_callouts: 4
checkpoint_project: yes      # mandatory mini-project at end
interview_section: yes
dry_run_tables: yes          # step-by-step traces required
complexity_comparator: yes   # side-by-side brute vs optimal with timing
boss_challenge: yes          # one hard synthesis problem combining multiple concepts
```

**Tone:** Demanding but supportive. "This is the hard part. Here is every tool you need."

---

# TIER ASSIGNMENT HEURISTICS

When the Course Input Structurer assigns tiers, use these heuristics:

| Signal | Tier |
|--------|------|
| Topic is pure syntax/recall for the audience | TIER_1 |
| Topic has gotchas but is learnable in one session | TIER_2 |
| Topic appears in interviews OR has subtle production bugs | TIER_3 |
| Topic is interview-critical AND has multiple sub-problems | TIER_4 |
| Topic combines 3+ concepts that interact non-trivially | TIER_4 |
| Topic is frequently asked at FAANG companies | TIER_3 or TIER_4 |

---

# SPECIAL RULES FOR TIER BOUNDARIES

## When a topic spans two tiers

If a class covers both a simple sub-topic (TIER_1) and a complex one (TIER_3):
- Use the HIGHER tier for the overall class
- Add a "Fast Track" callout box for the simple sub-topics

## TIER_1 pre-assessment gate

For all TIER_1 classes, show a 5-question pre-assessment BEFORE content:
- If score ≥ 80%: show a 1-page summary + gotchas only
- If score < 80%: show full class content

## TIER_4 checkpoint projects

Every TIER_4 class must end with a checkpoint mini-project that:
- Uses the topic just learned
- Also requires at least one concept from a prior class (creates connections)
- Has a clear acceptance criterion (not "practice this")

---

# HOW AGENTS USE THIS FILE

| Agent | How It Uses This Matrix |
|-------|------------------------|
| Course Input Structurer | Assigns `class_tier` to each block; sets duration, MCQs, examples from tier config |
| Interactive Learning Designer Pro | Reads `class_tier` from CLASS_BLOCK; scales all output sections; enforces tier-specific requirements |
| Page Auditor | Reads `class_tier`; applies tier-appropriate quality gates (doesn't demand TIER_4 quality from TIER_1 pages) |
| Section Patcher | Reads `class_tier` from audit; applies tier-appropriate patch depth |
| Course Outline Generator | References tier heuristics when recommending tier for each class |
