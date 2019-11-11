Github Cheats
---

Visit GitHub at [GitHub](http://github.com)

- Checkout code from master to a local branch

  `git checkout -b [BRANCH NAME]`

- Rename local branch name

  `git branch -m`

- Push local branch name to remote

  `git push origin [BRANCH NAME]`

- delete remote branch from repository

  `git push origin :[BRANCH NAME]`

- delete local branch from repository

  `git branch -D [BRANCH NAME]`

- commit changes to the local branch

  `git commit -m "[STORY NAME] description"`

- add modified/new/deleted files to the local branch

  `git add .`

- Update all remote-tracking branches from the remote repo

  `git fetch origin`

- Grab the commits that have been pushed to the master branch since you created your development branch and then apply your local commits on top of the commits to  `origin/master`

  `git rebase origin/master`

- switch to the master

  `git checkout master`

- Update the master branch by grabbing the latest changes from the remote and merge them into the local master branch

  `git pull`

- merges of development branches to the master

  `git merge --no-ff [BRANCH NAME]`

- Push your commits to the remote repo

  `git push origin master`

- Reset hard

  `git reset --hard`
