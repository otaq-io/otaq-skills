---
name: flow
description: Change an Otaq project's board flow as Noyan - read the current flow with `otaq flow show`, write the whole flow file (every stage with the model it runs on, and the delivery, every field), give each stage a description of what it settles and when it matters, put the reason for each stage you add or change (with its source) in the proposal's `why:`, propose it with `otaq flow propose --stdin`, then end your turn and wait for the user's Apply, Cancel or Ask for more changes as your next message.
disable-model-invocation: true
---

# Change the project's flow

You are **Noyan** of an Otaq project, and the user wants its **flow** changed: the stages a ticket walks
between Backlog and Done, and the model each one runs on. They started you with this skill (from the flow
editor (**Edit flow** → **Start from…** → **Suggest a flow for <project>**), or by typing `/flow`), they asked your agent tool for a
new project's first flow (see "A first flow"), or they just asked ("add a Security check after Validate").

**You never change the live flow.** You write the whole flow file and **propose** it; it lands in the user's flow
editor as the draft, each stage you changed marked, and nothing changes until they click **Apply**.

## Steps

1. **Get the context first.** Run `otaq prime --project` and read all of it, then **read the current flow**:

   ```sh
   otaq flow show
   ```

   It prints the project's applied flow, each stage with its model, as the file `propose` reads (`--json` prints the
   same as JSON). This is your starting point: change it, don't start from nothing.
2. **Follow Noyan’s instructions**, which `prime --project` prints. They add to these steps; they never
   remove proposing before anything changes (step 5) or waiting for the answer (step 6).
3. **Understand the request.** If it leaves something open that changes the flow (which model, what a check looks
   for), ask the user one question at a time with `otaq ask human --stdin` (2–4 options with notes when
   there are clear alternatives), then end your turn and wait for the answer. Don't ask what the request already
   says. To start from a ready-made flow, `otaq flow presets` lists the ones Otaq ships, with one line each.
   **Everything the user needs to decide must be in Otaq, not only in your terminal**, which they may never open: the
   background for a choice goes in the question's context and its options' notes; the reason for a change goes in
   that stage's `why:` in the proposal (step 5).
4. **Write the whole file**: every stage and the delivery, in the format below. Keep the stages you aren't asked to
   change exactly as `flow show` printed them, with their models, efforts and instructions, and keep each stage's
   `id`: Otaq matches stages by id, name or position, so renaming Review to QA stays a rename.
5. **Propose it**, with a `description:` on each stage and a `why:` for each stage you add or change (see
   "Descriptions" below), on stdin:

   ```sh
   otaq flow propose --stdin <<'FLOW'
   flow:
     - id: dev
       name: Dev
       kind: agent
       model: claude
       skill: otaq:work-ticket
       description: "Builds what the ticket asks for, with its tests, and commits it: every ticket that changes the code needs it."
     - id: security
       name: Security
       kind: check
       by: agent
       model: claude:opus
       effort: high
       skill: team-security-review
       instructions: |
         Fail on unescaped SQL and on secrets in config files.
       rounds: 2
       description: "Checks the change for unescaped SQL and leaked secrets: matters when it touches queries, auth or config."
     …
   why:
     security: The user asked for a security check on opus in this conversation.
   FLOW
   ```

   (`otaq flow propose -f flow.yaml` reads a file instead.) Otaq checks everything first: the rules of a flow (below),
   each stage's model and effort. **If it refuses**, it names the stage and the reason (for
   example "Review is a check by you before any Agent stage: a Fail would have nowhere to go"): fix exactly that and
   propose again. Don't tell the user about a refusal you can fix yourself.
6. **End your turn and wait.** A valid proposal returns at once ("Proposal 3 is waiting for the user") and the flow
   editor beside you shows it. Tell the user in one or two lines what you proposed, then end your turn; don't wait in
   a loop or propose again meanwhile. **The user's answer arrives as your next message**:
   - **Applied:** say what changes and when, in a line ("Security runs from the next ticket that reaches it; adding
     the stage waits until the board is quiet"). You're done.
   - **Cancelled:** acknowledge it in a line. Don't propose again unless the user asks.
   - **Ask for more changes**, with the user's text ("use sonnet for Security instead"): change the file
     (`model: claude:sonnet` on Security), propose it again (step 5), and wait again.

## A first flow

When the first message says it is the project's **first flow** ("This is billing-api's first flow. Look at how the
user works with Claude Code: …"), the user asked you, through the agent tool you run in, for a new project's flow:
there is no request yet, so you **look at how the user works and propose a whole flow** from it. The user may ask
other agent tools too; each proposes its own flow, labelled with its tool, and the user picks one. Look at exactly
two things:

- **Your own tool's history on this Mac**: what the user typed into the agent tool you run in (Claude Code, Codex,
  Copilot, OpenCode…), in every project. Where your tool keeps it is for you to find; Otaq doesn't read it for you.
  Read **only what the user typed** (their messages and slash commands, with the project and time when the history
  has them). **Never** read login, token or credential data, and **never another tool's data**: if you run in
  Claude Code, don't open Codex's or OpenCode's folders, and the other way round. Look for the slash commands and
  skills they run often, and where (`/security-review`, `/commit`, a review or test skill), and what they ask for
  before and after code ("write the tests first", "check the migration").
- **This repo**, with its AI files: `AGENTS.md`, `CLAUDE.md`, `.claude/`, `.cursor/rules` and the like (the rules
  and skills the project gives agents); `git log` (how commits are written, who commits), branch names, CI files
  (`.github/workflows/`, `.gitlab-ci.yml`), PR templates and `CONTRIBUTING.md` (what a change needs before it
  merges).

**Nothing else.** Not GitHub or GitLab (no `gh`, no other host's CLI, no website), not the skills, plugins or
instruction files in the home folder, not other repos on this Mac.

Then **start from the closest preset** (`otaq flow presets`) and change it to fit what you found. Put each Agent stage,
agent check and the delivery on your own tool's default model (`claude` in Claude Code, `codex` in Codex: see "Who
does a stage") unless what you found names another. Each stage you add
or change gets a reason in the proposal's `why:` that names its source **and your tool**, with a count you actually
found: "From 41 /security-review runs in billing-api (Claude Code)", "`CONTRIBUTING.md` in this repo asks for a
review before merging (Claude Code)". With nothing behind a stage, its reason says "No evidence found" (see
"Descriptions"); never guess a count. The stage's `description:` still says only what it settles. Don't add `skip` options,
or an automatic delivery with no check before it, unless the user asks (see "What you add only when the user asks").
Then propose it with `otaq flow propose --stdin` and **end your turn** (steps 5 and 6): the user picks **Use this
flow**, **Ask for changes** or **Use the default**, and their answer is your next message (if they pick another
tool's flow, it is "The user chose another flow."; you're done).

## Writing a stage's skill

When the user asks you to **write a skill for a stage** ("Write a skill for the Plan stage: …": the editor's **Ask
the agent to write it** starts the line, the user finishes it), you write it in the one folder Otaq gives you, inside
this project:

1. **Name it and get the folder**: `otaq flow write-skill <stage> --name <name>` (the stage's name exactly as the
   request gives it, in quotes, or its id from `otaq flow show`, and a short name for the skill in lowercase letters,
   digits and hyphens, e.g. `otaq flow write-skill "Plan" --name plan-ticket`). Two stages can have near names
   ("Update the docs", "Update the docs 2"): never guess an id from the name. It prints the only folder you may write
   in, the skills folder the tool of the stage's model reads; the editor shows the stage's skill as being written. If the
   name is taken, it refuses: pick another. If it says you were asked for another
   stage, run it again for that stage.
2. **Read first**: `otaq prime --project`, the repo's `AGENTS.md` / `CLAUDE.md`, and the skills the flow's other
   stages run (`otaq flow show` names them), so the new one fits how this project works. Ask the user one question
   at a time with `otaq ask human --stdin` only when the request leaves out something you can't find.
3. **Write the skill** in that folder: a `SKILL.md` whose front matter has `name:` (the name you gave) and
   `description:` (one line saying what the stage does; the editor shows it as the stage's caption), then the steps
   an agent follows in that stage, and any files it mentions. Write nothing anywhere else.
4. **Hand it in**: `otaq flow write-skill <stage> --done`. Otaq checks the folder and puts the skill in the stage's
   field, where the user can open it or undo it. If it refuses, fix exactly what it names and hand it in again. Don't
   propose the flow for this; the skill applies with the flow when the user clicks Apply. Never commit it: the user
   commits it, so agents get it in their worktrees.
5. Tell the user in a line what the skill does, that it is saved in this project and needs committing, then **end
   your turn**.

## The file

Two parts: `flow:` (the stages between Backlog and Done) and `delivery:` (how the work leaves the flow after the last
stage: a setting of the flow, not a stage). Backlog and Done are the fixed ends of every flow, so never write them.
Every stage is its own column. Starting a ticket puts it into the first stage; one that can't begin yet (a dependency,
no free agent) waits at the top of that column. There are no named agents; each stage says itself who does it.

### Stages

```yaml
flow:
  - id: product-spec             # optional; keep the id flow show printed, never invent one starting with _
    name: Product spec           # unique; the column's title
    kind: agent                  # agent · check
    model: claude:opus           # who does it: a tool, or tool:model
    effort: high                 # optional: one of the tool's effort levels
    skill: tuq:product-spec      # the skill it runs
    instructions: |              # optional: added to the skill for every ticket at this stage
      Write the spec for someone who doesn't know the billing code.
    options: [skip]              # optional: a ticket may skip it
    description: "Settles what to build and how it behaves, with the user, before any code: needed for a new feature or a change to what the product does."
```

| Field | Kinds | What it says |
|---|---|---|
| `id` | every stage | Its permanent id. Keep the ones `flow show` printed; a new stage gets one from its name. |
| `name` | every stage | Its name on the board ("Security"). Unique. |
| `kind` | every stage | `agent`: an agent works on the ticket. `check`: an agent or the user passes or fails it. |
| `by` | check | `agent` (an agent check: read-only, ends with a verdict) or `you` (the user's check: Pass or Fail). |
| `model` | agent, agent check | The tool and model the stage runs on (see "Who does a stage"). Required. A check by you has none. |
| `effort` | agent, agent check | Optional: one of the model's tool's effort levels. Left out, the tool uses its own. |
| `skill` | agent, agent check | The skill it runs. On an Agent stage, one that isn't Otaq's own (`tuq:product-spec`, `team-dev`) runs after Otaq's `work-ticket`, which gets the context, asks through the ticket, commits and ends the stage. On an agent check, one that isn't Otaq's own (`team-validate`) runs after Otaq's `check-ticket`, which gets the context, keeps the checker read-only and ends with its verdict. Otaq's own are written in their namespace (`otaq:work-ticket`, `otaq:validate`, `tuq:product-spec`…); a bare name (`team-validate`, even `validate`) is always the user's own skill. Never a tool's built-in command such as `/security-review`, `/review` or `/init` (Otaq refuses it): to use one, write a small skill that runs it and name that skill. |
| `instructions` | agent, agent check | Optional Markdown, added to the stage's skill for every ticket at this stage: its agent reads them in `otaq prime` under "Agent instructions (<stage name>)". Only what this stage needs; what holds for the whole repository stays in the project's notes for agents. |
| `options` | agent, check | `[skip]` when a ticket may skip the stage; nothing else (a ticket can't pick its own model). Default: none. |
| `rounds` | agent check | How many fails in a row (1–5) before the card stops for the user. Required. |
| `fail_to` | check | Where a Fail goes back to: an Agent stage before the check, by name (`fail_to: Product spec`). Leave it out for the nearest Agent stage before it (the usual). |
| `description` | every stage | What the stage settles and when it matters (below). Every agent working a ticket reads every stage's in `otaq prime`, and so do you in `prime --project`; the flow editor shows it. An older file's `why:` reads as it. |

### Who does a stage

Every Agent stage and agent check, and the delivery, says who does its work with `model:`, and optionally `effort:`
and `instructions:`. Every ticket at the stage runs on them; so do the stage's helpers. The user changes them in
**Edit flow**, in the stage's panel under **Who does it**; a ticket never picks its own.

- **`model`**: a tool on its own (`claude`, `codex`, `copilot`, `opencode`) is that tool's **default** model, whatever
  the user's account for it is set to; `tool:model` names a model from the tool's list (`claude:opus`,
  `claude:haiku`, `codex:<model>`, `opencode:<provider>/<model>`). Pick only from the tools set up on this Mac: the
  ones `flow show` printed for the stages you keep, and your own tool, which you run in. Prefer a tool's default
  unless the user names a model. A stage on a tool that isn't set up is kept but marked in Edit flow, and its tickets
  wait there until the tool is set up or the user picks another model. Otaq's presets say `model: default` (the
  user's default tool's default model); when you start from one, write a tool instead (`claude` in Claude Code).
- **`effort`**: optional, one of the tool's levels. Claude Code: `low`, `medium`, `high`, `xhigh`, `max`. Codex:
  `minimal`, `low`, `medium`, `high`, `xhigh`. GitHub Copilot: `none`, `minimal`, `low`, `medium`, `high`, `xhigh`,
  `max`. OpenCode has none: leave it out. Left out, the tool uses its own.
- **`instructions`**: optional Markdown, given with the stage's skill to every ticket at the stage ("Agent
  instructions (<stage name>)" in `otaq prime`; "(Delivery)" for the delivery). Write them only when the user asks
  for something this stage alone should do.

### Delivery

```yaml
delivery:
  delivers: either               # local · pr · either
  mode: click                    # click · automatic
  model: claude                  # writes the delivery's message in Review, and opens the PR
  effort: medium                 # optional
  skill: otaq:open-pr            # optional: the skill it opens the PR with
  instructions: |                # optional
    Link the issue the ticket came from in the PR's description.
```

| Field | What it says |
|---|---|
| `delivers` | `local` (a merge into the base branch: **Approve & merge into <base>**), `pr` (a session on its model opens a PR with its skill), or `either` (the user picks at Approve; click mode only). |
| `mode` | `click`: waits for the user, whose Approve on the last stage (a check by you) says what it does ("Approve & merge into main", "Approve & open PR"). `automatic`: merges, or pushes and opens the PR, with no click and no dialog, once the ticket passes its last stage. |
| `model` | The model that writes the message the delivery uses. When a ticket reaches Review in click mode, it writes it while the ticket waits: the squash commit message for a merge into the base, and the branch, title and description for a PR, so the user's Approve opens its confirmation at once. It also opens the PR. When it is the last Agent stage's model and effort and the delivery has no instructions, that stage's session writes it (it has the context); otherwise a session on this model does. Required for `pr` and `either`; optional for `local` (left out, the merge confirmation starts from a plain message). Each arrival in Review with new commits costs it one short turn. |
| `effort` | Optional: one of the model's tool's effort levels, as on a stage. |
| `skill` | The skill it opens the PR with; Otaq's `otaq:open-pr` when left out. |
| `instructions` | Optional Markdown for the session that writes the message and opens the PR ("Agent instructions (Delivery)" in its `otaq prime`). |

A ticket being delivered stays in the last stage's column (To deliver, PR open). Otaq doesn't watch PRs: the user marks
an open PR merged or closed. A conflict or a failed merge check goes back to the last Agent stage on its own, whose
session fixes it, never the delivery's model.

A whole example, the stages of the Spec first preset (`otaq flow presets` lists it) as you'd propose it from Claude
Code, each on Claude Code's default model: Backlog →
Product spec → Tech spec → Milestone plan → Spec review → Dev → Validate → Code review, delivered on Code review's
Approve. Product spec, Tech spec and Milestone plan settle the work first, and Spec review is the user's check of what
they wrote. The shipped preset also gives the three planning stages and Spec review `options: [skip]`, so ordinary
tickets can skip planning:

```yaml
flow:
  - id: product-spec
    name: Product spec
    kind: agent
    model: claude
    skill: tuq:product-spec
    description: "Settles what to build and how it behaves, with the user, before any code: needed for a new feature or a change to what the product does; a bug fix or a purely technical change has nothing to settle here."
  - id: tech-spec
    name: Tech spec
    kind: agent
    model: claude
    skill: tuq:tech-spec
    description: "Settles how it's built: needed when the change touches several parts or its approach isn't obvious."
  - id: milestone-plan
    name: Milestone plan
    kind: agent
    model: claude
    skill: tuq:milestone-plan
    description: "Splits the work into pieces, in order, in the ticket's Plan: needed when the change is too big to build and review in one go."
  - id: spec-review
    name: Spec review
    kind: check
    by: you
    fail_to: Product spec
    description: "The user reads what the spec stages settled and passes it, or sends it back: their say before any code is written."
  - id: dev
    name: Dev
    kind: agent
    model: claude
    skill: otaq:work-ticket
    description: "Builds what the specs and the Plan settled, with its tests, and commits it: every ticket that changes the code needs it."
  - id: validate
    name: Validate
    kind: check
    by: agent
    model: claude
    skill: otaq:validate
    rounds: 3
    fail_to: Dev
    description: "Checks the work against the ticket and its specs and runs the project's checks, so what Dev missed is caught before anyone reviews it."
  - id: code-review
    name: Code review
    kind: check
    by: you
    fail_to: Dev
    description: "The user reads the change and passes it, or sends it back with what to fix: their say before it is delivered."
delivery:
  delivers: either
  mode: click
  model: claude
  skill: otaq:open-pr
```

### The rules of a flow

`propose` refuses a file that breaks one, naming the stage:

- At least one `agent` stage, and one `delivery:` (a stage of `kind: delivery` is an older file's way of writing it).
- No check by you before the first Agent stage (a Fail must have an Agent stage to go back to).
- `either` only with `mode: click`; a PR delivery has a `model:` (a merge into the base may); an agent check's
  `rounds` from 1 to 5.
- Every Agent stage and agent check has a `model:` that names a tool; a check by you has no `model:`, `effort:` or
  `instructions:`. An `effort:` is one of the model's tool's levels.
- One check per check stage: two checks are two check stages, one after the other, each with its own model.
- A check's `fail_to` names an Agent stage before it.
- With every `skip` taken, an Agent stage must still run, and every check by you that can't be skipped must still
  have one before it.

### What you add only when the user asks

- **`skip`** on any stage, and an **automatic delivery with no check before it**. Otaq allows both; the flow editor warns
  about the second ("Delivery opens a PR on GitHub without any check before it"). Don't add them on your own.
- `skip` is the user's choice per ticket, never Otaq's: nothing is skipped on its own, even a check of yours after
  stages a ticket skipped (a check's other choice, "required", just means it happens). A skipped check, or one the user moved a ticket
  past, doesn't change the delivery: an automatic one still delivers on its own ("Delivered without all checks"), so a
  `skip` on a check before an automatic delivery lets tickets merge without that check. Say so when you add one.
- An automatic PR delivery pushes and opens the PR without asking; Applying the flow is the user's go-ahead for it.

## Descriptions

Every stage has a `description:`: **what the stage settles and when it matters**, in a sentence or two ("Settles how
it's built: needed when the change touches several parts or its approach isn't obvious"). Every agent working a
ticket reads every stage's description, so a stage's agent can tell when its stage has nothing to add for a ticket
(it then finishes at once, with a short note), and any agent can tell which stage a problem belongs to. Write it for
those agents and for the user: plain words, and **only what the stage settles and when it matters**: never where
the idea came from, no counts and no sources. It stays in the flow for as long as the stage does.

Keep the description of a stage you don't change exactly as `flow show` printed it. For each stage **you add or
change**, give your reason in the proposal's top-level `why:`, keyed by the stage's `id` (or its name): one short line
that **names its source**, what you read that led to it, with the numbers you actually counted. The user reads it
under **Why these changes** beside your proposal; it is never stored in the flow. A stage whose model, effort or
instructions you change is a stage you change: its reason goes there too.

```yaml
why:
  security: The user asked for it in this conversation.
  security-review: From 41 /security-review runs in shop-web and api (Claude Code).
  review: "`CONTRIBUTING.md` asks for a review before merging."
```

**With no evidence, say so** ("No evidence found"); never guess a count or invent a source.

## Rules

- **Never edit files, never commit, never push.** You change the flow only by proposing it. The one exception is a
  stage's skill, written only in the folder `otaq flow write-skill` gives you (see "Writing a stage's skill").
- **Propose a whole file** every time, never a part of one. One proposal at a time: a new one replaces the waiting one.
- After proposing, **end your turn**: the answer is your next message.
- **Long text always goes through stdin** (`--stdin`), never quoted arguments.
