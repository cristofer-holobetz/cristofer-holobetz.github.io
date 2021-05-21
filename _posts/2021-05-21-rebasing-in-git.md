---
layout: post
title: "Using git rebase to Undo Large File Commits"
---

# Hi

**This is the first post,** almost 3 years after the website was created.

Here we'll record how I fixed a commit problem involving large data files. This post's purpose is a form of note-taking, and I don't yet fully understand most of what I actually did.

First, a description of the problem:

I have been using git as a VCS for one of my current projects. The project involves large data files, and I converted one of these into a json file for ease of access.

After doing so, I forgot to add the directory it's saved in to the .gitignore. Because I'm a novice with git, I didn't realize that there is a 100 mb limit to file sizes allowed when pushing to github.

Even so, I thought that retroactively adding the file to my .gitignore, would solve the problem. Alas, this was not the case. Even after adding the folder to the .gitignore, I found that pushing to github always failed with the same problem: 

    ╰─$ git push origin master
    Counting objects: 126, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (125/125), done.
    Writing objects: 100% (126/126), 49.59 MiB | 1.44 MiB/s, done.
    Total 126 (delta 67), reused 0 (delta 0)
    remote: Resolving deltas: 100% (67/67), completed with 1 local object.
    remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
    remote: error: Trace: 10671f87ba9a06e8279a26a11ab5c8382d6470b150d3de71cf273a17d9a16450
    remote: error: See http://git.io/iEPt8g for more information.
    remote: error: File rn2_spike_times.json is 254.09 MB; this exceeds GitHub's file size limit of 100.00 MB
    To https://github.com/cristofer-holobetz/path_equivalence_neighbor_bias.git
     ! [remote rejected] master -> master (pre-receive hook declined)
    error: failed to push some refs to 'https://github.com/cristofer-holobetz/<project_name>.git'


The reason for this turned out to be that even when large files are removed from the current commit, the fact that they exist in history means that git will try to push them all. It will look through the history of commits and see that one of them includes a gigantic file even if it no longer exists.

To fix this, I conducted an interactive rebase a la this 2020 Medium post by Erin Hoffman: [Tutorial: Removing Large Files from Git](https://medium.com/analytics-vidhya/tutorial-removing-large-files-from-git-78dbf4cf83a).

Here's a quick summary to help me internalize the process.

1. First, I used the second scenario, where the problem commit was multiple commits in the past.

2. I used ```git log``` to see the entire history of my commits and find the hash for the one that originally tried to push the large data file.

3. Once I found that hash, I took note of the last GOOD commit, ie the one immediately preceeding that one. In this case it started with ```3ce4...```.

4. At this point, I intiated a rebase using
    
    git rebase -i 3ce...
    
This opened a file in vim with a list of commit messages and their associated hashes.

5. I edited the file so that the BAD commit had the word 'edit' in front of it, making sure to leave the word 'pick' in front of all the others - __this allowed me to change history of the bad commit while preserving the good ones.__

6. After doing this, I quit Vim with :wq

7. Next, I used the command
    ```
    git commit --amend
    ```
This is the command that allowed me to amend the history of the bad commit. In effect, it let me 'redo' that commit and therefore undo the staging of the large file ```rn2_spike_times.json```. This is crucial, because the presence of this large file in the commit history was exactly the thing causing my ```git push```s to fail.

8. Next, I used
    ```
    git rm --cached rn2_spike_times.json
    git commit --amend -C HEAD
    ```
to actually remove the file.

9. I then removed the bad file using 
    ```
    git rm --cached rn2_spike_times.json
    ```
10. Finally, I attempted to finish the rebase using
    ```
    git rebase -continue
    ```
Frustratingly, this didn't work, even though my reference article ended here. Instead, I got the error:
    ```
    (my_venv) ╭─cristoferholobetz@Cristofers-MacBook-Pro ~/Documents/science/collaborators/<collaborator>/2020/<project_name> (9b99cfc*) 
    ╰─$ git rebase --continue
    error: The following untracked working tree files would be overwritten by merge:
    	notes/todo_path_eq
    Please move or remove them before you merge.
    Aborting
    Could not apply cd3df92... Will add colorbar later. Don't need to normalize across all trajectories. Next step is to add directions and trajectory boundaries, then on towards quantification.
    ```
    ```
    ╰─$ git rebase --continue
    error: The following untracked working tree files would be overwritten by merge:
    	notes/todo_path_eq
    Please move or remove them before you merge.
    Aborting
    Could not apply cd3df92... Will add colorbar later. Don't need to normalize across all trajectories. Next step is to add directions and trajectory boundaries, then on towards quantification.
    ```
    ```
    ╰─$ git status                                                                    141 ↵
    interactive rebase in progress; onto 323ee6d
    Last command done (1 command done):
       edit 2de8430 Figure out how to save valid_spike_times as json files. In total file is only 266 mb, so this seems the way to go for now.
    Next commands to do (24 remaining commands):
       pick 122a4c9 Got cell firing locations for desired trajectories and cells.
       pick f5aab2a Moved rn2_spike_times.json to the data folder so it's seen by .gitignore.
      (use "git rebase --edit-todo" to view and edit)
    You are currently editing a commit while rebasing branch 'master' on '323ee6d'.
      (use "git commit --amend" to amend the current commit)
      (use "git rebase --continue" once you are satisfied with your changes)
    
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)
    
    	deleted:    rn2_spike_times.json

    Untracked files:
      (use "git add <file>..." to include in what will be committed)
    
    	figures/
    	notes/
    	rn2_spike_times.json
    ```


I eventually found that I had to clean my untracked files, so:

11. I cleaned the untracked files using

    ```
    git clean -f -d
    ```
this yielded
    ```
    Removing figures/.ipynb_checkpoints/
    Removing figures/all_trajectories/
    Removing figures/linearized_trajectories/
    Removing notes/
    Removing rn2_spike_times.json
    ```
12. I used ```git rebase --continue``` again:
    ```
    ╰─$ git rebase --continue
    error: Your local changes to the following files would be overwritten by merge:
    	notes/todo_path_eq
    Please commit your changes or stash them before you merge.
    Aborting
    Could not apply 4be4a14... Removed notes from tracking.
    ```
13. Then
   ```
    ╰─$ git status                                                                      1 ↵
    interactive rebase in progress; onto 323ee6d
    Last commands done (33 commands done):
       pick 4be4a14 Removed notes from tracking.
       pick cbbe645 Added figures and notes folders to gitignore.
      (see more in file .git/rebase-merge/done)
    Next command to do (1 remaining command):
       pick 4be4a14 Removed notes from tracking.
      (use "git rebase --edit-todo" to view and edit)
    You are currently editing a commit while rebasing branch 'master' on '323ee6d'.
      (use "git commit --amend" to amend the current commit)
      (use "git rebase --continue" once you are satisfied with your changes)
    
    nothing to commit, working tree clean

    ```
14. Then finally 
    ```
    ╰─$ git rebase --continue
    Successfully rebased and updated refs/heads/master.
    ```
15. I commited the changes:
    ```
    ╰─$ git commit -m "Cleaned out untracked notes, rn2_spike_times and figures."
    On branch master
    nothing to commit, working tree clean
    ```
16. then pushed origin to master
    ```
    ╰─$ git push origin master
    Counting objects: 124, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (123/123), done.
    Writing objects: 100% (124/124), 16.42 MiB | 1.57 MiB/s, done.
    Total 124 (delta 69), reused 0 (delta 0)
    remote: Resolving deltas: 100% (69/69), completed with 1 local object.
    To https://github.com/cristofer-holobetz/project_name.git
       323ee6d..a846870  master -> master
    ```