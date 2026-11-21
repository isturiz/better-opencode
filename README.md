# better-opencode

Small, standalone additions for OpenCode V2. Inspired by
[better-claude](https://github.com/midudev/better-claude).

## Commit

Send `@commit` in an OpenCode session to open the commit menu. You can include a
scope in the same prompt, for example `@commit for staged changes only`.

| Option | What it does |
| --- | --- |
| **Commit only** | Commit on the current branch without pushing. |
| **Commit on a new branch** | Create a branch and commit there without pushing. |
| **Message only** | Return a copyable message without changing the repository. |
| **Commit + push** | Commit on the current branch and push to the destination shown in the preview. |

Review the proposed message, files, and destination before approving any write.
If the staging area conflicts with the proposed scope, the skill stops rather
than resetting it. Dismissing the menu or using a client that cannot display it
does not authorize any changes.

If no remote is configured, **Commit + push** offers the three local options
again without staging or committing anything. You can create the initial
commit on the current branch or a new branch. Pushing an initial commit
requires a verified remote whose target branch does not exist yet. If
there are existing outgoing commits, the push preview includes them.

### Message format

Messages are written in English. For one request, specify a style in the prompt
(for example, `@commit use a bracketed type prefix`) or mention an installed
formatting-only skill alongside `@commit`. Otherwise, the skill follows
documented repository conventions, then consistent recent commit history, and
defaults to Conventional Commits.

`@commit` doesn't look for or load formatting skills on its own. Formatting
overrides can't authorize Git actions or change the mode, scope, approvals, or
safety checks.

### Install

Copy the entire [commit skill directory](.opencode/skills/commit/) into
`.opencode/skills/` in the target project, or into
`~/.config/opencode/skills/` to use it across projects. Keep its directory name
`commit` and check that you are not replacing an existing skill with that name.
No copy is needed to try it from this repository.

Use a client that supports OpenCode's interactive `question` tool, such as its
terminal UI, and an agent that is allowed to ask questions and run Git.
This is an agent skill, not a Git policy enforcement tool: your OpenCode
permissions and Git hooks still apply. There is no `/commit` command to install.
