---
title: "[AWS] Spring Boot + AWS RDS (MySQL) 배포 및 연동"
excerpt: "AWS의 RDS 서비스를 사용하여 MySQL을 배포해보자"

categories:
  - AWS
tags:
  - [AWS, RDS, Spring Boot]

permalink: /aws/aws-springboot-rds-mysql/

toc: true
toc_sticky: true

date: 2022-12-20
last_modified_at: 2022-12-21
---

<img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-1.png" width="10%">

## **RDS(Relational Database Service)**
* <span style="color:red">관계형 데이터베이스</span>를 이용할 수 있는 서비스
* DB 설정, 패치, 백업 등 시간 소모적인 관리 작업을 AWS에서 처리
* RDBMS 클라우드 서비스 : Amazon Aurora, MySQL, MariaDB, PostgreSQL, Oracle 등을 지원
<br><br>

## **1. RDS Database 생성하기**
데이터베이스 생성 방식 : 표준 생성<br>
엔진 옵션 : MySQL<br>
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-2.PNG" width="90%"></p>
<br>

템플릿 : 프리 티어<br>
설정 : 사용자 이름, 마스터 암호로 실제 DB에 접근하니 잘 기억해두자.<br>
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-3.PNG" width="90%">
<img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-4.PNG" width="90%"></p><br>

DB 인스턴스 클래스 : 프리 티어 선택 시, 자동으로 버스터블 클래스로 설정된다.<br>
스토리지 : 프리 티어로 이용하기 때문에 최솟값인 20GB로 설정하고, 자동 조정 기능은 임계값 초과 시 요금이 발생할 수 있으니 비활성화 시켜준다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-5.PNG" width="90%"></p><br>

연결 : 외부 컴퓨터로 접속 가능하게 "예"로 설정
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-6.PNG" width="90%"></p><br>

데이터베이스 인증 : 암호 인증<br>
데이터베이스 옵션 : 초기 데이터베이스 이름을 설정해야 한다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-7.PNG" width="90%"></p><br>

## **2. 보안 그룹 생성하기**
<br>1. 생성한 데이터베이스를 클릭해, 상세 화면으로 이동한다.
<br>2. VPC 보안 그룹을 클릭해, 보안 그룹페이지로 이동한다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-8.PNG"></p>
<br>3. "보안 그룹 생성"을 클릭합니다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-9.png"></p>
<br>4. 보안 그룹을 생성하고 누구나 접속 가능하게 설정해준다. (추후에 변경 예정)
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-10.png"></p>
<br>5. DB 인스턴스 수정 페이지에서 생성한 보안 그룹으로 변경한다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-11.png"></p><br>

## **3. Workbench에서 DB 연동하기**
<br>1. DB의 앤드포인트를 복사한다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-12.png"></p>

<br>2. 복사한 앤드포인트를 Hostname에 붙여 넣고 설정한 Username과 Password를 입력한뒤 연결한다.
<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-13.png"></p><br>


## **Spring Boot에서 DB 연동하기**

<br><span style="font-size:120%"> **1. application.properties 설정**</span>

<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-14.png"></p><br>

<span style="font-size:120%"> **2. 프로젝트 실행**</span>

<p align="center"><img src="../../assets/images/posts/aws/aws-springboot-rds-mysql/aws-springboot-rds-mysql-15.png"></p><br>
Run(실행)했을 때 작동한다면 연동은 끝이다!

<hr/>

참고자료<br>
<a href="https://velog.io/@u-nij/Spring-Boot-AWS-RDS-MySQL-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0">https://velog.io/@u-nij/Spring-Boot-AWS-RDS-MySQL-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0<br>