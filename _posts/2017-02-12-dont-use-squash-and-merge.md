---
layout: post
title: Don't use squash and merge
---

As software engineers, we care about detail. We care about precision. We care about cleanliness and purity. So when we're accepting a pull request in GitHub and we're offered the option to Squash and Merge, to take the contribution and compress it into one clean commit right on the stack of commits, we're tempted to elect that option. There are several reasons I hope will persuade you to avoid using this option except in the most constrained environments.

## Lost History

Both squash-and-merge and rebase will re-write history, but sqash-and-merge is more aggressive, collapsing and reducing a series of commits into a single commit. Sometimes, a change is readily represented as a single commit, and it was only originally committed as several commits due to factors completely unrelated to the software product itself. It was mistakes made by one developer or artifacts of some editor or code management scheme. These extrinsic factors are rarely the driving force behind multiple commits in a pull request. Much more often, the history of commits represents a thought process. Collapsing the commits destroys that history and makes that thought process essentially irretrievable. Some have argued that the commits do remain in the requesting user's repo or with that user, but in practice, tracing a change back to commits outside of the canonical code repository is a much higher burden, eliminating all but the most ambitious cases of forensics.

This lost history includes lots of important detail about how a solution was reached, such as separating a refactoring from a functional change. This separation of concerns helps isolate changes such that they can be considered separately.

But the history retention and separation of concerns doesn't mean the changes can't be considered in concert. That's what a merge commit is all about. A merge commit effectively takes the aggregate effect of a series of changes and combines them into a single commit. So with a merge commit, you get the best of both worlds, a single commit showing the result of the change, plus the history of how that change came into being.

## Lost Attribution

In addition to lost history of the changes to the source code, a squash-and-merge also loses important aspects of attribution. Because multiple commits are combined, lost are the metadata about those commits, specifically the authors, commit dates, date of merge, and other related details such as which issues or pull requests were referenced. Only if the squash commit somehow manages to capture these details is this important information not lost. And even if those details are captured, because a squashed commit doesn't have a structured way to store this information, it's essentially irretrievable without human intervention.

## Introduction of Merge Conflicts

In some workflows, it's possible to have multiple pull requests in at the same time, some of which depend on the acceptance of other pull requests. A contributor may author these pull requests in such a way that they are reconciled to work together. If, however, the target project has a policy (or whim) to use squash-and-merge, the acceptance of the first pull request will likely invalidate the dependent request, as the commit history is no longer related and resolved conflicts are re-introduced.

# Conclusion

Merge commits are in nearly every way superior. You can always squash a merge commit later, but you can never take a squash commit and revive it into its constituent changes. The only downside is that more commits are present and add to the activity of the history. This activity is often of value and is rarely just noise.

So don't squash and merge. Humanity and your future self will thank you for it.
