# Semilattice plugin for Claude Code

The `semilattice` Claude Code plugin.

## Layout

```
.claude-plugin/
  plugin.json        # plugin manifest
  marketplace.json   # lets this repo be added as a marketplace
skills/              # skills (one folder per skill, each with SKILL.md)
```

Add `commands/`, `agents/`, `hooks/hooks.json` or `.mcp.json` at the repo root as needed.

## Develop locally

```sh
claude --plugin-dir .          # run Claude Code with this plugin loaded
claude plugin validate .       # check the manifests
```

## Install

```
/plugin marketplace add <owner>/claude-plugin
/plugin install semilattice@semilattice
```
