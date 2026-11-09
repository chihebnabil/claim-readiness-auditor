# Disclosures

These are the commitments this Skill makes about data, permissions and limits. Read this file when the user asks what the Skill can access, what it changes, or how it handles their data. Answer from it in plain words.

## Data handling

- **Stays on the NAS.** Every step is carried out by the NAS's registered experts. The Skill makes no network requests and sends no data to outside services.
- **Only the folders the user chooses.** The Skill scans only the folders confirmed in Required input, and never widens the search on its own.
- **No credentials.** The Skill never asks for insurer, store or email logins, passwords or account numbers, and it never stores any.
- **No profiling.** Asset values, receipts and policy details are used only to build the report in front of the user. Nothing is kept between runs unless the user saves the report.

## Permissions and changes

- **Read-only by default.** Steps 1 to 6 only read files. The Skill never moves, renames, tags, edits or deletes the user's files, and it never creates folders.
- **One optional write.** Step 7 saves the report as a new Markdown file, only when the user asks. It goes through the platform's approval step, and the Skill never overwrites an existing file.
- **No direct NAS commands.** The Skill only delegates to registered experts. Each expert runs its own actions under the platform's permission and approval rules.

## Accuracy and limits

- **Not insurance advice.** The score measures how complete the evidence is. It does not predict whether an insurer will pay, and it does not replace reading the policy or asking the insurer.
- **Based only on what was found.** Every finding comes from a real file returned by an expert. Missing evidence is reported as missing, a check that could not run is reported as "not checked", and uncertain asset matches are flagged as "possible match".
- **Recognition can miss things.** Blurry serial plates, handwritten receipts and unusual policy wording may not be read correctly. Users should confirm important findings against the source files listed in the report.
- **Evidence outside the NAS is invisible.** Receipts kept only in email or online accounts count as missing until they are saved to a scanned folder.
- **Runs on request.** The Skill checks the folders each time the user asks; it does not run in the background.

## Package contents

This package contains only Markdown: `SKILL.md` and the files in `references/`. It includes no scripts, executables, binaries, credentials, personal data or third-party materials. The sample in the example report uses invented data.
