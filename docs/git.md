---

template:      article
reviewed:      2016-05-20
naviTitle:     Git
title:         Git on fortrabbit
lead:          A quick intro to Git, how to set it up and how to use it on fortrabbit.
group:         Kitchen_sink

keywords:
    - ssh key
    - public key
    - git
    - ssh
    - Putty
    - msysGit
    - Cygwin
    - Windows
    - Git Bash


seeAlsoLinks:
    - ssh-keys
    - deployment
    - dashboard
    - terminology
    - github
    - bitbucket
    - app


tags:
    - beginner

---

Git is a distributed version control system. It's quite popular to work with text files — think in software development. It's fast, features data integrity and support for non-linear workflows (branching). Git was developed by Linux kernel developers (including Linus Torvalds). It's free and it mostly happens on the command line — think in the terminal.

Git can help you to collaborate on code projects, keep track of your code changes and rollback to certain points in time if needed. It comes with:

* A history of all files included, so you can undo all changes
* Powerful file merging on line base which makes collaboration easy

With fortrabbit you also use Git to deploy code from your local machine to our remote machines.



## Learn Git

To successfully use fortrabbit you should be familiar with Git standard operations and concepts — `commit`, `push`, `pull`. If you don't know Git yet, go ahead learn it. You will profit from it — either you and using fortrabbit or not. It's a cornerstone of todays software development. It's not very intuitive at start, but very very handy when you know it a bit better. There are many good tutorials out there in the interwebs to get started. For example:

* [Offical docs from Git SCM](https://git-scm.com/doc)
* [Try Git in your browser for free](https://try.github.io/levels/1/challenges/1)
* [Guides & ebook from Git Tower](https://www.git-tower.com/learn/)
* [Get Git right from Atlassian](https://www.atlassian.com/git/)
* and [many more](http://lmgtfy.com/?q=learn+git)



## Install Git

To use Git, you need to have it installed on your local machine. You might already have Git? Open a Terminal session and type ``git --version`.


### Git on Mac Os & Linux

Good news: you most likely already have it installed.


### Git on Windows

Good news: you can work it out. There are multiple version floating around, we recommend to download and install Git from the **[official Git website](https://git-scm.com/downloads)**. This one comes with the "Git Bash".

* [StackOverflow: Difference between Gits for Windows](http://stackoverflow.com/questions/22310007/differences-between-git-scm-msysgit-git-for-windows)
* [Beanstalk Git Windows install guides with SSH key setup](http://guides.beanstalkapp.com/version-control/git-on-windows.html#installing-Git)




## Installing your public SSH keys

BRAVO: You know how to use Git and you have installed on your machine. There is one more thing: You need to install and use SSH keys to make it work with fortrabbit. Please see the dedicated article: [SSH key setup help and troubleshooting](/ssh-keys) to get that working.


## Using Git with fortrabbit

Once you have everything setup and working you can have a look at how to use Git with fortrabbit:

### Using the remote Git repo as a version control system

Each [App](app) comes with a dedicated Git remote repo. This server-side repository can act as a backup for your code and as the base for collaboration. All fortrabbit Git repos are private for you and your team.


### Using Git for deployment

On top of the standard usage we also use Git to deploy your code. Your ``git push`` on the master branch will update the remote repository and trigger a chain of processes which will finally distribute your code changes to your public [Apps](app). Please see our [deployment article](deployment) for detailed informations and advanced usage.


## Git desktop app GUIs

Most people probably use Git on command line level (bash/terminal). But you will also find GUIs (desktop Apps) to manage Git. Those help to get started and to see visually what's going on:

* [Git Desktop GUIs list](https://git-scm.com/downloads/guis)


### Git is not GitHub

Sometimes people confuse Git with GitHub. [GitHub](https://github.com) is a popular provider of hosted Git repositories. GitHub has extended Git workflows with neat communication tools around the basic Git usage. Most notable is the "pull request" workflow. The fortrabbit [Dashboard](dashboard) does not offer such project communication tools — it's barebone Git only. You can however build your own workflow which includes a hosted Git repo on GitHub ([integration guide](/github)) or Bitbucket ([integration guide](/bitbucket)) or any another Git repo provider.
