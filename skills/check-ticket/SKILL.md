---
name: check-ticket
description: Check the Otaq ticket named by $OTAQ_TICKET in a check stage whose skill is another (a task skill such as the user's team-validate, which Otaq types after the first turn) - get the context with `otaq prime --validate` and end the first turn, then follow the task skill for what to check and this skill for how - stay a read-only judge (no edits, no commits, output only in $OTAQ_RESOURCE_DIR), never ask the user (an open point goes into a fail verdict, or a send-back for an earlier stage's problem), and always end with a short verdict through `otaq check pass|fail --stdin`.
disable-model-invocation: true
---

# Check an Otaq ticket

The agent of an Otaq ticket (its key is in `$OTAQ_TICKET`, for example `SHOP-29`) submitted its work. You are a
**checker** in one of the flow's check stages: a fresh session in the ticket's worktree, started by Otaq to judge that
work before the ticket goes on.

Your stage names a skill of its own under "## Your stage" in `otaq prime --validate` (the user's `team-validate`, a
`tuq:` skill, shown as your tool calls it, such as `/team-validate` or `$team-validate`). That skill is your stage's
**task skill**: Otaq types it into this session once your first turn ends. The task skill decides **what** is checked
and how it is worked out: which checks to run, what to look at, what counts as a problem. **This skill decides how the
check meets Otaq**: you stay a read-only judge, you never ask the user, and you always end with a verdict. A task skill
is written for a plain terminal, so where the two disagree about how, this skill wins; about what, the task skill
wins. The Agent instructions and the project's notes apply to both.

**Your check ends only when you run `otaq check pass --stdin` or `otaq check fail --stdin`.** A summary in this chat
reaches no one: the board, the user and the ticket's agent see only what your verdict carries. Ending your turn
without a verdict (once the task skill has run) leaves the ticket waiting for the user, with no result.

## Steps

1. **First turn: get the context, then end your turn.** Run `otaq prime --validate` and read all of it: the ticket's
   description, the user's **Agent instructions** and the project's notes, your stage and its skill, the earlier
   stages, the **comments** (the agent's comment on its work, the user's, and earlier rounds' verdicts), the diff and
   the linked tickets. Open the reports from you that `prime --validate` lists under "Reports from you · read these
   before you start" (`otaq report SHOP-29 1`). Then check nothing yet, give no verdict yet, and **never start the
   task skill yourself**: end your turn with one line saying you have the context. Otaq types the task skill into this
   session next. A verdict before then is refused: "Your stage's skill `/team-validate` hasn't run yet: end your turn
   now, and Otaq types it next." Don't judge yet whether your check has anything to look at: that is the task skill's
   call.
2. **Follow the task skill**, step by step, for what to check. Where it names its subject (its argument, "the
   change", "the current branch"), the subject is this ticket's change: the diff `prime --validate` shows, judged
   against its description and comments there. Where the task skill says to run its own `git diff` against a base,
   judge the diff `prime --validate` shows instead. What `prime --validate` lists as approved earlier is context:
   never fail the ticket for it. Its steps meet Otaq this way:
   - **Changing things:** where it says to fix what it finds, format, commit, install or update anything, don't.
     Report the problem instead: it becomes a line of a fail verdict, and the ticket's agent fixes it.
   - **Asking:** where it asks the user something (a question in this terminal, the tool's own question tool), don't
     ask. Decide it yourself from the ticket, or put the open point into a fail verdict; a problem that belongs to an
     earlier stage goes back to it (see "A problem that belongs to an earlier stage").
   - **Ending:** where it ends by reporting in this chat ("Pass.", a list of findings) or by suggesting the next
     skill, give your verdict instead (step 3). The next stage runs the next skill; never start it yourself.
   - **Nothing to check:** when the task skill finds nothing this change touches, pass at once with a short note
     saying so ("Nothing to check: no migration in this change"). That is an ordinary pass, never a skip: only the
     user skips a stage.
   - **Other skills it names** it may call as it says: a skill that was started may call the skills it names.
3. **Give the verdict** with the comment on stdin, then end your turn. The verdict in one or two lines, then one line
   per finding with its file and line (`src/cart.ts:42: the total ignores the discount`), and in a few words what you
   ran and what you relied on (at most 2,000 characters). A run's output stays in its log, never in the comment.

   ```sh
   otaq check fail --stdin <<'VERDICT'
   Fail: src/cart.ts:42: the total ignores the discount the description asks for.
   `npm test` at a1b2c3d: 1 failed, src/cart.test.ts:10; log: `$OTAQ_RESOURCE_DIR/cart-tests.log`
   VERDICT
   ```

   or, when nothing fails:

   ```sh
   otaq check pass --stdin <<'VERDICT'
   Pass: the migration is reversible and the old rows still load.
   Ran: `npm run migrate:check` at a1b2c3d; log: `$OTAQ_RESOURCE_DIR/migrate.log`
   VERDICT
   ```

   Only problems with this ticket's change fail it. A gap covered by a linked ticket, or not asked for by this ticket,
   is a `Note:` line, never a failure. **What you couldn't check yourself** (it needs a real browser, a device, or a
   service you can't reach) goes in `--manual-check "<what to check>"`, one per check, on `check pass` or
   `check fail`: the user sees them at review.

## A problem that belongs to an earlier stage

A product gap no Product spec settled, or an approach a Tech spec should have decided, isn't the agent's to fix. Don't
fail the ticket for it and never move it back yourself: send it back, naming that earlier stage (`prime --validate`
lists every stage under "## The flow"), with your reason on stdin, instead of giving a verdict, then end your turn:

```sh
otaq ask human --send-back "Product spec" --stdin <<'QUESTION'
Should Product spec decide what happens to refunds over 30 days?
Product gap: refunds over 30 days aren't specified, and the change picks one rule without saying why.
QUESTION
```

This is the only way a checker reaches the user. Anything else that is open goes into your verdict.

## Rules

- **Change nothing:** don't edit, create or delete files, don't commit, don't push, and don't run commands that change
  tracked files or anything outside the worktree (no installs, no migrations, no formatting fixes), whatever the task
  skill says. If tracked files change or a new commit appears while you run, your result is discarded. Writing your
  check runs' output into `$OTAQ_RESOURCE_DIR` is the one exception. Starting the app with `otaq run start --wait` is
  not a change; stop what you started (`otaq run stop`) before your verdict, and never stop a Run the user started.
- **Never ask the user** with `otaq ask human`, in this terminal or with the tool's own question tool: a checker gives
  a verdict, or sends the ticket back to an earlier stage.
- Don't file tickets and don't run `otaq stage next`: the user creates tickets from your notes.
- **Long output goes to a file** in `$OTAQ_RESOURCE_DIR` (`npm test > "$OTAQ_RESOURCE_DIR/tests.log" 2>&1`), never
  in the worktree; search it with `grep` or `tail`, never print it in full, and name it in your verdict. Attach only a
  document you wrote with `--report-file`.
- **Long text always goes through stdin** (`--stdin`), never quoted arguments.
- **Never start a skill on your own initiative.** Otaq types the task skill; a skill that was started may call the
  skills it names.
- Always finish with `otaq check pass --stdin` or `otaq check fail --stdin`.
