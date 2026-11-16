---
name: table-of-content
description: >
  Use this skill to insert sections, subsections, or an agenda slide into a PowerPoint presentation using UpSlide MCP,
  or to refresh the UpSlide table of contents after the deck changed.
  Trigger whenever the user wants to create, add, insert, or set up sections or dividers in a presentation using UpSlide.
  Examples: "add a section called Introduction", "insert sections for my deck", "crée des sections dans ma présentation",
  "ajoute un divider de section", "insert a subsection", "set up my agenda slide", "add sections to my PowerPoint",
  "refresh the table of contents", "update the agenda", "mets à jour le sommaire".
  Always use this skill when the user mentions sections, subsections, dividers, or agenda slides in the context of UpSlide or PowerPoint.
  If the user doesn't provide section names or enough details, ask before proceeding.
compatibility: "Requires the UpSlide connector and an active PowerPoint session"
---

# 📋 UpSlide Table of Content

Insert sections, subsections, or agenda slides into a PowerPoint presentation via UpSlide MCP, and refresh the table of contents.

---

## Step 1: Gather required information

Before calling any tool, make sure you have:

| Field | Required? | Notes |
|---|---|---|
| `type` | Yes | `section`, `sub_section`, or `agenda` |
| `index` | Yes | 1-based slide index where to insert |
| `title` | Yes (for section/sub_section) | Not needed for `agenda` |
| `numberedSection` | No | Boolean, only for `section`. **Always `true` by default unless the user explicitly says otherwise** |
| `appendixSection` | No | Boolean, only for `section` |

**If any required information is missing, ask the user before proceeding.** In particular:

- If no section names are provided: ask "Quels sont les noms des sections que tu veux créer ?" (or in English: "What are the names of the sections you want to create?")
- If no insertion index is provided: ask at which slide position(s) to insert

For multiple sections at once: collect all section names and their insertion positions before starting.

A refresh needs no information: go straight to Step 2.

---

## Step 2: Call UpSlide MCP

1. Call `UpSlideListCapabilities` with the domain `table_of_content`. It returns the capability that inserts one table-of-content element and the one that refreshes the table of contents, with their ids and input schemas. Do not name this call to the user.
2. Call `UpSlideInvokeCapability` with the id of the right capability, and `args` built from its input schema. Pass enum values exactly as the schema lists them. Never guess a capability id: ids depend on the user's UpSlide version.

To insert a section, the `args` look like:

```json
{
  "type": "section",
  "index": 2,
  "title": "Introduction",
  "numberedSection": true,
  "appendixSection": false
}
```

If the schema asks for other arguments (for example the presentation to work on), fill them from the context.

Note: `numberedSection` is `true` by default for all sections. Only set it to `false` if the user explicitly asks for an unnumbered section.

### Type reference

| User says | `type` value |
|---|---|
| "section", "divider de section", "section slide" | `section` |
| "subsection", "sous-section", "sub-section divider" | `sub_section` |
| "agenda", "table of contents", "sommaire" | `agenda` |

### Multiple sections

If the user wants several sections, insert them **in reverse order of index** (highest index first) so that earlier insertions don't shift later slide positions. Or insert in order if the user provides final positions accounting for slides added.

If the insert capability's schema offers a `refreshToc` argument, set it to `false` on every insertion, then call the refresh capability once at the end.

### Refresh

When the user asks to refresh or update the table of contents, call the refresh capability. It updates the agenda slide, sections, subsections, footers, reminders and breadcrumbs.

---

## Step 3: Wait for completion

`UpSlideInvokeCapability` usually returns the final result. If its response says the task is still running, call `GetUpSlideMcpTaskStatus` with the `taskId` it gives, every 2 seconds (never more often), until the status is `done`, `failed`, or `doneWithErrors`.

If status is `failed` or `doneWithErrors`, report the error message to the user clearly.

---

## Step 4: Confirm to the user

Once done, confirm what was inserted:
- Section name
- Slide position
- Type (section / subsection / agenda)
- Whether it was numbered

If multiple sections were inserted, summarize all of them in a single message. After a refresh, confirm that the table of contents was refreshed.

---

## Edge cases

- **Index out of range**: UpSlide will return an error. Ask the user how many slides are in the deck and retry with a valid index.
- **Agenda type**: Does not require a title. If the user provides one anyway, ignore it (the `agenda` type doesn't accept a title field).
- **Appendix section**: Only relevant if the user explicitly says "appendix". Otherwise default to `false`.
- **Unnumbered section**: Only use `numberedSection: false` if the user explicitly requests it. The default is always numbered.
- **No table of contents yet**: a refresh fails when the presentation has no UpSlide table of contents. Offer to insert an `agenda` slide first.
- **Agenda not set up in the template**: if UpSlide answers that the presentation does not have the UpSlide Agenda set up, relay its explanation to the user; do not retry.
