I'm using [git-some](https://github.com/GROSSWEBER/git-some) to quickly create commits.

1. Create a simple repo with master and topic, where topic is the PR branch.

   ```sh
   agross@firiel ~/partial-pr
   $ git init
   Initialized empty Git repository in /Users/agross/partial-pr/.git/

   agross@firiel ~/partial-pr @master
   ± git some 2 && git checkout -b topic
   [master (root-commit) f364c19] master: A.txt
   1 file changed, 1 insertion(+)
   create mode 100644 A.txt

   [master c18c0dc] master: B.txt
   1 file changed, 1 insertion(+)
   create mode 100644 B.txt

   Switched to a new branch 'topic'

   agross@firiel ~/partial-pr @topic
   ± git some 3
   [topic f493219] topic: C.txt
   1 file changed, 1 insertion(+)
   create mode 100644 C.txt

   [topic f2e5d38] topic: D.txt
   1 file changed, 1 insertion(+)
   create mode 100644 D.txt

   [topic e281a01] topic: E.txt
   1 file changed, 1 insertion(+)
   create mode 100644 E.txt
   ```

1. Upload to GitHub

   ```sh
   agross@firiel ~/partial-pr @topic
   ± git remote add origin git@github.com:agross/partial-pr.git

   agross@firiel ~/partial-pr @topic
   ± git push --all origin
   Enumerating objects: 15, done.
   Counting objects: 100% (15/15), done.
   Delta compression using up to 4 threads
   Compressing objects: 100% (9/9), done.
   Writing objects: 100% (15/15), 1.13 KiB | 384.00 KiB/s, done.
   Total 15 (delta 4), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (4/4), done.
   To github.com:agross/partial-pr.git
   * [new branch]      master -> master
   * [new branch]      topic -> topic
   ```

1. Now open a PR on GitHub. It'll be [#1](https://github.com/agross/partial-pr/pull/1).

1. Pretend we only want to integrate parts of the PR (file `D.txt` in my case)

   1. Switch to target branch.

      ```sh
      agross@firiel ~/partial-pr @topic
      ± git checkout master
      Switched to branch 'master'
      ```

   1. Download the PR's commits.

      ```sh
      agross@firiel ~/partial-pr @master
      ± git fetch origin refs/pull/1/head
      From github.com:agross/partial-pr
       * branch            refs/pull/1/head -> FETCH_HEAD
      ```

   1. Perform a local recursive merge, but do not commit.

      ```sh
      agross@firiel ~/partial-pr @master
      ± git merge --no-commit --no-ff FETCH_HEAD
      Automatic merge went well; stopped before committing as requested

      agross@firiel ~/partial-pr @master|MERGING^
      ± git status
      On branch master
      All conflicts fixed but you are still merging.
        (use "git commit" to conclude merge)

      Changes to be committed:
      	new file:   C.txt
      	new file:   D.txt
      	new file:   E.txt
      ```

   1. All changes from the [`topic` branch](https://github.com/agross/partial-pr/tree/topic)
      are now available. Let us remove `C.txt` and `E.txt`.

      ```sh
      agross@firiel ~/partial-pr @master|MERGING^
      ± git reset HEAD C.txt E.txt

      agross@firiel ~/partial-pr @master|MERGING?^
      ± git status
      On branch master
      All conflicts fixed but you are still merging.
        (use "git commit" to conclude merge)

      Changes to be committed:
      	new file:   D.txt

      Untracked files:
        (use "git add <file>..." to include in what will be committed)
      	C.txt
      	E.txt
      ```

   1. With `C.txt` and `E.txt` being unstaged, continue the merge. They will not
      be part of the merge and can be removed.

      ```sh
      agross@firiel ~/partial-pr @master|MERGING?^
      ± git merge --continue
      [master db4675a] Merge commit 'refs/pull/1/head' of github.com:agross/partial-pr

      agross@firiel ~/partial-pr @master?
      ± git clean -f
      Removing C.txt
      Removing E.txt

      agross@firiel ~/partial-pr @master
      ± ls
      total 12K
      drwxr-xr-x   6 agross staff 192 May 11 17:11 .
      drwx------+  7 agross staff 224 May 11 17:09 ..
      drwxr-xr-x  16 agross staff 512 May 11 17:11 .git
      -rw-r--r--   1 agross staff  19 May 11 17:09 A.txt
      -rw-r--r--   1 agross staff  19 May 11 17:09 B.txt
      -rw-r--r--   1 agross staff  19 May 11 17:11 D.txt
      ```

   1. Upload the merge commit to GitHub. This will close the PR.

      ```sh
      agross@firiel ~/partial-pr @master
      ± git push origin master
      Enumerating objects: 4, done.
      Counting objects: 100% (4/4), done.
      Delta compression using up to 4 threads
      Compressing objects: 100% (2/2), done.
      Writing objects: 100% (2/2), 326 bytes | 326.00 KiB/s, done.
      Total 2 (delta 1), reused 0 (delta 0), pack-reused 0
      remote: Resolving deltas: 100% (1/1), completed with 1 local object.
      To github.com:agross/partial-pr.git
         c18c0dc..db4675a  master -> master
      ```
