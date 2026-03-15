# LinkedIn Post — Steal This Claude Skill #3

---

"Just run a dedupe" is the worst advice in HubSpot ops.

I just finished deduplicating a portal with 10,000+ duplicate pairs. Tools like Koalify are fantastic at the mechanics — but they can only match the patterns you tell them to look for.

Every portal has its own weirdness. Credential suffixes in names (MBA, SHRM-CP, PHR). Company rebrands where the same person has emails at two completely different domains. Gmail dot variations. Job changers with the same name at a new company.

If you don't know which patterns exist in YOUR portal, you'll merge 800 records and miss 4,000 more.

So I built a Claude skill that scans your contact export and classifies every duplicate pattern it finds before you merge anything.

Here's how it works:

→ Export your contacts from HubSpot (include email, name, company, phone)
→ Open Claude, attach the SKILL.md file and your CSV
→ Tell it: "Run the dedupe pattern scan on the attached export."
→ It discovers which patterns exist in your data and quantifies each one
→ Then it outputs a 4-pass resolution plan — high-confidence auto-merges first, then company-confirmed matches, then a review queue, then cleanup

The key: it's layered. Pass 1 merges change the dataset. Pairs that were ambiguous become obvious after the first layer resolves. You re-scan between each pass.

It also generates the specific Koalify rules to prevent each pattern from recurring.

Grab it free: https://github.com/dataopsgroup/hubspot-skills/tree/main/dedupe-patterns

What other skills would you like me to share?

#HubSpot #RevOps #DataOps #Deduplication #CRM

---

## Posting Notes

- **Character count**: ~1,230 (tight but under 1,300)
- **Image**: Screenshot the demo.html showing the Pass 1 tab with the cascade diagram visible at top. The "merge → re-scan → merge → re-scan" visual tells the story.
- **Link**: https://github.com/dataopsgroup/hubspot-skills/tree/main/dedupe-patterns
- **Best posting time**: Tuesday or Wednesday, 7-8am
- **Koalify note**: This post name-checks Koalify positively. If they see it, that's a potential partnership conversation.
