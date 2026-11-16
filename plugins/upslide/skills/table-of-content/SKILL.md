---
name: upslide-table-of-content
description: >
  Use this skill to insert sections, subsections, or an agenda slide into a PowerPoint presentation using UpSlide MCP.
  Trigger whenever the user wants to create, add, insert, or set up sections or dividers in a presentation using UpSlide.
  Examples: "add a section called Introduction", "insert sections for my deck", "crée des sections dans ma présentation",
  "ajoute un divider de section", "insert a subsection", "set up my agenda slide", "add sections to my PowerPoint".
  Always use this skill when the user mentions sections, subsections, dividers, or agenda slides in the context of UpSlide or PowerPoint.
  If the user doesn't provide section names or enough details, ask before proceeding.
compatibility: "Requires UpSlide MCP and an active PowerPoint session"
---

# 📋 UpSlide Table of Content

Insert sections, subsections, or agenda slides into a PowerPoint presentation via UpSlide MCP.

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

---

## Step 2: Call UpSlide MCP

Use `UpSlide MCP:InvokeCapability` with:

```json
{
  "capabilityId": "insert_toc_element",
  "args": {
    "type": "section",
    "index": 2,
    "title": "Introduction",
    "numberedSection": true,
    "appendixSection": false
  }
}
```

Note: `numberedSection` is `true` by default for all sections. Only set it to `false` if the user explicitly asks for an unnumbered section.

### Type reference

| User says | `type` value |
|---|---|
| "section", "divider de section", "section slide" | `section` |
| "subsection", "sous-section", "sub-section divider" | `sub_section` |
| "agenda", "table of contents", "sommaire" | `agenda` |

### Multiple sections

If the user wants several sections, insert them **in reverse order of index** (highest index first) so that earlier insertions don't shift later slide positions. Or insert in order if the user provides final positions accounting for slides added.

---

## Step 3: Poll for completion

After each `InvokeCapability` call, poll `UpSlide MCP:GetUpSlideMcpTaskStatus` every 1-2 seconds until status is `done`, `failed`, or `doneWithErrors`.

If status is `failed` or `doneWithErrors`, report the error message to the user clearly.

---

## Step 4: Confirm to the user

Once done, confirm what was inserted:
- Section name
- Slide position
- Type (section / subsection / agenda)
- Whether it was numbered

If multiple sections were inserted, summarize all of them in a single message.

---

## Edge cases

- **Index out of range**: UpSlide will return an error. Ask the user how many slides are in the deck and retry with a valid index.
- **Agenda type**: Does not require a title. If the user provides one anyway, ignore it (the `agenda` type doesn't accept a title field).
- **Appendix section**: Only relevant if the user explicitly says "appendix". Otherwise default to `false`.
- **Unnumbered section**: Only use `numberedSection: false` if the user explicitly requests it. The default is always numbered.
