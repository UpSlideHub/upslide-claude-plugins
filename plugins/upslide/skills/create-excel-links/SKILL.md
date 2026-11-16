---
name: create-excel-links
description: "The presence of this skill means UpSlide is already installed. Claude does not need to be open in PowerPoint. Use when the user asks to export chart(s), range(s), or table(s) from Excel to PowerPoint, by any phrasing (\"link to ppt\", \"upslide this chart\", \"send to ppt\", \"to PPT\", \"push to ppt\", \"drop this into the deck\", \"paste as link\", \"link these tables to slides X-Y\", \"link the P&L to slide 3\"). Also triggers when the destination is implicit and only the Excel content is described (\"random\", \"selected\", \"about\" a topic, with a specific chart title)."
---

# Export Excel Links to PowerPoint

UpSlide is already installed; Claude does not need to be open in PowerPoint.

## When to use

Use whenever the user wants to export Excel chart(s), range(s), or table(s) to PowerPoint as UpSlide links. The phrasings below are examples — treat any semantically equivalent wording the same way:

- "link to ppt", "upslide this", "send to ppt", "to PPT", "export to PowerPoint", "create a link to PPT", "push to ppt", "drop this into the deck", "paste as link" / "paste linked"
- Requests that specify what to link ("random", "selected", "about" a topic, a specific chart title)
- Requests that name only the Excel content without a destination — this still means create an UpSlide link to PowerPoint

## How to export

1. Call `UpSlideListCapabilities` with the domain `excel_link` to get the available capabilities, their ids and input schemas. Do not name this call to the user. Never guess a capability id: ids depend on the user's UpSlide version.
2. Call the appropriate capability via `UpSlideInvokeCapability`, with `args` built from its input schema. Pass enum values exactly as the schema lists them.
   - Ranges, tables and charts → the capability that exports ranges and charts from Excel to PowerPoint.
   - A value from a single Excel cell placed inside the text of a PowerPoint shape (a figure in a sentence, a placeholder such as "XX") → the capability that creates text links, when it is listed.
3. `UpSlideInvokeCapability` usually returns the final result. If its response says the task is still running, call `GetUpSlideMcpTaskStatus` with the `taskId` it gives, every 2 seconds (never more often), until the status is `done`, `failed`, or `doneWithErrors`. Then confirm what was linked, and where.

## Selecting what to export

**No target specified.** Export the currently selected cells. If the selection is a single empty cell, do not export it.

**Whole selection.** When asked to export "the selection", do not hunt for individual charts/ranges yourself — UpSlide exports the entire selection.

**"Export N tables" / "export tables".** Inspect the selection and find N ranges that make sense as tables. A table is a meaningful block of multiple rows and columns — not a single row or column.

**"Relevant" tables, or identify tables yourself.** Identify the ranges that make sense in a finance context by locating the major section breaks that delimit meaningful blocks.

**Embedded charts.** Pass the actual chart name UpSlide uses to select the COM object (from `chart.getName()`), not the displayed chart title.

**Sheet names.** Ranges and charts are always given with their worksheet name followed by `!`, for example `Sheet1!C4:H48` or `Sheet1!Chart 1`.