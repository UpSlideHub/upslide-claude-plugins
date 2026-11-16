---
name: upslide-logo-insertion
description: "Insert one or more brand logos into the active PowerPoint presentation using UpSlide MCP. Use this skill whenever the user wants to add, insert, or place a company logo or brand mark in a slide. Trigger when the user says things like 'insert the logo of X', 'add logos for X and Y', 'put the McKinsey logo on this slide', 'insère le logo de X', 'ajoute les logos de X et Y', 'mets le logo de Z sur la slide', or 'insert logos for these companies'. Always trigger when one or more company or brand names are mentioned alongside a request to insert, add, or place a logo. Requires UpSlide MCP - Develop to be connected and an active PowerPoint presentation open with UpSlide active."
---

# 🖼️ Logo Insertion Skill

Insert brand logos into the active PowerPoint presentation using the UpSlide MCP `insert_logos` capability.

## Requirements
- UpSlide MCP connected (`UpSlide MCP - Develop`)
- An active PowerPoint presentation open with UpSlide active

---

Before to start call UpSlide MCP to find the session ID by your own.

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

Call `UpSlide MCP - Develop:InvokeCapability` with `capabilityId: "insert_logos"`:

```json
{
  "capabilityId": "insert_logos",
  "upSlideSessionId": "<session_id_from_context>",
  "args": {
    "brandWebDomains": ["brand1.com", "brand2.com", "brand3.com"]
  }
}
```

This tool runs **asynchronously**. After invoking, poll `GetUpSlideMcpTaskStatus` every 1-2 seconds until status is `"Done"`, `"Failed"`, or `"DoneWithErrors"`:

```json
{
  "upSlideSessionId": "<session_id_from_context>"
}
```

---

## Step 3 — Confirm to the User

Once done, confirm:
- Which logos were successfully inserted (by brand name)
- Which failed, if any, with a suggestion to add them manually
- Never mention the session ID or any internal token

---

## Error Handling

| Error | Action |
|---|---|
| Session ID not found in context | Ask user to click "Connect to MCP" in the UpSlide ribbon, then retry |
| Domain not recognized | Inform user the logo could not be found; suggest an alternative domain if known |
| `"Failed"` status | Report which brand(s) failed; proceed with successful ones |
| `"DoneWithErrors"` status | Report the partial result — which worked, which did not |
| More than 20 domains | Split into batches of 20 and run sequentially |