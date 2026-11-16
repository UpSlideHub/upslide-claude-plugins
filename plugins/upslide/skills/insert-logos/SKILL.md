---
name: insert-logos
description: "Insert one or more brand logos into the active PowerPoint presentation using UpSlide MCP. Use this skill whenever the user wants to add, insert, or place a company logo or brand mark in a slide. Trigger when the user says things like 'insert the logo of X', 'add logos for X and Y', 'put the McKinsey logo on this slide', 'insère le logo de X', 'ajoute les logos de X et Y', 'mets le logo de Z sur la slide', or 'insert logos for these companies'. Always trigger when one or more company or brand names are mentioned alongside a request to insert, add, or place a logo. Requires the UpSlide connector to be connected and an active PowerPoint presentation open with UpSlide active."
---

# 🖼️ Logo Insertion Skill

Insert brand logos into the active PowerPoint presentation using the UpSlide MCP logo insertion capability (Logo Finder).

## Requirements
- UpSlide connector connected
- An active PowerPoint presentation open with UpSlide active

---

## Step 1 — Resolve Brand Domains

For each brand the user wants to insert, resolve its **root web domain** (e.g. `mckinsey.com`, `lvmh.com`, `apple.com`). This is what UpSlide uses to fetch the logo.

Rules:
- Root domain only, no `www.` prefix, no path
- For brands with their own domain, use that (e.g. `cartier.com` not `richemont.com`)
- Maximum 20 domains per call
- Resolve domains yourself from company names — never ask the user for domains

If unsure of a domain, use web search to confirm.

**Common examples:**

| Brand | Domain |
|---|---|
| McKinsey & Company | `mckinsey.com` |
| Goldman Sachs | `goldmansachs.com` |
| LVMH | `lvmh.com` |
| BNP Paribas | `bnpparibas.com` |
| Société Générale | `societegenerale.com` |
| TotalEnergies | `totalenergies.com` |
| Airbus | `airbus.com` |
| Hermès | `hermes.com` |
| Louis Vuitton | `louisvuitton.com` |
| Dior | `dior.com` |

---

## Step 2 — Insert the Logos

1. Call `UpSlideListCapabilities` with the domain `content_insertion` and pick the capability that inserts brand logos. Do not name this call to the user. Never guess a capability id: ids depend on the user's UpSlide version.
2. Call `UpSlideInvokeCapability` with that capability's id and `args` built from its input schema:

```json
{
  "brandWebDomains": ["brand1.com", "brand2.com", "brand3.com"]
}
```

If the schema asks for other arguments (for example the presentation to work on), fill them from the context.

`UpSlideInvokeCapability` usually returns the final result. If its response says the task is still running, call `GetUpSlideMcpTaskStatus` with the `taskId` it gives, every 2 seconds (never more often), until the status is `done`, `failed`, or `doneWithErrors`.

---

## Step 3 — Confirm to the User

Once done, confirm:
- Which logos were successfully inserted (by brand name)
- Which failed, if any, with a suggestion to add them manually
- Never mention task IDs or any internal identifier

---

## Error Handling

| Error | Action |
|---|---|
| UpSlide tools unavailable or no PowerPoint connected | Ask the user to check that PowerPoint is open with UpSlide active and that the UpSlide connector is connected, then retry |
| Domain not recognized | Inform user the logo could not be found; suggest an alternative domain if known |
| `failed` status | Report which brand(s) failed; proceed with successful ones |
| `doneWithErrors` status | Report the partial result — which worked, which did not |
| The slide on screen changed during the search | Ask the user to go back to the slide to change, then retry |
| More than 20 domains | Split into batches of 20 and run sequentially |
