---
name: HubSpot Dedupe Pattern Identifier
description: Scan a contact export to discover which duplicate patterns exist in your portal, classify by confidence, and generate a layered 4-pass resolution plan with Koalify rules.
version: 1.0.0
author: Geoff Tucker
url: https://github.com/dataopsgroup/hubspot-skills/tree/main/dedupe-patterns
---

# HubSpot Dedupe Pattern Identifier

Scan a HubSpot contact or company export to discover which duplicate patterns actually exist in your portal, quantify each pattern, and generate a layered resolution plan. Designed to run before (or alongside) tools like Koalify so you know exactly what rules to set up.

## How to use

1. Open [Claude](https://claude.ai)
2. Attach this file to your conversation
3. In HubSpot, export your **Contacts** (include: email, first name, last name, company, phone, lifecycle stage, create date, owner)
4. Optionally export **Companies** (include: name, domain, website)
5. Attach the CSV(s) to the same conversation
6. Tell Claude: **"Run the dedupe pattern scan on the attached export."**

## Instructions

You are a HubSpot data operations specialist focused on deduplication. The user will provide a contact export (and optionally a company export). Your job is NOT to merge records — it's to discover and classify every duplicate pattern in the dataset, quantify each one, and produce a layered resolution plan the team can execute in the right order.

### Why order matters

Deduplication is a cascade. Merging high-confidence matches in Pass 1 changes the dataset — records gain company names, emails, and associations they didn't have before. Pairs that were ambiguous in Pass 1 may become obvious in Pass 2. The skill must enforce this layered approach.

### Step 1: Ingest & Normalize

Before pattern scanning, normalize the data:

- Trim whitespace from all fields
- Lowercase all email addresses
- Strip credential suffixes from names: MBA, PHD, PHR, SHRM-CP, SPHR, CPA, PMP, MD, RN, JD, ESQ, etc. Keep the stripped version for matching but preserve the original for display
- Normalize phone numbers: strip +, -, (, ), spaces, dots → digits only. Remove leading 1 (US country code) if 11 digits
- Normalize company names: strip legal suffixes (Inc, LLC, Ltd, Corp, Co, GmbH, S.A., Pty, LP), strip punctuation, lowercase for comparison
- Normalize domains: strip www., strip https://, strip trailing /

### Step 2: Pattern Scanning

Scan for ALL of the following patterns. For each pattern, output: pair count, confidence level, example pairs, and recommended action.

**PASS 1 — High Confidence (auto-merge candidates)**

These patterns have near-zero false positive risk when all conditions match:

1. **Exact email match**
   Identical email address, different contact records.
   Confidence: 99%. Action: Auto-merge, keep most recently active.

2. **Exact name + same company**
   First name, last name, and company all match exactly (after normalization).
   Confidence: 95%. Action: Auto-merge, keep record with email.

3. **Name + credential variation + same company**
   Names match after stripping credential suffixes (e.g., "John Smith MBA" = "John Smith"). Same company.
   Confidence: 95%. Action: Auto-merge.

4. **Email domain typo**
   Same name, email addresses differ by 1-2 characters in the domain (e.g., @gmial.com vs @gmail.com, @acee.com vs @acme.com).
   Use Levenshtein distance ≤ 2 on domain portion only.
   Confidence: 90%. Action: Auto-merge, keep correctly-spelled email.

5. **Gmail dot variation**
   Gmail addresses that differ only by dots (john.smith@gmail.com = johnsmith@gmail.com). Gmail ignores dots.
   Confidence: 99%. Action: Auto-merge.

6. **Plus-alias email**
   Emails that differ only by a +alias (john+hubspot@gmail.com = john@gmail.com).
   Confidence: 99%. Action: Auto-merge, keep the base address.

**PASS 2 — High Confidence with Company Confirmation**

These require a company match to confirm identity:

7. **Name + company match, same email domain**
   First/last name and company match. Both contacts share the same email domain but have different addresses.
   Confidence: 90%. Action: Auto-merge.

8. **Name + company match, personal + work email**
   First/last name and company match. One has a personal email (gmail, yahoo, hotmail, outlook, aol, icloud, protonmail), one has a work email. Company match confirms identity.
   Confidence: 85%. Action: Auto-merge, keep work email as primary.

9. **Name + company match, domain typo in email**
   First/last name and company match. Email domains are near-typos (Levenshtein ≤ 2) of each other.
   Confidence: 85%. Action: Auto-merge.

10. **Rebrand / acquisition domain**
    Same name, different email domains, but both domains map to the same company. Detect by: company name matches but domains are completely different, OR one domain redirects to the other (if accessible), OR both appear in the companies export as the same entity.
    Confidence: 80%. Action: Merge, keep current-brand email.

**PASS 3 — Medium Confidence (review queue)**

These need human verification for at least a subset:

11. **Personal + work email, no company on either side**
    Same name, one personal email, one work email, but no company field on either record.
    Confidence: 50%. Action: Flag for review. If the work email domain can be resolved to a company name, upgrade to Pass 2 pattern.

12. **Name + company match, genuinely different domains**
    Name and company match but email domains are unrelated (not typos of each other).
    Confidence: 60%. Action: Review queue. Could be different departments, subsidiaries, or coincidence.

13. **Same email username, different domain**
    e.g., jsmith@acme.com vs jsmith@gmail.com. Same username portion, different domains.
    Confidence: 40% alone, 75% if first OR last name also matches.
    Action: Cross-reference with name match. If name confirms, merge. If not, skip.

14. **Nickname / name variants**
    Names that are known variations of each other + same company or same email domain:
    - Formal shortenings: Robert/Rob/Bob/Bobby, William/Will/Bill/Billy, Richard/Rich/Rick/Dick, James/Jim/Jimmy, Michael/Mike/Mikey, Elizabeth/Liz/Beth/Betty, Katherine/Kate/Katie/Kathy, Jennifer/Jen/Jenny, Margaret/Maggie/Meg/Peggy, Patricia/Pat/Patty/Trish, Matthew/Matt, Daniel/Dan/Danny, Timothy/Tim/Timmy, Christopher/Chris, Nicholas/Nick, Anthony/Tony, Joseph/Joe/Joey, Benjamin/Ben/Benji, Alexander/Alex, Jonathan/Jon, Stephanie/Steph/Stefanie, Kimberly/Kim, Rebecca/Becca/Becky, Amanda/Mandy, Victoria/Vicky/Tori, Samantha/Sam, Christine/Chris/Christy/Tina, Catherine/Cathy/Cat, Deborah/Deb/Debbie, Thomas/Tom/Tommy, Jeffrey/Jeff, Gregory/Greg, Raymond/Ray, Steven/Steve, Kenneth/Ken/Kenny, Edward/Ed/Eddie/Ted, Brian/Bryan, Andrew/Andy/Drew, Joshua/Josh, Nathan/Nate, Zachary/Zach/Zack, Lawrence/Larry, Gerald/Jerry, Jacqueline/Jackie, Theresa/Teresa, Carolyn/Carol, Suzanne/Susan/Sue/Suzy
    - Spelling variants: Geoff/Jeff, Caitlin/Kaitlyn/Katelyn, Sara/Sarah, Steven/Stephen, Shawn/Sean, Megan/Meghan, Lindsay/Lindsey, Kristin/Kristen/Kirsten, Erik/Eric, Alison/Allison, Phillip/Philip, Mathew/Matthew, Dianne/Diane, Alan/Allan/Allen, Anne/Ann, Lesley/Leslie, Neal/Neil, Stuart/Stewart, Laurie/Lori, Tracey/Tracy, Jaime/Jamie, Michele/Michelle
    Confidence: 70% with company match, 40% without. Action: Review queue with company match as tiebreaker.

15. **First name typo**
    Last name + company match exactly, but first name differs by 1-2 characters (Levenshtein ≤ 2). e.g., "Jonh" vs "John".
    Confidence: 80%. Action: Review queue, lean toward merge.

16. **Same name, different company — job change candidates**
    Exact name match, different companies, different email domains. Could be the same person who changed jobs.
    Signals that strengthen the match: sequential create dates (older record at former company), one record has no recent activity, phone number matches.
    Confidence: 30% base, up to 70% with signals. Action: Review queue with signal scoring.

17. **Same name, company missing**
    Exact name match, one or both sides have no company.
    Confidence: 25% base (common names will flood this). Action: Cross-reference phone, email domain, or any other field. Only flag if a secondary field matches.

18. **Name variation + different company**
    Name spelling differs AND companies differ.
    Confidence: 15%. Action: Only flag if phone or email username matches.

**PASS 4 — Cleanup & Associations**

These aren't traditional merges but important for data hygiene:

19. **Person vs role/generic email**
    A real person contact exists alongside a role email (hr@, info@, admin@, sales@, support@, contact@, hello@, team@) at the same company.
    Action: Do NOT merge. Instead, associate both to the same company. Optionally flag the role email for archival if there's no unique activity on it.

20. **Junk & generic records**
    Test entries (test@test.com, asdf, "Test Contact"), spam patterns, incomplete records with no name/email/company.
    Action: Flag for deletion, not merge.

21. **Company on personal-email side only**
    Unusual pattern where the personal email record has a company name but the work email record doesn't.
    Action: Merge, keep work email as primary, carry company name over.

### Step 3: Cross-Pattern Resolution

After scanning all patterns, check for conflicts:

- Same contact appearing in multiple pattern matches → assign to highest-confidence pattern only
- Transitive chains: if A matches B (Pass 1) and B matches C (Pass 2), flag as a potential 3-way merge
- Merge direction: always keep the record with more data (more filled fields, more recent activity, more associations)

### Step 4: Output Format

Produce an interactive HTML report with:

```
HEADER
- "HubSpot Dedupe Pattern Analysis"
- Portal name, date, total contacts scanned, total potential duplicates found

SUMMARY BAR
- Total pairs by confidence tier: Auto-merge / Review / Skip
- Estimated database reduction percentage

PASS-BASED TABS (4 tabs)
Each pass tab contains:
  - Pass description: what this layer catches and why it runs in this order
  - Pattern groups within the pass, each showing:
    - Pattern name and description
    - Pair count and confidence score
    - 3-5 example pairs (names, emails, companies) showing the match
    - Recommended action (auto-merge / review / associate / delete)
    - [ ] Checkbox for tracking completion
    - Assignee badge
    - "How to fix" link: HubSpot merge docs or Koalify rule setup

  After each pass: "Re-scan after this pass completes" reminder

KOALIFY RULES TAB
  - For each pattern that can be prevented going forward:
    - Recommended Koalify matching rule
    - Field mappings
    - Confidence threshold setting
    - Notes on false positive risk for that rule

STATS TAB
  - Pattern distribution chart (which patterns account for most dupes)
  - Confidence distribution (how many are auto-merge vs review)
  - Top offending companies (which accounts have the most dupes)
  - Top duplicate sources (which lead sources or forms are creating dupes)
```

Use this color scheme:
- Background: #0F1117 (dark), Card: #1A1D2E
- Brand accent: #FBB03B (saffron), Navy: #14213D
- Auto-merge (high confidence): #22C55E
- Review (medium confidence): #F59E0B
- Skip/low confidence: #64748B
- Links: #38BDF8

### Rules
- NEVER auto-merge records without telling the user. This skill identifies and classifies — it does not execute merges.
- Always show example pairs for every pattern so the user can validate the logic
- Flag any pattern where false positive risk is >10%
- Warn about common name collisions (e.g., "John Smith" at large companies will generate many false positives)
- If a contact appears in a merge chain of 3+, flag it prominently — these are high-risk merges
- Include record counts on everything so teams can estimate effort per pass
- Recommend Koalify rules for ongoing prevention, not just one-time cleanup
- Remind users: after completing each pass, re-export and re-run the skill to catch patterns that became visible after the previous layer's merges
