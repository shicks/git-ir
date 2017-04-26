# git-ir
Git interactive rebase on steriods. Handles multiple branches at once.

## Usage

```
git ir --onto NEWROOT BRANCH1 BRANCH2...
git ir --onto NEWROOT --from OLDROOT1 --from OLDROOT2 --all
```

Required arguments include a single `--onto` commit, and at least one existing
branch (though the latter may be specified via `--all`, which include all child
branches of `--from` commits).

This brings up an editor for interactive rebase, which allows all standard
rebase commands, plus some extras.

## Extended Rebase Commands

In addition to the standard `pick`, `reword`, `edit`, `squash`, `fixup`, and
`exec`, four more commands are allowed:

* `exec!` (or `!`): similar to `exec`, but re-pushes itself upon failure.
  Continuing the rebase will immediately re-run the command, and immediatey
  halt if it continues to fail.
* `branch` (or `b`): moves a branch. All passed-in branches will result in
  a `b` line being added to the initial todo list. Removing the command will
  not delete the branch, but will leave it unchanged. No branches are actually
  moved until after all commits are successfully rebased.
* `push` (or `(`): stores the current commit onto a stack, allowing to handle
  non-linear groups of branches. The initial todo list will have these added as
  needed to preserve the original tree structure. Indentation is also inserted
  to make the structure clearer.
* `pop` (or `)`): pops the most recently pushed commit and resets to it.

## Example

Assume the following initial tree:

```
               A              B    C
a----b----c----d----e----f----g----h
 \         \
  \     D   \          E         F
   i----j    k----l----m----n----o
    \         \
     \     G   \     H
      p----q    r----s

```

where uppercase letters represent branches, and lowercase represent commit
hashes. The command

```
git ir --onto A --from f --from c --all D
```

would result in the following initial todo list:

```
(
  pick g
  branch B
  pick h
  branch C
)
(
  pick k
  (
    pick l
    pick m
    branch E
    pick n
    pick o
    branch F
  )
  (
    pick r
    pick s
    branch H
  )
)
(
  pick i
  pick j
  branch D
)
```

Branches `B` and `C` were rebased starting at commit `g` (skipping `e` and `f`)
because `f` was listed as a `--from` root. Branches `E`, `F`, and `H` were
similarly rooted at `c`. Branch `D` was rooted at `a`, which is the closes
common ancestor with the `--onto` commit. Branch `G` was not rebased at all
because it was neither explicitly specified, nor reachable from any `--from`
commit.

At this point, the commits can be rearranged and the tree structured changed
arbitrarily. If we rearrange a bit,

```
(
  pick g
  branch B
  pick h
  branch C
  pick i
  pick j
  branch D
)
(
  pick k
  pick m
  (
    pick l
    branch E
    pick o
    branch F
    pick r
    pick s
    branch H
  )
)

```

then the following is the result:

```
               A    B    C         D
a----b----c----d----g----h----i----j
 \              \
  \     G        \          E    F         H
   p----q         k----m----l----o----r----s
```
