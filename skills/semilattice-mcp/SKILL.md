---
name: "semilattice-mcp"
description: "Plan and run audience simulations through the Semilattice MCP. Use when someone wants to predict how a modelled audience responds to a product, feature, price, message or live page — decomposing a PRD, spec or idea into testable claims, choosing goals and user models, pricing the whole job, running the slice they pick, and reading the results back."
---

# Semilattice MCP

Semilattice predicts how a modelled audience answers. Your job is to turn what the
user wants to learn into simulations, tell them what the whole job costs, run the
part they choose, and report what came back.

Treat the input as a set of claims about users and compile it into predictions.
Every claim can be tested. Price the full job, plan the slice the user wants, keep
the rest visible.

## What the pieces are

**Project** — a container. Call `create_project`, then pass its `project_id` to
every `create_simulation`. Simulations join the project's latest round, and
`get_project` reads the rounds plus the synthesis of any round that has finished.
Create one whenever the work needs more than a single simulation.

**Simulation** — one goal, its inputs, and one or more user models. Two modes:
`research` (survey-style, called **User Research** when talking to the user) and
`journey` (a browser agent against a live URL, called **User Journey**). Both are
prediction engines; plan with equal confidence in either.

**User model** — an audience. `list_user_models` returns ids, accuracy figures,
`simulacrum_count` and `simulation_engine`.

**Engines** — `anacreon` and `answers-1` are the two engines behind the user models,
and they pace a run differently. An `anacreon` simulation takes roughly 15 minutes
whatever its length; `answers-1` answers 4 questions at a time at about 30 seconds
each, so its wait scales with the question count. An audience is covered by one
engine or the other, never both, so the engine is not a choice you make: it follows
from which model matches the audience. `list_user_models` reports each model's own
accuracy.

**Claim** — any assertion or assumption about users that the input makes or depends
on. A claim with variants (five headlines, three price points) is one claim
carrying an option set.

## How a simulation is built

You choose a goal, fill its inputs, and pick models. A pipeline then writes research
questions from your inputs, and survey questions from those. **The inputs are your
control surface**: put everything the question writer needs into them — the full
option list, the concept as it should be described to respondents, the context that
makes the question sharp.

`update_simulation_plan` can rewrite the research question, the hypothesis and the
whole survey question list while status is `planned`. Use it to repair a plan, not
to author one: if the *inputs* were wrong, create a replacement simulation rather
than rewriting every question by hand. Passing `survey_questions` replaces the
entire list. There is no delete over MCP, so say plainly that the abandoned
simulation is still in the project for the user to remove in the web app.

Mechanics every plan has to respect:

- Every survey question is an independent, single-choice, Likert-scale item.
- The whole population answers every question. No conditional logic, no subgroups,
  no cross-tabulation, no rankings, no open text.
- Comparing N options means N items scored on the same scale and compared
  afterwards. Never call this head-to-head or forced-choice when describing it —
  say "compares" or "scores against each other".
- Respondents have no lived history with the product. Claims are tested as
  reactions to described concepts and as stated intent, never as reports of past
  usage. "Users are satisfied with X" compiles to "does X as described appeal", and
  you should say so.
- Respondents are never asked to role-play another audience. A claim about audience
  B needs a model of B, and a claim about a subgroup needs a model of that
  subgroup — questions cannot filter the population.

## Planning: decompose, type, model, pack, price

Five steps, in order, before any tool call.

### Decompose — turn the input into claims

List every claim the input makes. A PRD is a bundle of claims; a sentence is a
smaller bundle.

Include the implicit ones: that users want the category at all, that they read the
notification, that the named competitor is the real alternative. These are often the
riskiest claims precisely because nobody wrote them down.

Turn decisions into option sets. Where the input has committed to one option and
plausible alternatives exist, the claim is "the committed option beats the
alternatives", and the option set is the committed option plus the alternatives it
displaced — only options someone would actually ship, never a straw option added to
pad a set. A document that has decided things is a configuration: list the decisions
it made and, for each, the options it did not take.

**Decisions are separable by default.** Test each one's options as its own claim,
and the best option per decision composes into the best configuration. Plan a
combination test only where two decisions plausibly change each other's answer —
because one alters the stakes of the other, or both express the same underlying
concern. A combination test is an ordinary option set whose options are the
combinations, so three autonomy levels across two tasks is one claim carrying six
options and costs the product of the two counts. Name the reason whenever you
propose one.

Prefer claims that vary a decision over claims that describe a topic. Where the user
names a target outcome, work backwards: the claims are the configurations that could
produce it.

When the input is a direction rather than a document ("let's explore whether…"),
generating the claims the user has not stated is part of the work, and your first
response is an assumption map rather than a plan. Sort what the idea rests on into
three groups and say them: what you can simulate (never ask the user these — testing
them is the product), what only the user can answer, and what you will hold fixed in
the stimulus with its alternatives backlogged. Plan after the map, not instead of it.

### Type — decide how each claim is tested

Attitudinal claims — would they, do they prefer, do they care — become `research`
simulations.

Behavioural claims — can they, do they, where do they stall — become `journey`
simulations when a live URL exists in what the user gave you. With no live URL, test
the attitudinal version in research mode and say that is what you did.

Call `list_simulation_schema(mode, goal)` before creating a simulation with a goal
whose inputs you have not seen.

### Model — decide who answers, per claim

Match the audience, not the engine. A model built for the user's organisation was
made from their own data: use it for every claim about their audience. Reach for a
public model only for an audience their own models do not cover, and where several
public models cover that audience, prefer the one with the higher reported accuracy
for the mode you are running (`accuracy_research` or `accuracy_journey`).

One model per claim about one audience. Do not fan such a claim out across several
models to triangulate, and do not plan a pilot on one engine followed by a rerun on
the other — the engines never cover the same audience. The one multi-model case is a
claim about how audiences differ ("US buyers behave like UK ones"): there the same
simulation runs on each audience and the answers are compared.

When a claim is really one audience's belief about another audience, say so, and
plan the direct test on whichever model covers the second audience. If no model
covers them, record it as a limit of the work. Role-playing is not a workaround.

### Pack — fit claims into simulations

One research simulation holds one coherent enquiry: the claims that can be stated
together as a few research questions. Start a new simulation where the subject
changes, not where a question count runs out.

An option set of any size stays in one simulation. Never silently test a subset of
an option list.

Goals that take a single subject (`validate_concept`, `assess_change_risk`) compare
nothing. To compare alternatives, restructure into a list-taking goal
(`test_messaging`, `prioritise_features`), plan one simulation per alternative, or
use `something_else`.

One journey simulation tests one claim.

**A stimulus commits you.** When a question has to describe the concept to
respondents, every detail in that description — price, cadence, length, format —
becomes a tested claim, but only in the form described. The headline result is
conditional on that exact configuration, and you say so. Details the user never
decided go into the stimulus deliberately rather than by improvisation, and the
backlog entry for them reads "the alternatives to X", never "X untested". X was
tested, in one form.

### Price — state the budget, let the user set it

After decomposing you know the full compile: N testable claims, M simulations,
roughly T minutes. Say it in one sentence whenever the input contains more than the
user asked about: "this PRD contains 14 testable claims; all of it is about 45
minutes."

Wait classes, stated whenever you propose a plan: interactive (`answers-1`, a couple
of minutes), considered (`anacreon` simulations in parallel, about 15 minutes),
extended (journey simulations, long question lists, or sequential batches — up to an
hour or two). Nothing you plan should take a day. "Quick check" is a tempo
instruction: keep the plan small and say what you left out, and never switch to a
model the audience does not match.

Then plan the slice the user chose. Everything unplanned stays visible: close with
the untested claims as a short list, one line each, that they can pick from later.

## What to ask before you create anything

Four things. Usually they are stated or safely inferable — ask about exactly the ones
that are not, with short enumerated options where the choice is enumerable. There is
no fixed limit on questions, but never ask about something already in front of you,
and never ask the user something a simulation can answer.

- **Scope.** When the decomposed surface is meaningfully larger than the obvious
  opening move — whether it came from a substantial artefact or from a one-line
  direction — always ask before creating: everything, the riskiest few, or one area.
  Never silently default to a subset. "Plan this from the attached PRD" does not
  name a slice.
- **Tempo.** When the honest plan has a genuinely slow route, offer the choice: the
  full version and its wait, or the smaller faster version and what it leaves out.
- **Referent.** When the user's words do not match the input — "test the launch
  plan", and the document contains no launch plan — say what you cannot find and ask
  what they mean. Never guess a referent into existence.
- **Intent.** When the input is a direction and the design forks on why they want
  the thing — a lead-generation podcast is tested on buyers with credibility probes,
  a brand podcast on a wider audience without them — ask the fork.

A short transactional request carrying a clear question ("test whether this tagline
lands with UK developers") gets one sharp simulation and no questions at all.

## The call sequence

If the Semilattice tools are not available in this conversation, the MCP server is
not connected. Say that and give the user both the server URL,
`https://mcp.semilattice.ai`, and the setup guide at
`https://docs.semilattice.ai/mcp/connect`, which has the steps for each client and
the list of clients that cannot connect at all. Stop there and do not plan
simulations you cannot run.

1. `list_user_models`, and `list_simulation_schema(mode)` for the goal list. Call
   `list_simulation_schema(mode, goal)` for the input keys of every goal you have
   not used before, all in one response.
2. `create_project` when the work needs more than one simulation. Pass its
   `project_id` to every `create_simulation` so they stay together.
3. `create_simulation` for each planned simulation, batched into one response rather
   than one call per turn.
4. Each comes back `status='planning'`. Poll `get_simulation`; planning usually
   takes 10 to 20 seconds.
5. At `planned`, show the user the generated plan and get their confirmation before
   running. If they ask to see the questions, or their words imply it, list every
   survey question in full, in order, with its answer options. Do not sample,
   summarise or truncate.
6. `update_simulation_plan` for the edits they ask for, and nothing else.
7. `run_simulation`, once per simulation, each with an explicit yes for that
   simulation. It spends something the user owns. Never read a yes into an ambiguous
   request. **Start every run in the batch before you wait on any of them.** A
   round's synthesis is written once, when the first batch of its simulations all
   reach `completed`; anything you run after that lands in a round whose synthesis
   has already been written and will not be rewritten.
8. Poll `get_simulation`: wait about `estimated_seconds` before the first call, then
   at most once a minute. Polling more often does not make a run finish sooner.
9. At `completed`, read `synthesis`, `markdown`, and the per-question
   `answer_distribution` (research) or `simulation_results` (journey).

Journey plans include `url_checks`, a pre-flight check of each test URL. If any
check is not `ok` — unreachable, bot_blocked, login_wall — tell the user before
running, because the simulation will likely fail on that URL.

`share_simulation` makes a simulation viewable by anyone with the link. Ask first,
every time, and never as part of a batch of other work.

If `create_simulation` returns an error, do not retry the same call more than once.

## Reading results back

Read the data before you answer. Never quote a number you have not seen in a result,
and never answer from your own earlier summary when `get_simulation` is one call
away.

Report each claim against the option set it was tested with, and say what the result
is conditional on: the audience it ran against, and the stimulus exactly as
described. Where you tested decisions separately, the composed winner — the best
option from each decision, described as a single concept — is a prediction rather
than a finding. Say so, and offer it as the next simulation. If it scores as the
parts predicted, the decisions were separable; if it scores below them, the gap is
where they interact.

When a new version of the document arrives, diff it against what was tested: plan
against the claims that changed and the claims the last results left uncertain, not
the document from scratch. An option that lost does not return to the option set.

## What this MCP does not do

Say so plainly rather than working around it.

- **No artefact upload.** The document lives in your conversation, so what a
  simulation needs from *it* has to reach the simulation through the goal's inputs.
  The organisation's own context is different: it is attached server-side when a
  simulation is created, inheriting the project's. Do not restate who the company
  is, what it sells or who its audience is in your inputs — that arrives on its
  own, and repeating it only crowds out the detail the question writer needs.
- **No round control.** Simulations join the project's latest round. `get_project`
  reads every round and the synthesis each one reached, but you cannot start a
  round, retitle one, or regenerate a synthesis. A project's whole life over MCP
  happens in one round, so plan the work you want synthesised together, run it
  together, and tell the user that a genuinely new phase wants a new project.
- **No chat history.** You cannot read what the user and the in-app planner said to
  each other, or write into that conversation. If the user has been working in the
  web app, ask them what changed rather than assuming the project tells you.
- **No respondent-level reads.** `get_simulation` returns distributions and the
  synthesis, not how individual respondents answered.
- **No cross-tabulation, subgroup filtering, ranking or open text.** See the
  mechanics above; these are properties of the survey, not gaps in the API.

## Vocabulary

A **user model** is an audience; the simulated people inside one are **simulacra**
or **respondents**. A **round** is one phase of the work — its simulations and one
synthesis. An **artefact** is a document attached to a round in the web app. A
**synthesis** is what a finished round concluded. `anacreon` and `answers-1` are the
two simulation engines. The modes are **User Research** and **User Journey** when
you name them to the user. These are Semilattice's words; do not substitute another
product's.
