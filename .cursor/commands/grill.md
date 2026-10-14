---
name: "/grill"
id: "grill"
category: "Workflow"
description: "Passo 1 del flusso: analisi con grilling, glossario e ADR"
---

Read and follow the `grilling` and `domain-modeling` skills (`.cursor/skills/grilling/SKILL.md` and `.cursor/skills/domain-modeling/SKILL.md`), then start the analysis of the change described below.

Before the first question:

1. Check that the current branch is `spec/<change-name>`. If it is not, stop and tell the user to create it from an up-to-date `main`.
2. Read `GLOSSARY.md`, `docs/adr/` if it exists, and the specs in `openspec/specs/` related to the topic.

During the session, write only `GLOSSARY.md` and `docs/adr/`. Do not create OpenSpec artifacts and do not write code.

**Input**: The argument after `/grill` is the change description to analyze.
