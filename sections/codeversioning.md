### [[↑]](../README.md#toc) <a name='codeversioning'>Questions about code versioning:</a>

#### Branching in HG and in Git
> Why is branching with Mercurial or git easier than with SVN?

**Expert Answer:**

**The Short Answer:** 
In Git, a branch is just a lightweight, 40-character text file pointing to a commit hash, whereas SVN physically copies the entire directory structure on the server.

**The Deep Dive:** 
SVN was designed around a centralized, directory-based structure. When you create a branch in SVN, the server essentially executes a massive `cp -r` command, copying your entire project into a `/branches/new-feature` folder. This is slow and heavy. 
Git uses a directed acyclic graph (DAG) of snapshots. A branch in Git is literally just a tiny file (e.g., `.git/refs/heads/feature-x`) containing the SHA-1 hash of the commit it points to. Branching takes less than a millisecond, happens entirely locally, and takes zero extra disk space regardless of how large the repository is. This encourages a culture of frequent, disposable feature branches.

**The Trade-offs (Pros/Cons):**
* **Pros (of Git branching):** Instantaneous; zero disk footprint; entirely offline.
* **Cons (of Git branching):** The concept of abstract pointers moving around a DAG can be conceptually difficult for beginners compared to SVN's physical folders.

**Code Example:**
```bash
# In Git, this command completes in < 1 millisecond. 
# It just creates a text pointer.
git checkout -b new-feature

# You can actually see the "branch" is just a hash!
cat .git/refs/heads/new-feature
# Output: 5f95c9f850b29849206d8a3952f1e29deed8e9cf
```

#### DVCS
> What are the pros and cons of distributed version control systems like Git over centralized ones like SVN?

**Expert Answer:**

**The Short Answer:** 
Distributed Version Control Systems (DVCS) give every developer a full, offline backup of the repository, offering immense speed and reliability, but suffer when handling massive binary files.

**The Deep Dive:** 
In SVN, the server is the single source of truth. To view history or commit, you need a network connection. 
In Git (DVCS), when you clone a repository, you download the *entire history of the project*. 
*   **The Pros:** Operations like `git log`, `git commit`, and `git diff` are instantaneous because they happen entirely on your local SSD. If GitHub goes down, development doesn't stop. If the central server is destroyed, any developer's laptop serves as a perfect backup.
*   **The Cons:** Because every clone downloads the full history, adding large, frequently changing binary files (like PSDs or 3D models) will bloat the repository size forever, making cloning painfully slow.

**The Trade-offs (Pros/Cons):**
* **Pros (Git):** Offline capability; blazing fast operations; no single point of failure.
* **Cons (Git):** Terrible for huge binary assets (requires Git LFS); steep learning curve regarding local vs. remote staging areas.

**Code Example:**
```bash
# These commands run instantly in Git because all history is local.
# No network request is made.
git log
git diff HEAD~1
git commit -m "Offline work"
```

#### GitFlow and GitHubFlow
> Could you describe GitHub Flow and GitFlow workflows?

**Expert Answer:**

**The Short Answer:** 
GitFlow is a strict, multi-branch model for scheduled releases, while GitHub Flow is a lightweight, trunk-based model optimized for Continuous Deployment.

**The Deep Dive:** 
*   **GitFlow:** A rigid model utilizing long-lived branches (`master`, `develop`) and specific short-lived branches (`feature`, `release`, `hotfix`). It is excellent for shrink-wrapped software or mobile apps where version `1.2.0` is cut, tested for weeks, and shipped. It is often too heavy and slow for modern web SaaS.
*   **GitHub Flow:** A trunk-based development model. There is only one long-lived branch (`main`). Developers branch off `main`, open a Pull Request, get it reviewed, merge back to `main`, and deploy to production *immediately*.

**The Trade-offs (Pros/Cons):**
* **Pros (GitFlow):** Highly structured; provides a safe `release` staging area; easy to support multiple historical versions.
* **Cons (GitFlow):** Causes "merge hell" when `feature` branches stay open for weeks; slows down velocity.
* **Pros (GitHub Flow):** Fast, continuous delivery; prevents merge conflicts by encouraging small, daily merges.

**Code Example:**
```bash
# GitHub Flow: Keep it simple and ship it.
git checkout -b feature/login-page
git commit -am "Add login page"
git push origin feature/login-page
# Open PR -> Merge to main -> CI/CD automatically deploys!
```

#### Rebase
> What's a rebase?

**Expert Answer:**

**The Short Answer:** 
A rebase moves a sequence of commits to a new starting point, rewriting history to create a perfectly linear project timeline without messy merge commits.

**The Deep Dive:** 
If you branch off `main` and make 3 commits, but meanwhile your team merges 10 commits into `main`, your branch is now out of date. 
If you `git merge main`, Git creates a new "merge commit" that ties the histories together, causing a messy, train-track history graph. 
If you `git rebase main`, Git temporarily sets aside your 3 commits, updates your branch to match the newest `main`, and then "re-plays" your 3 commits on top. 

**The Trade-offs (Pros/Cons):**
* **Pros:** Creates a beautiful, linear, readable history graph; makes `git bisect` much easier to use.
* **Cons (The Golden Rule):** Rebasing rewrites history by generating new commit hashes. You must **never** rebase a branch that has been pushed and shared with others, or you will cause catastrophic conflicts for your teammates.

**Code Example:**
```bash
git checkout feature-branch

# Fetch the latest code from the team
git fetch origin

# Replay my local feature commits ON TOP of the latest main
git rebase origin/main

# Because the hashes changed, I must force-push my branch
git push origin feature-branch --force-with-lease
```

#### Merging in HG and in Git
> Why are merges easier with Mercurial and Git than with SVN and CVS?

**Expert Answer:**

**The Short Answer:** 
Git tracks entire filesystem snapshots, whereas SVN tracks file modifications (deltas), giving Git a much deeper mathematical understanding of the repository's history graph to resolve conflicts automatically.

**The Deep Dive:** 
SVN sees history as a linear set of patches applied to files. If a file is heavily modified or renamed in SVN, it often loses track of the file's origin, resulting in a "Tree Conflict" that requires painful manual resolution.
Git, on the other hand, tracks *content snapshots*. When Git performs a merge, it looks at the common ancestor commit, the tip of your branch, and the tip of the target branch. By performing a 3-way merge on these complete snapshots, it is incredibly smart at resolving conflicts automatically—even if a file was moved to a completely different directory in one branch while being edited in another.

**The Trade-offs (Pros/Cons):**
* **Pros (of Git's snapshot merging):** "Merge anxiety" disappears; developers branch and merge multiple times a day effortlessly; automatically handles file renames.
* **Cons:** The underlying algorithms (like the recursive merge strategy) can be opaque when a complex conflict *does* occur.

**Code Example:**
```bash
# Git handles this complex scenario effortlessly:
# Branch A: Renames user.go to account.go
# Branch B: Modifies a function inside user.go
# Merging them will automatically apply the changes to account.go!
git merge branch-b
```
