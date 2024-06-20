---
title: How to Move a Git Repository with Multiple Branches to a New Server Using a Bash Script
date: 2019-11-16 19:10:00 +0700
categories: [Devops, Git]
tags: [Devops, Git, Tools, "Scripts"]
pin: false
---

# Introduction

You want to move a repository to a new git server, but your repo has too many branches to move them one by one.

# Solution

We will use a simple bash script to get the list of branches from the original repo's remote (`origin`) and push them to the new repo's remote (`dest`).

## Syntax

First, let's define the syntax for our script:

```bash
./sync.sh path-to-directory origin dest
```

In this script:

- `origin` and `dest` are the two remotes set in the local repo you cloned.
- `path-to-directory` is the relative path to the directory containing your local repo.

## Step 1: Clone the Original Repo

Clone the original repo:

```bash
git clone git://path-to-your-repo
cd your-repo
git fetch --all
```

## Step 2: Add the New Repo Remote

Add the URL of the new repo to your local repo:

```bash
git remote add dest git://path-to-your-remote-dest
```

## Step 3: Get the List of Branches

Get the list of branch names from the original remote. We need to remove the remote part from the branch name, e.g., `origin/master` becomes `master`:

```bash
git branch -r | grep -v '\->' | grep 'origin/' | sed "s/origin\///g"
```

## Step 4: Sync the Branches

Sync all branches using a while loop:

```bash
function sync {
  while read line
  do
    writeHR;
    echo "Checking out $line";
    git checkout $line;
    git pull origin $line;
    git pull dest $line;
    git push dest $line;
  done;
}
```

## Complete Script

Combining the steps above, we have the complete script:

```bash
#!/bin/bash

function writeHR {
  echo '---------------------'
}

function sync {
  while read line
  do
    writeHR;
    echo "Checking out $line";
    git checkout $line;
    git pull $src $line;
    git pull $dest $line;
    git push $dest $line;
  done;
}

cd $1;
echo "You are currently on `realpath .`"

src="$2";
dest="$3";

echo "Fetching remote: $src";
git fetch $src;
echo "Fetching remote: $dest";
git fetch $dest;

git branch -r | grep -v '\->' | grep $src/ | sed "s/$src\///g" | sync
```

Here, we use `$src` and `$dest` as the names of the two remotes. These variables are passed into the script when it's called:

```bash
./sync.sh path-to-directory src-remote dest-remote
```

You can find this script on Gist: [https://gist.github.com/thanhtunguet/bac5a6b6fc21a6b0ada66cc7fee1e776](https://gist.github.com/thanhtunguet/bac5a6b6fc21a6b0ada66cc7fee1e776)
