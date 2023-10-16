---
title: "[DB] TodoList ERD 설계"
excerpt: "TodoList ERD 설계를 해보자."

categories:
  - DB
tags:
  - [DB]

published: true

permalink: /db/todolist-erd-design/

toc: true
toc_sticky: true

date: 2023-05-26
last_modified_at: 2023-05-26
---

새로 시작한 TodoList 프로젝트에서 Todo 기능을 맡았다.<br>

API 명세서를 작성하기 전에 Todo 테이블을 설계해보자.

<br><br>

## **요구 사항 분석**
<hr />

### **Todo**

1. 사용자는 원하는 날짜에 여러 Todo를 등록할 수 있다.  
2. 새로운 Todo를 등록하려면 제목(필수), 이모지(필수), 반복 요일(선택)을 입력해야 한다.
3. 사용자는 Todo의 제목, 반복 요일, 완료 여부, 이모지를 수정할 수 있다.

<br>

### **Used_Emoji**

Used_Emoji 테이블은 아래 사진과 같이 사용자가 최근에 사용한 이모지 데이터를 저장하는 테이블이다.
<br>
<img src="../../../assets/images/posts/development/db/db-todolist-erd-design/db-todolist-erd-design-1.PNG" width="40%"><br>

1. 사용자가 Todo 등록/수정에 사용한 이모지를 등록한다.
2. 사용자가 사용한 이모지들을 설정한 최대 갯수만큼 저장하여 최근에 사용한 이모지 목록을 제공한다.

<br><br>

## **ERD 설계**

<img src="../../../assets/images/posts/development/db/db-todolist-erd-design/db-todolist-erd-design-2.PNG" width="90%"><br>

아직 다른 도메인에 대한 설계가 진행되지 않아서 앞으로 업데이트 할 예정이다.
