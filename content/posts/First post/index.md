---
title: Upgraded my website to Hugo!
date: 2025-02-01
lastmod: 2025-02-02
draft: false
tags:
  - website
---

I was tired of my old website with broken CSS and I wasn't working on it. But I discovered this fabulous tool called [Hugo](https://gohugo.io/), which is an amazing program that transforms regular markdown files into websites using templates! I went with the [Blowfish](https://blowfish.page/) one as it is pretty and well documented.

The source code is available on Github, and the final website is in the "deploy" branch.
{{< github repo="auxbh/auxbh.fr" >}}

I was inspired by NetworkChuck's [video](https://www.youtube.com/watch?v=dnE7c0ELEH8).
Below is a modified version of his [Mega Script](https://blog.networkchuck.com/posts/my-insane-blog-pipeline/#maclinux-1) without the Python part. It asks if you would like to input a custom commit message and is modified to my needs.

```sh
#!/bin/bash
set -euo pipefail

# Change to the script's directory
cd "/path/to/hugosite"

# Step 1: Build the Hugo site
echo "Building the Hugo site..."
if ! hugo; then
    echo "Hugo build failed."
    exit 1
fi

# Step 2: Add changes to Git
echo "Staging changes for Git..."
if git diff --quiet && git diff --cached --quiet; then
    echo "No changes to stage."
else
    git add .
fi

# Step 3: Ask for a custom commit message
echo "Enter a custom commit message (or press Enter to use the default message):"
read -r user_commit_message

# Set the commit message
if [[ -z "$user_commit_message" ]]; then
    commit_message="New Blog Post on $(date +'%Y-%m-%d')"
else
    commit_message="$user_commit_message"
fi

# Step 4: Commit changes with a dynamic message

if git diff --cached --quiet; then
    echo "No changes to commit."
else
    echo "Committing changes..."
    git commit -m "$commit_message"
fi

# Step 5: Push all changes to the main branch
echo "Deploying to GitHub Main..."
if ! git push origin main; then
    echo "Failed to push to main branch."
    exit 1
fi

# Step 6: Push the public folder to the Deploy branch using subtree split and force push
echo "Deploying to Temp Deploy..."
if git branch --list | grep -q 'temp-deploy'; then
    git branch -D temp-deploy
fi

if ! git subtree split --prefix public -b temp-deploy; then
    echo "Subtree split failed."
    exit 1
fi

if ! git push origin temp-deploy:deploy --force; then
    echo "Failed to push to deploy branch."
    git branch -D temp-deploy
    exit 1
fi

git branch -D temp-deploy

echo "All done! Site synced, processed, committed, built, and deployed."
```

Then, a webhook request is from Github to my VPS, making it pull the latest code from the Deploy branch.
This is made using [webhook](https://github.com/adnanh/webhook), and a tutorial is available [here](https://maximorlov.com/automated-deployments-from-github-with-webhook/).