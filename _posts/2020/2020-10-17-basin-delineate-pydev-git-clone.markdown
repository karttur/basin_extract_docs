---
layout: post
title: git clone basin_extract
categories: basin_delineate_setup
excerpt: "git clone basin_extract from GitHub for use in Eclipse"
tags:
  - Eclipse
  - Basin extract
  - git
  - clone
image: ts-mdsl-rntwi_RNTWI_id_2001-2016_AS
date: '2020-10-17 11:27'
modified: '2020-10-17 T18:17:25.000Z'
comments: true
share: true
figure3: github-framework_karttur_03_Git-repo-alternatives
figure301: github-framework_karttur_0301_Git-repo-alternatives
---

## Introduction

The python package _basin_extract_ is freely available as a [repository on GitHub](https://github.com/karttur/basin_extract/).

If you just want to download _basin_extract_ from GitHub, just go ahead and you are done with this post. The rest of this post deals with cloning either a stable version or a development version. The latter is intended for those who want to participate in the code development.

## Prerequisits

If you want to use _basin_extract_ with the IDE <span class='app'>Eclipse</span> you must have setup <span class='app'>Eclipse</span> as outlined in the [previous](../basin-delineate-eclipse-setup) post. If you want to participate in the development you should also have setup the [git version control system](https://karttur.github.io/git-vcs/).

## Cloning alternatives

First you need to decide if you want to participate in the development, or even develop your own code. If you want to do either of those you should start by [forking the original repo to your own GitHub account](https://karttur.github.io/git-vcs/git/git-forks/). If you do not need your own development version you can skip the forking and just clone the _basin_extract_ from Karttur's repo.

### Fork project to your own repo

Login to your account on [GitHub](https://github.com/), or create a free account if you do not have one. While being logged in navigate to [Karttur's _basin_extract_ repo](https://github.com/karttur/basin_extract). In the upper right corner, below the top menu row, there is a button for <span class='textbox'>Fork</span>. Click on it, and GitHub creates a fork (copy), then opens it in your own account.

<figure>
<img src="../../images/github_fork_stulturum_basin_extract.png">
<figcaption> GitHub forked repo.
</figcaption>
</figure>

![swithc_branch](../../images/github-switch-branch.png){: .pull-right}
The forked repo has two branches. Change to the branch _dev_ by clicking the branch button, as illustrated to the right.

With your own fork of the project _basin_extract_ you are ready to get a local clone and starting coding. You can then push your improved/customized code to your won fork, and then push

<br />

### Local clone

Regardless if you want to clone the stable (_main_) or development (_dev_) version, you have some different options.

1. Clone using <span class='app'>Eclipse</span> at startup
2. Clone from inside <span class='app'>Eclipse</span>
3. Clone using git command line and then import separately into <span class='app'>Eclipse</span>

If you do not intend to use <span class='app'>Eclipse</span> use the third alternative.

#### Eclipse startup clone

![new-workspace-folder](../../images/eclispe_create_new_folder_in_workspace.png){: .pull-right}
This is probably the easiest way to get the whole project setup in <span class='app'>Eclipse</span>. Start <span class='app'>Eclipse</span> and when the IDE launcher page open, create a new folder in your workspace directory. When the welcome pages opens, select the alternative _Checkout projects from Git_ (illustrated below).

<figure>
<img src="../../images/eclipse_welcome_select_git.png">
<figcaption> From Eclipse Welcome page select _Checkout projects from Git_.
</figcaption>
</figure>

The next window that opens, <span class='window'>Select Repository Source</span> just highlight [Clone URI] and click <span class='button'>Next</span>.

<figure>
<img src="../../images/eclipse_select_repo_source.png">
<figcaption> Eclipse Select Repository Source - highlight [Clone URI] and click Next.
</figcaption>
</figure>

In the <span class='window'>Source Git Repository</span> window, paste the URI to the GitHub repo of _basin_extract_. Either Karttur's original repo (if you are not pushing any changes back online) or your own fork (to allow staging and committing changes to the online repo). Then click <span class='button'>Next</span>.

<figure>
<img src="../../images/eclipse_source_git_repo.png">
<figcaption> Eclipse Source Git repo.
</figcaption>
</figure>

In the Branch selection window you should have two alternatives, _dev_ and _main_. Choose the one that suit your needs (_main_ if you are user, _dev_ if you are a developer). Click <span class='button'>Next</span>.

<figure>
<img src="../../images/eclipse_git_branch_selection.png">
<figcaption> Eclipse Git Branch Selection.
</figcaption>
</figure>

Set the path to the directory you just created as the target <span class='textbox'>Directory</span>. _DO NOT_ accept the default target path suggested by <span class='app'>Eclipse</span>, under your user and then <span class='file'>git</span>. This will lead to conflicts when updating git itself. You can also change the <span class='textbox'>Initial branch</span> to _dev_ (or another branch). But keep the standard _origin_ for the <span class='textbox'>Remote name</span>.

<figure>
<img src="../../images/eclipse_git_clone_local-dir.png">
<figcaption> Eclipse Git Local Destination.
</figcaption>
</figure>

In the next window (_Select a wizard to use for importing projects_) click the radio button for <span class='textbox'>Importing Existing Eclipse Projects</span>. Then <span class='button'>Next</span>.

<figure>
<img src="../../images/eclipse_select_import_wizard.png">
<figcaption> Eclipse Select a wizard to use for importing projects.
</figcaption>
</figure>

And you should have reached the last window, <span class='window'>Import Projects</span> and this time you can happily click <span class='button'>Finish</span>.

<figure>
<img src="../../images/eclipse_import_project_finish.png">
<figcaption> Eclipse Finish Import projects.
</figcaption>
</figure>

When the project is imported, <span class='app'>Eclipse</span> looks for a Python interpreter.

<figure>
<img src="../../images/eclipse_python_interpreter_missing.png">
<figcaption> Eclipse python projects requires a Python interpreter.
</figcaption>
</figure>

Click the <span class='button'>Manual config</span> and you will get to to the Preferences window and the menu set to identifying Python interpreters.

<figure>
<img src="../../images/eclipse_python_interpreter.png">
<figcaption> Eclipse Preferences, identifying python interpreters.
</figcaption>
</figure>

The Python interpreter you need is described in the parallel post on [Eclipse for Basin extraction](../basin-delineate-eclipse-setup/).

#### Clone from inside Eclipse

You can clone a git repo from within an existing <span class='app'>Eclipse</span> project. Start an empty <span class='app'>Eclipse</span> project. Get the GIT perspective. Either from the menu system:

<span class='menu'>Window -> Perspective -> Open Perspective -> Other ... </span>.

<figure>
<img src="../../images/eclipse_window_perspecive_other.png">
<figcaption> Eclipse Menu path to different perspectives.
</figcaption>
</figure>

<figure>
<img src="../../images/eclipse_perspectives_other.png">
<figcaption> Eclipse menu of perspectives.
</figcaption>
</figure>

You can also use the search tool and start writing "git" and locate _git perspective_.

<figure>
<img src="../../images/eclipse_search_git.png">
<figcaption> Eclipse search.
</figcaption>
</figure>

In the <span class='tab'>Git repositories</span> view you should click the alternative _Clone a Git repository_ (if no text appears slide the cursor over the icons to get the alternative _Clone a Git repository and add the clone to this view_).

<figure>
<img src="../../images/github-framework_karttur_03_Git-repo-alternatives.jpg">
<figcaption> Eclipse git perspective.
</figcaption>
</figure>

<figure>
<img src="../../images/github-framework_karttur_0301_Git-repo-alternatives.jpg">
<figcaption> Eclipse git perspective.
</figcaption>
</figure>

This will open the same <span class='window'>Source Git Repository</span> window as in the previous section and the steps that then follows are the same. But <span class='app'>Eclipse</span> does not recognise the imported repo as a PyDev project. You have to define the PyDev project and then team up the git repo packages. How to do that is covered in the post [Setup Eclipse teamed with GitHub repository](https://karttur.github.io/geoimagine/develop/develop-github-eclipse/).
