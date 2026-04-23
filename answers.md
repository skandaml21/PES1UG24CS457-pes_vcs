# Phase 5: Branching and Checkout

## Q5.1: How to implement `pes checkout <branch>`

A branch in PES-VCS is stored as a file inside `.pes/refs/heads/` that contains a commit hash.

To implement `pes checkout <branch>`:

1. Check if the branch exists in `.pes/refs/heads/`
2. Update `.pes/HEAD` to point to the selected branch:
   ref: refs/heads/<branch>
3. Read the commit hash from the branch file
4. Load the commit object and extract the tree hash
5. Traverse the tree and update the working directory files accordingly

This operation is complex because it must safely overwrite files and avoid losing uncommitted changes.

---

## Q5.2: Detecting dirty working directory

A dirty working directory means there are uncommitted changes.

To detect this:

1. Compare index entries with working directory files using:
   - file size
   - modification time (mtime_sec)
2. Compare index with the latest commit (HEAD)

If any mismatch is found, the working directory is considered dirty.

In such cases, checkout should refuse to proceed to prevent data loss.

---

## Q5.3: Detached HEAD

Detached HEAD occurs when `.pes/HEAD` contains a commit hash instead of a branch reference.

In this state:
- New commits can still be created
- But no branch points to them

These commits may become unreachable.

To recover, the user can create a branch pointing to the current commit:
`pes branch <new-branch>`

---

# Phase 6: Garbage Collection

## Q6.1: Finding unreachable objects

To identify unreachable objects:

1. Start from all branch heads in `.pes/refs/heads/`
2. Traverse all reachable objects:
   - commits → parents → trees → blobs
3. Store visited hashes in a hash set
4. Delete objects not in the visited set

For a repository with 100,000 commits and 50 branches, traversal may involve hundreds of thousands of objects.

---

## Q6.2: Why GC is dangerous during commit

Garbage collection can cause issues if it runs concurrently with a commit.

Race condition:
1. A commit creates new objects
2. Before references are updated, GC runs
3. GC considers new objects unreachable
4. GC deletes them

To avoid this, Git uses:
- locking mechanisms
- reference logs
- delayed garbage collection
