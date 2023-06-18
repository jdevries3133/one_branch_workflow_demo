# One Branch Demo

If you just use one development branch, when you rebase your branch on `main`,
the commits that have been merged onto main just, "disappear," from your dev
branch when you merge them.

## `your_dev_branch`

This is the branch you'd use all day every day. In real life when working on
teams, I name this branch, `jack`.

It looks like this, right now:

```text
commit 74673b5f2dc13b36e6bbeda3e1f8bc1327342a60 (your_dev_branch)
Author: Jack DeVries <jdevries3133@gmail.com>
Date:   Sun Jun 18 09:31:31 2023 -0400

    chore: #1

commit 637a2cc2ef0535f780831454ed137fd617b65eb6
Author: Jack DeVries <jdevries3133@gmail.com>
Date:   Sun Jun 18 09:31:21 2023 -0400

    fix: #1

commit 5bd8f4ab6e480a13172a0c36fad98ef2813ecd46
Author: Jack DeVries <jdevries3133@gmail.com>
Date:   Sun Jun 18 09:31:10 2023 -0400

    feat: #2

commit 471acb0168cef468946aa1c4c6320f938cb1558b (ship_feature_1)
Author: Jack DeVries <jdevries3133@gmail.com>
Date:   Sun Jun 18 09:30:58 2023 -0400

    feat: #1

commit 98b73e1b6f7ab7810655efa69d065ba989df1f23
Author: Jack DeVries <jdevries3133@gmail.com>
Date:   Sun Jun 18 09:24:50 2023 -0400

    initial commit
```

## Sending Code to Code Review

This is how I'd send changes off to code review. I create a branch from
`your_dev_branch`;

```bash
git checkout your_dev_branch
git checkout -b ship_feature_1
```

Then, I perform;

```bash
git rebase -i $(git merge-base main HEAD)
```

Let's break that down.

- `git merge-base main HEAD` - this is the commit where `your_dev_branch`
  checked out from master originally. In this repo, it's `98b73e1`, the initial
  commit.
- `git rebase -i` initiates an interactive rebase

Basically, we're interactively rebasing over the base of the branch. In a team
environment, this is quite different from `git rebase main` or `git rebase -i
main`, because if you've pulled, the version of `main` that you're now rebasing
on has new changes. Now, at some point you will want to `git rebase main` to
sync `your_dev_branch` with upstream. But here, our use-case is to take control
of the history of the branch, so we want to rebase over the merge-base to do
that as surgically as possible. The scope of our changes is now completely
limited to the changes on `your_dev_branch`, and no surprises will happen (like
merge conflicts) from new code being pulled in from` main`.

Anyway, remember that we're now on `ship_feature_1` - the goal here is to drop
all the commits except the one we want to ship. At that point, you're left with
a branch with a single commit from `your_dev_branch` atop `main`, which can be
sent out for code review.

### Out-of-Order Code Review

Notice that we have "ship" branches `ship_feature_1`, and `ship_fix_1`, but
that doesn't correspond to the order that we made the actual commits on
`your_dev_branch`! A great advantage to this workflow is that the order you
create changes and the priority with which you send them to code review are
decoupled. This is especially useful if you have an experimental DX improvement
that helps you personally, and you want to refine it on your branch before
putting it up for code review.

## After Code Review

I think this is the part that most people are scared of, and it's why I wanted
to make this demo rather than just writing a doc! I think there's an impression
that this single-branch workflow will get out of hand, but it actually scales
incredibly well. Consider where we are right now;

- `feature_1` from `your_dev_branch` has been merged
- `fix_1` has also been merged, which hop-scotched over `feat_2`
- this documentation commit has been merged from a different dev branch,
  simulating the work of a colleague landing on the main branch
- there are 2 remaining unmerged commits on `your_dev_branch`

This is the kitchen sink! What the hell happens now? Well, don't worry, it's
actually super simple;

```bash
git checkout your_dev_branch
git rebase main
```

And voila, you're done!

Git will kindly let you know exactly which refs are skipped during the rebase
process.

```
➜  one_branch_demo git:(your_dev_branch) grb main
warning: skipped previously applied commit f899409
warning: skipped previously applied commit 11b6165
Successfully rebased and updated refs/heads/your_dev_branch.
```

These are the commits from `your_dev_branch` that have landed on `main` since
the last time you rebased. Thus, the one-branch workflow has the added benefit
of providing you with visibility over which commits are in code review, and
you'll quickly notice when commits land on the main branch!

