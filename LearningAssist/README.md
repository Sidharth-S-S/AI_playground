# Learning Pro Pipeline — Interactive Course Generator

A fully automated, topic-agnostic course generation system powered by GitHub Copilot agents. Generate complete interactive HTML training courses from a single prompt — no manual content creation required.

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Pipeline Architecture](#pipeline-architecture)
- [User Input Reference](#user-input-reference)
- [Tier System](#tier-system)
- [Agent Reference](#agent-reference)
- [Output Structure](#output-structure)
- [Usage Examples](#usage-examples)
- [Resume & Skip Options](#resume--skip-options)
- [Topic-Specific Agents](#topic-specific-agents)
- [Design Decisions](#design-decisions)
- [Limitations & Tips](#limitations--tips)

---

## Overview

The Learning Pro Pipeline chains 6 stages to turn a minimal user prompt (e.g., `topic=Python; audience=interview-prepper`) into a complete set of interactive HTML training pages with:

- In-browser code playgrounds (Skulpt, sql.js, Monaco, etc.)
- FAANG-level MCQ assessments with code-trace and scenario questions
- Tabbed interface (Concepts / Code Lab / Patterns / Mistakes / MCQ)
- Tier-scaled content depth (45-minute quick recall to 120-minute expert deep-dives)
- Automated quality audits and surgical patching

**Supported Topics:** Python, SQL, C++, JavaScript/React, NodeJS, System Design — or any technical subject.

---

## Quick Start

### Minimum viable input (2 fields)

```
topic=Python; audience=interview-prepper
```

### Using the prompt file

Open `.github/prompts/generate-course.prompt.md` in VS Code and invoke via Copilot Chat.

### Invoking the orchestrator directly

Ask Copilot to run the **Course Master Orchestrator** agent:

```
@workspace /agent Course Master Orchestrator
topic=Python; audience=interview-prepper; depth=deep
```

---

## Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  USER INPUT: topic + audience (+ optional fields)               │
└──────────────────────────────┬──────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 0: Course Outline Generator                               │
│  → Auto-generates full curriculum from minimal input             │
│  → Output: generated/course-inputs/<slug>-course-outline.md      │
└──────────────────────────────┬───────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 1: Course Input Structurer                                │
│  → Converts outline to tier-calibrated CLASS_BLOCK manifest      │
│  → Output: generated/course-inputs/<slug>-manifest.md            │
└──────────────────────────────┬───────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 2: Interactive Learning Designer Pro (per class)           │
│  → Generates standalone HTML page with playground + MCQs         │
│  → Output: generated/<topic>-interactive-pages/<class>.html      │
│  → Batched by tier priority (TIER_4 → TIER_1)                   │
└──────────────────────────────┬───────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 3: Page Auditor (per class)                               │
│  → Checks quality gates; outputs patch list if failures found    │
│  → Output: generated/audits/<class>-audit.md                     │
└──────────────────────────────┬───────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 4: Section Patcher (only if audit has FAILs)              │
│  → Surgical HTML fixes; preserves all passing content            │
│  → Output: Patched HTML (in-place)                               │
└──────────────────────────────┬───────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│  STAGE 5: Master Index Generator                                 │
│  → Links all pages with section grouping and navigation          │
│  → Output: generated/<topic>-interactive-pages/index.html        │
└──────────────────────────────────────────────────────────────────┘
```

---

## User Input Reference

### Required Fields (minimum 2)

| Field | Description | Examples |
|-------|-------------|----------|
| `topic` | The technical subject to teach | `Python`, `SQL`, `C++`, `React`, `NodeJS`, `System Design`, `Rust` |
| `audience` | Target learner profile | `interview-prepper`, `beginner`, `returner`, `advancer`, `career-switcher`, `data-analyst` |

### Optional Fields

| Field | Default | Description | Examples |
|-------|---------|-------------|----------|
| `depth` | `deep` | Content depth level | `shallow`, `standard`, `deep`, `expert` |
| `total_classes` | Auto-calculated | Number of classes in the course | `20`, `35`, `40` |
| `language_version` | Latest stable | Language/framework version | `Python 3.11`, `C++20`, `ES2024` |
| `focus_areas` | Full coverage | Comma-separated focus topics | `DSA, OOP, functional`, `web-dev, databases` |
| `exclude_topics` | None | Topics to skip | `GUI, pygame`, `legacy-patterns` |
| `interview_companies` | Generic | Shape problem selection for these companies | `Google, Meta, Amazon` |
| `time_per_class` | `90` | Target minutes per class | `60`, `90`, `120` |
| `output_folder` | Auto-generated | Override output directory | `generated/my-custom-folder` |

### Control Fields (pipeline behavior)

| Field | Default | Description | Examples |
|-------|---------|-------------|----------|
| `skip_stages` | None | Skip specific stages | `audit,patch` |
| `resume_from` | `stage0` | Resume from a specific stage | `stage2`, `stage3` |
| `start_class` | `1` | Start generating from this class | `15` |
| `end_class` | Last class | Stop generating at this class | `25` |
| `manifest_path` | Auto | Path to existing manifest (for resume) | `generated/course-inputs/python-manifest.md` |

---

## Tier System

The **Universal Course Depth Matrix** defines four complexity tiers that automatically scale content:

| Tier | Label | Duration | MCQs | Code Examples | Interview Problems | Checkpoint Project |
|------|-------|----------|------|---------------|-------------------|-------------------|
| **TIER_1** | Recall & Quicken | 45 min | 10 | 4 | 0 | No |
| **TIER_2** | Standard | 75 min | 15 | 6 | 2 | No |
| **TIER_3** | Deep Mastery | 90 min | 20 | 8 | 5 | Optional |
| **TIER_4** | Expert & Interview-Critical | 120 min | 25 | 10 | 7 | **Mandatory** |

### Tier Assignment Heuristics

| Signal | Assigned Tier |
|--------|---------------|
| Pure syntax/recall for the audience | TIER_1 |
| Has gotchas but learnable in one session | TIER_2 |
| Appears in interviews OR has subtle production bugs | TIER_3 |
| Interview-critical AND has multiple sub-problems | TIER_4 |
| Combines 3+ concepts that interact non-trivially | TIER_4 |
| Frequently asked at FAANG companies | TIER_3 or TIER_4 |

### TIER_4 Special Features

- Mandatory four-pass generation (no shortcuts)
- Dry-run tables (step-by-step code traces)
- Complexity comparator (brute-force vs optimal with timing)
- Boss challenge (hard synthesis problem)
- Checkpoint project (mandatory mini-project)

---

## Agent Reference

### Universal Agents (any topic)

| Agent | Stage | Purpose | File |
|-------|-------|---------|------|
| Course Outline Generator | 0 | Auto-generates complete curriculum | `CourseOutlineGenerator.md` |
| Course Master Orchestrator | All | Chains all stages end-to-end | `CourseMasterOrchestrator.md` |
| Course Input Structurer | 1 | Converts outline to tier-calibrated manifest | `CourseInputStructurer.md` |
| Interactive Learning Designer Pro | 2 | Generates HTML training pages | `InteractiveLearningDesignerPro.md` |
| Page Auditor | 3 | Quality gates and patch list generation | `PageAuditor.md` |
| Section Patcher | 4 | Surgical fixes for failed quality gates | `SectionPatcher.md` |
| Universal Course Depth Matrix | Config | Tier definitions and scaling rules | `UniversalCourseDepthMatrix.md` |

### Topic-Specific Agents (Python)

| Agent | Purpose | File |
|-------|---------|------|
| Python Course Input Structurer | Python-specific structurer with Classes 1-35 mapping | `PythonCourseInputStructurer.md` |
| Python Interactive Learning Designer Pro | Python-specific page generation with Skulpt | `PythonInteractiveLearningDesignerPro.md` |
| Python Page Auditor | Python-specific quality gates | `PythonPageAuditor.md` |
| Python Section Patcher | Python-specific surgical fixes | `PythonSectionPatcher.md` |
| Course Depth Matrix | Python-specific class→tier mapping | `CourseDepthMatrix.md` |

### Topic-Specific Agents (C++)

| Agent | Purpose | File |
|-------|---------|------|
| C++ Course Input Structurer | C++ DSA-focused structurer | `CppCourseInputStructurer.md` |
| C++ Interactive Learning Designer Pro | C++ page generation with Compiler Explorer | `CppInteractiveLearningDesignerPro.md` |

### Agent Selection Logic

The orchestrator auto-selects agents based on the `topic` field:

| Topic | Structurer | Designer | Auditor | Patcher |
|-------|-----------|----------|---------|---------|
| Python | Python Course Input Structurer | Python Interactive Learning Designer Pro | Python Page Auditor | Python Section Patcher |
| C++ | C++ Course Input Structurer | C++ Interactive Learning Designer Pro | Page Auditor | Section Patcher |
| Any Other | Course Input Structurer | Interactive Learning Designer Pro | Page Auditor | Section Patcher |

---

## Output Structure

```
generated/
├── course-inputs/
│   ├── <topic-slug>-<audience>-course-outline.md    # Stage 0
│   └── <topic-slug>-<audience>-manifest.md          # Stage 1
├── <topic-slug>-interactive-pages/
│   ├── index.html                                    # Stage 5
│   ├── <topic>-class-1-<module-slug>.html           # Stage 2
│   ├── <topic>-class-2-<module-slug>.html
│   └── ...
└── audits/
    ├── <topic>-class-1-audit.md                      # Stage 3
    └── ...
```

### HTML Page Structure

Each generated page is a standalone HTML file with:

- **5 tabbed sections:** Concepts | Code Lab | Patterns & Idioms | Common Mistakes | MCQ Assessment
- **Hero section:** Module name, class number, duration, MCQ count, learning outcomes
- **Interactive playground:** Topic-appropriate engine (Skulpt, sql.js, Monaco, etc.)
- **MCQ system:** Submit/score/reset with explanations and progress tracking
- **Responsive design:** Mobile-friendly with proper media queries
- **No external dependencies** (except CDN for playground engines)

---

## Usage Examples

### Example 1: Full Python Interview Prep Course

```
topic=Python; audience=interview-prepper; depth=deep
```

Generates ~30-35 classes covering Python fundamentals through advanced DSA, with FAANG-level interview problems at every class.

### Example 2: SQL for Data Analysts (shorter course)

```
topic=SQL; audience=data-analyst; depth=standard; total_classes=20
```

Focused 20-class course on SQL queries, joins, window functions, and optimization.

### Example 3: React for Career Switchers

```
topic=React; audience=career-switcher; depth=deep; total_classes=25; language_version=ES2024
```

Complete React course assuming general programming knowledge but no frontend experience.

### Example 4: System Design (FAANG Interview Focus)

```
topic=System Design; audience=interview-prepper; depth=expert; total_classes=20; interview_companies=Google, Meta, Amazon
```

Expert-level system design course shaped by real FAANG interview patterns.

### Example 5: C++ DSA Course

```
topic=C++; audience=interview-prepper; depth=deep; total_classes=30; language_version=C++20; focus_areas=DSA, STL, templates
```

C++ course with DSA focus using modern C++20 features.

### Example 6: Quick Recall Course (narrow scope)

```
topic=Python; audience=returner; depth=shallow; total_classes=10; focus_areas=async, generators, decorators
```

Short refresher for experienced developers returning to Python.

---

## Resume & Skip Options

### Resume a partial run

If generation was interrupted at Stage 2, Class 15:

```
topic=Python; audience=interview-prepper; resume_from=stage2; manifest_path=generated/course-inputs/python-interview-prepper-manifest.md; start_class=15
```

### Generate only a subset of classes

```
topic=Python; audience=interview-prepper; resume_from=stage2; start_class=20; end_class=30
```

### Skip quality pass (faster, no audit/patch)

```
topic=SQL; audience=beginner; depth=standard; skip_stages=audit,patch
```

### Re-audit and patch existing pages

```
topic=Python; audience=interview-prepper; resume_from=stage3; manifest_path=generated/course-inputs/python-manifest.md
```

---

## Design Decisions

### Why topic-agnostic?

The original pipeline was Python-only. Upgrading to topic-agnostic eliminates duplicating agents for every new subject. The universal agents detect the topic from the CLASS_BLOCK and adapt playground engines, color themes, and code styles automatically.

### Why tier-based scaling?

Not all topics deserve equal depth. A CSS basics lesson and a distributed systems lesson have wildly different cognitive loads. The 4-tier system ensures simple topics get lean 45-minute pages while interview-critical topics get full 120-minute deep-dives with boss challenges and dry-run tables.

### Why batch by tier priority (TIER_4 first)?

If generation is interrupted, the most valuable (hardest, interview-critical) pages already exist. TIER_1 pages are trivial to regenerate.

### Why audit + patch instead of regenerate?

Full regeneration wastes tokens on sections that were already correct. The auditor identifies exactly what failed, and the patcher fixes only those sections — preserving 80-95% of correct content.

### Why standalone HTML (no build step)?

Each page is a single `.html` file that works by opening it in any browser. No npm, no bundler, no server required. CDN-loaded playground engines (Skulpt, sql.js) handle interactivity.

### MCQ Quality Approach

Questions follow a strict distribution:
- **Recall (≤30%):** Basic knowledge checks
- **Code-reading (≥35%):** "What does this output?" with real code traces
- **Application (≥20%):** "Which approach solves this?"
- **Edge-case (≥15%):** "What happens when..." (boundary conditions)

This matches FAANG interview patterns and avoids the "which statement is correct" trap.

---

## Limitations & Tips

### Token/Context Limits

- TIER_4 pages are large (~3000-4000 lines). The Designer Pro uses a mandatory 4-pass generation for these.
- Courses with 30+ classes should always be generated in batches of 5-6.
- The orchestrator handles batching automatically — but if you invoke agents manually, respect batch sizes.

### Best Practices

1. **Start with the orchestrator** — don't invoke individual agents unless debugging
2. **Let tier auto-assignment work** — the matrix heuristics are calibrated for interview prep
3. **Use resume for iteration** — if you want to regenerate specific classes, use `start_class`/`end_class`
4. **Audit before publishing** — even manually-invoked Designer Pro pages benefit from an auditor pass
5. **Topic-specific agents are more polished** — for Python and C++, the dedicated agents have deeper templates

### Known Constraints

- Playground engines require internet access (CDN-loaded)
- Very niche topics (e.g., "COBOL", "VHDL") may produce less interview-relevant content
- The outline generator works best when `audience` is descriptive — "interview-prepper" produces better structure than "student"

---

## File Manifest

```
.github/
├── agents/
│   ├── CourseOutlineGenerator.md          # Stage 0 — curriculum generation
│   ├── CourseMasterOrchestrator.md        # Master — chains all stages
│   ├── CourseInputStructurer.md           # Stage 1 — universal structurer
│   ├── InteractiveLearningDesignerPro.md  # Stage 2 — universal page designer
│   ├── PageAuditor.md                     # Stage 3 — universal quality auditor
│   ├── SectionPatcher.md                  # Stage 4 — universal surgical patcher
│   ├── UniversalCourseDepthMatrix.md      # Config  — tier definitions
│   ├── CourseDepthMatrix.md               # Config  — Python-specific tier map
│   ├── PythonCourseInputStructurer.md     # Python-specific structurer
│   ├── PythonInteractiveLearningDesignerPro.md  # Python-specific designer
│   ├── PythonPageAuditor.md               # Python-specific auditor
│   ├── PythonSectionPatcher.md            # Python-specific patcher
│   ├── CppCourseInputStructurer.md        # C++-specific structurer
│   ├── CppInteractiveLearningDesignerPro.md     # C++-specific designer
│   ├── interactive-learning-designer.agent.md   # Legacy designer
│   ├── design_cpp_course_curriculum.md    # C++ curriculum designer
│   └── complete-cpp-course-foundations-to-expert-delivery-agent.md  # C++ delivery
├── prompts/
│   ├── generate-course.prompt.md          # Single entry-point prompt
│   ├── generate-cpp-dsa-class.prompt.md   # C++ DSA class prompt
│   ├── generate-output.prompt.md          # Generic output prompt
│   └── ...
└── copilot-instructions.md                # Workspace-level instructions
```

---

## Version History

| Date | Change |
|------|--------|
| 2026-06-29 | Full universal pipeline created: OutlineGenerator, Orchestrator, universal Structurer, Auditor, Patcher, DepthMatrix |
| 2026-06-28 | All 35 Python interactive pages generated with FAANG interview quality |
| 2026-06-27 | InteractiveLearningDesignerPro upgraded to topic-agnostic |
| Earlier | Python-specific and C++-specific agents created |

---

## License

Internal training content. See repository root for licensing terms.
