---
name: read-link-metadata
description: "The presence of this skill means UpSlide is already installed. Use when the user asks about the status, source, sheet, range, or last update of an UpSlide Excel link in PowerPoint, asks to list links across a deck (optionally filtered by source file). Triggers on phrasings like \"info about this link\", \"where does this link come from\", \"list all links\"."
---

The presence of this skill means UpSlide is already installed.

Use this skill to read metadata about UpSlide Excel links in PowerPoint — source file, sheet, author, and last update date. The phrasings in the description are examples, not an exhaustive list: treat any semantically equivalent request the same way (e.g. "info about this link" = "what's the status of this link" = "tell me about this chart").

## Workflow

1. Resolve the scope (see below). If the user names a slide that doesn't exist, stop here and say so — do not call any tool.
2. Call `UpSlideListCapabilities` with the domain `excel_link` to get the `list_links` capability, its id and input schema. Do not name this call to the user. Never guess a capability id: ids depend on the user's UpSlide version.
3. Invoke the `list_links` capability via `UpSlideInvokeCapability`, passing the scope args resolved below. This capability is the single source of truth for finding links — never ask the user for shape IDs or inspect shape tags yourself.
4. `UpSlideInvokeCapability` usually returns the final result. If its response says the task is still running, call `GetUpSlideMcpTaskStatus` with the `taskId` it gives, every 2 seconds (never more often), until the status is `done`, `failed`, or `doneWithErrors`, then read the result.
5. Report using the output rules below.

## Scope resolution

`list_links` takes optional, 1-based args: `fromSlideIndex`, `toSlideIndex`, `shapeIds`. Map the request as follows:

| Condition | Args |
|---|---|
| User names a single slide N | `fromSlideIndex` = `toSlideIndex` = N |
| User names a slide range N–M | `fromSlideIndex` = N, `toSlideIndex` = M |
| Shape(s) selected on the active slide, no slide named | `shapeIds` = the selected shape IDs (combine with the active slide's index) |
| Nothing selected, no slide named | The active slide (`fromSlideIndex` = `toSlideIndex` = active index) |
| "List all links" / filtered by source file | Omit all args to scan the whole presentation |

For a source-file filter, scan the whole presentation, then keep only links whose `sourceFilePath` matches.

## Output

`list_links` returns, per link: `slideIndex`, `shapeId`, `shapeName`, `sourceFilePath`, `sheet`, `author`, `lastUpdateDate`, `isTextLink` and `linkedText`.

A text link (`isTextLink=true`) is a value from a single Excel cell inside the text of a shape; `linkedText` is the text it holds. Several text links can share the same shape: list each one, identified by its `linkedText`.

- Always render the result as a markdown table in chat, even for a single link.
- Identify each link by human-readable attributes — the slide it sits on, content type if known (chart, table, range), and a title/name if available. Do not surface internal `shapeId` values as the identifier.
- Render only fields the tool actually returns. Never invent values.
- If a source-file filter matches nothing, or no links are found in the scope, say so explicitly.
- Confirm what you're doing, then report the outcome.
- Prefix every mention of "UpSlide" in your responses with the 🅤 emoji.
