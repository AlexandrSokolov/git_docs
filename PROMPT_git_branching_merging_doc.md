# Prompt: Build the Git Branching & Merging Workflow Doc

> Hand this file back to Claude when I'm ready to build the doc. It captures the
> scope, my context, and the repo-specific rules so we don't re-derive them.

---

## Role & goal

Act as my Git workflow mentor. Produce a **reference document** (Word `.docx`,
matching the style of my other project docs) covering branching and merging
best practices for the `allocadia/azuqua` repo — structured around **concrete
cases, traps, and best practices**, not abstract theory.

The doc is for me and eventually the rest of the Professional Services team, so
it must assume a reader who is competent with Git basics but has **not** worked
with disciplined multi-branch promotion workflows before.

## My context (calibrate to this)

- Strong backend background (Java/Maven/Spring). Frontend and Git-workflow
  corner cases are where I'm thin — flag those proactively.
- I came from GitLab and from **solo / small-team** work, where I committed and
  pushed freely and never had to think about history rewrites, promotion
  tracking, or shared-branch etiquette. So I have **gaps around corner cases**,
  not around the basics.
- I habitually **commit and push together** from a GUI (IntelliJ IDEA).
- I like: short statements, worked examples with **real commands**, and the
  **IntelliJ GUI equivalent** alongside each CLI command.
- I reason from first principles and want the *why*, not just the *what*. Build
  understanding before rules.

## Repo invariants the doc must respect (do not contradict these)

- Three branch types: `master` = production, `staging` = UAT/customer testing
  (~6 weeks ahead of master), and persistent per-form branches (one per form,
  reused for months).
- **Never merge `staging` → `master`.** Promote individual form branches only.
- Promotion of infra changes cut from a non-master base is done by
  **cherry-pick onto a fresh branch off master**, never a branch PR.
- New forms can go straight to master; existing live forms go through staging
  first. (This policy point is under review with Liz — treat as current, flag as
  provisional.)
- Liz (emckeown) holds merge authority; I cannot self-merge past her. She
  defaults to **plain merge commits**. All three merge types are enabled in
  the repo.
- I keep a **backup branch before any risky Git op** and use targeted
  `git add <file>`, not `git add -A`.

## Required structure

For each topic: **(1) the mechanic / mental model → (2) worked example with
commands + IntelliJ equivalent → (3) the trap(s) → (4) the best-practice rule.**
Use a running example form (e.g. `greenhouseCoupa`) throughout so examples chain
together. Include ASCII commit graphs where they clarify ancestry.

## Content checklist (cover all; expand each into a case)

1. **The one distinction everything rests on:** rewriting history makes *new
   hashes*; merging *preserves* old hashes as ancestors.
2. **`git branch --contains <hash>` as a promotion-tracking tool** — how it
   works (ancestry/reachability), and why I rely on it to see where a commit has
   landed (form branch / staging / master).
3. **Why merge commits beat squash *for my workflow*** — squash/rebase-merge
   rewrite and break `--contains`; plain merge commits preserve it. (Note this
   is a deliberate trade-off, not a universal rule — squash has its place on
   teams that don't track by hash.)
4. **Interactive rebase (`git rebase -i`)** — the to-do list model; the verbs
   `pick` / `squash` / `fixup` / `reword` / `drop`; deleting a line == drop
   (footgun in the raw editor). IntelliJ's Log → "Squash Commits" and
   "Interactively Rebase from Here" as the safer GUI path.
5. **The safe-rewrite window** — rebase is safe until commits are **merged into
   a shared branch** (staging/master) or someone else works off my branch.
   "Not pushed yet" is just the simplest safe case; pushed-but-solo-and-unmerged
   is still safe.
6. **The iron rule** — never rewrite a branch after it's been merged somewhere;
   spell out exactly what breaks (staging pointing at an orphaned hash, blind
   `--contains`, duplicate/conflicting history on the next promotion).
7. **`--force-with-lease` vs `--force`** — lease refuses instead of silently
   eating someone's commit; the stale-info rejection and how to recover
   (`fetch` → inspect → reconcile → re-push). The multi-machine self-collision
   case (rewrite on laptop, old commits on another machine).
8. **The clean-history-without-losing-tracking recipe** — commit messily → one
   local `rebase -i` cleanup *before the first PR* → freeze hashes → merge-commit
   to staging then master → `--contains` works at every step.
9. **Cross-references to promotion mechanics already established:** why a PR from
   a branch cut off `staging` silently drags all of staging's history, and why
   cherry-pick-onto-master is the fix. (Tie the branching hygiene here back to
   the merge-tracking theme.)

## Traps to make sure land (dedicated callouts)

- Squash silently breaks hash-based promotion tracking.
- Deleting a line in the raw `rebase -i` editor deletes the commit.
- Plain `--force` can delete a colleague's unfetched commit.
- Rewriting after merge = the orphaned-reference failure mode.
- Number of commits is **irrelevant to the Jenkins incremental build** (diff is
  by changed file paths, not commit count) — kill this misconception early so
  the reader doesn't optimize for the wrong thing.

## Style

- Match my existing `.docx` project docs in tone and formatting.
- Prose over bullet-walls; commands in code blocks; one running example.
- Every CLI command gets its IntelliJ GUI equivalent.
- Lead with the mechanic, then the rule — never rules without the why.

## When we build it

Ask me first whether anything in the repo invariants has changed (especially the
new-form-to-master policy, still pending with Liz), then draft section by section
with my confirmation between sections — my usual incremental rhythm.
