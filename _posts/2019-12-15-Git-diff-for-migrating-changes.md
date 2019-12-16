---
layout: post
title: Git diff for migrating changes between two git repositories
---
 
Let's assume you are maintaining two different git repositories for your project.
One repo is your internal repo where you do all kinds of commits, this repo could be your daily driver repo where you don't care about commit messages. The other repo of your project is a public repo where you don't want to expose all the minor commit messages and where you want each commits to be sensible and publicly audit-able. Problem when maintaining this workflow is how to port changes from one repo to another. As you may know git is a distributed version control system and on a fundamental level, all of your changes are in the form of diffs, so there is a solution.
 
So this is a workflow for porting your changes from your internal repo to public repo.
 
[//]: # <!–break–>
 
This assumes you have cloned your public Github repo from url https://github.com/your_public_repo and want to add changes to this repo from your internal git repo with url
 
`https://<repo_url>/internal_repo`
 
### Pushing Bitbucket repo changes to github repo changes
 
#### Go to folder containing  github repo with command :
```
cd <github_repo>
```
 
#### Add additional remote origin ( now repo will have two origins )
```
git remote add INTERNAL_REPO https://<repo_url>/internal_repo
```
 
#### Check diff for both
```
git diff -G. master INTERNAL_REPO/master
```
 
####  Make patch from diff
```
git diff -G. master INTERNAL_REPO/master > testPatch.patch
```
 
#### Actually apply Patch
```
git apply testPatch.patch
```
 
#### add and commit changes
```
git add --all
git commit -m "commit msg"
```
#### Push changes to origin
```
git push origin master
```
 
Let me know if you have encountered a similar problem and this workflow helped you in some way. Also I am open to suggestions of better alternatives for this exact situation. 
