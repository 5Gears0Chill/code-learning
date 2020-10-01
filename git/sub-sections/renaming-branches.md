# Renaming Git Branches

The process defines the methodology used to rename a branch on local and remote should a name change be required.

Step 1: Rename the local branch to the new name 

```shell
git branch -m <old_name> <new_name>
```

Step 2: Delete the old branch on remote - where `<remote>` is, for example, origin

```shell
git push <remote> --delete <old_name>
```

Note a shorter way to complete step 2 is as follows

```shell
git push <remote> :<old_name>
```

Step 3: Push the new branch to remote

```shell
git push <remote> <new_name>
```

Step 4: Reset the upstream branch for the `new_name ` local branch

```shell
git push <remote> -u <new_name>
```

# Renaming the Remote Branch Only

In this option, we will push the branch to the remote with the new name, while keeping the local name as is

```shell
git push <remote> <remote>/<old_name>:refs/heads/<new_name> :<old_name>	
```

# Important Things to Note

When you use the `git branch -m` (move), Git is also **updating** your tracking branch with the new name.

> ```shell
> git remote rename legacy legacy
> ```

`git remote rename` is trying to update your remote section in your configuration file. It will rename the remote with the given name to the new name, but in your case, it did not find any, so the renaming failed.

**But** it will not do what you think; it will rename your **local** configuration remote name and **not** the remote branch.

---

**Note**: Git servers might allow you to rename Git branches using the web interface or external programs (like **Sourcetree**, etc.), but you have to keep in mind that in Git all the work is done locally, so it's recommended to use the above commands to the work.

---

## Change Log

- [01-10-2020] - Added strategy for renaming branches