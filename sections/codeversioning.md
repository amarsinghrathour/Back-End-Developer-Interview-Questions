### [[↑]](../README.md#toc) <a name='codeversioning'>Questions about code versioning:</a>

#### Branching in HG and in Git
> Why is branching with Mercurial or git easier than with SVN?

**Expert Answer:**
In SVN, branching is literally copying the entire directory structure to a new folder on the server. It is slow and creates heavy server-side directories. 
In Git, a branch is just a lightweight pointer (a text file containing a 40-character SHA-1 hash) to a specific commit. Branching takes less than a millisecond, is completely local, and takes zero extra disk space regardless of the repository size. This fundamental difference makes branching a cheap, everyday operation in Git, encouraging feature branches and experimentation.

#### DVCS
> What are the pros and cons of distributed version control systems like Git over centralized ones like SVN?

**Expert Answer:**
*   **Pros (Git):** You have a full, offline copy of the entire repository history. You can commit, branch, diff, and rebase without an internet connection. It is exponentially faster because operations are local. It prevents single points of failure (if GitHub goes down, every developer still has a full backup).
*   **Cons (Git):** The learning curve is steep because developers must understand concepts like the staging area (index) and local vs remote repositories. Large binary files can bloat the local repository forever since every client downloads the full history (though Git LFS mitigates this).
*   **Centralized (SVN):** Simpler to understand (it's just "the server" and "my files"). Better for massive game development repositories with gigabytes of binary assets.

#### GitFlow and GitHubFlow
> Could you describe GitHub Flow and GitFlow workflows?

**Expert Answer:**
*   **GitFlow:** A strict branching model utilizing multiple long-lived branches (`master`, `develop`) and short-lived branches (`feature`, `release`, `hotfix`). It is highly structured and excellent for software with scheduled, versioned releases (e.g., shrink-wrapped software, mobile apps). However, it is often too heavy and slow for modern web backends.
*   **GitHub Flow:** A lightweight, trunk-based development model. There is only one long-lived branch (`main` or `master`). Developers branch off `main`, commit, open a Pull Request, get it reviewed, merge back to `main`, and deploy immediately. It is ideal for SaaS and Continuous Deployment (CD).

#### Rebase
> What's a rebase?

**Expert Answer:**
`git rebase` is the process of moving or combining a sequence of commits to a new base commit. 
If you branch off `main` and make 3 commits, but meanwhile `main` has moved forward, a rebase takes your 3 commits and "re-plays" them on top of the newest `main`. 
*   **Advantage:** It creates a perfectly linear, clean project history without messy merge commits.
*   **Danger:** It rewrites history (changes commit hashes). You should *never* rebase a branch that has already been pushed and shared with other developers, or you will cause a chaotic divergence.

#### Merging in HG and in Git
> Why are merges easier with Mercurial and Git than with SVN and CVS?

**Expert Answer:**
Git tracks *content* (snapshots of the whole file system tree), whereas SVN tracks *file changes* (deltas). When Git merges, it looks at the common ancestor commit, the tip of your branch, and the tip of the target branch, and performs a 3-way merge on the snapshots. It is incredibly smart at resolving conflicts automatically. SVN struggles with merges because it lacks this deep, snapshot-based graph understanding of history, often leading to manual conflict resolution nightmares, especially if files were moved or renamed.
