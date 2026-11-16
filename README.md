# UpSlide plugins for Claude

This repository is a Claude plugin marketplace. It holds the **`upslide`** plugin, which gives Claude
everything it needs to work with UpSlide in PowerPoint and Excel:

- the **UpSlide connector** (MCP server `https://api.upslide.com/mcp`),
- the **UpSlide skills**, which teach Claude how to use it for common tasks.

Install the plugin once: the connector and every skill come together, and new versions reach you
through plugin updates. You no longer download skill files one by one.

## Where it works

The UpSlide skills act on the document open in **Claude for PowerPoint** and **Claude for Excel**
(Claude for Microsoft 365), with the UpSlide add-in installed. A plugin is saved to your Claude
account, so you install it once, from the Claude desktop app or claude.ai, and it is available in
the PowerPoint and Excel add-ins.

## Install — one user

1. In Claude, open **Customize > Plugins > Add marketplace**.
2. Paste `https://github.com/UpSlideHub/upslide-claude-plugins`.
3. Install the **upslide** plugin.
4. Open **Customize > Connectors**, select **UpSlide** and click **Connect**, then sign in with your
   UpSlide account.

In Claude Code, the same install is one command:

```
/plugin install upslide --marketplace UpSlideHub/upslide-claude-plugins
```

## Install — a whole organisation (Team and Enterprise plans)

An Owner of the Claude organisation:

1. Opens **Organization settings > Plugins & skills > Add > Sync from GitHub** and syncs
   `https://github.com/UpSlideHub/upslide-claude-plugins`.
2. Sets the **upslide** plugin to *Installed by default* or *Required*.
3. Adds the UpSlide connector in **Organization settings > Connectors**.

Each member then connects the UpSlide connector with their own UpSlide account
(**Customize > Connectors > UpSlide > Connect**).

If the Owner has turned off *User-created skills*, members cannot add the marketplace themselves:
the Owner path above is the only one.

## Updates

When a skill changes, UpSlide publishes a new plugin version in this repository. Installed plugins
pick it up through Claude's plugin updates: there is nothing to download again.

## Repository layout

```
.claude-plugin/marketplace.json        the marketplace: lists the plugins in this repository
plugins/upslide/
├── .claude-plugin/plugin.json         plugin name, version and description
├── .mcp.json                          the UpSlide connector
└── skills/<skill-name>/SKILL.md       one folder per skill
```

## For maintainers

- Every change to a skill or to the connector must **bump `version`** in
  `plugins/upslide/.claude-plugin/plugin.json`: installed plugins only update when the version
  changes.
- A skill's folder name and its `name` field are the slash command users type (for example
  `/insert-logos`). Do not rename a published skill.
- Skills must not hard-code UpSlide capability ids: the catalogue depends on the user's add-in
  version. They name the domain and let Claude find the capability with `UpSlideListCapabilities`.
- Check the structure before pushing: `claude plugin validate .`
