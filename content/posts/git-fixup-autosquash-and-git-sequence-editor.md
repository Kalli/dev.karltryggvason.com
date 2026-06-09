+++
date = '2026-06-09T09:45:44+02:00'
draft = false
title = 'Git: --fixup --autosquash and GIT_SEQUENCE_EDITOR'
tags = ["productivity", "git"]

+++

A while back a colleague told me about the [`--fixup`](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---fixupamendrewordcommit) option for git commit and its cousin [`--autosquash`](https://git-scm.com/docs/git-rebase#Documentation/git-rebase.txt---autosquash). I've long been a proponent of squashing and fixing up commits; it means you can commit work earlier which in turn means less chance of data loss as well as smaller diffs being easier to reason about. With the fixup and autosquash options you do the re-organising work up front and save yourself the manual re-ordering of commits in the editor (no more of the vim `yy`, `dd` `p` tango).

More recently, a coding agent (a different kind of colleague I suppose), taught me a new trick that complements the above nicely. Passing the [`GIT_SEQUENCE_EDITOR=true` env variable](https://git-scm.com/docs/git#Documentation/git.txt-GITSEQUENCEEDITOR) to a `git rebase -i HEAD~<RANGE>` command skips the opening the git editor at all, saving precious seconds.

A full (somewhat contrived) example might look like this[^1]: 

```sh
# create some commits
echo "foo" > foo.txt
git add foo.txt && git commit -m "add foo"
echo "bar" > bar.txt
git add bar.txt && git commit -m "add bar"
echo "foo" > foo.txt
# adds a fixup commit that will be squashed into the "add foo" commit above
git add foo.txt && git commit --fixup=HEAD~1
# rebases the three commits into two, skipping the editor
GIT_SEQUENCE_EDITOR=true git rebase -i HEAD~3 --autosquash
```

Addendum: I assumed `true` was a git-specific flag, but it's actually a standard shell command, and, it turns out, two things at once. My shell (zsh) ships `true` as a builtin, but there's also a standalone `/usr/bin/true` binary on disk. Both do the same trivial job (nothing, successfully, exit code 0), which is why it works as a no-op editor: git "*opens*" it, it touches nothing, and the rebase proceeds with the default plan.

```sh
type -a true
true is a shell builtin
true is /usr/bin/true
```

I'm sure I'll find some more uses for it and its sibling `false`!


[^1]: This clean skip only works because `--fixup` discards the commit message. If you use `--squash` or have a reword in the range, git still opens your editor for the message. Set `GIT_EDITOR=true` too if you want to accept the defaults there as well.
