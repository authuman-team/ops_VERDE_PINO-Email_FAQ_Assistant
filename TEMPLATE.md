# Authuman Client Solution Template

## How to use this template
1. Create a new repo from this template
2. Rename the repository to `client-<name>-<solution>`
3. Replace placeholder values marked with <...>
4. Import the workflow JSON into n8n
5. Update `workflowVersion` and environment

---

## Mandatory initial decisions (must be filled)

- Client name:
- Solution name:
- Environment: pilot | production
- System of record:
- Re-processing allowed: yes | no
- Data contains personal data: yes | no
- Recall vs cost priority:
- OCR required: yes | no

---

## Files you must update
- workflows/solution.vX.X.json
- docs/README.md
- docs/decision-log.md
- docs/changelog.md
- config/env.example.md

---

## Rules
- Prompts live in `docs/prompts.md`
- Runtime logic lives in n8n
- Notion is descriptive only
- Any behavior change = changelog entry
