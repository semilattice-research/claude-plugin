# Semilattice

Simulate how users respond to product changes, concepts and decisions,
without leaving Claude.

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

### Claude.ai and Cowork

1. Add the Semilattice plugin from the plugin directory.
2. Open the plugin's **Connectors** tab and connect Semilattice.
3. Sign in with your Google email. This creates a Semilattice account if you don't
   have one.

### Claude Code

```
/plugin marketplace add semilattice-research/claude-plugin
/plugin install semilattice@semilattice
```

Then sign in:

1. Run `/mcp`. You should see the Semilattice server listed as disconnected.
2. Select it, press Enter and choose **Authenticate**.
3. Complete the sign-in flow with your Google email. This creates a Semilattice
   account if you don't have one.

If you already added the server by hand with
`claude mcp add --transport http semilattice https://mcp.semilattice.ai`, remove it
with `claude mcp remove semilattice` so Claude sees one set of tools.

## Use

Ask Claude to test something with users, for example "test whether this tagline
lands with UK developers" or "plan simulations for the claims in this PRD". The skill
loads, proposes a plan and its cost in time, and asks before running anything.

## Data

The plugin itself stores nothing. Everything it does goes through the Semilattice MCP
server at `mcp.semilattice.ai`, which receives:

- **Simulation inputs:** the goals, concepts, option lists and context Claude writes
  into each simulation, which can include text drawn from documents you share with
  Claude.
- **Page URLs** for User Journey runs, which Semilattice visits to test the flow.
- **Your account identity**, through the OAuth sign-in.

Simulations and their results are saved to your Semilattice account.

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

## Licence

MIT. See [LICENSE](LICENSE).
