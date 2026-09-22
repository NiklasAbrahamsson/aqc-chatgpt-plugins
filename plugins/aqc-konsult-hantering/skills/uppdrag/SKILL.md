---
name: uppdrag
description: "Spara och publicera ett konsultuppdrag via Konsultmatch, sätt användaren som kontaktperson, skapa rätt HubSpot-deal och returnera en färdig sammanfattning för utskick med den fullständiga publika ansökningslänken synlig."
---

# /uppdrag

## Purpose

Use this skill when the user provides a new consulting assignment and wants it saved, publicly shared, and prepared for outreach.

The workflow is:

1. Hand the assignment over to Konsultmatch `create_assignment` ("Skapa uppdrag") and let the platform own extraction, skill normalization, geocoding and language resolution.
2. Publish the saved assignment and set the user as public contact person.
3. Retrieve the public application URL.
4. Ask the user whether they want to proceed with the HubSpot part.
5. Only if the user confirms, create a HubSpot deal populated from the saved assignment.
6. Return an email-ready Swedish summary with the full public URL visible.

## Step 1 — Skapa uppdrag

**This skill does not define its own extraction rules.** `create_assignment` and the Konsultmatch `extract_assignment` prompt are the single source of truth for how posting text becomes a structured assignment. Follow those instructions as written — including where they contradict habits from earlier versions of this skill.

Do not pre-summarize, restructure or clean up the posting before handing it over. Pass the source as it stands.

### Preferred path — structured `assignment`

1. Fetch the `extract_assignment` prompt from the Konsultmatch MCP server.
2. Run it against the raw posting text.
3. Call `create_assignment` with the resulting `assignment` object, plus `sourceUrl` when the assignment came from a web page.

This path is preferred: the prompt is versioned server-side, so the skill stays correct without edits here whenever the extraction changes.

### Fallback — raw `text`

Only if the `extract_assignment` prompt cannot be fetched or run, call `create_assignment` with `text` set to the raw posting and let the platform extract server-side. Note in the final report that the fallback was used.

Pass exactly one of `assignment` or `text` — never both.

### Rules that come from the tool

These are the tool's rules, restated here only so they are not lost. If the tool's own instructions differ, the tool wins.

- `description` and `notes` must be **verbatim** from the source. Never summarize, shorten or rewrite them. Only obvious chrome — greetings, signatures, navigation — may be trimmed.
- Never guess or fabricate a value. A missing, uncertain or suspicious value (an email that may belong to a contact person or salesperson rather than the consultant, an inferred date, an assumed city or rate) is asked about, not filled in.
- If the tool returns `{ needsUserInput: true, missingFields: [...] }`, **nothing was saved**. Relay each message to the user, collect the answers, then call the tool again with the confirmed values. Do not fill the gaps yourself.
- Skill normalization against the knowledge graph, city geocoding and language resolution happen platform-side. Do not attempt them in this skill.

After a successful save, the saved assignment is the source of truth for every later step. Read it back with `get_assignment` when fields are needed.

## Step 2 — Publish and set contact person

Use `manage_assignment_sharing` with `action: "publish"` on the saved assignment id. Set `responsible` to the invoking user (name or email of the admin) in the same call so the public page shows the right contact; if the platform rejects the combination, follow up with `action: "update_settings"`.

Notes:
- `publish` returns the public URL. A first publish mints the address; republishing reuses it, and an `OPEN` assignment moves to `IN_PROGRESS`.
- Do not set `audienceTagLabels` unless the user asked for a restricted audience — an unintended `[]` reopens a restricted link to everyone.
- Set `deadlineDate` only when the source states one.
- Never fabricate the public URL. It comes from the tool response or from `get_assignment` (`publicSharing`).

## Step 3 — Ask before HubSpot

Konsultmatch is always handled automatically through the preceding steps. Before using any HubSpot tool, always ask the user explicitly whether they want to proceed with the HubSpot part. Do not create, update or associate any HubSpot records until the user confirms.

If the user declines or does not confirm, skip HubSpot and continue with the email output. Report that HubSpot was not run because the user did not confirm.

## Step 4 — HubSpot

If the user confirmed and HubSpot tools are available, create a deal representing the saved assignment.

The deal should:
- be based on the saved assignment, not on a separately reinterpreted version of the posting;
- use the customer and assignment information as Konsultmatch saved it;
- associate with the correct company/customer when a reliable match exists;
- associate with the relevant contact when appropriate and supported;
- avoid creating duplicate companies or contacts merely because a match is uncertain.

If an exact customer/company association cannot be established confidently, do not invent one.

## Step 5 — Email output

Return an email-ready Swedish summary that can be pasted into HubSpot.

The output should normally include:
- a short subject suggestion;
- role/title;
- customer/context when appropriate;
- location;
- assignment period;
- a short description;
- key requirements;
- a clear call to action;
- the FULL public application URL on its own visible line.

The email summary is the one place where the assignment may be condensed — the saved `description` in Konsultmatch stays verbatim regardless.

Example URL presentation:

Ansök här:
https://example.com/public/assignment/123

Do NOT output:

[Ansök här](https://example.com/public/assignment/123)

Do NOT replace the URL with shortened text.

## Response format

After successful execution, report:

### Uppdrag sparat
- Titel
- Kund
- Plats
- Period
- Extraktion: prompt / server-side fallback
- Publicerad: Ja
- Kontaktperson: namn
- Publik länk: full URL

### HubSpot
- Deal skapad: Ja/Nej
- Kund associerad: Ja/Nej
- Kontakt associerad: Ja/Nej

### Mailtext

Provide the ready-to-send Swedish email text with the complete public URL visible.

## Failure handling

If one step fails:
- do not claim that it succeeded;
- report exactly which step failed;
- continue with independent steps when safe;
- never fabricate a public URL, HubSpot deal id, assignment id, customer association, or other result.

If `create_assignment` returns `needsUserInput`, stop and ask — do not proceed to publishing or HubSpot, because nothing was saved.

If HubSpot is unavailable, still save and publish the assignment when Konsultmatch is available, and clearly state that the HubSpot deal could not be created.

## Trigger

Use this skill when the user invokes `/uppdrag` or clearly asks to save/publish a supplied consulting assignment using this workflow.
