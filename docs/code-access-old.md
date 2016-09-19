---

template:       article
reviewed:       2016-01-08
naviTitle:      Code access — Old Apps
title:          Different code deployment methods compared - Old Apps
lead:           How you can authenticate to access code on fortrabbit.
dontList:       true
oldApp:         true
group:          Kitchen_sink

keywords:
    - ssh key
    - git
    - git-deployment
    - ssh
    - FTP
    - FTPs

seeAlsoLinks:
    - git-deployment
    - ssh-sftp-old-app
    - collaboration
    - directory-structure
    - app
    - private-composer-repos

tags:
    - beginner

---


## Authentication methods

Public SSH key authentication is how you identify yourself on fortrabbit when deploying code. We assume that you are familiar with the general concept and usage. On fortrabbit, you manage your public SSH keys in the [Dashboard](dashboard).


### Account SSH keys

**This is the recommended method:** You store and manage your public SSH keys with your user Account on fortrabbit. This way you always have up-to-date code access on each App you own or you are collaborating with. It also makes managing collaboration easy — add/remove collaborators and code access is handled "automagically".


### App-only SSH keys

In certain cases you might want to add code access to an App without the need to register a new Account with fortrabbit. One case is some hectic ad-hoc hotfix scenario (good luck!), another case is that you have some advanced deployment with a third party continuous integration service bot going on. So you can install additional App-only custom public SSH keys with each App. You manage those App-only SSH keys in the Dashboard.

## Legacy SSH & SFTP access

Old Apps can also access the Apps via [SSH/SFTP](ssh-sftp-old).