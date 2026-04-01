---
name: android-git-skill
description: Enforces safe, predictable, and standardized source control practices for Android repositories. Use when executing Git commands, staging changes, or managing version control operations.
---

# Android Gravity Skill: Git Operations

## Description
This skill enforces safe, predictable, and standardized source control practices when working with Android repositories.

## Instructions
When activated, you MUST adhere to the following rules:

### 1. General Safety & Approvals
- **Explicit Confirmation Before Pushing:** Never execute a `git push` or any command that modifies the remote repository without obtaining explicit confirmation from the user first.
- **No Squashing:** Do not squash commits unless explicitly requested. Maintain the commit history exactly as requested by the user.
- **No Force Pushing:** Never use `git push --force` or `-f` on shared branches (like `main`, `master`, or `develop`).

### 2. Commit Standards
- **Conventional Commits:** Ensure commit messages are clear, concise, and follow the Conventional Commits format:
  - `feat(ui): added login screen`
  - `fix(auth): resolved null pointer on token refresh`
  - `chore(deps): bump compose version`
- **Atomic Commits:** Keep commits atomic. A single commit should represent a single logical change. Do not bundle unrelated changes (e.g., a bug fix and a new feature) into the same commit.

### 3. Branching Strategy
- **Branch Naming:** Create branches using descriptive prefixes:
  - `feature/login-screen`
  - `bugfix/crash-on-startup`
  - `hotfix/api-token-expiration`
- **Main/Master Protection:** Do not merge code into `main` or `master` branches directly. Work on feature branches and create Pull Requests.

### 4. Android-Specific Ignore Rules
- Ensure the `.gitignore` is properly configured and respected. Never stage or commit:
  - `local.properties` (contains local SDK paths and secrets)
  - `build/` or `.gradle/` directories
  - `*.jks` or `*.keystore` files (unless explicitly tracking debug keystores)
  - `.idea/` workspace files

### 5. Conflict Resolution
- Always prefer `git rebase` over `git merge` when updating local feature branches with changes from the main branch to maintain a linear history, unless the team's specific workflow dictates otherwise.

## Triggers
Activate this skill whenever:
- Executing Git commands (`git commit`, `git push`, `git merge`, `git rebase`).
- Staging changes or modifying `.gitignore`.
- The user asks you to manage version control operations or branching.
