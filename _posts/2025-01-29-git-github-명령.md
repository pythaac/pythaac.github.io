---
layout: post
title: Git/Github 명령
date: 2025-01-29 01:03 +0900
toc: true
comments: true
categories:
- 수첩
tags:
- Git
- Github
description: Git 또는 Github 관련 명령어 모음
---

### main branch 외 모든 local branch 삭제
```bash
git checkout main && git branch | grep -v "main" | xargs git branch -D
```