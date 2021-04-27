---
layout: post
title: Make a git repo
categories: basin_delineate_setup
excerpt: "Make a GitHub repo for _basin_delineate_, clone it locally and add the PyDev project and push it back to GitHub"
tags:
  - Eclipse
  - Basin extract
  - git
  - push
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2020-10-17 11:27'
modified: '2020-10-17 T18:17:25.000Z'
comments: true
share: true
fig1: eclipse_ide_launcher
fig2: eclipse_welcome_page
fig3: eclipse_menu_open-other-perspective
fig4: eclipse_window_select_perspective
fig5: eclipse_select_interpreter_ortho-py36
---

## Introduction

The python package _basin_extract_ is freely available as a [repository on GitHub](https://github.com/karttur/basin_extract/). This post is a manual for how it was constructed. The post is thus only of interest if you want to create a similar repo.

## Prerequisits

You need to have a [GitHub](https://github.com) account and you must have [Installed git for command line](https://karttur.github.io/git-vcs/git/git-commandline-install/).

## _basin_extract_ at GitHub

This post outlines how to create a [GitHub](https://github.com) repo with a default _main_ branch and a _dev_(elopment) branch using [git command line tool](https://karttur.github.io/git-vcs/git/git-commandline-install/). You have to register a free account with [GitHub](https://github.com) and then [create a repository](https://karttur.github.io/git-vcs/git/git-github/). The name of the repo with the _basin_extract_ code is called _basin_extract_. For a more detailed instruction on creating a new repo, see the post [Remote repositories with GitHub](https://karttur.github.io/git-vcs/git/git-github/).

![github-settings-menu-SSH-keys](../../images/github-code-alts.png){: .pull-right}
When the new repo is created, click on the green button <span class='button'>Code</span>. There are (in November 2020) 4 alternatives for cloning, plus the additional options [Open with GitHub Desktop] and [Download ZIP]. Copy the text for the clone alternative [SSH], as shown in the figure to the right.

## Clone to local repo

Open a <span class='app'>Terminal</span> window on your local machine and <span class='terminalapp'>cd</span> to the parent directory where you want to create the _clone_ of your remote (GitHub) repository. Then type (but do **not** execute):

<span class='terminal'>$ git clone </span>

If you followed the instructions in the previous section, you can now just paste the SSH link from your clipboard to the command line:

<span class='terminal'>$ git clone git@github.com:karttur/basin_extract.git</span>

Execute the command and the remote repo ("basin_extract") should clone to your local machine. If this is the first time you use the SSH key, you will be prompted the following statement and question:

```
Cloning into 'basin_extract'...
The authenticity of host 'github.com (140.82.118.4)' can't be established.
RSA key fingerprint is [.....].
Are you sure you want to continue connecting (yes/no)?
```

Answer with a full <span class='terminal'>yes</span>

The cloning will only take a few seconds, and the result is reported at the terminal prompt:

```
Cloning into 'basin_extract'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), 12.54 KiB | 2.51 MiB/s, done.
```

Change direction (<span class='terminalapp'>cd</span>) to the newly cloned repo and list (<span class='terminalapp'>ls</span>) the content:

<span class='terminal'>$ cd basin_extract</span>

<span class='terminal'>$ ls</span>

if you instead type (<span class='terminalapp'>ls -a</span>) also hidden files will be listed, including <span class='file'>.git</span>.

<span class='terminal'>$ ls -a</span>

The post on [Local git control](https://karttur.github.io/git-vcs/git/git-local-use) explains how to [Check and setup user name](https://karttur.github.io/git-vcs/git/git-local-use).

You can check the remote repo associated with your local clone:

<span class='terminal'>git remote -v</span>

```
origin	git@github.com:karttur/basin_extract.git (fetch)
origin	git@github.com:karttur/basin_extract.git (push)
```

Check out the branches and history:

<span class='terminal'>$ git log \-\-all \-\-decorate \-\-oneline \-\-graph</span>

```
* a5a7152 (HEAD -> main, origin/main, origin/HEAD) Initial commit
```


For the _clone_ we are working with the _HEAD_ pointer points towards two new items: _origin/main_ and _origin/HEAD_. As _origin_ can be seen as an alias for the primary remote repo, the pointing is towards the remote (GitHub) repo. As _HEAD_ is pointing both towards the local _main_ and the remote _main_, our local repo is in sync with the GitHub repo.

## Create/edit .gitignore file

Open/create the file <span class='file'>.gitignore</span> using for example the terminal editor <span class='terminalapp'>pico</span>.

<span class='terminal'>$ pico .gitignore</span>

Add the following items to ignore:

```
.DS_Store
*.pyc
__pycache__/
```

The first item (.DS_Store) is an internal Mac OSX file, the second (*.pyc) is a wildcard for ignoring all python cache files and the last item (\_\_pycache\_\_/) is the directory holding the python cache files.

Hit [ctrl]+[X] to exit <span class='terminalapp'>pico</span> and save the edits by pressing <span class='terminal'>Y</span> when asked.

### Stage and commit I

With edits done to the <span class='file'>.gitignore</span> you can _stage_ (add) and _commit_ (lock in and save) the changes to your local git repository:

<span class='terminal'>git add .<br>
git commit \-m \'created .gitignore\'</span>

As this is the first time you _stage_ something you have to use the separate <span class='terminal'>git add .</span> command.

If all the files that you want to commit have already been staged before, you can shorten the two commands to:

<span class='terminal'>git commit \-am \'created .gitignore\'</span>

## Create or copy PyDev project

The repo (_basin_extract_) only contains the <span class='file'>README, LICENSE</span> and <span class='file'>.gitignore</span> files (plus <span class='file'>.git</span>). The Python package that you want to develop with the help of git must be either created inside the repo or, if you have already started the package, copied into it. As I have already built a first version of the Python package _basin_extract_, I copy the complete structure into the repo. The initial structure of _basin_extract_ thus becomes (leaving out the content of <span class='file'>.git</span> and adding comments):

```
.
|____README.md # initial README markdown (md) text file
|____.gitignore
|____.git
|____basin_delineate # This is the Eclipse project (containing the package)
| |____.project # definition of the Eclipse project
| |____.pydevproject # definition of the Eclipse PyDev project
| |____basin_extract # The main python package
| | |______init__.py # Python default initiation script file
| | |____basin_extract.py # The main python module
| |____params # Python package for setting parameters
| | |____be_params.py # python module for setting parameters
| | |______init__.py # Python default initiation script file
| | |____be_xml.py # python package for reading xml parameters
| |____ds_manage # python package for spatial data source (ds) management
| | |______init__.py # Python default initiation script file
| | |____datasource.py # python module for patial data source (ds) management
```

### Stage and commit II

Stage and commit the complete project in your local clone of the repo _basin_extract_.

<span class='terminal'>git add .<br>
git commit \-m \'initial commit of PyDev project\'</span>

Then check the status of your repo:

<span class='terminal'>$ git status</span>

```
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

Or with the more decorated option:

<span class='terminal'>$ git log \-\-all \-\-decorate \-\-oneline \-\-graph</span>

```
* 1430b14 (HEAD -> main) initial commit of PyDev project
* a5a7152 (origin/main, origin/HEAD) Initial commit
```

### Push to origin

Push the committed changes to your remote repo on GitHub:

```
git push  <REMOTENAME> <BRANCHNAME>
```

<span class='terminal'>$ git push origin main</span>

```
Enumerating objects: 1139, done.
Counting objects: 100% (1139/1139), done.
Delta compression using up to 4 threads
Compressing objects: 100% (1104/1104), done.
Writing objects: 100% (1138/1138), 8.10 MiB | 3.79 MiB/s, done.
Total 1138 (delta 229), reused 0 (delta 0)
remote: Resolving deltas: 100% (229/229), done.
To github.com:karttur/basin_extract.git
   a5a7152..1430b14  main -> main
```

<span class='terminal'>$ git status</span>

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

### Create local dev branch

Create a local development (_dev_) branch.

<span class='terminal'>$ git checkout -b \"dev\"</span>

```
Switched to a new branch 'dev'
```

<span class='terminal'>$ git branch [-r] [-a]</span> [-r] to see only remote branches, [-a] to see all branches.

```
* dev
  main
```

#### A minute edit

Do some small editing to any of the files in the repo, just for illustration really. As all files are already staged, you can stage and commit in one command:

<span class='terminal'>$ git commit -a -m \"update be_params.py\"</span>

```
[dev 4cc0223] update be_params.py
 1 file changed, 96 insertions(+), 97 deletions(-)
 ```

<span class='terminal'>$ git log \-\-all \-\-decorate \-\-oneline \-\-graph</span>

```
* 4cc0223 (HEAD -> dev) update be_params.py
* 1430b14 (origin/main, origin/HEAD, main) initial commit of PyDev project
* a5a7152 Initial commit
```

_HEAD_ is now pointing at the branch _dev_ and the remote repo (with the alias _origin_) is out of sync still pointing at _masin_.

### Upload branch

Upload the branch _dev_ to the repo

<span class='terminal'>$ git push origin dev</span>

```
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 4 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 975 bytes | 975.00 KiB/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
remote:
remote: Create a pull request for 'dev' on GitHub by visiting:
remote:      https://github.com/karttur/basin_extract/pull/new/dev
remote:
To github.com:karttur/basin_extract.git
 * [new branch]      dev -> dev
```

<span class='terminal'>$ git log \-\-all \-\-decorate \-\-oneline \-\-graph</span>

```
* 4cc0223 (HEAD -> dev, origin/dev) update be_params.py
* 1430b14 (origin/main, origin/HEAD, main) initial commit of PyDev project
* a5a7152 Initial commit
```

_HEAD_ is pointing towards the branch _dev_, also in the remote repo (_origin/dev_).

Return to the online (remote) [GitHub repo _basin_extract_](https://github.com/karttur/basin_extract).

<figure>
<img src="../../images/github_basin_extract_two_branches.png">
<figcaption> GitHub repo after pushing local branch "dev".
</figcaption>
</figure>

![swithc_branch](../../images/github-switch-branch.png){: .pull-right}
Change to the branch _dev_ by clicking the branch button, as illustrated to the right. The page for the _dev_ branch (below) tells you that _This branch is 1 commit ahead of main._. Thus we know that all has worked out as we intended.  

<br />
<br />
<br />

<figure>
<img src="../../images/github_basin_delineate_dev-branch.png">
<figcaption> GitHub repo showing the "dev" branch.
</figcaption>
</figure>
