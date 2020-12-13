---
layout: post
title: Prefering main to master
---

I recently noticed that git and Github and others have been transitioning away from the use of `master` as the default branch name in source code repos, preferring instead `main`. Since new repos were being created with this new default, I decided it would be worthwhile to unify my existing repos to match the future expectation. Since I maintain hundreds of repos, I'd like to automate the steps necessary to make the transition.

[Xonsh](https://xon.sh/) is my shell of choice, so I set out to write the routine. I quickly came up with the following:

```
def retire_master():
    git checkout master
    git pull
    git checkout -b main
    gpo
    hub api -X PATCH /repos/@(get_repo_url()) -F default_branch=main
    git push origin :master
    git branch -d master

aliases['retire-master'] = retire_master
```

This routine depended on two existing routines I already had:

```
def git_push_new_branch():
    head = $(git rev-parse --abbrev-ref HEAD).rstrip()
    git push --set-upstream origin @(head)

aliases['gpo'] = git_push_new_branch

def get_repo_url():
    raw = $(git remote get-url --push origin).rstrip()
    return raw.replace('https://github.com/', '')
```

With this in the `.xonshrc`, I could readily invoke `retire-master` in a repo and it would take care of making sure the latest code was present, create the new branch locally, push that branch to the origin, then invoke the Github API to set the default branch to this new branch, then finally to delete the old branch remotely and locally.

Unfortunately, I discovered that deleting the default branch has consequences for repos with outstanding pull requests - all of the requests to that branch get closed summarily.

So I added a bit more sophistication to update any outstanding pull requests:

```
def move_prs(repo):
    import json
    prs = json.loads($(hub api /repos/@(repo)/pulls?base=master))
    nums = [pr['number'] for pr in prs]
    for pr_num in nums:
        hub api -X PATCH /repos/@(repo)/pulls/@(pr_num) -F base=main
```

This routine, given a repo path, would query for any pulls targeting the master branch and then update each PR. Then, one small change to the retire function puts it all together:

```
def retire_master():
    git checkout master
    git pull
    git checkout -b main
    gpo
    repo = get_repo_url()
    move_prs(repo)
    hub api -X PATCH /repos/@(repo) -F default_branch=main
    git push origin :master
    git branch -d master
```

Now, running the `retire-master` command will readily make the switch in any repo. I've switched several today and will get through the rest as I encounter them.
