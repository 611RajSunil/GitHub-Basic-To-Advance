# How GIT Works in Production

---

## Table of Contents

- [Introduction](#introduction)  
- [Rebasing](#rebasing)  
  - [Why Rebasing Exists](#why-rebasing-exists)  
  - [The Limitations of Merge](#the-limitations-of-merge)  
  - [Why Non-Linear History Becomes a Problem](#why-non-linear-history-becomes-a-problem)  
  - [What is Rebasing](#what-is-rebasing)  
  - [Rebasing: Concept, Flow, and Integration](#rebasing-concept-flow-and-integration)  
  - [Merge vs Rebase: Structure and Intent](#merge-vs-rebase-structure-and-intent)  
  - [**Demo:** Rebase in Practice](#demo-rebase-in-practice)  
    - [Step 1: Base setup](#step-1-base-setup)  
    - [Step 2: Create feature branch](#step-2-create-feature-branch)  
    - [Step 3: Parallel development (creating branch divergence)](#step-3-parallel-development-creating-branch-divergence)  
    - [Step 4: Rebase feature onto main (changing the base)](#step-4-rebase-feature-onto-main-changing-the-base)  
    - [Step 5: Conflict during commit replay](#step-5-conflict-during-commit-replay)  
    - [Step 6: Inspect and resolve conflict](#step-6-inspect-and-resolve-conflict)  
    - [Step 7: Final state and integration](#step-7-final-state-and-integration)  
    - [Step 8: Clean-up and observe final history](#step-8-clean-up-and-observe-final-history)  
  - [Production Workflow and Rebase in Practice](#production-workflow-and-rebase-in-practice)  
  - [When to Use Rebase vs Merge](#when-to-use-rebase-vs-merge)  
- [Interactive Rebase](#interactive-rebase-concept-and-intent)  
  - [Usage 1: Rebase + reshape commits](#usage-1-rebase--reshape-commits)  
  - [Usage 2: Reshape recent commits (local cleanup)](#usage-2-reshape-recent-commits-local-cleanup)  
  - [When to Use Rebase vs Interactive Rebase](#when-to-use-rebase-vs-interactive-rebase)  
  - [**Demo:** Interactive Rebase in Practice](#demo-interactive-rebase-in-practice)  
    - [Step 1: Base setup](#step-1-base-setup-1)  
    - [Step 2: Create working branch](#step-2-create-working-branch)  
    - [Step 3: Iterative development (noisy commits)](#step-3-iterative-development-noisy-commits)  
    - [Step 4: Identify need for cleanup](#step-4-identify-need-for-cleanup)  
    - [Step 5: Start interactive rebase](#step-5-start-interactive-rebase)  
    - [Step 6: Modify replay instructions, resolve, and finalize history](#step-6-modify-replay-instructions-resolve-and-finalize-history)  
    - [Step 7: Integration into main](#step-7-integration-into-main)  
- [Squash Merge](#squash-merge-concept-and-context)  
  - [Behavior and History Representation](#squash-merge-behavior-and-history-representation)  
  - [**Demo:** Squash Merge in Practice (GitHub + PAT)](#demo-squash-merge-in-practice-github--pat)  
    - [Step 1: Create Private Repository + Setup Authentication (PAT)](#step-1-create-private-repository--setup-authentication-pat)  
    - [Step 2: Initialize Local Repository and Configure Remote (PAT)](#step-2-initialize-local-repository-and-configure-remote-pat)  
    - [Step 3: Create Feature Branch and Push Changes](#step-3-create-feature-branch-and-push-changes)  
    - [Step 4: Create Pull Request (PR)](#step-4-create-pull-request-pr)  
    - [Step 5: Perform Squash and Merge](#step-5-perform-squash-and-merge)  
    - [Step 6: Deletion and Cleanup](#step-6-deletion-and-cleanup)  
- [Cherry-pick](#cherry-pick-concept-and-intent)  
  - [**Demo:** Cherry-pick in Practice](#demo-cherry-pick-in-practice)  
    - [Step 1: Base setup](#step-1-base-setup-2)  
    - [Step 2: Create feature branch and commits](#step-2-create-feature-branch-and-commits)  
    - [Step 3: Critical fix on main (source of cherry-pick)](#step-3-critical-fix-on-main-source-of-cherry-pick)  
    - [Step 4: Introduce conflicting change on feature](#step-4-introduce-conflicting-change-on-feature)  
    - [Step 5: Identify commit to cherry-pick](#step-5-identify-commit-to-cherry-pick)  
    - [Step 6: Cherry-pick into feature branch](#step-6-cherry-pick-into-feature-branch)  
    - [Step 7: Resolve conflict](#step-7-resolve-conflict)  
    - [Step 8: Final state](#step-8-final-state)  
- [Git Integration & History Strategies: Revision](#git-integration--history-strategies-revision)  
- [Conclusion](#conclusion)  
- [References](#references)  

---

## Introduction

This is **Advanced History & Integration Strategies (final part)** of the Git Masterclass, where we move beyond basic branching and merging to focus on **how Git history is structured, refined, and integrated in real-world workflows**.

In Foundations and Collaboration & Production Workflows, we covered:

* core Git concepts and commit fundamentals  
* branching, merging, and working with remotes  
* collaboration using Pull Requests  

In this Advanced History & Integration Strategies (final part), we shift focus from *creating history* to **shaping and controlling history**.

This becomes critical in production systems, where commit history must remain **clean, readable, and easy to reason about over time**.

We will cover:

* **Rebase** to align branches and maintain linear history  
* **Interactive Rebase** to clean and restructure commits before sharing  
* **Squash (platform-level)** to control how changes are recorded during integration  
* **Cherry-pick** to selectively apply specific commits across branches  

Each concept is explained both **theoretically and through hands-on demos**, focusing on real-world usage patterns and trade-offs.

> This part completes the Git Masterclass by bridging the gap between **using Git** and **using Git effectively in production workflows**.

---

## Rebasing

### Why Rebasing Exists

So far, we have learned how to **create branches**, **make commits**, **merge changes**, and collaborate using **Pull Requests**. This allows teams to build and integrate changes safely.

However, one important aspect is still missing:

> **How do we keep commit history readable and easy to reason about?**

This question becomes critical when **integrating changes across branches**, where multiple development timelines come together and shape the final history.

---

### The Limitations of Merge

In practice, development is incremental. Developers make multiple small commits while building changes. When this history is merged as-is, two distinct types of issues emerge.

**1. Commit-level issues (content quality)**
These relate to how individual commits are written:

* low-quality commit messages such as *“fix”*, *“update”*, or *“final-final”*
* intermediate commits reflecting trial-and-error

**2. History-level issues (structure)**
These relate to how commits are connected:

* merge commits introduce branching and joining
* history becomes **non-linear (multiple paths between commits)**

> **Important distinction:** Rebase addresses only **history structure**, not commit content.
> It does not fix commit messages or remove intermediate commits.
> While it may result in fewer commits than a merge (by avoiding a merge commit), it does **not reduce or eliminate the original commits**.

A **non-linear history** means there are **multiple possible paths between certain commits**, typically introduced by merge commits (which have more than one parent). This makes the commit graph harder to follow.

For example, after a merge:

~~~text
A --- B --- C --- F -------- M
            \               /
             D ----------- E
~~~

From **M to C**, you can reach C in two ways:

* M → F → C
* M → E → D → C

This is what makes the history **non-linear**.


> Merge preserves structure as it happened; rebase restructures it without altering content.

> **Note:** In plain English, *linear* means something that follows a **straight line**, with a clear start and end. *Non-linear* means it does not follow a single straight path and instead has **branches or multiple possible routes**.
> The same idea applies to Git: a *linear history* follows a single sequence of commits, while a *non-linear history* contains branching and merging, creating multiple paths between commits.

> **Important:** Not all merges create non-linear history.
> A **fast-forward merge** does not create a new commit, so history remains linear.
> Non-linearity is introduced only when Git creates a **merge commit**, which has **multiple parents** and therefore introduces multiple paths in the commit graph.

---

### Why Non-Linear History Becomes a Problem

Consider a typical workflow where multiple developers work in parallel and changes are integrated using merge:

~~~text
A --- B --- C --- F -------- M
            \               /
             D ----------- E
~~~

Here, work diverges at **C**, progresses independently (**F** and **D → E**), and is later combined using a merge commit **M**. Because **M has two parents**, the history now contains **multiple paths between commits**.

For example, from **M to C**, there are two valid paths:

* M → F → C
* M → E → D → C

This is what makes the history **non-linear**. The structure is no longer a single sequence, but a graph with branching and joining.

In smaller projects, this is usually manageable because there are fewer branches and merges. However, in real-world systems with multiple developers, frequent branching (feature, bugfix, release), and continuous integration, this structure becomes significantly more complex.

It is important to note that:

> **Merge preserves the exact sequence of how work happened, including divergence and integration points**

This makes the history **accurate**, but introduces structural complexity. As branches diverge and merge repeatedly:

* the graph accumulates multiple branching and joining points
* commits may be reachable through different paths
* the notion of “what came next” is no longer obvious

This has practical consequences. Reading history now requires interpreting a graph rather than following a straight line. Code reviews become harder because changes are spread across branches and merge commits. Debugging becomes more complex since identifying when a change was introduced may involve navigating multiple paths. Even tooling like `git log` or `git bisect` requires more context in such histories.

Over time, the issue is not correctness of history, but **cognitive load**. The history remains technically accurate, but becomes harder to read, reason about, and work with at scale.

> **Why care about history if the working branch will be deleted?**
>
> Even though the working branch is temporary, the **commits created on that branch do not disappear**. When the branch is merged (via merge, rebase, or squash), those commits or their final representation become part of the **main branch history**, which is permanent and shared.
>
> This history is critical because it is used for:
>
> * **understanding what changed and why** (context for future developers)
> * **debugging and root cause analysis** (e.g., `git blame`, `git bisect`)
> * **code reviews and audits** (clear, meaningful commits improve traceability)
> * **long-term maintainability** of the codebase
>
> If the history is noisy or poorly structured, the main branch becomes **hard to read, reason about, and debug over time**.
>
> > The branch may be temporary, but its history lives on in the main line of development.

---

### What is Rebasing

Let’s say you created a branch `feature/login` (this could be a feature, bugfix, or hotfix branch) from `main` (this could be any primary branch such as `main`, `production`, `release`, etc.).

In most of our examples so far, we followed a simple flow where work happens on a branch and is eventually merged back into `main`. This can give the impression that changes always move in one direction, from the feature branch to the primary branch. However, in real-world development, this is only part of the picture.


While you are working on your branch, the primary branch (such as `main`) continues to receive new commits. In most production environments, this happens through **pull requests**, since the primary branch is typically **protected and does not allow direct commits**.

Over time, this means that your branch, which was created from an earlier point, is now behind the latest state of the primary branch. At that stage, before continuing further development or before integrating your work back, you would typically want to **bring the latest changes from the primary branch into your branch** so that you are working on the most up-to-date system state.

---

There are **two main ways** to bring changes from the primary branch into your working branch, along with a couple of **less common approaches**.


1. **Merge (`git merge main`)**

   * integrates changes from the primary branch into your branch while **preserving the exact history of how development occurred**, including any divergence between branches

   * retains the **branching structure**, meaning the commit graph continues to reflect parallel lines of development

   * a **merge commit is created only when there is divergence**, i.e., both branches have new commits after their common ancestor

     * this merge commit has **two parents** (one from each branch), which introduces multiple paths in the commit graph
     * as a result, history becomes **non-linear**, clearly showing where branches diverged and were later integrated

   * if there is no divergence (for example, your branch has no new commits beyond the common ancestor):

     * Git performs a **fast-forward merge**, where the branch pointer simply advances
     * no merge commit is created, and the history remains **linear**, even though a merge operation was performed

---

2. **Rebase (`git rebase main`)**

   * achieves the same end goal of bringing your branch up to date, but instead of merging histories, it **moves your branch onto the latest state of the primary branch and reapplies your work on top of it**
   * this makes it appear as if your work was originally developed from the latest version of the primary branch, rather than from an older base

   Internally, Git performs the following sequence of operations:

   * identifies commits in your branch that are **not present in the primary branch**

   * temporarily removes those commits from your branch

   * shifts the branch reference to the **latest commit of the primary branch**

   * reapplies the removed commits one by one on top of this new base

   * because these commits are **recreated during replay**, they receive **new commit hashes**, even though the underlying changes remain the same

   The end result is that:

   * your branch contains:

     * all the latest changes from the primary branch
     * your work applied on top in sequence
   * the history becomes a **single linear progression**, with no visible branching structure or merge commits

---

> For completeness, there are a couple of additional approaches, but they are **not commonly used for this purpose**:
>
> * **Cherry-pick (`git cherry-pick <commit>`)**
>   Used when you want to bring **specific commits** from `main` into your branch, rather than the entire set of changes. This is a selective operation and not a full synchronization strategy. We will **cover this in detail later in this lecture**.
>
> * **Reset (`git reset --hard main`)**
>   Moves your branch to match the state of `main`, effectively **discarding all commits on your branch that are not present in `main`**. This is a destructive operation and is typically used only in recovery or cleanup scenarios. We have **already discussed this earlier in the course**.

---

This leads to the key conceptual difference:

* **Merge** preserves how development actually happened, including divergence and integration points
* **Rebase** restructures history to present a clean, linear sequence of changes as if development happened in a single flow

Rebasing does not change the final code, but it changes how the history is **structured and interpreted**.

> **Note:** Rebasing does not modify the content of commits; it only **repositions them onto a new base**, recreating them with new hashes. It does not combine, remove, or alter commits by itself. Any such modifications require **interactive rebase**, which provides explicit control over how commits are replayed.

---

## Rebasing: Concept, Flow, and Integration

Rebasing is a technique used to **move a branch onto a new base commit**.

When we say:

> rebase `feature/login` onto `main`

we mean:

> take the commits created in `feature/login` and **replay them on top of the latest state of `main`**, making it appear as if the work started from that newer point.

---

Consider the following evolution:

```text
Initial state (branch created from C):

A --- B --- C   (main, feature/login)

After parallel development:

A --- B --- C --- F   (main)
            \
             D --- E   (feature/login)
```

* `C` is the **common ancestor**
* `main` has progressed to **F**
* `feature/login` has additional commits **D, E**
* both branches have **diverged after C**

---

Rebasing integrates this work by **changing the base of the feature branch**:

* identify commits not present in `main` → **D, E**
* temporarily remove these commits
* shift the branch reference from **C → F**
* replay those commits one by one on top of **F**

```text
After rebase:

A --- B --- C --- F --- D' --- E'   (feature/login)
```

This leads to the following outcomes:

* the base of the branch is now **F instead of C (from a history perspective)**
* commits are **recreated** as **D', E'** (new hashes)
* the branch appears as if it was developed from the latest `main`
* the previous divergence is no longer visible

Although it now looks like the branch started from `F`, this is a **result of history rewriting**, not an actual change in the original creation point.

The resulting history is **linear**, meaning there is a single path through commits with no branching structure, improving readability without altering the final code.

> Rebase rewrites history. To anyone inspecting the commit graph, it appears as if the branch started from the latest `main`, and the original divergence (C → D → E) is no longer visible. Internally, Git still keeps references to the previous state for some time, which can be inspected or recovered using **`git reflog`**. However, this information is **local and temporary**, and not part of the shared history.

---

## Merge vs Rebase: Structure and Intent

> **Note:** In earlier examples, we merged feature branches into `main`, which can suggest a one-way flow. In practice, it is equally common to **bring `main` into a feature branch** to stay aligned with the latest system state and reduce conflicts.

When integrating divergent branches, both merge and rebase can be used to **bring the latest changes from `main` into a working branch (`feature/login`)**. They produce the **same final code**, but differ in how history is represented.

---

### Merge (preserves true history)

```text
Before merge:

A --- B --- C --- F   (main)
            \
             D --- E   (feature/login)

After `git merge main` on feature branch:

A --- B --- C --- F
            \       \
             D --- E --- G   (feature/login)
```

* original commits (**D, E**) are preserved
* a **merge commit (G)** is created on the feature branch
* the merge commit has **two parents** → `F` (main) and `E` (feature)
* history reflects actual parallel development

Merge is faithful to reality. It preserves divergence and shows exactly how work evolved. However, over time this can introduce:

* multiple merge commits
* visible branching structures
* interleaved timelines

which can make history harder to read at scale.

---

### Rebase (reshapes history for clarity)

```text
Before rebase:

A --- B --- C --- F   (main)
            \
             D --- E   (feature/login)

After `git rebase main`:

A --- B --- C --- F --- D' --- E'   (feature/login)
```

* commits are **recreated (D', E')** on a new base
* no merge commit is introduced
* history becomes **linear and sequential**
* it appears as if the branch started from **F**

Rebase presents the same final code as merge, but **restructures history into a clean progression of changes** applied on top of the latest `main`.

> **Note:** When bringing changes from `main` into a feature branch, both `merge` and `rebase` are executed from the feature branch (`git checkout feature/login`). The difference is whether you **preserve history (merge)** or **rewrite it (rebase)**.
> Since rebasing rewrites history, it should not be used on shared branches, as it can cause divergence and conflicts for other collaborators.

---

After rebasing, the feature branch already contains the latest changes from `main`, so integration into `main` becomes straightforward:

```bash
git checkout main
git merge feature/login
```

Since `main` is now **behind** `feature/login` with no divergence, Git performs a **fast-forward merge**:

```text
A --- B --- C --- F --- D' --- E'   (main, feature/login)
```

* no merge commit is created
* no additional conflict resolution is required
* `main` simply advances to the tip of `feature/login`

Any conflicts that could have occurred are handled during the **rebase phase**, when commits are replayed.

---

## Demo: Rebase in Practice

### Step 1: Base setup

```bash
git init git-rebase-demo  
cd git-rebase-demo  

echo "Application initialized" > app.txt  
git add app.txt  
git commit -m "chore: initialize application"  

echo "Updated application header styling" >> app.txt  
git commit -am "style: improve application header styling"  

echo "Added footer section to application" >> app.txt  
git commit -am "feat: add footer section"  
```

> Once a file has been staged at least once, `git commit -am` can be used to automatically stage and commit subsequent modifications to that file in a single step.

> In this demo, we are **committing directly to the `main` branch** to establish a simple base state.
> In real-world or production repositories, you typically **do not start by committing directly to `main`**. Instead, you would first ensure your local repository is up to date with the remote:

```bash
git fetch  
git pull  
```

> This ensures you are working with the **latest changes from the remote repository**, especially when multiple developers are contributing.
> We covered this in detail in Part-2, particularly in the **“Working with Remotes”** section.

```text
A --- B --- C   (main)
```

---

### Step 2: Create feature branch

~~~bash
git checkout -b feature/login  
~~~

~~~bash
A --- B --- C   (main, feature/login)
~~~

---

### Step 3: Parallel development (creating branch divergence)

At this stage, we intentionally make changes on both branches to **create divergence**, so that we can later **rebase `feature/login` onto `main`** and observe how conflicts are handled.

**On `main`**

~~~bash
git checkout main  

echo "CloudWithVarJosh teaches real-world DevOps" > cwvj.txt  
git add cwvj.txt  
git commit -m "feat: introduce CWVJ learning philosophy content"  
~~~

**On `feature/login`**

~~~bash
git checkout feature/login  

echo "CloudWithVarJosh focuses on production-grade training" > cwvj.txt  
git add cwvj.txt  
git commit -m "feat: add CWVJ positioning for login module"  

echo "Login module initial structure added" >> app.txt  
git add app.txt  
git commit -m "feat: scaffold login module structure"  
~~~


**State now**

~~~bash id="7hx6ne"
A --- B --- C --- F   (main)
            \
             D --- E   (feature/login)
~~~

* Both branches have progressed independently after **C**
* Same file `cwvj.txt` created on both branches
* Different content ensures a **deterministic rebase conflict**

~~~bash
git log --oneline --all --graph

* 992834f (HEAD -> feature/login) feat: scaffold login module structure
* 106738e feat: add CWVJ positioning for login module
| * 435fd70 (main) feat: introduce CWVJ learning philosophy content
|/
* 10f8cee feat: add footer section
* 4dac7f3 style: improve application header styling
* 511fcc3 chore: initialize application
~~~

* Both branches have progressed independently after the common base
* `main` contains commit **F**, while `feature/login` contains **D and E**
* Same file `cwvj.txt` created independently on both branches
* Different content ensures a **deterministic rebase conflict**

> **About `git checkout` vs `git switch`**
>
> Throughout this course, we have been using `git checkout`, but Git also provides a modern command `git switch` for the same purpose.
>
> `git checkout` is an older, multi-purpose command used to switch branches, restore files, and move to specific commits (detached HEAD), which can make its intent confusing.
>
> `git switch` is designed only for switching branches, making it clearer and safer for beginners.
>
> In modern workflows, `git switch` is preferred, but `git checkout` is still widely used in production and commonly seen in real-world projects.
>
> You can use either for this demo, the behavior remains the same.

---

### Step 4: Rebase feature onto main (changing the base)

```bash
git checkout feature/login  
git rebase main  
```

We run rebase from `feature/login` because **rebase rewrites the current (checked-out) branch**.

* `main` remains unchanged
* `feature/login` is moved from base **C → F** (from a history perspective)
* commits **D, E** are temporarily removed and prepared for replay

> Rebase does not pull changes into your branch; it **moves your branch onto a new base and reapplies your commits**

> `rebase-i` in the prompt indicates an in-progress rebase using Git’s internal sequencer. It does not necessarily mean an interactive rebase was explicitly invoked.

---

### Step 5: Conflict during commit replay

```bash
CONFLICT (add/add): raj.txt  
```

Conflict occurs while replaying commit **D**.

* `main` already has `cwvj.txt` (from commit **F**)
* `feature/login` also created `cwvj.txt` (commit **D**)
* both commits introduce the same file independently with different content
* Git cannot automatically decide which version to keep

> A conflict is not specific to merge or rebase.
> It occurs whenever **changes cannot be cleanly combined during integration or replay**

In this case, it is an **add/add conflict**:

* the same file is created independently in both branches
* the file contents differ

**When would conflict NOT occur?**

* different files modified → no conflict
* same file, different non-overlapping lines → usually auto-merged
* identical changes → no conflict

> During rebase, conflicts are resolved **commit by commit**, as each commit is replayed.

---

### Step 6: Inspect and resolve conflict

~~~bash
cat raj.txt  
~~~

~~~text
<<<<<<< HEAD
Demo about Git Masterclass
=======
We are focuses on production-grade training
>>>>>>> feat: add RAJ positioning for login module
~~~

* `HEAD` → current base (`main` / F)
* incoming change → commit being replayed (D)

Resolve manually (example: combine):

~~~text
We learn real-world DevOps  
We focuses on production-grade training  
~~~

~~~bash
git add raj.txt  
git rebase --continue  
~~~

* commit **D → D'** recreated with resolved content
* remaining commits (**E → E'**) replay automatically

> In rebase, conflicts are resolved **per commit during replay**, not all at once

---

### Step 7: Final state and integration

After rebase:

```bash
A --- B --- C --- F --- D' --- E'   (feature/login)
```

* base successfully shifted to **F**
* history rewritten as a **linear sequence**
* conflict resolution embedded into new commits

Now integrate into `main`:

```bash
git checkout main  
git merge feature/login  
```

```bash
A --- B --- C --- F --- D' --- E'   (main, feature/login)
```

* fast-forward merge
* no merge commit
* no additional conflicts

> Conflicts were already resolved during rebase, so merge becomes a **simple pointer movement**

> Since there is **no divergence between branches**, Git performs a **fast-forward merge**, which simply **moves the `main` pointer forward** to the tip of `feature/login`. No content merging happens here; the final state of `cwvj.txt` is exactly what was **resolved earlier during the rebase conflict**.

---

### Step 8: Clean-up and observe final history

~~~bash
git branch -d feature/login  
git log --oneline --graph --all  
~~~

~~~bash
* 6f2aaac (HEAD -> main) feat: scaffold login module structure
* a59a70a feat: add CWVJ positioning for login module-RebaseConflict
* 435fd70 feat: introduce CWVJ learning philosophy content
* 10f8cee feat: add footer section
* 4dac7f3 style: improve application header styling
* 511fcc3 chore: initialize application
~~~

At this point, notice that the history appears as a **single straight line of commits**, with no visible branching or merge commits. It looks as if all changes were developed sequentially on `main`, even though in reality the work was done on a separate feature branch.

This is the direct outcome of rebasing. During rebase, we **moved `feature/login` onto the latest state of `main` and replayed its commits**, resolving conflicts along the way. As a result, when we merged back into `main`, Git performed a **fast-forward merge**, which simply moved the branch pointer forward without creating any merge commit or performing additional content merging.

It is important to contrast this with what we would have typically done without rebase. To bring the latest changes from `main` into `feature/login`, we could have run:

~~~bash id="g3x9zt"
git checkout feature/login  
git merge main  
~~~

This would have introduced a **merge commit on the feature branch**, preserving the branching structure. Later, when merging `feature/login` back into `main`, another merge (or fast-forward depending on state) would occur, resulting in a history that explicitly shows divergence and integration points.

A typical merge-based history would look like:

~~~bash
*   c139484 (HEAD -> main) merge commit
|\
| * 9ffc105 feat: introduce CWVJ learning philosophy content
* | 0e406ef feat: scaffold login module structure
* | 0825b83 feat: add CWVJ positioning for login module
|/
* fff769c feat: add footer section
* f3830bd style: improve application header styling
* 3e7c0d6 chore: initialize application
~~~

In this structure, the branching is explicit. While this accurately reflects how development happened, it introduces **non-linear history**, which can become increasingly complex as more developers and branches are involved.

With rebasing, we chose a different approach. Instead of preserving the branching structure, we **rewrote the feature branch history before integration**, so that:

* all changes from `main` were already part of `feature/login`
* conflicts were resolved earlier during development
* integration back into `main` became a simple fast-forward

This results in a **clean, linear history**, which is significantly easier to read, review, and debug, especially in large codebases with many contributors.

> Rebasing does not remove the fact that branching occurred; it removes the *evidence of divergence* from the commit history by rewriting it into a linear sequence.

---

### Production Workflow and Rebase in Practice

> In this demo, we merged `feature/login` into `main` locally to clearly demonstrate how rebase leads to a **fast-forward merge**.

In real-world workflows, this process differs slightly:

* the `main` branch is usually **protected**

  * you **cannot push directly to `main`**
  * all changes are integrated via **Pull Requests (PRs)**

---

The typical production flow is:

* work on your feature branch (`feature/login`)
* rebase your branch with the latest `main`
* push your feature branch to the remote
* create a **Pull Request (PR)** from `feature/login` → `main`

After approval:

* the platform (GitHub, GitLab, etc.) performs the integration  
* the outcome depends on the **strategy selected in the Pull Request**:

  * **Create a merge commit** → always creates a merge commit, even if fast-forward is possible (GitHub behavior)  
  * **Rebase** → commits are replayed onto the target branch, resulting in linear history without a merge commit  
  * **Squash** → all commits are combined into a single commit on the target branch  

---

### When to Use Rebase vs Merge

In practice, your goal is to keep your work **aligned with the latest state of `main` before integration**, while ensuring collaboration remains safe.

---

* **Rebase (on working branches)**

  * used to **bring the latest changes from `main` into your branch**
  * moves your branch onto the **latest commit of `main`**
  * replays your commits on top, creating a **linear history**
  * ensures your work is based on the **current system state before integration**
  * helps resolve conflicts **early during development**
  * avoids unnecessary merge commits in the feature branch

  > This is why we rebase **onto a shared branch (`main`)**: not to change `main`, but to ensure your work is built on top of the latest version of it.

---

* **Why shared branches are not rebased**

  * rebasing **rewrites commit history (new hashes)**
  * shared branches are used by **multiple developers simultaneously**
  * rewriting history would cause:

    * divergence across local clones  
    * duplicate commits after sync  
    * broken or confusing histories for collaborators  

  * requires **force push**, which can overwrite others’ work

  > Because of this, shared branches like `main` are treated as **stable, immutable history** and are **not rebased**.

---

* **Why rebasing feature branches is usually safe**

  * feature branches are typically **owned by a single developer**
  * history rewrite affects **only that developer’s local branch**
  * no other collaborators depend on that history
  * allows developers to **clean and structure commits before sharing**

  * however, if a feature branch is **shared between multiple developers**:

    * rebasing will rewrite history for everyone  
    * can lead to confusion and conflicts  
    * in such cases, teams typically **avoid rebasing and use merge instead**

---

* **Merge (into shared branches like `main`)**

  * used to **integrate completed work into shared history**
  * does **not rewrite existing commits**
  * preserves collaboration safety across developers
  * provides a **stable and predictable integration mechanism**

---

> Rebase is used to **prepare and align your branch**, while merge is used to **safely integrate that work into shared history**.

> The choice is not about correctness of code, but about **how history is managed for collaboration and maintainability**.

---

## Interactive Rebase: Concept and Intent

In the previous section on rebase, we saw that when we run:

```bash
git rebase main
```

Git takes **all commits from the current (checked-out) branch that are not present in the target branch (`main`)**, temporarily removes them, moves the branch to the latest commit of `main`, and then **replays those commits one by one** on top of it.

This process is **automatic and deterministic**:

* all eligible commits are replayed
* order is preserved
* no opportunity to modify commits during replay

> Rebase operates on the **current branch**, not specifically on “feature” or “main”. These are just examples. In practice, rebase can be run from any branch onto any other branch.
> However, since rebase **rewrites history (new commit hashes)**, it is typically used on **working branches owned by an individual developer**. Rebasing shared branches (like `main`) is avoided because it would **disrupt history for all collaborators**.

> Rebase = automatic replay of all commits on a new base

While this is useful for aligning your branch and achieving a linear history, it has limitations:

* you cannot **combine related commits** into a single logical unit  
* you cannot **reorder commits** to improve readability  
* you cannot **edit commit messages or modify commit content during replay**  
* you cannot **remove unnecessary or noisy commits**  

In real-world development, commits are often:

* iterative (WIP, fixes, small adjustments)
* noisy or overly granular
* not structured for code review or long-term readability

Before sharing such work, especially via a pull request, it is often desirable to **reshape the commit history** into a cleaner and more meaningful sequence.

This is where interactive rebase comes in.

---

### Usage 1: Rebase + reshape commits

```bash
git checkout feature/login  
git rebase -i main
```

> Rebase operates on the **current (checked-out) branch**, so we explicitly switch to `feature/login`.

Git performs the same rebase operation, but instead of directly replaying commits, it opens a **todo list of commits** that will be replayed, allowing you to:


* **Reorder commits** (rearrange lines in todo list): present changes in a logical sequence for better readability and review
* **Combine commits (squash / fixup)** (`squash`, `fixup`): group related work into a single meaningful unit instead of noisy incremental commits
* **Modify commit messages** (`reword`): make commit history clear, descriptive, and useful for future understanding
* **Edit commit content** (`edit`): fix or refine changes before sharing, without creating extra commits
* **Remove commits entirely** (`drop` or delete line): eliminate unnecessary or incorrect changes from history before it is shared



```text
Before:

A --- F --- G   (main)
        \
          B --- C --- D   (feature/login)
```

* feature branch was created from **F**
* `main` has progressed to **G**
* commits **B, C, D** represent iterative work (feature → fix → refactor)

Assume we choose to **squash and clean history** during interactive rebase:

```text
pick   B feat: add login UI  
squash C fix: adjust login UI alignment  
squash D refactor: cleanup login module  
```

* `pick` → keep commit **B as the base commit**
* `squash` → combine commits **C and D into B**
  > squash means: “merge this commit into the previous commit”
* changes from C and D are **not lost**, they become part of the final commit

Internally, during replay, Git effectively does:

* apply **B**
* take **C → merge into B**
* take **D → merge into B + C**

```text
After:

A --- F --- G --- X   (feature/login)
```

* branch is now **aligned with latest main (G)**

* commits **B, C, D → X (single logical commit)**

* history is both **updated and cleaned**

* base moved from **F → G**

* commits were not just replayed, but **transformed during replay**

> Rebase = automatic replay
> Interactive rebase = **controlled replay (with transformation)**

> In this example, we used `squash` for commits C and D, merging them into B, resulting in a single combined commit **X**.
> However, interactive rebase allows other reshaping options as well. For example, if you had only modified commit messages (using `reword`), you would still have three separate commits, but rewritten as **B′, C′, D′** with updated messages.


> Without interactive rebase, the result would have been:
> `A --- F --- G --- B' --- C' --- D'`

---

### Usage 2: Reshape recent commits (local cleanup)

```bash
git rebase -i HEAD~3
```

Instead of referencing another branch (`main`), this command operates on the **last N commits of the current branch**.

* no alignment with `main` is performed
* only the selected commits are **reshaped locally**

```text
Before:

A --- B --- C --- D   (feature/login)
```

* commits **B, C, D** represent iterative work

  * B → feature
  * C → fix
  * D → refactor

Assume we choose to **clean this into one logical commit**:

```text
pick   B feat: add login UI  
squash C fix: adjust login UI alignment  
squash D refactor: cleanup login module  
```

Internally:

* apply **B**
* take **C → merge into B**
* take **D → merge into B + C**

```text
After:

A --- X   (feature/login)
```

* commits **B, C, D → X (single logical commit)**
* history is **cleaned and simplified**
* base remains unchanged (still rooted at A)

---

This gives us two clear ways to use interactive rebase:

* `git rebase -i main` → **align with latest base + clean commits**
* `git rebase -i HEAD~N` → **clean recent commits only (no alignment)**

---

Interactive rebase is primarily used to:

* prepare a branch for **pull request and review**
* convert **noisy development history into meaningful commits**
* ensure the shared history is **clean, readable, and intentional**

> Interactive rebase is not about changing what was done, but about **presenting it in a structured and meaningful way before sharing**.

---

## When to Use Rebase vs Interactive Rebase

Both rebase and interactive rebase operate on the same principle of **replaying commits on a new base**, but they serve different purposes in a typical workflow.

* **Use rebase (`git rebase main`) when the goal is alignment**

  * bring your branch up to date with the latest `main`
  * incorporate upstream changes before continuing development
  * resolve conflicts early at the commit level
  * maintain a **linear history without merge commits**

* **Use interactive rebase (`git rebase -i main`) when the goal is cleanup**

  * restructure commits before sharing code
  * combine related changes into **logical units (squash/fixup)**
  * remove noisy or unnecessary commits
  * improve commit messages and overall readability

In practice, both are often used together:

* rebase ensures your branch is **based on the latest state of main**
* interactive rebase ensures your commits are **clean and meaningful**

> Rebase answers: “Is my branch up to date?”
> Interactive rebase answers: “Is my history ready to be shared?”

> **When should you use rebase vs interactive rebase?**
> Interactive rebase is **more powerful**, but it’s **not always needed**. If your goal is to **bring your branch up to date**, a normal rebase is **faster and safer**. Interactive rebase is used when you want to **shape and clean your commit history**. It provides **more control**, but also requires **manual decisions**, so it is used **intentionally, not by default**.

---

## Demo: Interactive Rebase in Practice

### Step 1: Base setup

```bash
git init git-interactive-rebase-demo  
cd git-interactive-rebase-demo  

echo "Initial app setup" > app.txt  
git add app.txt  
git commit -m "chore: initial setup"  
```

```text
A   (main)
```

* `main` represents **shared, stable branch**
* no history rewriting will be performed here

---

### Step 2: Create working branch

```bash
git checkout -b feature/login  
```

```text
A   (main, feature/login)
```

* all development happens on **feature branch**
* safe to rewrite history here (locally owned)

---

### Step 3: Iterative development (noisy commits)

```bash
echo "Add login UI" >> app.txt  
git commit -am "feat: add login UI"  

echo "Fix login UI alignment" >> app.txt  
git commit -am "fix: adjust login UI alignment"  

echo "Refactor login UI code" >> app.txt  
git commit -am "refactor: cleanup login UI code"  
```

```text
A --- B --- C --- D   (feature/login)
```

* multiple small commits representing iterative development
* history is **functionally correct but not clean**

---

### Step 4: Identify need for cleanup

```bash
git log --oneline  
```

```text
D refactor: cleanup login UI code  
C fix: adjust login UI alignment  
B feat: add login UI  
A chore: initial setup  
```

* commits **B, C, D** are logically related
* represent a single feature developed incrementally
* better represented as **one clean commit**

---

### Step 5: Start interactive rebase

```bash
git rebase -i HEAD~3  
```

* `HEAD~3` selects last 3 commits (**B, C, D**)
* Git opens editor with:

```text
pick B feat: add login UI  
pick C fix: adjust login UI alignment  
pick D refactor: cleanup login UI code  
```

> Order is **oldest → newest**, even though `git log` shows newest first

> Different selections are possible depending on the requirement:
>
> * `HEAD~2` → last 2 commits
> * `HEAD~5` → last 5 commits
> * `git rebase -i <commit-hash>` → from a specific commit onward
>
> The range determines **which part of history you want to rewrite**

---

### Step 6: Modify replay instructions, resolve, and finalize history

```text
Before interactive rebase:

A --- B --- C --- D   (feature/login)
```

We now control how commits are replayed.

```text
pick   B feat: add login UI  
squash C fix: adjust login UI alignment  
squash D refactor: cleanup login UI code  
```

Git provides multiple operations:

```text
p, pick    = use commit  
r, reword  = use commit, but edit the commit message  
e, edit    = use commit, but stop for amending  
s, squash  = use commit, but meld into previous commit  
```

* `pick` → keep commit as the **base commit**

  * chosen because **B best represents the feature intent**
  * subsequent commits will be merged into this

* `squash` → combine commit into the previous one

  * changes from **C and D are NOT lost**
  * they become part of the final combined commit

> `fixup` is similar to squash, but **discards the commit message**
> `reword` allows editing only the commit message
> `edit` pauses to modify commit content

---

Git now combines commits and prompts for the **final commit message**:

```text
feat: add login UI  

fix: adjust login UI alignment  
refactor: cleanup login UI code  
```

Clean it:

```text
feat: implement login UI with alignment fixes and cleanup  
```

* multiple commits now form **one logical unit**
* history becomes **clean and meaningful**

---

```text
After interactive rebase:

A --- X   (feature/login)
```

* commits **B, C, D → X**
* new commit created → **new hash**
* history rewritten as a **single linear unit of work**

---

#### Alternative (when commits should remain separate)

```text
pick   B feat: add login UI  
reword C fix: correct login UI alignment for responsive layout  
pick   D refactor: cleanup login UI code  
```

```text
Before:

A --- B --- C --- D   (feature/login)

After interactive rebase:

A --- B --- C' --- D   (feature/login)
```

* commits are **not combined**, but can be **refined individually**
* `reword` keeps the commit but **updates its message (C → C')**, resulting in a new hash

```text
Before:
fix: adjust login UI alignment  

After:
fix: correct login UI alignment for responsive layout  
```

* `pick` keeps commits **B and D unchanged in intent** (hashes may still change due to history rewrite)
* useful when commits represent **distinct logical steps** and should remain separate
* preserves **granularity of changes** while improving clarity and reviewability

> In actual Git flow, you mark `reword` first, and Git then opens an editor to modify the message. Here we show the final intent directly for clarity.

> Interactive rebase is not always about reducing commits; it is about **making each commit meaningful and intentional**

---

### Step 7: Integration into main

```bash
git checkout main  
git merge feature/login  
```

```text
A --- X   (main, feature/login)
```

* fast-forward merge
* no merge commit
* clean, linear history preserved

In production environments, `main` is typically a **protected branch**, so direct merges like this are not allowed. Instead, the workflow is:

* push the feature branch to remote:

```bash
git push origin feature/login  
```

* raise a **Pull Request (PR)** from `feature/login` → `main`
* CI checks and code review are performed
* once approved, the PR is merged

Because we used **interactive rebase earlier**:

* the commit history is **clean and minimal (A → X)**
* reviewers see **one logical unit of work**, not multiple noisy commits
* improves **review clarity and maintainability**

The merge performed via PR will typically result in:

* **fast-forward merge** (if linear history is enforced), or
* **squash merge** (common in many teams, still results in a single clean commit)

> Interactive rebase ensures that what you share is **intentional history**, not raw development noise, making collaboration and code review significantly more effective.

> **About `git checkout` vs `git switch`**
>
> Throughout this course, we have been using `git checkout`, but Git also provides a modern command `git switch` for the same purpose.
>
> `git checkout` is an older, multi-purpose command used to switch branches, restore files, and move to specific commits (detached HEAD), which can make its intent confusing.
>
> `git switch` is designed only for switching branches, making it clearer and safer for beginners.
>
> In modern workflows, `git switch` is preferred, but `git checkout` is still widely used in production and commonly seen in real-world projects.
>
> You can use either for this demo, the behavior remains the same.

---

### Key Takeaways for Interactive Rebase

* interactive rebase is used on **working branches before PR**
* used to **clean and structure commits**, not to sync
* results in **better review and maintainability**
* main remains **stable and untouched by history rewrites**

> Interactive rebase is a **pre-integration cleanup step**, not an operation performed on shared branches.

---

## Squash Merge: Concept and Context
> In this section, **“squash merge” refers to the platform-level integration strategy** used during PR merge.
> This distinction is intentional to avoid confusion with the `squash` operation used in **interactive rebase**, which is a **local history rewrite**.

In the previous section, we worked with **interactive rebase**, which can be used in two ways:

* `git rebase -i main`: align with another branch while reshaping commits during replay
* `git rebase -i HEAD~N`: reshape commits locally without integrating with another branch

In both cases, we used operations like:

* `squash` / `fixup`: combine multiple commits into one
* `reword`: improve commit messages

These operations are performed **locally on a working branch**, allowing you to **control how commits are replayed and structured before sharing code**.

> The `squash` used in interactive rebase operates only on **local branch history**, irrespective of whether commits are being aligned with another branch or simply cleaned up.

In real-world workflows, code is not integrated by merging locally into `main`. Instead:

* changes are pushed to a remote repository
* a **Pull Request (PR)** is created
* integration is performed on the **platform (GitHub, GitLab, Bitbucket)**

At this stage, platforms provide multiple **merge strategies**, which determine how the final history is recorded.

> Squash merge is one such strategy and is a **remote-side operation performed during PR merge**, not a local history editing step.

This leads to a critical distinction:

* **Interactive rebase (with squash)**: local history reshaping before sharing
* **Squash merge**: integration-time history consolidation on the platform
* interactive rebase prepares and structures your branch
* squash merge controls how that work is recorded in shared history

---

## Squash Merge: Behavior and History Representation

When integrating a feature branch via a Pull Request on platforms like GitHub or GitLab, there are **three integration options available** (as labeled in GitHub UI):

1. **Create a merge commit** (merge commit strategy)
   3-way merge execution, merge commit creation, full history preservation

2. **Rebase and merge** (rebase strategy)
   commits rebased and replayed individually onto target branch, linear history without a merge commit

3. **Squash and merge** (squash strategy)
   all source commits combined into one, resulting in a single commit on the target branch

> These are **platform-level integration strategies**, not local Git operations.
> Notably, **squash is not available as a direct local integration operation**, it is performed during PR merge.

> **Note on naming:** Platforms like GitHub use terms such as **“Create a merge commit”**, **“Rebase and merge”**, and **“Squash and merge”**.
> However, only the first option actually performs a true Git *merge commit*.
> The other two modify how commits are applied before integration.

> **Terminology simplification (used in this course):**
>
> * Merge → Create a merge commit
> * Rebase → Rebase and merge
> * Squash → Squash and merge

> **Important:** Rebasing a working branch onto `main` is commonly performed **locally by developers** to keep their branch up to date before creating a PR.
>
> However, the **“Rebase and merge” option on platforms** refers to an **integration strategy during PR merge**, and is not always the default choice across teams. Many teams prefer **merge or squash** for integration, depending on their workflow and history preferences.

---

Instead of redefining them further, we will now **observe how each strategy affects history**, with a focus on **squash**.

---

Consider a feature branch developed iteratively:

```text
A --- B --- C   (main)
        \
          D --- E --- F   (feature/login)
```

* branching point: feature created from **B**, not latest `main`
* divergence state: `main` advanced to **C**, feature progressed independently
* commit nature: `D` (implementation), `E` (fixes), `F` (refactor/cleanup)
* history quality: reflects **development process**, not final logical structure

---

#### 1. **Merge (merge commit strategy)**

```text
A --- B --- C -------- M   (main)
        \              /
          D --- E --- F
```

* merge behavior: **3-way merge** using common ancestor
* merge commit: new commit **M** with two parents
* platform behavior: **merge commit created even if fast-forward is possible**
* history shape: **non-linear**, branch structure preserved
* commit preservation: all commits (`D, E, F`) retained as-is

---

#### 2. **Rebase (rebase strategy)**

```text
A --- B --- C --- D' --- E' --- F'   (main)
```

* rebase behavior: commits **rebased onto latest `main` and replayed sequentially**
* commit transformation: `D, E, F → D', E', F'` (**new commit hashes**)
* history shape: **linear**, no merge commit created
* commit preservation: all commits retained but **recreated (history rewritten)**

---

#### 3. **Squash (squash strategy)**

```text
A --- B --- C --- X   (main)
```

* squash behavior: `D, E, F → X` (**single consolidated commit**)
* history shape: **linear**, no merge commit created
* abstraction level: feature represented as **one logical change**
* message control: commit message **defined at merge time**

Example:

```text id="sqmsg"
feat: implement login UI with alignment fixes and cleanup
```

---

## Demo: Squash Merge in Practice (GitHub + PAT)

### Step 1: Create Private Repository + Setup Authentication (PAT)

* Repository name: `raj-git-practice`
* Visibility: **Private**

When working with **enterprise applications**, you will almost always deal with **private repositories**. Public repos are rare in enterprise environments due to security and IP concerns.

For authentication in this demo, we will use a **GitHub Personal Access Token (PAT)**.

* Earlier in the course, we used **SSH**, which is commonly used by **individual developers** for interacting with repositories.
* Here, we intentionally switch to **PAT** to demonstrate **token-based authentication over HTTPS**.
* In real enterprise setups, you will often see **SSO-based authentication** integrated with identity providers.

  In such setups:

  * SSO handles **user authentication and identity verification**
  * Git operations are typically performed over **HTTPS**, using **tokens issued after SSO authentication**
  * SSH may still be used in some environments, but access is often **centrally managed or restricted**

---

#### Create the PAT

GitHub → Settings → Developer Settings → Personal Access Tokens

* Create **Fine-grained token**
* Name: `raj-squash-demo`
* Expiration: 7 days (policy-dependent in real setups)
* Repository access: **Selected repositories**
* Select: `raj-git-practice`

Permissions:

* Click **Add permissions → Repositories → Contents**
* Set **Contents → Read and Write**

> This is the **minimum required permission** to clone, push, and pull code.

> Copy the token immediately. It will not be shown again.

---

### Step 2: Initialize Local Repository and Configure Remote (PAT)

```bash
git init raj-git-practice  
cd raj-git-practice  

echo "Initialize RAJ platform" > app.txt  
git add app.txt  
git commit -m "chore: initialize RAJ platform"  

echo "Add RAJ core configuration" >> app.txt  
git commit -am "feat: add RAJ core configuration"  
```

```text
A --- B   (main)
```

Configure the remote repository and authenticate using the PAT:

```bash
git remote add origin https://github.com/Our GitHub Account name /Our Github account repo name.git  
git remote set-url origin https://<GITHUB-PAT>@github.com/Our GitHub Account name /Our Github account repo name.git  
```

* repository is initialized locally with **baseline commits (A, B) on `main`**
* remote is added using **HTTPS (not SSH)**
* PAT is embedded in the URL for **HTTPS-based authentication with the private repository**

---

### Step 3: Create Feature Branch and Push Changes

Create a feature branch from the current state:

```bash
git checkout -b feature/raj-auth  
```

Continue work on the feature branch:

```bash
echo "Add RAJ authentication module" >> app.txt  
git commit -am "feat: add RAJ authentication module"  

echo "Fix RAJ authentication validation logic" >> app.txt  
git commit -am "fix: adjust RAJ authentication validation"  
```

```text
A --- B   (main)
        \
          D --- E   (feature/raj-auth)
```

* feature branch created from **B**
* `main` remains unchanged after branching
* feature branch progresses with **D, E**
* commits reflect **incremental development (feature → fix)**

---

Push both branches to the remote repository:

```bash
git push -u origin main  
git push -u origin feature/raj-auth  
```

* establishes upstream tracking for both branches
* publishes complete state (**A, B on main and D, E on feature**)
* enables creation of a **Pull Request (feature/cwvj-auth → main)**

---

### Step 4: Create Pull Request (PR)

On GitHub:

* Navigate to the repository `raj-git-practice`
* Create a Pull Request: `feature/raj-auth` → `main`

GitHub will display:

```text
D feat: add RAJ authentication module  
E fix: adjust RAJ authentication validation  
```

* PR shows the **complete commit history of the feature branch**

* reviewers can:

  * inspect **individual commits** (via *Commits* tab)
  * review the **combined diff** (via *Files changed* tab)
  * provide feedback at **file or commit level**

* CI pipelines (if configured) will run:

  * build, test, lint checks
  * enforce quality gates before merge

* GitHub also shows:

  * **checks status** (e.g., all checks passed)
  * **mergeability status** (no conflicts with base branch)
  * available **integration strategies**:

    * Create a merge commit
    * Squash and merge
    * Rebase and merge

> Open the dropdown and select **“Squash and merge”**.
> Notice that GitHub explicitly indicates that the **2 commits will be combined into a single commit** in the base branch.

> **Note:** Platforms use labels like “Squash and merge” and “Rebase and merge”.
> These do not always represent a true Git “merge”.
> Only the **Merge** option creates a merge commit.
> Squash and Rebase change how commits are applied before integration.

---

### Step 5: Perform `Squash and Merge`

On the PR page:

* Choose **“Squash and merge”** (instead of “Create a merge commit”)
* From the dropdown, select **Squash and merge**

> Notice that GitHub indicates the **2 commits will be combined into a single commit** in the base branch.

GitHub prepares a **single combined commit** from all PR commits:

* takes the **net effect (diff) of D and E**
* applies it on top of the latest `main`
* opens a commit message editor

Example:

```text
feat: implement CWVJ authentication with validation fixes and cleanup
```

* this message should represent the **entire feature as a single logical unit**
* replaces multiple commit messages with a **clean, meaningful summary**

Confirm merge.

```text
A --- B   (main)
        \
          D --- E   (feature/cwvj-auth)

Squash (platform)

A --- B --- X   (main)
```

> Squash merge ignores how the work evolved (D → E) and captures only the **final state of changes**.

---

### Step 6: Deletion and Cleanup

Once the PR is **approved and merged**:

* the changes from `feature/raj-auth` are now part of the **main line of development (`main`)**
* the feature branch has served its purpose and is typically **deleted**

```bash 
git branch -d feature/raj-auth             # delete locally  
git push origin --delete feature/raj-auth  # delete on remote  
```

* avoids clutter from stale branches
* ensures future work starts from **latest main**

---

Delete the repository from GitHub (demo cleanup):

* Navigate to repository **Settings**
* Scroll to **Danger Zone**
* Click **Delete this repository**
* Confirm by typing the repository name: `cwvj-git-practice`
* removes the demo repository from remote
* ensures no leftover resources remain after the exercise


> After integration, the feature branch is no longer needed because its changes are already represented in `main` as commit **X**.
> The repository itself can also be safely deleted once the demo is complete.

---

## Cherry-pick: Concept and Intent

What we have seen so far is the integration of an **entire branch into another branch**. We explored both **merge** and **rebase**, which are used to bring changes from one branch into another.

However, both of these operate at the **branch level**:

* they consider **all commits in the branch**
* they integrate the **entire line of development**

This raises an important practical question:

> what if you don’t want the entire branch, and instead only need a **specific commit** from that branch?

This is where **cherry-pick** comes in.

As the name suggests, cherry-pick allows you to **pick a specific commit and apply it onto another branch**, without bringing the rest of the branch history.

**When would you use cherry-pick?**

* apply a **critical bug fix** from `main` to a feature branch without merging everything  
* **backport fixes** from `main` to release or hotfix branches  
* move a **small, stable change** without bringing incomplete feature work  
* selectively reuse a **useful commit** from another branch without full integration

---

When we run:

```bash
git cherry-pick <commit-hash>
```

Git takes the **changes introduced by that commit** and applies them on the **current (checked-out) branch**, creating a **new commit with a new hash**.

This is fundamentally different from merge or rebase:

* **merge** → brings the **entire branch history**
* **rebase** → replays **all commits onto a new base**
* **cherry-pick** → applies **only selected commit(s)**

---

Consider the following scenario:

```text
A --- B --- C   (main)
      \
        D --- E   (feature/cwvj-auth)
```

* `main` contains commits **A, B, C**
* `feature/cwvj-auth` was created from **B**
* commit **C** contains a **critical bug fix** needed in the feature branch

Instead of merging or rebasing the entire branch, we selectively apply only that commit:

```text
A --- B --- C        (main)
      \
        D --- E --- C'   (feature/cwvj-auth)
```

* commit **C is applied as C'** on the feature branch
* a **new commit hash is created**, even though the change is identical
* only the selected commit is applied, not the entire branch

---

**Key Pointers:**
* cherry-pick operates on **individual commits, not branches**
* creates a **new commit (C')**, even if content is identical
* does not preserve original commit relationships or branch structure
* useful for applying a **critical fix without merging unrelated changes**
* commonly used to **backport fixes to release or hotfix branches**
* helps selectively move **stable changes across branches**
* if overused, can lead to **duplicate commits across branches**

> Cherry-pick extracts **what changed**, not **how the change was developed across the branch**.

---

## Demo: Cherry-pick in Practice

### Step 1: Base setup

```bash
git init raj-cherry-pick-demo  
cd raj-cherry-pick-demo  

echo "Initialize RAJ application" > app.txt  
git add app.txt  
git commit -m "chore: initialize RAJ application"  

echo "Add RAJ core module" >> app.txt  
git commit -am "feat: add RAJ core module"  
```

```text
A --- B   (main)
```

---

### Step 2: Create feature branch and commits

```bash
git checkout -b feature/raj-auth  

echo "Add RAJ authentication module" >> app.txt  
git commit -am "feat: add RAJ authentication module"  

echo "Refactor authentication flow" >> app.txt  
git commit -am "refactor: cleanup authentication flow"  
```

```text
A --- B   (main)
        \
          D --- E   (feature/raj-auth)
```

---

### Step 3: Critical fix on main (source of cherry-pick)

Now simulate a **bug fix directly on main**:

```bash
git checkout main  

echo "Fix authentication validation logic" >> app.txt  
git commit -am "fix: resolve authentication validation issue"  
```

```text
A --- B --- C   (main)
        \
          D --- E   (feature/raj-auth)
```

* commit **C** is a **critical fix**
* feature branch does not have this fix

---

### Step 4: Introduce conflicting change on feature

Modify the same logical area to **force a conflict**:

```bash
git checkout feature/raj-auth  

echo "Update authentication validation logic (feature version)" >> app.txt  
git commit -am "feat: update authentication validation logic"  
```

```text
A --- B --- C   (main)
        \
          D --- E --- F   (feature/raj-auth)
```

---

### Step 5: Identify commit to cherry-pick

> Ensure you are on the **feature branch**, since this is where the commit will be applied.

```bash
git checkout feature/raj-auth  
git log --oneline main  
git log --oneline --graph --all  
```

Example:

```text
a1b2c3 fix: resolve authentication validation issue  
```

* commit **C exists on `main`**, but is **not part of our current branch (`feature/cwvj-auth`)**
* we will **pick this commit from `main` and apply it onto the feature branch**

---

### Step 6: Cherry-pick into feature branch

```bash
git cherry-pick <commit-hash>  
```

* replace `<commit-hash>` with the **commit you want to pick from `main`**
* Git applies the changes from that commit onto the **current (feature) branch**

💥 Conflict occurs (if overlapping changes exist):

```text
CONFLICT (content): Merge conflict in app.txt
```

---

### Step 7: Resolve conflict

Edit `app.txt` manually and combine both changes logically.

Then:

```bash
git add app.txt  
git cherry-pick --continue  
```

---

### Step 8: Final state

```text
A --- B --- C   (main)
        \
          D --- E --- F --- C'   (feature/raj-auth)
```

* commit **C is applied as C'** (new hash)
* conflict was resolved during application
* feature branch now contains the **critical fix**
* original history remains unchanged

---

## Git Integration & History Strategies: Revision

| Operation              | Scope of Operation        | What It Does                                | History Impact                                | Commit Identity                 | Merge Commit          | Typical Use Case                                 | Risk / Caveat                                         |
| ---------------------- | ------------------------- | ------------------------------------------- | --------------------------------------------- | ------------------------------- | --------------------- | ------------------------------------------------ | ----------------------------------------------------- |
| **Merge**              | Entire branch             | Combines branch histories using 3-way merge | Non-linear history, preserves structure       | Original commits preserved      | Yes (or fast-forward) | Safe integration, preserve full development flow | History can become noisy and harder to navigate       |
| **Rebase**             | Branch commits (replayed) | Replays commits onto new base               | Linear history, rewritten commit sequence     | New commits (new hashes)        | No                    | Clean history before merging                     | Rewrites history, unsafe on shared branches           |
| **Interactive Rebase** | Commits within branch     | Reorders, squashes, edits commits locally   | Cleaner, curated history                      | New commits after rewrite       | No                    | Clean up commits before sharing                  | History rewrite requires force push                   |
| **Squash (Platform)**  | All commits in PR         | Combines all commits into single commit     | Linear, simplified history                    | Single new commit replaces many | No                    | Keep main clean, one commit per feature          | Loses granular commit history on main                 |
| **Cherry-pick**        | Individual commit(s)      | Applies specific commit onto current branch | Adds commit without altering branch structure | New commit with same changes    | No                    | Hotfixes, backports, selective changes           | Duplicate commits across branches, conflicts possible |

> Merge brings everything, rebase reshapes everything, squash compresses everything, cherry-pick selects exactly what you need.

---

## Conclusion

With this, we conclude the **Git Basic To Advance (Foundations → Advanced History & Integration Strategies)**.

In this (Advanced History & Integration Strategies) final part, we focused on how Git is not just a tool for tracking changes, but a system for **structuring, refining, and communicating history**.

We covered:

* **Rebase** for aligning branches and maintaining linear history  
* **Interactive Rebase** for cleaning and shaping commit history before sharing  
* **Squash (platform-level)** for controlling how changes are recorded during integration  
* **Cherry-pick** for selectively applying specific commits across branches  

Across all three parts, the key idea is:

> Git is not just about writing code, but about **representing how that code evolved in a way others can understand, review, and maintain**.

While the final code remains the same across different strategies, the **history you create directly impacts**:

* readability and code reviews  
* debugging and root cause analysis  
* collaboration across teams  
* long-term maintainability of the codebase  

The goal of this masterclass was not just to teach commands, but to help you develop the ability to:

* choose the right strategy based on context  
* understand trade-offs between approaches  
* work effectively in real-world, production environments  

> At this point, you should be comfortable not only using Git, but also **reasoning about history and making informed decisions about how it is structured**.

---

## References

* Git Documentation  
  https://git-scm.com/docs  

* Pro Git Book (Scott Chacon & Ben Straub)  
  https://git-scm.com/book/en/v2  

* GitHub Docs – About Pull Requests  
  https://docs.github.com/en/pull-requests  

* GitHub Docs – About Merge Methods  
  https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository  

* Atlassian Git Tutorials  
  https://www.atlassian.com/git/tutorials  

* Git Rebase Documentation  
  https://git-scm.com/docs/git-rebase  

* Git Cherry-pick Documentation  
  https://git-scm.com/docs/git-cherry-pick 
