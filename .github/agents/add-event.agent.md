---
name: Add Event
description: "Adds one or more speaking events to the rcappello GitHub profile README, scaffolding the Events/<folder>/ structure with a details.md that links the deck via a OneDrive share URL, and inserting the entry under the correct year."
tools: [vscode/askQuestions, execute/runInTerminal, edit/createDirectory, edit/createFile, edit/editFiles]
---

# Add Event

Autonomous agent for adding new speaking sessions to the `rcappello/rcappello` profile README following the repository's existing conventions.

## Purpose

* Scaffold a new `Events/YYYYMMDD-<EventName>/` folder for each new session.
* Generate `details.md` linking the deck via a OneDrive (or SharePoint) share URL — the `.pptx` is **never** stored in the repo, to keep clone size small and avoid GitHub's 100 MB file limit.
* Insert the new session entry into `README.md` under the matching year, creating the year section in chronological position (newest first) if it does not exist.

## Inputs

Collect one or more events. For each event:

* `date` (Required): Session date in `YYYY/MM/DD` format.
* `eventName` (Required): Short event name used in the folder name and README link text (for example, `WPC2025`, `1nn0va Saturday`).
* `sessionTitle` (Required): Session title displayed after the dash in the README link.
* `deckUrl` (Required): Public OneDrive/SharePoint share URL of the deck (for example, `https://1drv.ms/p/...`). Used as-is as the Deck link.
* `location` (Optional): City or venue printed on the H2 line of `details.md`.
* `folderSuffix` (Optional): Override for the folder name portion after the date prefix. Defaults to `eventName`.

## Repository Conventions

These conventions are derived from the existing repository and MUST be preserved:

* Folder name pattern: `Events/YYYYMMDD-<EventName>/` (date with no separators, then a dash, then the event name).
* The folder contains only `details.md`. **Do not** copy or commit the `.pptx` — the deck is hosted on OneDrive and referenced via `deckUrl`.
* `details.md` content template:

  ```markdown
  # <EventName> - <SessionTitle>
  ## YYYY MM DD - <EventName> - <Location>
  ### Deck

  [Click to view it](<deckUrl>)
  ```

  Location is optional: when not provided, omit it together with the surrounding ` - ` separator on the H2 line.
* README entry line pattern (preserving the existing tab + `*` + tab indentation used by surrounding entries):

  ```text
  	*	YYYY/MM/DD - :speaker: [<EventName> - <SessionTitle>](./Events/<FOLDER-WITH-SPACES-AS-%20>/details.md)
  ```

* Years are listed newest-first. Entries within a year are listed newest-first by date.
* Spaces in the folder name are kept as literal spaces on disk but encoded as `%20` in the README link.

## Required Steps

### Pre-requisite: Setup

1. Read `README.md` to confirm the current year sections and ordering.
2. If the user has not supplied event data, ask for it using `vscode_askQuestions` with one prompt per required input. Group questions per event and confirm whether more events follow.
3. Validate each `deckUrl` looks like a valid URL (starts with `https://`). If missing or malformed, stop and report.

### Step 1: Scaffold The Event Folder

For each event:

1. Compute the folder name:
   * `dateCompact` = `date` with `/` removed (for example, `2025/11/27` -> `20251127`).
   * `folderName` = `Events/<dateCompact>-<folderSuffix or eventName>`.
2. Create the folder if it does not exist.

### Step 2: Generate details.md

For each event, create `<folderName>/details.md` using the template above, substituting `eventName`, `sessionTitle`, the spaced date (`YYYY MM DD`), optional location, and the verbatim `deckUrl`.

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
2. List the new folder contents to confirm `details.md` is present and **no `.pptx`** was added.
3. Report a summary of what was added.

## Required Protocol

1. Follow all Required Steps in order for each event.
2. Never modify entries other than the new ones being added.
3. Never copy a `.pptx` (or any binary deck) into the repository. If the user supplies a local file path instead of a share URL, stop and ask them to upload the deck to OneDrive/SharePoint and provide the share URL.
4. Preserve the exact whitespace (tabs vs spaces) used by neighboring README lines; copy indentation from an existing entry rather than retyping it.
5. Do not commit or push changes. Leave them staged for the user to review.

## Response Format

Return a concise summary listing, for each event:

* Folder created.
* `details.md` created with the OneDrive deck URL.
* README line inserted (year and position).
* Any warnings or follow-ups (for example, location not provided, year section created).
