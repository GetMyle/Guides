Here at Myle we follow GitHub Flow:
 - [What is a good Git workflow?](https://help.github.com/articles/what-is-a-good-git-workflow)
 - [The Ever-Deployable GitHub Workflow](http://mettadore.com/2011/09/07/the-ever-deployable-github-workflow)

#### So, what is GitHub Flow? ####

* Anything in the master branch is deployable
* To work on something new, grab an issue from backlog (or create new one if it doesn't exist), create a descriptively named branch off of master that have prefix that equals to the issue number (i.e.: 11-new-oauth2-scopes)
* Commit locally and rebase your branch to master regularly. Push your branch to the server often - so we all know what’s being worked on
* When you need feedback or help, or you think the branch is ready for merging, open a [pull request](http://help.github.com/send-pull-requests/)
* Provide descriptive explanation about what changes are addressed in this pull request and also provide URL to issue that's being addressed
* After someone else has reviewed and signed off on the feature, you have to rebase the branch to master, then merge it into master
* Once it is merged and pushed to ‘master’, you can deploy

#### Important notes ####

* We rebase branches before merging into master
* Master is changed only by merging pull requests (green button on pull request form)
* We never push to master
* We never do merges manually, consider rebasing to get latest changes from master or another branch
* Try to avoid usage of `git pull`, use `git fetch` instead, because `pull` merges current bracnh if remote is different. We would like to avoid unnecessary merges.

#### We rebase branches before merging into master ####

The main reason is to keep commit history simple and clean (i.e. to avoid situation like [this](http://agentdero.cachefly.net/unethicalblogger.com/images/branch_madness.jpeg)) and for better pull request experience.

Rebasing is a git feature that allows replaying specific range of commits on top of another commit.

Let's imaging the following situation:

```
B - feature1
|
A - master
```

In meantime somebody merged into master another feature branch. So after we pull recent changes from origin we would see:

```
D - master
|\
| C
| |
| | B - feature1
|/ /
A--
```

In order to merge `feature1` branch into master, there is need to do a rebase. The following command will do that:

```
git checkout feature1
git rebase master
```

After that our branch tree will look like this:


```
B' - feature1
|
D - master
|\
| C
|/
A
```

What happened here: all commits from `feature1` branch (in our case it is a single commit `B`) were replayed on top of `master` branch. Now our `feature1` branch contains all recent master changes.

Commit `B'` has the same changes as `B`, but different hash.

It can happen that changes in `feature1` branch conflict with changes in master during rebase. Developer should resolve these conflicts.

Once rebase on latest master, a branch is good to be pull requested and, if approved, merged into master. The merge have to be non-fast-forward. The simplest way to merge is to click "Merge" button in GitHub. Our tree would look like:

```
E - master, feature1
|\
| B'
|/
D
|\
| C
|/
A
```

One more thing to note is that if you had a branch __pushed__  and then __rebased__ it locally, you have to __force push__ rebased version:

```
git checkout feature1
git push origin +feature1
```

__So the rule is: always rebase your feature branch on top of the latest master before merging.__

### A typical branch lifecycle (examples)

1. Create new feature branch

	```
	git checkout -b 21-cool-feature
	git fetch --all
	git reset --hard origin/master
	```

	__NOTE: if branch is created to address an existing issue, we have a convention in MYLE to prefix the branch with issue number. So in example above 21 - is issue number that `21-cool-branch` is going to address.__

2. Add commits to the branch

3. If `origin/master` has something new that we need in our branch we do rebase

	```
	git checkout 21-cool-feature
	git fetch --all
	git rebase origin/master
	```

Conflicts can happen during rebase. 

4. Periodically push to remote

	```
	git push origin 21-cool-feature 	// if the branch never was rebased
	git push origin +21-cool-feature 	// if the branch is rebased, so we have to force push
	```

5. Once feature is ready, it's time to create a pull request

Go to GitHub, find `21-cool-feature` branch and click "Create Pull Request" button. 

__NOTE: make sure that on this step you branch starts off `origin/master`, so it makes reviewers life easier to see changes in comparison with latest master.__

Provide explanation about changes the pull request addresses and add link to corresponding issue.

6. It's time for team members to review the pull request and suggest some fixes.

If there is need to do any fixes, keep committing and pushing them to `21-cool-feature` branch`.

Once pull request comments are addressed, leave a message in it, saying, that review is ready for next round.

8. Merge the branch once there is approval from team members

__NOTE: It's important to rebase on top of `origin/master` before clicking merge button.__

Merge is done by clicking green `Merge` button on pull request page. Right after merge, click "Delete branch" button to delete the branch.

### Helpful links
* [Merging vs. Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing/)
* [GitHub Tricks (cheat sheet)](https://github.com/tiimgreen/github-cheat-sheet)