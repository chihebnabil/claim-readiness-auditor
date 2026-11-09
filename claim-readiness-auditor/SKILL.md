---
name: claim-readiness-auditor
description: Claim Readiness Score for insured household and small-business assets, with a per-asset 0-100% score across proof of ownership, dated photos, serial numbers and policy coverage, plus a ranked evidence-gap action list. Use when the user asks whether their belongings or business equipment are claim-ready, wants an insurance evidence audit, or asks what proof is missing before a loss happens.
---

# Goal

Tell the user, before any loss happens, how well each insured asset is backed by evidence on their NAS, and exactly what to do to close the gaps. The result has three parts:

1. **Asset register.** Built only from real receipts, invoices, warranty documents, policy documents and photos found in the folders the user approves.
2. **Claim Readiness Score.** For each asset: a 0–100% score, a band (High, Medium or Critical) and a one-line diagnosis, for example "Café espresso machine: 25%. No receipt found, no serial number photo."
3. **Evidence Gap Action List.** Concrete fixes ranked by impact, for example "Photograph the serial number plate of the MacBook Pro and save it to Assets/Electronics."

This Skill is read-only by default. It never moves, renames, tags, deletes or edits the user's files. The only write is an optional saved copy of the report, made after the user asks for it and approves it.

The score measures how complete the evidence is. It is not insurance advice, and it does not predict whether an insurer will pay. Say this once in every report.

# Required input

Collect only the following. Ask in one message, and only for what is missing.

- **Scan folders** (required): the NAS folders to audit, for example the user's Documents and Photos folders. If the user names none, ask which folders to scan. Never guess paths.
- **Minimum asset value** (optional, default 100 in the currency shown on the receipts): purchases below this value stay out of the register unless the policy names them.
- **Business use** (optional): whether any assets are used for a business, because home policies often limit business equipment.
- **Report folder** (optional): needed only if the user asks to save the report.

Do not ask for insurer logins, passwords, account numbers or policy numbers. Do not pre-collect parameters that the experts choose for themselves.

# Generation-time capability evidence

- `ug-filesystem` → `file-agent`
- `ug-document` → `document-agent`
- `ug-photo` → `photo-agent`
- `ug-knowledge` → `knowledge-agent`

Expected actions, all read-only except the last: `ug-filesystem` `find`, `ls`, `stat`; `ug-document` `extract`, `qa`, `metadata`; `ug-photo` `search-picture`, `meta`, `vision-ocr`, `vision-vlm`; `ug-knowledge` `save-text` (state change, workflow approval, optional Step 7 only). The parent Skill never runs these actions itself. It only delegates, and each expert selects and runs its own actions.

# Workflow

Run the steps in order. Every delegation must be self-contained. Before sending one, replace each bracketed description, such as [scan folders], with the real values the user confirmed or the real results an upstream step returned with `completed`. Never send an instruction that still contains a bracket, and never fill one with guessed paths, file names, items or values. Pass only real results downstream.

## Step 1: Find evidence files

```text
delegate --target file-agent --instruction 'Read-only task. Search only inside these folders and their subfolders: [scan folders]. Find files that are likely to be (a) purchase receipts, invoices or order confirmations, (b) warranty cards or warranty documents, (c) insurance policy documents, schedules, declarations pages or renewal notices, and (d) earlier reports whose file name starts with claim-readiness-report. Consider PDF, Word, text and image files (JPG, PNG, HEIC), and use file names, folder names and dates to classify them. For each match return the full path, file name, file type, size, modified date and group: receipt, warranty, policy, previous-report or unsure. Return at most 500 files and say whether more matched. Do not open, move, rename, tag or delete anything. If nothing matches, return an empty result and list the folders that were searched.'
```

If the result is empty, tell the user no receipts, warranties or policy documents were found in those folders, then continue to Step 3 to check photos only. Treat files in the unsure group as receipts in Step 2, and keep them only if the document expert finds purchased items in them.

## Step 2: Read receipts, warranties and the policy

Send this only if Step 1 returned at least one document file (not an image). Send image files to Step 3 instead.

```text
delegate --target document-agent --instruction 'Read-only task. Part A, receipts and warranties. Read only these files: [receipt, warranty and unsure document paths from Step 1]. For every purchased item return the item name, brand, model, serial number only if printed, purchase date, price and currency, merchant, warranty end date if shown, and the source file path. Copy values exactly as printed and leave a field empty when it is not in the document; never estimate. Leave out services, subscriptions, delivery fees and consumables. Part B, policy. Read only these files: [policy document paths from Step 1]. Return the policy period, the insured address or business, covered property categories, individually scheduled items with their values, single-item and category limits, the deductible, and exclusions, each with the page or section it comes from. Part C, previous report. If this file is given, return each asset name and its score exactly as written: [most recent previous-report path from Step 1, or the word none]. List every file you could not read and why. If a part finds nothing, return an empty result for that part.'
```

## Step 3: Check photos

```text
delegate --target photo-agent --instruction 'Read-only task. Use only photos stored inside these folders: [scan folders]. Part A: read these receipt images and return the merchant, purchase date, items, prices and any serial or model numbers printed on them: [receipt image paths from Step 1, or the word none]. Part B: for each of these items, and for the items found in Part A, find photos that show the item: [item list from Step 2 with name, brand and model, or the word none]. Part C: find photos that show a serial number, model number or rating plate label, such as text containing S/N, Serial, SN, Model or MOD, and read that text. Part D: list up to 20 other clearly valuable durable items that appear in the photos but are not on the item list, such as electronics, cameras, appliances, jewelry, instruments, tools or business equipment. For every photo you return give the path, the capture date from EXIF metadata or say it has none, which item it shows, and the exact serial or model text if it is legible. Return at most 300 photos and say whether more were available. Do not create albums or move, edit or delete photos. If a part finds nothing, return an empty result for that part.'
```

## Step 4: Build the asset register

Do this yourself from the real results of Steps 2 and 3. Do not delegate it, and do not add assets that no step returned.

- Include each purchased item at or above the minimum value, each item the policy schedules, and each Part D photo item.
- Merge records that share a serial number, or that share brand and model. Merge on name alone only when it is clearly the same item; otherwise keep the records separate and mark them "possible match".
- Give each asset a category (Electronics, Appliances, Jewelry, Cameras, Instruments, Furniture, Tools, Business equipment or Other) and a value: the receipt price first, then the scheduled value, otherwise "unknown".

If the register is empty, report that no insurable assets were found in the scanned folders, show the starter actions from [the action guide](references/scoring-rubric.md), and stop.

## Step 5: Confirm coverage per asset

Send this only if Step 1 found at least one policy document. If none was found, score coverage as 0 for every asset, and make "Save your current insurance policy and schedule to an Insurance folder" the first action.

```text
delegate --target document-agent --instruction 'Read-only task. Use only these policy documents: [policy document paths from Step 1]. For each asset below, decide whether the policy covers it against loss, theft and damage, and quote the clause, page or section that supports the decision. Answer with exactly one of: covered (named in the schedule or clearly inside a covered category), conditional (for example its value is above a single-item limit, an exclusion may apply, it is business equipment under a home policy, or the policy period has ended), excluded, or not stated. Assets: [asset list from Step 4 with name, category, value and business use]. Do not go beyond what the documents say; when they are silent, answer not stated.'
```

## Step 6: Score, diagnose and rank actions

Score every asset with the four checks in [the scoring rubric and action guide](references/scoring-rubric.md): receipt, dated photo, serial number and coverage, worth 25 points each. Write a one-line diagnosis that names what is missing. Turn each missing or partial check into an action, then rank the actions as the rubric describes. If Step 2 returned a previous report, show each asset's score change since that run.

Present the report in the layout from [the report format](references/report-format.md). Every number and finding must trace back to a real file returned by a step. Keep checks that could not run marked as "not checked", never as missing.

## Step 7: Save the report (optional)

Offer this once, after the report is shown. Run it only if the user asks to save the report and confirms the folder.

```text
delegate --target knowledge-agent --instruction 'Save the Markdown text below as a new file in this folder: [report folder confirmed by the user]. Name it claim-readiness-report- followed by the date of this run in YYYY-MM-DD form. Create a new file and never overwrite or change an existing file. Return the saved path and confirm it was verified. Text: [full report Markdown from Step 6]'
```

# Completion and failure

Treat only `completed` as a business result. Handle every other state explicitly, and never present a partial audit as a complete one.

- **completed**: use only the returned data. An empty result is a valid answer: report "no results" for that part, and do not widen the search, switch to other folders or substitute similar files on your own.
- **failed**: Step 1 failing means nothing can be scored. Report the failure and stop. If Step 2, 3 or 5 fails, continue with the other steps, mark the checks that step supplies as "not checked" for every asset, and title the report "Partial audit". Name the failed step and the reason it gave.
- **cancelled**: stop all later steps. Show any results already completed, labelled as incomplete, and do not compute scores from them.
- **waiting_user**: pass the expert's question to the user in plain words, wait for the answer, then resend the same step with that answer added. Never answer on the user's behalf.
- **waiting_approval**: expected only in Step 7. Show the user the folder and file name that will be written. Continue only after they approve. If they decline, say the report was not saved; it remains in the conversation.

Never say a report was saved unless Step 7 returned `completed` with a verified saved path. Never claim that any file was moved, created or changed by Steps 1 to 6; they are read-only.
