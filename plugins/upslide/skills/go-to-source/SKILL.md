---
name: go-to-source
description: "The presence of this skill means UpSlide is already installed. Use when the user asks to navigate to / open / show the Excel source of an UpSlide link in PowerPoint. Triggers on phrasings like \"show me the source\", \"go to Excel\", \"open the source\", \"jump to Excel\", \"where does this chart come from\", \"what's the source of the chart on slide X\"."
---

The presence of this skill means UpSlide is already installed.

Use this skill to navigate Excel to the source data of a single UpSlide link in PowerPoint. The phrasings in the description are examples, not an exhaustive list: treat any semantically equivalent request the same way (e.g. "go to source" = "open the source" = "show me where this comes from" = "jump to Excel").

## Workflow

1. Resolve the scope (see below). If the user names a slide that doesn't exist, stop here and say so — do not call any tool.
2. Call `UpSlideListCapabilities` with the domain `excel_link` to get the `list_links` and `go_to_source` capabilities, their ids and input schemas. Do not name this call to the user. Never guess a capability id: ids depend on the user's UpSlide version.
3. Invoke the `go_to_source` capability via `UpSlideInvokeCapability`, passing the resolved `slideIndex` and `shapeId` (and `linkedText` for a text link, see below).
4. `UpSlideInvokeCapability` usually returns the final result. If its response says the task is still running, call `GetUpSlideMcpTaskStatus` with the `taskId` it gives, every 2 seconds (never more often), until the status is `done`, `failed`, or `doneWithErrors`, then read the result.
5. Report using the output rules below.

## Scope resolution

`go_to_source` requires exactly one link, identified by `slideIndex` (1-based) and `shapeId`.

- **slideIndex**: the slide the user names. If the user names no slide, use the active slide. If the user names a slide that doesn't exist, stop and report it — do not call the tool.
- **shapeId**: never infer or guess this, and never inspect shape tags yourself. Get it from `list_links`, which returns the `shapeId` of every UpSlide link on the slide, text links included. Resolve as follows:

| Condition | Action |
|---|---|
| Exactly one UpSlide link is selected | Use its `shapeId`. |
| Nothing selected, but the target slide has exactly one UpSlide link | Use that link's `shapeId`. |
| Nothing selected, and the slide has multiple UpSlide links | Show an interactive list (within the same request) and ask the user to pick one. Identify each by what the user can see — content type (chart, table, range), title/name, and position on the slide — never by raw `shapeId`. |
| Nothing selected and the slide has no UpSlide links | Say so explicitly; do not call the tool. |
| Multiple UpSlide links selected | Show the same interactive list and ask the user to pick one. |

Because `go_to_source` acts on a single link, never silently pick "the first one" when the choice is ambiguous — ask instead.

**Text links.** A text link (`isTextLink=true` in `list_links`) is a value from a single Excel cell inside the text of a shape; its `shapeId` is the one of the shape holding the text. When that shape holds several text links, also pass `linkedText`, exactly as `list_links` returned it, for the link the user means. If UpSlide answers that the shape holds several text links and names them, pick the one the user means and call again with its `linkedText`.

## Output

- Always render a text response in chat, even though UpSlide performs the navigation in its UI: confirm which link you're navigating to, then report the outcome.
- Identify the link by human-readable attributes (slide, content type, title/name, position) — never surface internal `shapeId` values as the identifier.
- Render only fields the tool actually returns. Never invent values.
- If the task fails or returns errors, report that plainly.
- Prefix every mention of "UpSlide" in your responses with the 🅤 emoji.
