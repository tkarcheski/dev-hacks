# Using `git fixup` to Combine Multiple Commits

## Introduction

This guide demonstrates how to use `git fixup` and interactive rebase to clean up your commit history by combining multiple commits into logical units. We'll walk through a practical example involving a quiz application to illustrate the process.

## Scenario Overview

Suppose you have the following commits in your project:

1. **Commit 1:** `abc123` - **Add basic structure of quiz feature**
   - Modifies:
     - `quiz.py` — Created with a basic structure.
     - `README.md` — Added a brief description.

2. **Commit 2:** `def456` - **Add question list and basic UI**
   - Modifies:
     - `quiz.py` — Added a list of questions.

3. **Commit 3:** `ghi789` - **Update README with setup instructions**
   - Modifies:
     - `README.md` — Added setup instructions.

4. **Commit 4:** `jkl012` - **Fix question list typo**
   - Modifies:
     - `quiz.py` — Corrected a typo in the list of questions.

5. **Commit 5:** `mno345` - **Add scoring functionality**
   - Modifies:
     - `quiz.py` — Implemented scoring functionality.

6. **Commit 6:** `pqr678` - **Fix setup instructions in README**
   - Modifies:
     - `README.md` — Corrected instructions.

### Objective

- **Combine Commit 4 into Commit 2**: Since Commit 4 fixes a typo introduced in Commit 2, we'll merge it into Commit 2.
- **Combine Commit 6 into Commit 3**: Commit 6 fixes the setup instructions added in Commit 3, so we'll merge it into Commit 3.

## Steps to Combine Commits Using Interactive Rebase

### 1. Start an Interactive Rebase

Begin an interactive rebase from the parent of the earliest commit you want to modify (`abc123`):

```bash
git rebase -i abc123^
```

This command opens an editor displaying all commits since `abc123`:

```
pick def456 Add question list and basic UI
pick ghi789 Update README with setup instructions
pick jkl012 Fix question list typo
pick mno345 Add scoring functionality
pick pqr678 Fix setup instructions in README
```

### 2. Modify the Rebase Instructions

In the editor, change the action for the commits you want to fix up:

- **Change `pick` to `fixup` for Commit 4 (`jkl012`):**

  This will merge it into Commit 2 (`def456`).

- **Change `pick` to `fixup` for Commit 6 (`pqr678`):**

  This will merge it into Commit 3 (`ghi789`).

Ensure that the `fixup` commits are placed immediately after their target commits. The modified rebase instructions should look like this:

```
pick def456 Add question list and basic UI
fixup jkl012 Fix question list typo
pick ghi789 Update README with setup instructions
fixup pqr678 Fix setup instructions in README
pick mno345 Add scoring functionality
```

### 3. Save and Exit

Save the changes and exit the editor. Git will proceed with the rebase, combining the `fixup` commits into their respective parent commits.

### 4. Resolve Any Conflicts (If Necessary)

If Git encounters any conflicts during the rebase, it will pause and allow you to resolve them:

```bash
# After resolving conflicts in files
git add <file>
git rebase --continue
```

### 5. Verify the Updated Commit History

Check your commit history to ensure the changes were applied correctly:

```bash
git log --oneline
```

The history should now look like:

```
mno345 Add scoring functionality
ghi789 Update README with setup instructions
def456 Add question list and basic UI
abc123 Add basic structure of quiz feature
```

### 6. Push the Changes

If you're working with a remote repository, force-push the updated history:

```bash
git push origin your-branch-name --force
```

**Note:** Force-pushing rewrites the remote history, which can affect other collaborators. Use it with caution.

## Explanation

- **Interactive Rebase (`git rebase -i`)**: Allows you to edit commits, reorder them, or squash them together.
- **`fixup` Action**: Tells Git to merge the commit into the one before it without keeping the commit message.
- **Commit Ordering**: Placing the `fixup` commit immediately after its target commit ensures they are combined correctly.
- **Conflict Resolution**: May be necessary if the changes in the commits conflict with each other.

## Best Practices

- **Communicate with Your Team**: Before rewriting history, inform your team to avoid conflicts.
- **Use `--autosquash` for New Fixups**: If you're creating new fixup commits, you can use `git commit --fixup <commit>` and `git rebase -i --autosquash` to automate the process.
- **Backup Branch**: Before performing the rebase, consider creating a backup branch in case you need to revert.

## Conclusion

By using interactive rebase and the `fixup` action, you can clean up your commit history, making it more readable and maintainable. This practice is especially useful before merging feature branches into the main branch.

## Additional Resources

- [Git Documentation - Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)
- [Understanding Git Rebase](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)
- [Git Commit --fixup and Autosquash](https://thoughtbot.com/blog/autosquashing-git-commits)
