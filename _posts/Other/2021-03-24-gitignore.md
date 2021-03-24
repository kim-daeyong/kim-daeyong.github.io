---
layout: post
title: gitignore를 이미 push된 repository에 추가할 때
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: Git
categories : [Other]
---

### gitignore를 이미 push된 repository에 추가할 때

git을 사용하다보면 gitignore를 후에 추가하거나 나중에 수정할때가 있다.  
이때 새로 변경된 gitignore를 추가하는 법  

```
$ git rm -r --cached .

$ git add .

$ git commit -m " commit "

$ git push origin master
```

로 추가해주면 된다.