---

template:      article
reviewed:      2016-12-20
title:         SSH + Git deployment workflow
naviTitle:     SSH + Git
lead:          How to use Git from the shell on fortrabbit — advanced worflow, special case.
dontList:      1
stack:        old
group:         platform

---

This workflow is for advanced developers to solve special cases. It is based on your SSH account and Git, which is installed on the shell. WARNING: Do not confuse this workflow with the regular [Git workflow](git-deployment)! This workflow should be used for solving edge cases, only!



## Problem

Deployment is not deployment. Sometimes the default is not enough.

* **Large binaries**: In the regular Git deployment, the file size is limited. If you want to keep your binaries as well under version control, this is a way to do it.
* **Special Hooks**: We already provide a mighty tool-set to implement a large variety of deployment scenarios. If this is not enough, here you can go wild.


## Solution

Use Git directly on the remote machine on fortrabbit via SSH.


## Usage

When you login to your SSH account, you can use Git directly from the terminal. As a side-effect, you can also use the SSH account as a git remote repository.


### Example workflow

```bash
# You firstly need to init a new Git repository. Login via SSH and setup an empty folder on the remote:
cd ~/htdocs/some-folder
git init .

# Now you can clone this folder locally and start doing Git's work:
git clone u-your-app@ssh1.eu1.frbit.com:~/htdocs/some-folder

# Go to the folder:
cd some-folder

# generate some files here and then:
git add -A
git commit -am 'My files'
git push -u origin master
```

### Using hooks

Hooks require a special "proxy" command. For example, if you want to utilize the `update` hook, you need to create a soft link to the proxy command `/usr/local/bin/git-hook-call` and create either an `update.sh` or `update.php` script, which is then called by the proxy.

```bash
cd ~/htdocs/some-folder/.git/hooks
ln -s /usr/local/bin/git-hook-call update
echo '<?php echo "Hello from update" ?>;' > update.php
```