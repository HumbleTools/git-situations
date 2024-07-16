# --force-if-includes EXPLAINED
Having trouble understanding the flag [--force-if-includes](https://git-scm.com/docs/git-push#Documentation/git-push.txt---no-force-if-includes) from the push command ? This article is for you.

## Prerequisites
You need to understand [--force-with-lease](https://git-scm.com/docs/git-push#Documentation/git-push.txt---force) first. `--force-if-includes` is intended to be used with it only, and makes sense only when rewriting history. If you are just adding commits, meaning the remote can fast-forward when you push, well... just push without forcing instead :)
Remember to only force-push when rewriting history on the remote.

Also, note that here the flag `--force-with-lease` is used WITHOUT any refname here. If you add a refname, the `--force-if-includes` flag will be ignored, as it would make no sense.

## When should I use it ?
Still reading ?  OK I hope you digested `--force-with-lease` first, you've been warned ! :)

The `--force-if-includes` flag should be used to perform a more secure force-push, to make sure you have integrated in your local changes the work someone else might have pushed on the remote. It means that both of these conditions should be met to make this flag useful : 
- You want to rewrite history (AKA force-pushing)
- You are working on a SHARED branch (AKA used by someone else)
- You do not want to loose any work from the remote even when force-pushing

## When should I NOT use it ?
- If you are not using [--force-with-lease](https://git-scm.com/docs/git-push#Documentation/git-push.txt---force)
- If you are working alone on the branch you want to push to
- If you rebased all your commits on top of the remote
- If just want to add commits on top of the remote

If any of these conditions are met, then just safe `push` to add commits. If rewriting your own history on a branch you are the only one working on for sure, just use `push --force` to rewrite history.

## Example usecase

The following example combines :
- using a shared branch 
- rewriting history
- including remote changes in local history
- preveting a bad force-push that would discard remote commits

```
[fig.1] Original situation

 A--B (origin/main)
  \
   C--D (main)
```
Here your work on main is C & D. Someone added B on origin/main.
If you perform *git push --force-with-lease --force-if-includes*, the push is rejected. So far so good : you prevented a force push with the second flag because B is not included into main, <u>even if you fetched it before</u>, as the schema is showing.

Now let's do a rebase to include B in your local history, between C and D. Because for some reason you decided it makes more sense to have them in that order.
The interactive rebase `git rebase -i A` is written like this : 
```
pick C
pick B
pick D
```

It ends up like this : 

```
[fig.2]

 A--B (origin/main)
  \
   C--B'--D' (main)
```
B and D have new hashes on main because they are copies written after C, and D' is now after B'.
Perform again `git push --force-with-lease --force-if-includes` 
```
[fig.3]

 A--C--B'--D' (origin/main)(main)
```
The change now is accepted because git sees B' as B integrated in main.

The flag `--force-with-lease` made sure the remote's HEAD was still on B when you pushed, preventing you from bypassing any unfetched commits ahead of B.

The flag `--force-if-includes` made sure that some form of B existed in the new history of main before accepting the force push, preventing you from discarding B when force-pushing, yet accepting a whole different history.

After [fig.2], a simple, secure `git push` would have been rejected because your local main branch has diverted. A `git push --force-with-lease` would have succeeded but would have been the same as doing a `git push --force`, as you fetched B before. B being the commit the flag has taken it's *lease* on, and the remote's HEAD being still on B, git would not have stopped you. `--force-if-includes` is intended for that specific purpose : adding a check to make sure the remote's HEAD is fully integrated locally.
