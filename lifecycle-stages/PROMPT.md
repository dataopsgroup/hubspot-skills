# How to run this skill

1. Open [Claude](https://claude.ai)
2. Attach the `SKILL.md` file to your conversation
3. Export from HubSpot:
   - **Contacts**: CRM → Contacts → Export (include lifecycle stage, create date, owner, lead source)
   - **Deals**: CRM → Deals → Export (include deal stage, create date, close date, amount, associated contact)
4. Attach both CSVs to the same conversation
5. Send this prompt:

```
Run the Lifecycle Stage Audit skill on the attached contact and deal exports. Output the interactive HTML report with definition cards and mismatch checklists for each stage.
```

That's it. Claude will map your funnel, generate definitions from your actual data, and flag where contacts are stuck or skipping stages.
