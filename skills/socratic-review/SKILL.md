---
name: socratic-review
description: Review GitHub PRs and leave inline comments written only as questions, so the author walks through their own code and finds the problem. Use when the user says /socratic-review, asks to review PRs "with questions", or wants review comments phrased as questions instead of findings.
---

Review one or more GitHub PRs. Find the real problems, then turn each one into a short inline comment that only asks questions. The reviewer sounds concerned and a little confused, not like they already know the answer. Each question should have an obvious honest answer ("no, it doesn't", "yes, it would") that leads the author to the bug themselves.

## 1. Find the PRs

- Use the PRs the user names. If they name an author or "assigned to me", list them with `gh pr list --author <login>` or `gh pr list --search "review-requested:@me"`.
- Check `baseRefName` on each. In a stacked PR, review only that PR's own diff: `git diff origin/<base>...origin/<head>`.
- Skip drafts and PRs not assigned to the user unless they ask for them.

## 2. Review

- Look for real problems: bugs, crashes, regressions, security gaps, missing cleanup, risky code with no tests. Skip style nits.
- For big or multiple PRs, run one review subagent per PR in parallel. Give it the diff command, the PR's goal, and what to focus on.
- Check every finding against the code at the PR head before keeping it. Drop anything you can't confirm.
- Keep must-fix and should-fix items. Leave minor items out unless the user asks for them.

## 3. Anchor each comment

- Put each comment on the line where the problem lives, on the new side of the diff.
- That line must be in the PR's diff, or the API rejects the comment. Check it against the hunks before posting.
- If the problem is a missing test, anchor it on the mock or the nearest test line.

## 4. Write the questions

Rules:

- Only questions. Never state the bug, the cause, or the fix. No severity labels, no "nit", no suggested code.
- Usually 2 or 3 short questions. The first sets up the scenario. The rest point at the piece of code that matters.
- Each question should have a clear "yes" or "no" answer that exposes the problem when the author answers honestly.
- Sound concerned and unsure, not like a lawyer cross-examining.
- Vary the openers. "What happens if…" is fine, but no more than about a third of the comments should start with it. Other openers: "is it possible…", "does … still…", "how does … handle…", "when …, does…", "could…", "is there anything that…", "where does … get…", "should this…", "do we know…".
- Lowercase all non-code text, including the first word of each sentence.
- Wrap all code in backticks: identifiers, props, hooks, message types, file names, literal values.

Before and after:

> before: What happens if this renders on the ProductDemo page? `SuggestedResponseGroup` calls `useChatState()`, which throws without a `ChatStateProvider`, and I don't see one on this route. Would the first suggested replies in full-screen Navigator mode send the call to the error page?
>
> after: what happens if this renders on the `ProductDemo` page? is that likely? does `useChatState()` work without a `ChatStateProvider`?

Examples, with varied openers:

> once the call ends, does anything clear these chips in the navigator chat? and should this send the same list the rail shows (`railResponses`), or is the raw list intended?

> if someone on a phone wants to type to the demo, is the navigator chat visible to them at that point? is there another way to open the composer?

> is `responses` the same array on every render? how often does this effect run during a call, and where does that leave someone who scrolled to the second chip?

> could the chips get wider without the row changing size, like when a font loads late? does anything here notice that?

> does `box` keep the same identity between renders? if `MediaTray` re-renders in the middle of a move, what does `lastBox` hold after that?

> when the chat unmounts while the demo is full screen on a phone, does anything tell the embed to leave full screen, like the framed path does with `overlay: false`?

> does `windowWidth` update if someone resizes their browser after the page loads? is it still in sync with `framed`?

> is checking the origin enough here, given that picks get sent back to `e.source`? what happens if a different same-origin window posts this message?

> since `MediaTray` is mocked here, is there a test that would catch it if the click area or the overlay cleanup broke?

## 5. Show the drafts first

List every draft comment with its `file:line`, grouped by PR. Post nothing until the user approves the wording.

## 6. Post

- Post one review per PR with `event: "COMMENT"`. Never approve or request changes unless the user asks.
- Confirm `commit_id` matches the PR's current `headRefOid`. If the head moved, recheck the lines first.
- Build the JSON payload in a file and post it:

```bash
gh api -X POST repos/<owner>/<repo>/pulls/<n>/reviews --input review.json
```

```json
{
  "commit_id": "<head sha>",
  "event": "COMMENT",
  "body": "",
  "comments": [
    { "path": "src/foo.tsx", "line": 42, "side": "RIGHT", "body": "does `bar` ...?" }
  ]
}
```

- A 422 "Git service unavailable" is usually transient. Check that nothing was posted (`gh api repos/<owner>/<repo>/pulls/<n>/comments`), then retry once.
- Afterwards, list your comments on each PR and confirm each one landed on its intended `path:line`. Report the review links.
