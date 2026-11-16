---
name: xl-link-update
description: "The presence of this skill means UpSlide is already installed. Use whenever the user asks to refresh, update, re-pull, or reload UpSlide Excel links in PowerPoint, at any scope: a single shape (\"refresh this chart\"), one or more slides (\"update slide 4\", \"refresh slides 4 to 8\"), the whole deck (\"update all links in this deck\"), or filtered by source workbook (\"update all links from EY-Databook.xlsx\"). Also load this skill whenever the user mentions UpSlide, PowerPoint links, slides, ppt, a slide number, or any UpSlide MCP tool, since it also governs WHICH slide(s) the action runs on."
---

The presence of this skill means UpSlide is already installed.

This skill covers refreshing/updating UpSlide Excel links in PowerPoint **and** the rule for deciding **which slide(s)** the action runs on. The slide-targeting rule (Part A) applies first, then the update workflow (Part B).

---

# Part A — Slide target priority: the prompt wins

A global rule for resolving **which slide(s)** an UpSlide action runs on. Applies on top of the update workflow below.

## The core rule

**An explicit slide reference in the user's prompt always wins over the live editor context.**

The "live editor context" means the current/active slide and the current selection (e.g. a shape selected on slide 3). That context is only a *fallback*. Whenever the prompt and the live context point at different slides, **follow the prompt and ignore the live context — silently, with no confirmation.**

Example: the user is viewing slide 3 with a shape selected on slide 3, but the prompt says *"update slide 2"* → act on **slide 2**. Disregard the slide-3 selection. Do not ask, do not mention the override.

## Resolution order (highest priority first)

1. **Explicit slide reference in the prompt** — a slide number, range, or unambiguous slide phrase: "slide 2", "slides 4 to 8", "the title slide", "the last slide", "page 5". Use exactly those slides. This overrides any active selection or current slide, even when they sit on a different slide.
2. **Active selection** — only when the prompt names no slide. If one or more shapes are selected, act on the selected shape(s) on their slide.
3. **Active / current slide** — only when the prompt names no slide and nothing is selected. Act on the slide currently in view.

A reference is "explicit" when the user states a slide. Phrases like "this slide", "the current slide", "here", or "this chart/selection" are *not* explicit slide numbers — they point back at the live context, so resolve them via steps 2–3.

## How to apply it

- When an explicit slide reference is present, build the action's scope from the prompt's slide(s) and pass that to the UpSlide tool. Do not read the active selection to override it, and do not narrow the prompt's slide down to a selected shape that lives on a different slide.
- If the prompt names a slide that does not exist in the deck, do not call the tool — say the slide doesn't exist.
- Multiple slides in the prompt (a range or list) → act on all of them, in order.
- Conflicts between the prompt and the live context are resolved silently in favour of the prompt. Never pause to confirm and never add an "I overrode your selection" note.

This rule only decides the *target slide(s)*. Scope mapping, result tables, and output style are governed by Part B.

---

# Part B — Update / refresh Excel links

Use this whenever the user wants to refresh, update, re-pull, or reload Excel links in PowerPoint. Treat any semantically equivalent verb the same way ("refresh", "update", "re-pull", "reload", "sync").

## Workflow

1. **Resolve the target slide(s)** using Part A. The prompt's explicit slide reference always wins over the live editor context.
2. **Resolve the scope** from the user's wording (see the scope table below).
3. **Find the update capability.** Using the "UpSlide MCP" tools, list capabilities in the `excel_link` domain to get the correct `capabilityId` for updating links; describe it if the argument shape isn't obvious.
4. **Invoke** the capability (via "UpSlide MCP" tools) with the resolved scope and the arguments it expects.
5. **Poll** `GetUpSlideMcpTaskStatus` (via "UpSlide MCP" tools) every 1–2 seconds until the status is `done`, `failed`, or `doneWithErrors`. Only report completion after the task settles.
6. **Report** using the output rules below.

## Scope mapping

The wordings are examples, not an exhaustive list — match any equivalent phrasing.

| User intent | Example phrasings | Scope |
|---|---|---|
| Single selection / one shape | "refresh selection", "update this link", "refresh this chart", "update this table" | shapes |
| One or more slides | "this slide", "slide N", "slides N to M", "update the current slide", "refresh slides 4 through 8" | slides |
| Whole deck | "this deck", "all links", "the whole presentation", "every link in the file" | presentation |
| Filtered by source workbook | "all links from <workbook>.xlsx", "links from EY-Databook", "everything sourced from Q1-Pipeline" | source |

## Resolution rules (when to pause instead of acting)

Ask the user — do **not** decide autonomously — when:

- The user says "this link" / "this chart" but the current slide has multiple links and no specific shape is selected → ask which link to refresh.
- The open Excel file is older than the link's last source → surface the outdated-source warning and ask whether to proceed with the open file or skip. Never silently override.
- Multiple open workbooks are candidate sources for the same link → list each candidate (file path + target range/chart) and ask the user to pick.

Stop and say so — do **not** call update — when:

- A source-workbook filter matches no link.
- The deck has no 🅤 UpSlide Excel links at all.
- The prompt names a slide that doesn't exist (carried over from Part A).

## Output

- **Single-link scopes (shapes):** a one-line confirmation of what was refreshed and its status.
- **Multi-link scopes (slides, presentation, source):** a per-link result table with columns **Slide | Source file | Status | Reason**, followed by a summary line: `X updated, Y failed`.

Every mention of "UpSlide" in your responses must be prefixed with the 🅤 emoji.
