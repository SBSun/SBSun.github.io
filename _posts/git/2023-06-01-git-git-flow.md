---
title: "[Git] Git Flow 협업 방식 사용법"
excerpt: "Git Flow 협업 방식 사용법에 대해서 알아보자."

categories:
  - Git
tags:
  - [Git]

published: true

permalink: /git/git-flow/

toc: true
toc_sticky: true

date: 2023-06-01
last_modified_at: 2023-06-05
---

새로 시작하게 된 TodoList 팀 프로젝트에서 Git Flow를 통해 협업하는 방식을 사용해보기로 했다.<br>

전에 진행했었던 프로젝트에서는 master, develop, feature 브랜치로 나누어 작업을 했었지만 Git Flow에 대하 자세히 알아보지 않은 상태로 사용했었다. 이번에 Git Flow에 대해 자세히 알아보고 한 번 적용해보도록 하기록 했다.

<br><br>

## **Git Flow란?**
<hr />

Git Flow는 Branching Model 중 하나로 분리된 Branch를 통해서 '시간의 흐름'에 따라 관리하는 것이 Git Flow의 핵심이다.<br>

Git Flow의 특징은 브랜치가 5가지로 나뉜다는 것이다.<br>

**main(master)** : 서비스를 배포하는 역할을 하는 브랜치<br>

**develop** : feature에서 개발된 내용이 저장되는 브랜치. 새로운 기능을 개발하는 여러 개발자들은 develop 브랜치를 기준으로 feature 브랜치를 생성해서 작업을 한 다음 다시 develop 브랜치로 내용을 병합한다.<br>

**feature** : 새로운 기능 개발을 위해 생성하는 브랜치. develop 브랜치를 기반으로 생성되며 기능 개발이 완료된 다음 다시 develop 브랜치로 병합된다.<br>

**hotfix** : main 브랜치로 배포를 한 후에 버그가 생겼을 때 빨리 고치기 위한 브랜치<br>

**release** : 새로운 릴리즈를 생성하기 위해 만들어지는 브랜치. develop 브랜치를 기반으로 생성되며 develop 브랜치와 master 브랜치로 병합된다. 릴리즈 브랜치에는 릴리즈 준비 과정에서 발견된 버그 수정 사항 같은 패치들만 적용이 되며, 메이저 기능들은 추가되지 않는다.<br>

<img src="../../../assets/images/posts/devops/git/git-git-flow/git-git-flow-1.PNG" width="100%"><br>

**main, develop**은 **필수 브랜치**이지만 나머지 브랜치는 유지 보수를 목적으로 하는 선택적 브랜치다.<br> 진행하는 프로젝트와 스타일에 따라 커스터마이징하면 되는데 이번 프로젝트는 규모가 크지 않으므로 **main, develop, feature** 3개의 브랜치만 사용하기로 결정했다.

<br><br>

## **Git Flow 사용법**
<hr />

Git Flow를 사용하기 위해서는 git-flow를 설치해야 하는데 Windows 사용자라면, Bash를 설치할 때 기본으로 설치가 된다.

<br>

### **1. git init 실행**
git-flow를 사용하기 위해서는 git 저장소를 git-flow에 맞게 초기화를 해야한다.<br>
어떤 브랜치를 어떤 용도로 사용할 것인지를 명시하게 된다.

``` bash
$ git flow init -d
```

`-d` 옵션을 추가하면 위의 기본적인 브랜치들이 자동으로 생성되는데 처음엔 **main, develop** 브랜치만 생성된다.

<br>

### **2. feature 브랜치 사용하기**
버그 수정이나 기능 추가를 위해 feature 브랜치를 사용할 수 있다. 새로운 개발 브랜치(feature)를 생성하기 위해 다음 명령을 실행하면 된다.<br>

``` bash
#새로운 feature를 개발할 때
$ git flow feature start <feature name>
# git flow feature start todo_entity -> feature/todo_entity
```

이 명령을 실행하면 develop 브랜치를 기반으로 새로운 feature 브랜치가 생성된다. 이후 자동으로 생성된 feature 브랜치로 checkout 된다.<br>

이제 평소 git을 이용한 것처럼 feature 브랜치에 개발을 진행하면 된다.<br>

기능 개발이 완료됐다면 다음 명령을 실행하면 된다.

``` bash
$ git flow feature finish <feature name>
# git flow feature finish todo_entity
```

이 명령어를 실행하면 git-flow가 develop 브랜치로 checkout한 다음 개발한 feature 브랜치의 내용을 병합하고 feature 브랜치를 제거한다.<br>

하지만 이번 프로젝트에서는 PR(Pull Request) 과정이 필요하기 때문에 이 기능은 사용하지 않는다.<br>

Github에서 PR을 사용하려면 우선 원격 저장소에 push가 되어 있어야한다.<br>

``` bash
$ git flow feature publish <feature name>
# git flow feature publish todo_entity
```
publish 명령을 수행하면 원격 저장소에 feature 브랜치를 push한다.<br>

그 후에 개발한 내용을 commit하고 원격 저장소에 코드를 push한다.<br>

마지막으로 Merge pull request를 통해 todo_entity 브랜치의 내용을 upstream/develop 브랜치로 병합한다.<br>

PR 후에 develop 브랜치를 최신화하고 `finish` 명령어를 실행하여 로컬뿐만 아니라 원격에 있는 `feature/todo_entity` 브랜치를 제거한다.
``` bash
$ git flow feature finish <feature name>
# git flow feature finish todo_entity
```

<br>

### **feature 브랜치 사용법 요약**

``` bash
# 1. feature 브랜치 생성
$ git flow feature start todo_entity

# 2. 생성한 feature 브랜치 원격 저장소에 push
$ git flow feature publish todo_entity

# 3. 개발한 코드 commit 후 push
$ git commit -m 'Feat : Todo Entity 생성'
$ git push

# 4. upstream/develop 브랜치를 병합하여 최신화
$ git merge upstream/develop

# 5. Pull Request 생성 및 Merge

# 6. PR 요청이 수락되면 로컬 develop 브랜치를 최신화하고 로컬 및 원격 저장소에 있는 feature/todo_entity 브랜치 제거
$ git flow feature finish todo_entity
```

<hr />
참고자료<br>
<a href="https://jinwoo1990.github.io/git/git-flow-tutorial/">https://jinwoo1990.github.io/git/git-flow-tutorial/</a><br>