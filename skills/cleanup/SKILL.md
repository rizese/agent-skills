---
name: cleanup
description: Review the branch's changes and recommend what to optimize, refactor, or remove now that the feature works. Use when the user says /cleanup, or asks to clean up or refactor their changes before shipping.
---

Now that we've built our feature like we want, is this the best way to build it?
Run `git --no-pager diff "$(git merge-base origin/HEAD HEAD)"` to see all changes (committed, staged, and unstaged) relative to the default branch.

- Is there anything we can optimize?
- Is there anything to refactor?
- Is there anything we're doing we don't need to? Anything we can remove?
- Is there any code that's just weird? Or has a bad code smell?

Compile a list of options and make recommendations.
