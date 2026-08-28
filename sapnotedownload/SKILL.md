---
name: sapnotedownload
description: Download one or more SAP Notes from SAP for Me and recursively collect every prerequisite matching the requested software component and version range. Use when a user asks to download SAP Notes with dependency checks.
---

# SAPNOTEDOWNLOAD

Download all requested SAP Notes and their qualifying prerequisite Notes through the in-app browser, then place the downloaded files in the requested folder.

## Start every run

Before using SAP for Me:

1. Tell the user to sign in to their SAP account in the in-app browser and reply `ready` when authentication is complete. Never ask the user to send a password or other sign-in secret in chat.
2. Ask for any missing required inputs in one concise prompt:
   - **SAP Note numbers:** one or more root Note numbers, separated by commas, spaces, or new lines.
   - **Software Component:** for example, `S4FPSL`.
   - **From version** and **To version:** both are required and are matched exactly.
   - **Destination folder:** the local folder that should contain the downloaded Note files.
3. Do not ask again for values the user already supplied. Normalize Note numbers to digits, remove duplicates while preserving their order, and reject malformed values instead of guessing.

Use the in-app browser and the user's existing authenticated session. Construct every Note URL as:

```text
https://me.sap.com/notes/<NOTE_NUMBER>
```

## Traverse all roots and dependencies

Use one shared queue or stack for all supplied root Notes, plus shared `visited`, `downloaded`, and dependency-edge collections. This deduplicates Notes that are shared by multiple roots and prevents cycles.

For each unvisited Note:

1. Open its direct URL and verify that the displayed Note number matches the URL.
2. Open the **Correction** or **Correction Instructions** area and locate the **Prerequisites** table.
3. Read the complete HTML table through page evaluation. Do not rely on screenshot OCR or a viewport-limited accessibility snapshot, because off-screen rows can be missed.
4. Identify columns by their normalized header text, including **Software Component**, **From**, **To**, and **SAP Note/KBA**. Read every data row. Correctly expand HTML `rowspan`/`colspan`, or carry merged group values forward, so continuation rows inherit their Software Component, From, and To values.
5. Keep only rows whose Software Component, From, and To exactly equal the user's requested inputs. From and To are separate exact comparisons; do not treat them as an approximate range.
6. Extract every Note number from the qualifying rows' **SAP Note/KBA** cells. Do not infer Note numbers from titles or other columns.

Branch rules:

- If the page has no Prerequisites table, or no rows match the requested component/from/to values, end that branch.
- If a prerequisite Note number equals the current Note number, record it as a self-reference and do not follow it.
- For every different prerequisite not already visited, record the dependency edge and add it to the shared queue or stack.
- If a Note is unavailable, access is denied, or its table cannot be read completely, report that branch as incomplete; do not claim the dependency set is complete.

## Download and file handling

After the current Note's prerequisites have been captured:

1. Click the Note download control and choose the SAP Note/SNOTE download when a format choice is shown.
2. Wait for the download to finish and verify that a file for the current Note exists in the browser's download location. SAP Note archives commonly follow `000<NOTE_NUMBER>_00.SAR`; confirm the Note number rather than assuming success from the click.
3. Mark the Note downloaded only after verification. Close its temporary browser tab after a successful download. Keep a failed tab open only when it helps the user resolve the failure.
4. Move each verified file to the destination folder. Create the destination if needed, validate the exact source and destination paths, and never overwrite an existing file without the user's direction.

Only move files verified as belonging to the current run. Do not delete or move unrelated downloads.

## Completion report

Before declaring success, confirm that the queue is empty, every discovered non-self prerequisite was processed, every successfully processed Note has one verified destination file, and no duplicate was downloaded.

Report:

- the root Note numbers;
- the requested Software Component, From, and To values;
- the complete dependency tree or edge list for each root;
- all downloaded Note numbers and destination filenames;
- the destination folder;
- any skipped self-references, existing-file conflicts, access failures, or incomplete branches.
