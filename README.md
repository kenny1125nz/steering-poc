# Steering PoC

Research and design material for **steerable agent orchestration** based on recursive goal decomposition with upward legitimacy feedback.

The central proposal: rather than pre-planning an entire goal or relying only on local feedback, an agent decomposes goals recursively, treats each sub-goal as an atomic unit, and allows a sub-goal to challenge its own legitimacy when execution evidence contradicts it — propagating that signal upward for recalibration.

## Contents

- `concept.md` — concept paper and motivation
- `engineering-design.md` — proposed architecture
- `poc-stage1-design.md` — first proof-of-concept design
- `harness-model.md` — evaluation harness model
- `reading.md` — related work
- `review.md` — review notes

## Status

Research / proof-of-concept. This repository contains documents, not a runnable application.
