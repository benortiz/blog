---
title: "Finding Lost Commits"
date: 2016-06-15T16:59:16-08:00
showDate: true
draft: false
tags: ["git", "til"]
---
In the process of rebasing with preserving a merge commit (`git rebase -i -p base_branch`),
I somehow managed to lose two commits I was trying to squash together.

To find the lost commits, I tried to use a combination of git commands.
First, I tried using `git fsck --lost-found` which returned a list of
commits that were "dangling". I have no idea how these specific commits
got lost to time, but now they were found. I also checked `git reflog`
which keeps a log of each action and--it seems--its sha.

Unfortunately for me, I couldn't find the specific commit I was looking
to reset everything to in either of these. What I ended up doing was looking
through my terminal history for a previous time when I had run `git log`
and grabbed the commit id from there.

After that, it was a simple `git merge --ff-only commit_id` and all my
changes were back.

*wipes sweat from brow*
