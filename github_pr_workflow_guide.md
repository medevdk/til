# GitHub PR Workflow Setup Guide

This comprehensive guide covers initializing a repository from an empty directory, setting up remote repositories using the GitHub CLI (`gh`), executing feature-branch development, merging Pull Requests (PRs), resolving merge conflicts, and configuring LLM/AI coding tools to automate PR generation.

---

## 1. Initial Project Setup

Follow these steps from a clean, empty directory to set up local Git tracking, make an initial commit, and establish a private remote repository on GitHub.

```bash
# 1. Create and navigate into project directory
mkdir my-project && cd my-project

# 2. Initialize local repository with 'main' as default branch
git init -b main

# 3. Create initial placeholder file
echo "# My Project" > README.md

# 4. Stage and commit initial file
git add README.md
git commit -m "chore: initial commit"

# 5. Create a private GitHub repository and push local code using GitHub CLI
gh repo create my-project --private --source=. --remote=origin --push
```

---

## 2. Core Development & PR Workflow

Never commit directly to `main`. Treat `main` as read-only locally and execute all features, bug fixes, or experiments on separate branches.

### Step 1: Sync Main and Create a Feature Branch
Before starting new work, ensure your local `main` branch is up to date with `origin/main`.

```bash
git checkout main
git pull origin main
git checkout -b feature/add-user-login
```

### Step 2: Build and Commit Locally
Work on your feature and make small, incremental commits.

```bash
git add .
git commit -m "feat: implement login authentication handler"
```

### Step 3: Push Branch and Open a Pull Request
Push your feature branch to GitHub and trigger PR creation.

```bash
# Push branch to remote and set tracking upstream
git push -u origin feature/add-user-login

# Create PR and open it in your browser via GitHub CLI
gh pr create --web
```

### Step 4: Code Review and Merge on GitHub
1. Review code diffs and automated test results on the GitHub web interface.
2. Select **Merge pull request** (or **Squash and merge** depending on preference).
3. Click **Delete branch** on GitHub to remove the remote feature branch.

### Step 5: Sync Local Workspace and Clean Up
Bring the newly merged `main` code back down to your local environment and delete the finished local branch.

```bash
# Switch to local main
git checkout main

# Pull updated main containing merged PR
git pull origin main

# Delete local feature branch
git branch -d feature/add-user-login
```

---

## 3. Fast-Forward vs. Non-Fast-Forward Merges

When integrating branches, Git handles merges differently depending on commit history:

* **Fast-Forward Merge (`git merge`):**
  If `main` has no new commits since your feature branch was created, Git simply moves the `main` pointer forward to match your branch. No new merge commit is created.

  ```text
  Before:  A --- B (main) \ C1 --- C2 (feature)
  After:   A --- B --- C1 --- C2 (main, feature)
  ```

* **Non-Fast-Forward Merge (`git merge --no-ff`):**
  Git creates a dedicated merge commit linking both histories, even if `main` did not receive new commits. This preserves the explicit boundary of the feature branch. GitHub PRs default to this behavior.

  ```text
  Before:  A --- B (main) \ C1 --- C2 (feature)
  After:   A --- B ------------------- M (main)
                  \                  /
                   --- C1 --- C2 ----
  ```

---

## 4. Handling Merge Conflicts

Conflicts occur when `main` moves ahead on GitHub while you are working on a feature branch, and both touch the same lines of code.

### Resolution Steps:

```bash
# 1. Update your local main branch
git checkout main
git pull origin main

# 2. Switch back to your feature branch and pull in main
git checkout feature/add-user-login
git merge main
```

When Git flags conflicts, open the files and locate conflict markers:

```diff
<<<<<<< HEAD (Your feature branch code)
func ConnectDB() *sql.DB {
    return openSQLite("data.db")
}
=======
func ConnectDB() *sql.DB {
    return openCgoFreeSQLite("app.db")
}
>>>>>>> main (Incoming code from main)
```

Edit the code to resolve differences, remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`), then stage, commit, and push:

```bash
# 3. Stage resolved files
git add .

# 4. Finalize merge commit
git commit -m "fix: resolve merge conflicts with main"

# 5. Push updated branch to automatically refresh the open PR
git push origin feature/add-user-login
```

---

## 5. LLM / AI Coding Assistant Workflow

When delegating code generation to an LLM or automated coding agent, instruct the agent to run inside an isolated feature branch and open a PR rather than making direct commits to `main` or editing your uncommitted work.

### Standard Prompting Directive:

```text
Follow this exact workflow for any requested changes:
1. Ensure main is up to date: `git checkout main && git pull origin main`.
2. Create a new branch: `git checkout -b feature/<descriptive-name>`.
3. Implement and test all requested changes.
4. Commit changes using Conventional Commits formatting (e.g., feat:, fix:, refactor:).
5. Push the branch: `git push -u origin feature/<descriptive-name>`.
6. Open a PR using GitHub CLI:
   gh pr create --title "<title>" --body "<summary of changes, motivation, and testing steps>"
7. Stop and await human review. Do not merge directly to main.
```

### Iterative Feedback Loop:
If you review the PR on GitHub and identify issues or necessary adjustments, instruct the LLM:

> *"Check out branch `feature/<descriptive-name>`, address the code review feedback regarding error handling on line 45, and push the update."*

The LLM will push a new commit to the branch, and GitHub will instantly update the open PR for re-review.
