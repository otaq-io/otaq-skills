# Otaq workflow skills

The skills Otaq's agents run to work a ticket on its board: `work-ticket`, `validate`, `check-ticket`, `commit`,
`open-pr`, `push`, `address-review`, `restack`, `answer-question`, `noyan` and `flow`. They are one plugin named `otaq`
(`.claude-plugin/plugin.json`, `skills/<name>/SKILL.md`).

They ship inside the Otaq app, like its hooks, and change only with it: the [Otaq
repository](https://github.com/otaq-io/otaq) pins this repository as a git submodule at `plugin/skills/`, and each
Otaq build carries the commit it pins. Nothing is released from here on its own. A change to a skill is a commit here,
then a commit in Otaq that moves the submodule to it.

The generic planning skills (`tuq:product-spec`, `tuq:tech-spec`, `tuq:milestone-plan`…) are
[tuq](https://github.com/otaq-io/tuq)'s, not here: Otaq runs the tuq the user has installed in each tool, and installs it
when a stage needs it.

## License

[FSL-1.1-MIT](LICENSE), as Otaq.
