Having trouble understanding this flag from the push command [--force-if-includes](https://git-scm.com/docs/git-push#Documentation/git-push.txt---no-force-if-includes) ?

Here is an example to understand the --force-if-include flag. You need to understand --force-with-lease first.
--force-if-include is intended to be used with it only, and makes sense only when rewriting history. If you are just adding commits, meaning the remote can fast-forward when you push, well... just push without forcing :)

The following example combines : 
- rewriting history
- including remote changes
- preveting a bad force-push
- simple push not possible

```
[fig.1] Original situation

 A--B (orig/main)
  \
   C--D (main)
```
Here my work on main is C & D. Someone added B on orig/main.
If I perform *git push --force-with-lease --force-if-includes*, the push is rejected. So far so good : I prevented a force push with the second flag because B is not included into main, even if I fetched it.

Now let's do a rebase to include B in my history, between C and D. Because for some reason it makes more sense to have them in that order.
The interactive rebase *git rebase -i A* is written like this : 
```
pick C
pick B
pick D
```

It ends up like this : 

```
[fig.2]

 A--B (orig/main)
  \
   C--B'--D' (main)
```
B and D have new hashes on main because they are copies written after C, and D is now after B.
Perform again *git push --force-with-lease --force-if-includes* 
```
[fig.3]

 A--C--B'--D' (orig/main)(main)
```
The change is accepted because git sees B' as B integrated in main, this the additional check performed by --force-if-include gives a green light.
