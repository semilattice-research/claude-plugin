---
name: "semilattice-mcp"
description: "Plan and run audience simulations through the Semilattice MCP. Use when someone wants to predict how a modelled audience responds to a product, feature, price, message or live page — decomposing a PRD, spec or idea into testable claims, choosing goals, user models and segments, searching generated options for the best one, pricing the whole job, running the slice they pick, and reading the results back. Also use when someone asks for user or customer feedback, user testing, a survey, usability testing or market research; asks which headline, name, price, message or feature people prefer; or wants to know whether people would sign up, pay, switch or understand something."
---

# Semilattice MCP

Semilattice predicts how a modelled audience answers. Your job is to turn what the
user wants to learn into simulations, tell them what the whole job costs, run the
part they choose, and report what came back.

Treat the input as a set of claims about users and compile it into predictions.
Every claim can be tested. Price the full job, plan the slice the user wants, keep
the rest visible.

## What the pieces are

**Context** — the briefing on the user's whole area of work: their company, or
the product or area they own, or for an agency the client. Every project reads the
same one, and a project's own question, feature or hypothesis never goes in it.
`get_context` reads it. When the user has none, write one with them before you
plan anything: draft it with `create_context` from what they tell you and their
website or documents, and show them the draft before saving.

**Project** — a container. Call `create_project`, then pass its `project_id` to
every `create_simulation`. Create one whenever the work needs more than a single
simulation. Check `list_projects` first when the work may belong with a project
the user already has.

**Round** — one phase of a project: the artefact it tests, its simulations, and one
synthesis across them. Start one with `start_round` for a new version of a document
or a deep dive on one part of it, and attach what it tests with `attach_artefact`.
Keep adding to the round you are in while the same exploration continues. There is
no current round: pass `round_id` to `create_simulation`, or the simulation joins
the latest round. `get_project` reads every round, what it tests, and its synthesis.
`rename_round` sets a round's title, and a title you set is kept.

**Simulation** — one goal, its inputs, and one or more user models. Two modes:
`research` (survey-style, called **User Research** when talking to the user) and
`journey` (a browser agent against a live URL, called **User Journey**). Both are
prediction engines; plan with equal confidence in either.

**User model** — an audience. `list_user_models` returns ids, accuracy figures,
`simulacrum_count`, `simulation_engine` and `owned_by_org`. When the account has
no user models it returns a message instead: pass it on and do not plan
simulations that cannot run.

**Segment** — a user model cut from another by filtering its simulacra's
attributes, such as the 45-to-64-year-olds in a UK model. `get_user_model_attributes`
lists the attributes and their exact values; `create_segment` makes one, and it
then runs like any other user model, with its parent's accuracy. Only `anacreon`
models can be segmented, a segment needs at least 100 simulated users, and its
filters lock once a simulation has used it. Show the user the name and filters
before creating one.

**Engines** — `anacreon` runs every public user model; some organisations' older
models of their own run on `answers-1`. They pace a run differently. An `anacreon`
simulation takes roughly 15 minutes whatever its length; `answers-1` answers 4
questions at a time at about 30 seconds each, so its wait scales with the question
count. An audience is covered by one engine or the other, never both, so the engine
is not a choice you make: it follows from which model matches the audience.
`list_user_models` reports each model's engine and its own accuracy.

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
survey questions while status is `planned`. Use it to repair a plan, not to author
one: if the *inputs* were wrong, create a replacement simulation rather than
rewriting every question by hand. Change one question with `question_edits` (by
id), append with `add_questions`, drop with `delete_question_ids`: every question
you leave alone keeps its id. Passing `survey_questions` instead replaces the
entire list and reissues every id, so re-read the simulation before editing again.
The two cannot be combined in one call. For a user model in another language,
send `question_text_local` and `answer_options_local` with each question, since
that wording is what respondents are asked. A journey task's fields follow its
goal's `journey_task_shape` in `list_simulation_schema`: goal-directed tasks carry
an objective and success criteria, observation tasks an objective and a debrief
question. To remove the abandoned simulation, name it and ask the user, then call
`delete_simulation` with `confirmed=true`.

To compare a set of things — features, messages, prices, concepts — pass
`survey_design` to `create_simulation`. An `item_sets` entry gives each item its own
question on one shared scale, so the items can be compared; `custom_questions` are
asked word for word, so each must name what it asks about. The plan then holds
exactly one question per item plus one per custom question. If the question writer
returns a different count, the simulation ends `failed` with a `failure_reason`:
create it again. Without `survey_design` the writer may fold the options into one
"which of these" question that cannot be compared.

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
  B needs a model of B, and a claim about a subgroup needs a segment of the model
  that covers it — questions cannot filter the population.
- A simulation holds at most 40 survey questions. A larger option set is split
  across simulations of up to 40 items each.

## Choosing a strategy

Pick the narrowest strategy the user's words support, then plan it with the steps
below.

- **Single check.** One stimulus, one audience, one simulation. "Does this tagline
  land with UK developers?"
- **Compare.** Score the options the user gave against each other. "Which of these
  five headlines works best?"
- **Decompose.** Turn a document into claims and its decisions into option sets,
  price the whole job, and run the slice the user picks. "Plan simulations for
  this PRD."
- **Explore.** A direction with no document: map the assumptions first, then any
  strategy above. "Let's explore whether a podcast would work for us."
- **Widen.** One round of generated alternatives scored alongside the user's own.
  "Is there anything better than these?"
- **Search.** Generate, score, keep the winners, and generate again from them,
  over several rounds, then confirm the result. "Find the best headline."
  "Optimise this message." "Keep going until it stops improving."
- **Revise.** A new version of something already tested: test what changed and
  what the last results left uncertain (see **Reading results back**).

A simulation costs minutes, not a recruited panel, so the size of an option set is
a choice, not a given. Whenever a claim asks which of something is best and more
options could be written — headlines, names, messages, price points, bundles —
name the next strategy up in one sentence with its price, and let the user take
it or leave it. Words like "best", "optimise", "strongest" or "as many as it
takes" ask for Search directly: state its budget before anything runs.

### How Search runs

1. Generate variants along named dimensions — angle, benefit, tone, length,
   proof — not paraphrases of one idea. Each must be something the user could
   ship.
2. Score them in batches of up to 40 items on the same scale and stimulus. Put the
   best option so far in every batch, so batches can be compared through it.
3. Keep the top few. Generate the next round from what won, along the dimensions
   that won. An option that lost does not return.
4. Stop when a round's best no longer clearly beats the previous best, or the
   budget runs out.
5. Confirm: score the finalists together in one fresh simulation. The best of
   many scores is partly noise, so report the confirmed scores, not the round
   ones.

Price the whole search up front — rounds, simulations, hours — and keep it in one
project round. One yes to that budget covers every simulation inside it, so you do
not stop to confirm each batch. Ask again before going past it.

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
pad a set. Generated options under Widen or Search meet the same bar. A document
that has decided things is a configuration: list the decisions it made and, for
each, the options it did not take.

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
simulation runs on each audience and the answers are compared. Groups inside one
audience — age bands, regions — are compared as segments of the same model.

A claim about a subgroup of an audience runs on a segment of the model that covers
it. Reuse a segment from `list_user_models` when one matches; otherwise cut one
with `create_segment`. When the filters match fewer than 100 simulated users,
widen them with the user's agreement or record it as a limit of the work.

When a claim is really one audience's belief about another audience, say so, and
plan the direct test on whichever model covers the second audience. If no model
covers them, record it as a limit of the work. Role-playing is not a workaround.

### Pack — fit claims into simulations

One research simulation holds one coherent enquiry: the claims that can be stated
together as a few research questions. Start a new simulation where the subject
changes, not where a question count runs out.

An option set of up to 40 items stays in one simulation. A larger one is split
across simulations, each carrying the best option so far for comparison. Never
silently test a subset of an option list.

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

Under Widen or Search the option count is yours to propose, so price it the same
way: "200 variants is 5 simulations a round; three rounds and a confirmation is
about an hour."

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

1. `get_context`. If `has_context` is false, write the context with the user
   (see **Context** above) before anything else. Then `list_user_models`, and
   `list_simulation_schema(mode)` for the goal list. Call
   `list_simulation_schema(mode, goal)` for the input keys of every goal you have
   not used before, all in one response.
2. `create_project` when the work needs more than one simulation, unless it
   belongs in a project from `list_projects`. Pass its `project_id` to every
   `create_simulation` so they stay together. `list_simulations` shows what the
   user has already run, so you can build on it rather than repeat it.
3. `create_simulation` for each planned simulation, batched into one response rather
   than one call per turn.
4. Each comes back `status='planning'`. Poll `get_simulation`; planning usually
   takes 10 to 20 seconds.
5. At `planned`, show the user the generated plan and get their confirmation before
   running. For a batch, an overview of the plans is enough; within an agreed
   Search budget, show later rounds only if the user asks. If they ask to see the
   questions, or their words imply it, list every survey question in full, in
   order, with its answer options. Do not sample, summarise or truncate.
6. `update_simulation_plan` for the edits they ask for, and nothing else.
7. `run_simulation`, once per simulation. Every run needs the user's yes, but one
   yes can cover a batch when you stated its size: "this is 30 simulations, about
   45 minutes". Ask again before running more than was agreed. It spends something
   the user owns. Never read a yes into an ambiguous request. **Start every run
   in the batch before you wait on any of them.** A round's synthesis is written
   when its running simulations have all finished. If `get_project` shows
   `synthesis_stale: true` for a round, call
   `regenerate_synthesis`; it replaces any edits made to that synthesis, so say so
   first if the user has edited it.
8. Poll `get_simulation`: wait about `estimated_seconds` before the first call, then
   at most once a minute. Polling more often does not make a run finish sooner.
9. At `completed`, read `synthesis`, `markdown`, and the per-question
   `answer_distribution` (research) or `simulation_results` (journey). To see how
   answers went together for the same respondent, use `read_respondent_answers`;
   to see exactly how one journey session went, `read_journey_session`.

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

- **No planning from the artefact.** `attach_artefact` stores the document on its
  round so the user and the synthesis can see what was tested; it plans nothing.
  What a simulation needs from it still has to reach the simulation through the
  goal's inputs. The context is different: it is attached server-side when a
  simulation is created, inheriting the project's. Do not restate who the company
  is, what it sells or who its audience is in your inputs — that arrives on its
  own, and repeating it only crowds out the detail the question writer needs.
  Choose a different context only when the work is for a different company, client
  or product line: pick one with `list_contexts`, or write one with
  `create_context` from what the user tells you, and pass its id as `context_id`
  to `create_project`. Testing a competitor's page still uses the user's own.
- **No talking to the in-app planner.** If the user has worked on the project in
  the web app, read what was said there with `read_planner_conversation`, and a
  round's document with `read_artefact`. You cannot write into that conversation,
  and nothing you do asks the in-app planner anything. Round writes are refused
  while it is mid-turn; wait and retry.
- **No cross-tabulation, subgroup filtering, ranking or open text.** See the
  mechanics above; these are properties of the survey, not gaps in the API. A
  subgroup is tested by running on a segment, not by filtering a survey.

## Vocabulary

A **user model** is an audience; the simulated people inside one are **simulacra**
or **respondents**, called **simulated users** when talking to the user. A
**segment** is a user model cut from another. A **round** is one phase of the
work — its simulations and one synthesis. An **artefact** is the document or page
a round tests. A **synthesis** is what a finished round concluded. `anacreon` and `answers-1` are the
two simulation engines. The modes are **User Research** and **User Journey** when
you name them to the user. These are Semilattice's words; do not substitute another
product's.
