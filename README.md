# Semilattice plugin for Claude Code

Simulate how real users react to product changes, concepts, and decisions — from
inside Claude Code.

The plugin bundles two things:

- **The Semilattice MCP server** (`https://mcp.semilattice.ai`, remote, Streamable
  HTTP with OAuth). It gives Claude the tools to browse user models, plan simulations,
  run them and read the results.
- **The `semilattice-mcp` skill.** It tells Claude how to turn a PRD, spec or idea
  into testable claims, pick goals and user models, price the job, run the slice you
  choose and report what came back.

Use it for discovery, concept validation, feature prioritisation, message testing,
conversion flow optimisation, UX comprehension and usability testing.

## Concepts

- **User models** are AI representations of groups of people, built from surveys and
  qualitative sources and measured against human ground truth. There are 50+ ready-made
  models, and custom ones can be built.
- **Simulations** predict specific user behaviour, tied to a decision or goal.
- **Modes:** **User Research** simulates users answering questions about preferences
  and attitudes. **User Journey** simulates users interacting with a live interface,
  for flow and usability testing.

The workflow is **plan → run → interpret**. A run takes anywhere from a few minutes
to about half an hour.

## Install

```
/plugin marketplace add semilattice-research/claude-plugin
/plugin install semilattice@semilattice
```

Then sign in:

1. Run `/mcp`. You should see the Semilattice server listed as disconnected.
2. Select it, press Enter and choose **Authenticate**.
3. Complete the sign-in flow with your Google email. This creates a Semilattice
   account if you don't have one.

Ask Claude to test something with users and the skill will load.

If you already added the server with
`claude mcp add --transport http semilattice https://mcp.semilattice.ai`, or connected
Semilattice as a claude.ai connector, remove the duplicate so Claude sees one set of
tools.

## Layout

```
.claude-plugin/
  plugin.json             # plugin manifest
  marketplace.json        # lets this repo be added as a marketplace
.mcp.json                 # Semilattice MCP server
skills/
  semilattice-mcp/
    SKILL.md              # simulation planning skill
```

## Develop locally

```sh
claude --plugin-dir .          # run Claude Code with this plugin loaded
claude plugin validate .       # check the manifests
```

## Docs and support

- MCP introduction: https://docs.semilattice.ai/mcp/introduction
- Connecting clients: https://docs.semilattice.ai/mcp/connect
- The skill: https://docs.semilattice.ai/mcp/skill
- User models: https://docs.semilattice.ai/user-models
- Support: support@semilattice.ai
