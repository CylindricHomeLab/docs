---
title: 'Force Git to use LF everywhere'
taxonomy:
    category:
        - git
    tag:
        - git
---

## The Problem

The git developers are obsessed with the idea that Windows editors can't support LF and corrupt files by putting CRLF everywhere. This is pretty much untrue, and it's safe to use LF _everywhere_ now. This page explains how to make git just use LF and not try to convert anything to CRLF.

## The Solution

First, make sure the repository is is set to use LF for text files. Repeat these with `--global` to make the changes the default.

An AutoCRLF setting of `input` means that when committing new files, CRLF will be changed to LF. To just leave things alone and commit as-is, use `false`.

```sh
git config core.eol lf
git config core.autocrlf input

git config --global core.eol lf
git config --global core.autocrlf input
```

No make sure all local files are recreated with the correct line-endings:

```sh
git checkout-index --force --all
```

If there are still some files not reporting correct line-endings, remove everything from your local copy and update them:

```sh
git rm --cached -r .
git reset --hard
```