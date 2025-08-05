
#  Source Code Management

Everything starts with code. How you manage it determines everything else.

## Basic Questions

1.  **Q: What is Git?**

    **A:** Git is a distributed version control system (DVCS). This means that instead of having one central server that contains all the code history, every developer has a full copy of the repository, including its entire history, on their local machine. This makes it incredibly fast for operations like committing, branching, and merging, and it allows developers to work offline.

2.  **Q: What is the difference between `git fetch` and `git pull`?**

    **A:** This is a classic question.
    * `git fetch` downloads all the latest changes from the remote repository (like GitHub) but does *not* merge them into your local working branch. It updates your local knowledge of the remote repository's state.
    * `git pull` is actually a combination of two commands: `git fetch` followed by `git merge`. It fetches the changes and then immediately tries to merge them into the branch you are currently on.
    **Best Practice:** In most cases, it's safer to use `git fetch` first. This allows you to see what has changed on the remote before you decide to merge it into your local work.

3.  **Q: What is a `git commit`?**

    **A:** A commit is a snapshot of your repository at a specific point in time. It's the fundamental unit of work in Git. Each commit has a unique ID (a SHA-1 hash), an author, an email, a timestamp, and a commit message explaining the change. A good commit is small, atomic (represents a single logical change), and has a clear, descriptive message.

4.  **Q: What is a "branch" in Git?**

    **A:** A branch is simply a lightweight, movable pointer to a specific commit. It's a way to work on a new feature or a bug fix in isolation without affecting the main line of development (usually the `main` or `master` branch). Creating a branch is extremely cheap and fast in Git, which encourages their use for any new work.

5.  **Q: What is the purpose of a `.gitignore` file?**

    **A:** The `.gitignore` file is a text file that tells Git which files or directories to intentionally ignore. This is used for files that shouldn't be committed to the repository, such as compiled code (`.class`, `.exe`), log files, dependencies (`node_modules/`), or files containing secrets or local environment configurations.



## Intermediate Questions

1.  **Q: Explain the difference between `git merge` and `git rebase`.**
    
    **A:** Both commands are used to integrate changes from one branch into another, but they do it in very different ways.
    * **`git merge`:** This creates a new "merge commit" in the target branch that ties the histories of the two branches together. It's non-destructive; the existing branches are not changed. This results in a graph history that shows exactly when and where merges happened.
        * **Pro:** Preserves the exact history. It's simple and easy to understand.
        * **Con:** Can lead to a messy, cluttered commit history with lots of merge commits.
    * **`git rebase`:** This takes all the commits from your feature branch and "replays" them, one by one, on top of the target branch. It rewrites the project history to create a clean, linear sequence of commits.
        * **Pro:** Creates a clean, easy-to-read, linear history.
        * **Con:** It rewrites history. **You should never rebase a branch that has been pushed and is being used by other people**, as it will cause major problems for them. It's best used for cleaning up your own local feature branches before merging them.

2.  **Q: What is a "pull request" (or "merge request")?**

    **A:** A pull request (PR) is a mechanism for a developer to notify team members that they have completed a feature or bug fix on a separate branch and it's ready to be merged into the main branch. It's not a Git command, but a feature of Git hosting platforms like GitHub, GitLab, and Bitbucket. A PR provides a forum for code review, discussion, and running automated checks (via CI) before the code is integrated. It's the central workflow for collaborative software development.

3.  **Q: Describe a common Git branching strategy you have used.**

    **A:** One of the most common and effective strategies is **GitHub Flow**. It's simple and works very well for teams practicing continuous delivery.
    * The `main` branch is always considered deployable.
    * To work on anything new (a feature or a bug fix), you create a new, descriptively named branch off of `main` (e.g., `feature/add-user-login`).
    * You commit to this branch locally and regularly push your work to the same-named branch on the server.
    * When you're ready for feedback or to merge, you open a pull request.
    * After the PR is reviewed and approved, and the CI tests pass, it is merged into `main`.
    * Once merged into `main`, it should be deployed to production immediately.
    This strategy is lightweight and ensures that `main` is always in a good state.

4.  **Q: What is `git cherry-pick` and in what scenario would you use it?**

    **A:** `git cherry-pick` is a command that allows you to pick a specific commit from one branch and apply it onto another. Let's say you have a bug fix on a feature branch that needs to be applied to the `main` branch *right now*, but the rest of the feature branch is not ready to be merged. You can use `git cherry-pick <commit-hash>` to grab just that one commit and apply it to `main`. It's a powerful tool for when you need to pull in a specific change without taking the entire branch.

5.  **Q: What is the `reflog` and how can it save you?**

    **A:** The `git reflog` (reference log) is a record of all the changes to the tips of your branches and other references in your local repository. It's your safety net. If you ever do something you think is destructive—like accidentally deleting a branch or doing a rebase that you regret—the `reflog` contains the history of where your branches *used to be*. You can find the commit hash from the reflog and use `git checkout` or `git reset` to restore your work. It's saved me more than once!


## Advanced Questions

1.  **Q: What is GitFlow, and what are its pros and cons compared to a simpler strategy like GitHub Flow?**

    **A:** GitFlow is a more complex branching model that was designed for projects with scheduled release cycles. It involves several types of branches:
    * `main`: Contains production-ready code. Tagged for each release.
    * `develop`: The main integration branch for new features.
    * `feature/*`: Branched from `develop` for new work. Merged back into `develop`.
    * `release/*`: Branched from `develop` when preparing for a release. Only bug fixes are allowed here. Merged into both `main` and `develop`.
    * `hotfix/*`: Branched from `main` to fix urgent production issues. Merged into both `main` and `develop`.
    * **Pros:** Very structured, provides a robust framework for managing releases, excellent for projects with a versioned release schedule (e.g., desktop software).
    * **Cons:** Overly complex for most web applications that practice continuous delivery. The overhead of managing all the branches can slow teams down. GitHub Flow is generally preferred for teams that deploy frequently.

2.  **Q: What is a Git submodule, and what are the common pitfalls of using them?**

    **A:** A Git submodule allows you to embed another Git repository into a subdirectory of your main repository. It's a way to include and manage a dependency as a separate project.
    * **Common Pitfalls:** They can be very tricky to work with.
        * **Complexity:** They add a layer of complexity to your workflow. Commands like `git clone` and `git pull` don't automatically update the submodules; you need to use extra flags or commands (`git submodule update --init --recursive`).
        * **"Detached HEAD" State:** When you check out a submodule, you are often in a "detached HEAD" state, pointing to a specific commit rather than a branch. It's easy to make changes, commit them, and then lose them because no branch was pointing to them.
        * **Pinning:** The parent repo is pinned to a specific commit of the submodule. Updating the submodule requires a new commit in the parent repo, which can be cumbersome.
    * **Alternatives:** In many modern development ecosystems, it's often better to manage dependencies through a package manager (like npm, Maven, or Go modules) rather than using submodules.

3.  **Q: Explain how you would handle a large binary file (e.g., a video or a dataset) in a Git repository.**

    **A:** You generally shouldn't commit large binary files directly to Git. Git is designed to handle text files and it stores the full history of every file, so your repository size will explode, making it slow to clone and work with.
    * **The Solution: Git LFS (Large File Storage).** Git LFS is an extension to Git that replaces large files in your repository with small text pointers. The actual binary files are stored on a separate LFS server. When you check out a branch, Git LFS downloads the correct versions of the large files you need. This keeps your main Git repository small and fast, while still versioning the large assets. You configure it by telling it which file patterns to track (e.g., `git lfs track "*.psd"`).

4.  **Q: What is `git bisect` and how does it work?**

    **A:** `git bisect` is an incredibly powerful debugging tool used to find the exact commit that introduced a bug. It performs an automated binary search through your commit history.
    * **How it works:**
        1.  You start the process with `git bisect start`.
        2.  You give it a "bad" commit where the bug exists (e.g., `git bisect bad HEAD`).
        3.  You give it a "good" commit where the bug did not exist (e.g., `git bisect good v1.0`).
        4.  Git then checks out a commit roughly halfway between the good and bad commits.
        5.  You test for the bug. If it's still present, you tell Git `git bisect bad`. If it's gone, you tell Git `git bisect good`.
        6.  Git repeats this process, halving the number of commits to search each time, until it pinpoints the exact commit that introduced the issue. It's exponentially faster than checking out commits one by one.

5.  **Q: You have accidentally committed a file containing sensitive information (e.g., an API key) and pushed it to a public repository. What are the steps you must take?**

    **A:** This is a critical incident. Simply committing a fix that removes the file is not enough, because the secret is still in the repository's history.
    * **Step 1: Invalidate the Secret.** This is the most important and immediate step. Go to the service provider (e.g., AWS, Google Cloud) and revoke or delete the exposed key. Assume it has been compromised.
    * **Step 2: Remove the File from History.** You need to completely purge the file from the repository's history. The recommended tool for this is `git filter-repo` (which has superseded the older `git filter-branch` and BFG Repo-Cleaner). You would run a command like `git filter-repo --invert-paths --path path/to/your/secret-file`.
    * **Step 3: Force Push the Changes.** After you have rewritten the history locally, you must force push the changes to the remote repository to overwrite the old, compromised history (`git push origin --force`).
    * **Step 4: Communicate.** Inform your team about what happened and that they will need to perform a fresh clone or a force pull of the repository. Also, audit your systems to see if the compromised key was used. This is a serious situation that requires a swift and thorough response.

