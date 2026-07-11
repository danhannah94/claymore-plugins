---
description: Drive an epic through the Kiro-style spec pipeline — grounding → requirements (red/blue-teamed, LOCKED) → design with mermaid architecture (red-teamed) → tasks doc with requirement traceability, ready for /hl:ship. Accepts an epic name to start fresh, or an existing spec-doc reference to resume mid-pipeline.
argument-hint: "<epic-name | doc-ref> [--stage <n>]"
---

Run the `spec` skill for: **$ARGUMENTS**

Nine stages, each gating the next: Ground (fresh repo exploration) → Draft
requirements (EARS criteria, must/should/could, 🤖/🧑 check owners) → Human
refinement (optional) → Red-team requirements (fresh-agent fan-out: grounding,
completeness, testability, cross-epic) → Blue-team requirements (optional;
exit = doc LOCKED) → Design doc(s) per module boundary with mermaid diagrams +
requirement traceability → Human refinement (optional) → Red-team design
(security, cost, plan-consistency, operability, simpler-alternative,
requirements-fidelity) → Tasks doc in /hl:ship Form-A format with the
must-coverage check as the exit gate.

If $ARGUMENTS references an existing requirements/design doc, detect the
current stage from the doc's state (draft / red-teamed / LOCKED / designed)
and resume from there. If no $ARGUMENTS, ask which epic to spec.

**Skipping a stage is the human's call — recommend, don't decide.** No
blue-team at the design stage: scope pressure found in design becomes a
visible requirements amendment, never a silent cut.
