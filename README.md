# Claim Readiness Auditor

**A UGREEN AI NAS Skill that scores how claim-ready your insured assets are, before you ever need to file.**

People and small businesses often find out their insurance claim is weak only *after* an incident: the receipt is missing, there's no photo of the serial number, or it's unclear what the policy covers. Payouts get denied or cut. Most tools put a claim file together after the loss. This Skill works the other way: it finds the gaps beforehand so you can close them.

## What it does

1. **Scan & Map.** Uses `ug-filesystem` and `ug-document` to find receipts, warranty PDFs and policy documents, and `ug-photo` to find asset photos and read serial numbers where they're visible.
2. **Score & Diagnose.** Checks each insured asset against three pillars of evidence:
   - **Ownership:** a receipt or invoice
   - **Condition:** a dated photo with metadata
   - **Coverage:** confirmation that the policy covers it

   Each asset gets a **Claim Readiness Score (0–100%)** and a plain-language diagnosis, for example *"Espresso Machine: 30%. No receipt found, no serial photo."*
3. **Prioritize & Act.** Produces an **Evidence Gap Action List** ranked by priority, for example:
   - "Photograph the serial number of the MacBook Pro and save it to `/Assets/Electronics`"
   - "Download the receipt from Amazon and place it in `/Receipts/2026`"

## Why it's a Skill

It's a Markdown-defined workflow (`SKILL.md`) that orchestrates native NAS capabilities (`ug-photo`, `ug-document`, `ug-filesystem`). There's no executable app and no external cloud calls. **Everything runs on your NAS.** Scanning financial and asset data around the clock is only trustworthy when it stays local.

## Who it's for

- Homeowners with valuables
- Renters with electronics or jewelry
- Small businesses: a café with an espresso machine, a studio with cameras

## Demo flow

1. Start from a messy `/Documents` and `/Photos` on the NAS.
2. Run the Skill. The dashboard shows 8 assets: 3 high (>80%), 4 medium (40–79%) and 1 critical (<40%).
3. Drill in: *"Café Espresso Machine: 25%. No receipt, no serial photo."*
4. Follow the Action List: photograph the serial number and save it. Re-run, and the score jumps to 75%.
5. *"This claim would have failed. Now it won't."*

## Skill package

The Skill itself lives in [`claim-readiness-auditor/`](claim-readiness-auditor/). This is the folder that gets zipped and submitted.

```
claim-readiness-auditor/
├── SKILL.md                    # entry point: inputs, workflow, failure handling
└── references/
    ├── scoring-rubric.md       # four 25-point checks, bands, actions, ranking
    └── report-format.md        # dashboard and action list layout
```

The workflow delegates to the NAS's registered experts: `file-agent` (`ug-filesystem`), `document-agent` (`ug-document`), `photo-agent` (`ug-photo`) and `knowledge-agent` (`ug-knowledge`, only to save the report when the user asks). Every step is read-only except that optional save, which needs the user's approval.

To check and package it with the Uliya Skill Kit:

```bash
node uliya-skill-kit/scripts/check-skill.mjs claim-readiness-auditor
```

```bash
zip -r claim-readiness-auditor.zip claim-readiness-auditor
```

## Status

Built for the UGREEN AI NAS hackathon ("Automate Your Workflow" track).
