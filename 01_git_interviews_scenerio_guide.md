

# ******Git Scenario-Based Questions******

---

## **1. Branching & Merging**

---

### **1. You are working on `feature-A` branch, but a critical bug is reported in production. How will you handle this situation in Git?**

🔹 Problem: You’re in the middle of work on `feature-A`, but production needs an urgent fix.
🔹 Goal: Pause your current work safely, fix the bug, release, and then resume `feature-A`.

✅ Solution:

1. **Save current work (if uncommitted):**

   ```bash
   git stash   # saves WIP changes without committing
   ```

   or commit temporarily:

   ```bash
   git add .
   git commit -m "WIP: unfinished changes"
   ```

2. **Switch to production branch (main/master):**

   ```bash
   git checkout main
   git pull origin main
   ```

3. **Create a hotfix branch:**

   ```bash
   git checkout -b hotfix/critical-bug
   ```

4. **Fix the bug → commit → push:**

   ```bash
   git add .
   git commit -m "Fix: critical bug in production"
   git push origin hotfix/critical-bug
   ```

5. **Merge hotfix into `main` and deploy:**

   ```bash
   git checkout main
   git merge hotfix/critical-bug
   git push origin main
   ```

6. **Return to `feature-A` and continue work:**

   ```bash
   git checkout feature-A
   git stash pop   # or continue after temporary commit
   ```

📌 Diagram:

```
feature-A     ---●---● (paused)
main          ---●----● hotfix applied -> deployed
```

---

### **2. You have two branches: `develop` and `feature/login`. You want to merge changes from `develop` into `feature/login` but avoid merge conflicts later. What steps would you take?**

🔹 Problem: `develop` is ahead of `feature/login`. If you delay syncing, conflicts will pile up.
🔹 Goal: Regularly merge/rebase `develop` into `feature/login` to minimize conflicts.

✅ Solution:

1. Ensure both branches are updated:

   ```bash
   git fetch origin
   ```

2. Switch to feature branch:

   ```bash
   git checkout feature/login
   ```

3. Merge `develop` into it:

   ```bash
   git merge origin/develop
   ```

   (resolves conflicts now in smaller chunks)

   OR, for cleaner history:

   ```bash
   git rebase origin/develop
   ```

4. Resolve conflicts (if any), test, and push:

   ```bash
   git push origin feature/login
   ```

📌 Diagram:

```
develop:       ----●----●----●
feature/login: ----●----● (merge develop here)
```

---

### **3. You merged a branch but realized you merged the wrong one. How will you undo this merge safely?**

🔹 Problem: Wrong branch merged into main/develop.
🔹 Solution depends on whether it was **pushed** or not.

✅ Case 1: Merge is the latest commit and not pushed yet:

```bash
git reset --hard HEAD~1
```

✅ Case 2: Merge was pushed to remote:

```bash
git revert -m 1 <merge-commit-hash>
git push origin main
```

* `-m 1` means keep parent 1 (usually `main`) and remove merged branch changes.

📌 Diagram (revert):

```
main: ----●----●----● M (wrong merge)
                \
                 (revert commit removes M)
```

---

## **2. Conflict Resolution**

---

### **1. While merging, you encounter a huge number of merge conflicts in multiple files. How would you approach resolving them efficiently?**

🔹 Problem: Large conflict set = time-consuming + error-prone.
🔹 Strategy:

1. **Use a merge tool**:

   ```bash
   git mergetool
   ```

   Helps resolve conflicts interactively.

2. **Tackle file groups**:

   * First auto-resolve trivial ones:

     ```bash
     git checkout --ours file.txt
     git checkout --theirs file.txt
     ```
   * Then handle complex conflicts manually.

3. **Commit frequently** after resolving groups of conflicts (smaller checkpoints).

4. **Communicate with team** if conflicts are caused by major refactors.

---

### **2. You pull changes from the remote and find conflicts in your local branch. How do you decide whether to use `git merge` or `git rebase` in this case?**

🔹 Decision rule:

* **Use merge** if you want to preserve exact history (common in teams).
* **Use rebase** if you want a linear, clean history (use only if you haven’t shared your branch yet).

📌 Diagram:

**Merge**

```
main:    ---●----●
feature:     \----●----●
```

**Rebase**

```
main:    ---●----●
feature:           ●----●
```

---

### **3. You rebased your branch and pushed changes, but your teammate says the commit history is messed up. How do you fix it?**

✅ Steps:

1. If history is broken → **reset to remote**:

   ```bash
   git fetch origin
   git reset --hard origin/branch-name
   ```

2. Reapply your local changes via:

   ```bash
   git cherry-pick <commit-hash>
   ```

   OR redo work on a fresh branch.

3. Push again.

📌 Key Rule: Never rebase commits already pushed to shared branches.

---

## **3. Commit Management**

---

### **1. You committed a password/API key by mistake. How do you remove it from history while keeping the rest of the commits intact?**

✅ Solution:

1. Remove from latest commit:

   ```bash
   git reset --soft HEAD~1
   git reset HEAD <file>
   git commit --amend
   ```

2. If committed long ago, remove from entire history using:

   ```bash
   git filter-repo --path <file> --invert-paths
   ```

   or BFG Repo Cleaner.

3. Force push:

   ```bash
   git push origin --force
   ```

---

### **2. You committed changes in the wrong branch. How do you move that commit to the correct branch?**

✅ Solution:

1. Find commit hash:

   ```bash
   git log
   ```

2. Checkout correct branch:

   ```bash
   git checkout correct-branch
   git cherry-pick <commit-hash>
   ```

3. Remove commit from wrong branch:

   ```bash
   git checkout wrong-branch
   git reset --hard HEAD~1
   ```

---

### **3. You have 10 commits in your feature branch. The reviewer asked to squash them into a single commit before merging. How will you do it?**

✅ Solution:

```bash
git rebase -i HEAD~10
```

* Mark first commit as `pick`
* Mark the next 9 as `squash`
* Write a single new commit message

Push with force:

```bash
git push origin feature-branch --force
```

---

## **4. Collaboration**

---

### **1. You and your teammate are working on the same branch. After pulling, you see changes that overwrite your work. How will you recover your lost work?**

✅ Solution:

1. Use reflog to find lost commits:

   ```bash
   git reflog
   git checkout <commit-hash>
   ```
2. Create a new branch from that commit.
3. Merge recovered work back.

---

### **2. Your teammate force-pushed to a branch and your local branch is now behind. How will you sync it without losing your local commits?**

✅ Solution:

1. Fetch latest:

   ```bash
   git fetch origin
   ```
2. Rebase your local commits on top:

   ```bash
   git rebase origin/branch-name
   ```

---

### **3. You are working in a team where multiple people are merging PRs to the main branch. How do you make sure your branch is always up-to-date before merging?**

✅ Solution:

* Regularly rebase:

  ```bash
  git fetch origin
  git rebase origin/main
  ```
* Enable branch protection requiring branches to be up-to-date before merge.

---

## **5. Git Commands Under Pressure**

---

### **1. You accidentally ran `git reset --hard HEAD~1` and lost your latest commit. Can you recover it?**

✅ Solution:

1. Use reflog:

   ```bash
   git reflog
   git checkout <lost-commit-hash>
   ```
2. Restore by creating a branch:

   ```bash
   git checkout -b recovered-branch <hash>
   ```

---

### **2. You need to see the difference between your branch and `main` before merging. What commands will you use?**

✅ Solution:

```bash
git fetch origin
git diff origin/main..HEAD
```

or just summary:

```bash
git log origin/main..HEAD --oneline
```

---

### **3. You need to check which commit introduced a bug. How will you find it quickly?**

✅ Solution:
Use `git bisect` (binary search):

```bash
git bisect start
git bisect bad          # current buggy commit
git bisect good <hash>  # last known good commit
```

Git checks out middle commit → test → mark as `good` or `bad`.
Repeat until bug commit is found.

---

## **6. Advanced / Real-World Situations**

---

### **1. Your Git repository size increased drastically. How will you find and remove large files from history?**

✅ Solution:

1. Find big files:

   ```bash
   git rev-list --objects --all | sort -k 2 > all-files.txt
   ```
2. Remove large files with:

   ```bash
   git filter-repo --strip-blobs-bigger-than 10M
   ```
3. Force push cleaned repo.

---

### **2. The build server is failing because the .gitignore file is not working as expected. How do you debug and fix it?**

✅ Solution:

1. Check if file is already tracked:

   ```bash
   git ls-files | grep <filename>
   ```
2. If yes → remove from index:

   ```bash
   git rm --cached <file>
   ```
3. Add to `.gitignore` and commit.

---

### **3. You cloned a repo and found it has hundreds of stale branches. How will you find and delete all branches merged into main?**

✅ Solution:

1. Fetch all branches:

   ```bash
   git fetch -p
   ```

2. List merged branches:

   ```bash
   git branch --merged main
   ```

3. Delete locally:

   ```bash
   git branch -d branch-name
   ```

4. Delete remotely:

   ```bash
   git push origin --delete branch-name
   ```

5. Bulk cleanup:

   ```bash
   git branch --merged main | grep -v 'main' | xargs git branch -d
   ```

---
