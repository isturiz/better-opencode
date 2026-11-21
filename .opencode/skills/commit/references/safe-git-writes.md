# Safe Git writes

## Plan without writing

- Stop on detached HEAD, unmerged index entries, or a Git operation in progress
  (including merge, rebase, am, cherry-pick, revert, sequencer, or bisect).
  Stop if a lock or Git environment override makes the worktree or index
  uncertain. Both local write modes support an initial commit. When no tip
  exists, a new branch starts unborn too; say so in the preview.
- Preserve the existing index. A normal index commit consumes everything
  staged: the first group must explicitly include all already-staged changes.
  Never add a path that has both staged and unstaged changes. An explicitly
  approved staged-only commit may take the staged portion while leaving the
  unstaged portion untouched. If the plan would consume unrelated staged
  changes, combine independent staged groups, or require altering partial
  staging, stop and ask for a revised scope or user adjustment. Don't stage
  or split individual hunks yourself.
- Use exact repository-relative file paths, not directories, globs, or
  arbitrary pathspecs. Parse NUL-delimited path lists (`-z`), account for both
  sides of a rename, and pass paths as safely quoted separate arguments after
  `--` with `git --literal-pathspecs`. Stop if a path or its effect on the
  index isn't unambiguous, including collisions, path aliases, nested
  repositories, gitlinks, and affected intent-to-add or hidden index entries.
  A path must not match a directory on disk or in the index. Don't force-add
  ignored files. Safely quote every dynamic shell argument (paths, branch
  names, remote names/URLs, refspecs, messages); never use `eval`.
- For a new branch, propose a literal name (for example `fix/short-slug`).
  Reject checkout shorthand such as `@{-1}` before calling
  `git check-ref-format --branch`, and ensure the validated name is unused.
  Do not switch branches in any other mode.

## Resolve a push before approval

Only in **Commit + push**, and after the skill's early check for a configured
remote, resolve the current branch's configured upstream to one named remote
and one full `refs/heads/...` destination. With no upstream configured, the
only automatic proposal is `origin` and the same full branch ref; preview it
as a fallback, and don't set upstream tracking. Stop if the proposed remote
is missing, the upstream is local or malformed, or user intent or settings
such as `branch.<name>.pushRemote` or `remote.pushDefault` make the destination
ambiguous. Don't select a different destination by guess.

Check `git remote get-url --push --all` for exactly one effective push URL
(not the fetch URL). Reject remote groups, mirrors, nonempty `push.pushOption`
settings, or a custom transport or hook that can publish elsewhere. Preview the
remote, URL without credentials, and full destination ref. Query that exact
endpoint with read-only `git ls-remote --refs --exit-code`, not a cached
tracking ref or a different fetch endpoint. Check the exact ref column
(patterns can also match suffixes) and distinguish absence from lookup
failure. Stop if rewrites, redirects, or credential/setup writes prevent a
safe check against the intended endpoint.

If the ref exists, its advertised tip must be locally available and an ancestor
of the local tip. If there is no local tip yet, stop rather than trying to
replace an existing remote branch. List all existing outgoing commits
(`remote-tip..local-tip`), not just the ones this skill proposes. If the ref is
absent, mark it as a new remote branch. Account for all history reachable from
the local tip if there is one; if the local branch is unborn, only the new
commits are outgoing. Show existing outgoing commit IDs and subjects in the
preview and explicitly include them in the approval. Review outgoing changes,
not just subjects; stop if the range can't be reviewed or contains an obvious
secret. Don't truncate or guess the range, fetch, force-push, or silently fall
back to a local commit if preflight fails.

## Preview, approve, and execute

1. Show the worktree, current and target branch, each group's exact files and
   their staged, unstaged, or untracked portions (including anything left out
   on a partially staged file), and each full message. For a push, show the
   resolved destination and the outgoing commits above. Use a single-select
   `question` with `Approve` and `Cancel` to authorize that exact plan. Only
   an explicit `Approve` allows writes. Respect normal tool permissions; don't
   work around a denial.
2. After approval, recheck the worktree, branch/tip, operation state, entire
   index, and relevant changes, including the contents of selected untracked
   files. Compare against the inspected plan, not just filenames. Recheck new
   branch availability or the resolved push destination and advertised tip
   where applicable. If anything material differs, stop and get fresh approval
   for a revised plan before writing.
3. If requested, create the approved branch with `git switch --no-track
   --no-recurse-submodules --no-overwrite-ignore -c <new-branch>` with no
   start-point argument. This uses the current tip or, on an unborn branch,
   points HEAD at the new branch name until the initial commit. Verify HEAD,
   the index, and pending changes before staging anything.
4. For each approved group, confirm the expected state and stage only its
   exact unstaged or untracked paths with `git --literal-pathspecs add --
   <paths>`. Never use a broad add, reset, unstage, stash, or path-limited
   commit. Recheck the entire staged diff against the approved group (paths
   and content), then use `git commit --cleanup=verbatim -m <literal-message>`
   without bypassing hooks or signing. Verify the new commit's parent or root
   status, tree changes, message, and remaining index/worktree state against
   the plan. If boundaries can't be verified, don't commit further or push.
5. Push only after every approved commit succeeds. Recheck the local branch,
   verified final commit, effective push URL and remote configuration, and
   advertised destination tip and outgoing range. Stop if anything deviates
   from the plan. Use one non-forced refspec anchored to the verified final
   commit, for example `git push --no-follow-tags --recurse-submodules=no
   --no-set-upstream -- <remote> <commit-id>:<full-branch-ref>`. Don't push
   other refs, tags, or submodules, add server push options, or bypass hooks.

Stop on any failed step. Inspect the actual state using read-only commands
and report commits created (IDs and branch), push result, and any remaining
staging or branch change. If push fails after committing, report local
success and the push failure separately; if remote status is uncertain, say
so. Don't auto-retry, undo, or clean up.
