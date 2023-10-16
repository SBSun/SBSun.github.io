---
title: "[Git] Git Rebase를 사용하여 git-flow 커밋 이력 깔끔하게 만들기"
excerpt: "Git Rebase를 사용하여 git-flow 커밋 이력 깔끔하게 만들어보자."

categories:
  - Git
tags:
  - [Git]

published: true

permalink: /git/git-rebase-and-git-flow/

toc: true
toc_sticky: true

date: 2023-06-07
last_modified_at: 2023-06-07
---

## **Feature 개발 마무리 전 커밋 이력 깔끔하게 만들기**
<hr />

git-flow로 협업을 하는 도중에 불필요하게 분리되어 있는 커밋들을 보고 하나의 커밋으로 만들어서 PR을 보내야겠다고 생각했다.<br>

``` bash
$ git log --oneline
```
``` bash
8d73a8e (HEAD -> feature/daily-todolist, origin/feature/daily-todolist) Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수 수정 기능
a2037ba Feat : TodoList Update - TodoList의 Todo 개수 수정 기능 구현, Test Code 작성 중
7e11966 Fix : TodoListController API 정보 수정, Todo Test 수정
c94af3d Merge remote-tracking branch 'upstream/develop' into develop
```

`7e11966` 커밋을 시작점으로 돌아가 `a2037ba`, `8d73a8e` 커밋들을 하나로 합쳐 'Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수, Todo 완료 개수 수정 기능'으로 수정하고자 한다.

<br>

``` bash
$ git rebase -i 7e11966
```

``` bash
pick a2037ba Feat : TodoList Update - TodoList의 Todo 개수 수정 기능 구현, Test Code 작성 중
pick 8d73a8e Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수 수정 기능

# Rebase 7e11966..8d73a8e onto 7e11966 (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
```

`git rebase -i <돌아가고 싶은 커밋 시점>` 을 통해 돌아가고 싶은 커밋 시점 이후의 커밋 내용을 수정

<br>

``` bash
pick a2037ba Feat : TodoList Update - TodoList의 Todo 개수 수정 기능 구현, Test Code 작성 중
squash 8d73a8e Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수 수정 기능
```

pick 으로 되어 있는 부분을 squash 아니면 s로 수정<br>
**이때 squash 나 s로 변경한 부분 위에 합칠 수 있는 커밋이 존재해야 한다.**

<br>

``` bash
# This is a combination of 2 commits.
# This is the 1st commit message:

Feat : TodoList Update - TodoList의 Todo 개수 수정 기능 구현, Test Code 작성 중

# This is the commit message #2:

Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수 수정 기능
```

위와 같은 로그들이 출력 되면 해당 커밋 내용을 수정하기 원하는 내용으로 수정

<br>

``` bash
# This is a combination of 2 commits.
Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수, Todo 완료 개수 수정 기능
```

위의 커밋 메시지들을 지우고, 원하는 커밋 메시지로 수정한 뒤 저장하고 편집창을 나간다. (esc + :wq)

<br>

``` bash
6c0b428 (HEAD -> feature/daily-todolist, origin/feature/daily-todolist) Feat : TodoList 수정 기능 구현 - TodoList의 Todo 개수, Todo 완료 개수 수정 기능
7e11966 Fix : TodoListController API 정보 수정, Todo Test 수정
```

로그를 다시 확인하면 `7e11966`을 시작점으로 남아있고 `a2037ba`, `8d73a8e` 커밋들이 `6c0b428` 커밋으로 합쳐진 것을 확인할 수 있다.

<br>

``` bash
$ git push -f origin feature/daily-todolist
```

마지막으로 원격 저장소에 강제로 push를 진행한다.

<hr />
참고자료<br>
<a href="https://jinwoo1990.github.io/git/git-flow-tutorial/">https://jinwoo1990.github.io/git/git-flow-tutorial/</a><br>