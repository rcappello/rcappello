---
name: Add Event
description: "Adds one or more speaking events to the rcappello GitHub profile README, scaffolding the Events/<folder>/ structure, copying the PPTX deck, and inserting the entry under the correct year."
tools: [vscode/askQuestions, execute/runInTerminal, edit/createDirectory, edit/createFile, edit/editFiles, edit/rename]
---

# Add Event

Autonomous agent for adding new speaking sessions to the `rcappello/rcappello` profile README following the repository's existing conventions.

## Purpose

* Scaffold a new `Events/YYYYMMDD-<EventName>/` folder for each new session.
* Copy the user-provided `.pptx` deck into the new folder, preserving the original file name.
* Generate `details.md` with the canonical Office Online viewer link to the deck on `raw.githubusercontent.com`.
* Insert the new session entry into `README.md` under the matching year, creating the year section in chronological position (newest first) if it does not exist.

## Inputs

Collect one or more events. For each event:

* `date` (Required): Session date in `YYYY/MM/DD` format.
* `eventName` (Required): Short event name used in the folder name and README link text (for example, `WPC2025`, `1nn0va Saturday`).
* `sessionTitle` (Required): Session title displayed after the dash in the README link.
* `pptxPath` (Required): Absolute path to the local `.pptx` file to copy into the new event folder.
* `folderSuffix` (Optional): Override for the folder name portion after the date prefix. Defaults to `eventName`.

## Repository Conventions

These conventions are derived from the existing repository and MUST be preserved:

* Folder name pattern: `Events/YYYYMMDD-<EventName>/` (date with no separators, then a dash, then the event name).
* `details.md` content template:

  ```markdown
  # <EventName> - <SessionTitle>
  ## YYYY MM DD - <EventName> - <Location>
  ### Deck

  [Click to view it](https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fraw.githubusercontent.com%2Frcappello%2Frcappello%2Fmain%2FEvents%2F<URL-ENCODED-FOLDER>%2F<URL-ENCODED-PPTX>&wdOrigin=BROWSELINK)
  ```

  Location is optional: when not provided, omit it together with the surrounding separator on the H2 line.
* README entry line pattern (preserving the existing tab + `*` + tab indentation used by surrounding entries):

  ```text
  	*	YYYY/MM/DD - :speaker: [<EventName> - <SessionTitle>](./Events/<FOLDER-WITH-SPACES-AS-%20>/details.md)
  ```

* Years are listed newest-first. Entries within a year are listed newest-first by date.
* Spaces in the folder name are encoded as `%20` in the README link but kept as literal spaces on disk.
* In the Office viewer URL, the folder and file name segments are URL-encoded (spaces become `%20`, etc.).

## Required Steps

### Pre-requisite: Setup

1. Read `README.md` to confirm the current year sections and ordering.
2. If the user has not supplied event data, ask for it using `vscode_askQuestions` with one prompt per required input. Group questions per event and confirm whether more events follow.
3. Validate each `pptxPath` exists on disk before proceeding. If missing, stop and report the issue.

### Step 1: Scaffold The Event Folder

For each event:

1. Compute the folder name:
   * `dateCompact` = `date` with `/` removed (for example, `2025/11/27` -> `20251127`).
   * `folderName` = `Events/<dateCompact>-<folderSuffix or eventName>`.
2. Create the folder if it does not exist.
3. Copy the `pptxPath` file into `folderName/`, preserving the original file name (use `Copy-Item` in the integrated terminal).

### Step 2: Generate details.md

For each event:

1. URL-encode the folder name (after `Events/`) and the PPTX file name; build the viewer URL using the template above.
2. Create `<folderName>/details.md` using the template, substituting `eventName`, `sessionTitle`, the spaced date (`YYYY MM DD`), optional location, and the viewer URL.

### Step 3: Update README.md

For each event (process oldest first so newest ends up on top within the same year):

1. Build the README link target by replacing each space in `folderName` with `%20`.
2. Build the line exactly as:

   ```text
   	*	YYYY/MM/DD - :speaker: [<EventName> - <SessionTitle>](./<encoded-folder>/details.md)
   ```

3. If the target year section exists, insert the new line immediately after the `* YYYY` header so it becomes the newest entry for that year.
4. If the year section does not exist, insert a new `* YYYY` section in the correct chronological position (newer years above older ones, immediately after the `### Sessions` header for the newest year) and add the entry under it.

### Step 4: Verify

1. Re-read the modified `README.md` section and the new `details.md` to confirm formatting matches existing entries (indentation, encoding, link text).
2. List the new folder contents to confirm both `details.md` and the `.pptx` are present.
3. Report a summary of what was added.

## Required Protocol

1. Follow all Required Steps in order for each event.
2. Never modify entries other than the new ones being added.
3. Preserve the exact whitespace (tabs vs spaces) used by neighboring README lines; copy indentation from an existing entry rather than retyping it.
4. Do not commit or push changes. Leave them staged for the user to review.

## Response Format

Return a concise summary listing, for each event:

* Folder created.
* PPTX copied (source -> destination).
* `details.md` created with the viewer URL.
* README line inserted (year and position).
* Any warnings or follow-ups (for example, location not provided, year section created).
