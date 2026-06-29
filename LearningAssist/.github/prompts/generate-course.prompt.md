---
agent: agent
description: "One-shot course generation pipeline. Provide topic + audience + depth and the orchestrator generates the entire course end-to-end."
tools: [read, edit]
---

# Learning Pro Pipeline — Single Entry Point

Invoke the **Course Master Orchestrator** agent with the following inputs:

```
topic={{topic}};
audience={{audience}};
depth={{depth}};
total_classes={{total_classes}};
language_version={{language_version}};
focus_areas={{focus_areas}};
interview_companies={{interview_companies}}
```

## Quick Examples

### Python Interview Prep (full auto)
```
topic=Python; audience=interview-prepper; depth=deep
```

### SQL for Data Analysts
```
topic=SQL; audience=data-analyst; depth=standard; total_classes=20
```

### React for Career Switchers
```
topic=React; audience=career-switcher; depth=deep; total_classes=25; language_version=ES2024
```

### System Design (FAANG)
```
topic=System Design; audience=interview-prepper; depth=expert; total_classes=20; interview_companies=Google, Meta, Amazon
```

### C++ DSA Course
```
topic=C++; audience=interview-prepper; depth=deep; total_classes=30; language_version=C++20; focus_areas=DSA, STL, templates
```

## Pipeline Stages (auto-executed)
1. **Course Outline Generator** → raw course content
2. **Course Input Structurer** → tier-calibrated manifest
3. **Interactive Learning Designer Pro** → HTML pages (batched by tier)
4. **Page Auditor** → quality gate checks
5. **Section Patcher** → surgical fixes
6. **Master Index** → navigation page

## Resume a Partial Run
```
topic=Python; audience=interview-prepper; resume_from=stage2; manifest_path=generated/course-inputs/python-interview-prepper-manifest.md; start_class=15
```
