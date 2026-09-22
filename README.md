# Niklas Plugins

Personal plugin marketplace for ChatGPT and Codex. The included plugin is a
skills-only plugin for AQC consultant work, with the `uppdrag` skill for
structuring and managing assignments.

## Included plugin

- Plugin: `aqc-konsult-hantering`
- Marketplace: `aqc-plugins`
- Skill: `uppdrag`
- MCP server: none

## Install on ChatGPT desktop

The desktop app reads plugins from marketplaces registered locally with the
Codex CLI. The marketplace is cached on each computer, so installation must be
done separately on every computer.

### Install from this local folder

From the root of this repository:

```sh
codex plugin marketplace add .
codex plugin add aqc-konsult-hantering --marketplace aqc-plugins
```

### Install from the public GitHub repository

On another computer:

```sh
codex plugin marketplace add NiklasAbrahamsson/aqc-chatgpt-plugins
codex plugin add aqc-konsult-hantering --marketplace aqc-plugins
```

Because the repository is public, GitHub login is not required to install it.
GitHub login is only needed for pushing changes. A private repository would
require Git credentials on every computer that installs it.

### Check the installation

```sh
codex plugin marketplace list
codex plugin list
```

Restart the ChatGPT desktop app after installing or updating the plugin.

If `codex plugin` is not recognized, update the Codex CLI:

```sh
npm install -g @openai/codex@latest
```

## Update the plugin

After changes are pushed to GitHub, refresh the marketplace on each computer:

```sh
codex plugin marketplace upgrade aqc-plugins
```

Restart the desktop app afterward. When publishing a new plugin version, bump
the `version` field in
`plugins/aqc-konsult-hantering/plugin.json` before pushing.

## Use it in ChatGPT web

A local marketplace, even when its GitHub repository is public, is not
automatically available in the ChatGPT browser version. There are two browser
options:

### Publish to a workspace

This requires a Business or Enterprise workspace where you are an admin:

1. Open ChatGPT and go to the workspace's Plugins area.
2. Select **Personal**.
3. Find `AQC konsult-hantering` and open its three-dot menu.
4. Select **Publish**.

After publishing, the plugin is available to members of that workspace on the
web and on other devices. The exact menu labels may vary as the ChatGPT UI is
updated.

### Submit to the public Plugins Directory

If the plugin should be available outside your workspace, submit it to the
public Plugins Directory for OpenAI review. A GitHub marketplace repository by
itself is not a public-directory submission.

This plugin has no MCP server, so it does not need a public HTTPS endpoint.
Likewise, it contains no hooks that need to be deployed separately.

## Publish changes to GitHub

From the repository root:

```sh
git add .
git commit -m "Update AQC consultant plugin"
git push
```

The repository is public at:

https://github.com/NiklasAbrahamsson/aqc-chatgpt-plugins
