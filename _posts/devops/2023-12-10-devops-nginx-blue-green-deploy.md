---
title: "[DevOps] Nginx를 사용한 Blue/Green 무중단 배포"
excerpt: "Nginx를 사용한 Blue/Green 무중단 배포"

categories:
  - DevOps
tags:
  - [DevOps, Deploy]

published: true

permalink: /devops/nginx-blue-green-deploy/

toc: true
toc_sticky: true

date: 2023-12-10
last_modified_at: 2023-12-10
---

## 배경

---

Github Actions와 Docker를 사용하여 배포 자동화 환경을 구축했지만, 배포를 하는 동안에는 애플리케이션이 종료되는 문제가 존재했습니다. 긴 시간은 아니지만 새로운 배포 버전 서비스가 실행되기 전까지 기존에 배포되어 있는 서비스가 중단됩니다. 때문에 서비스를 중단하지 않고 새로운 버전을 배포할 수 있는 무중단 배포를 구현하게 되었습니다.

<br>

## Nginx를 이용한 Blue/Green 무중단 배포 방법

---

Nginx는 외부의 요청을 받아 서버로 요청을 전달하는 **리버스 프록시** 기능이 존재합니다.
클라이언트의 요청을 Nginx가 받고, Nginx는 WAS에 클라이언트의 요청을 전달하는 **클라이언트 → nginx(웹서버) → WAS** 구조가 됩니다.

### Blue/Green 배포 방식

<img src="../../assets/images/posts/devops/devops-nginx-blue-green-deploy/devops-nginx-blue-green-deploy-1.gif" width="70%"><br>

트래픽을 한번에 구버전에서 신버전으로 옮기는 방법입니다. Blue/Green 배포 전략에서는 **현재 운영중인 서비스의 환경을 Blue라고 부르고, 새롭게 배포할 환경을 Green**이라고 부릅니다.

Blue 서버가 배포되어 있는 상태에서 Green 버전의 배포가 진행된다면, Green 버전 배포가 성공적으로 완료가 되었을 때, 트래픽을 Green으로 전환시키고, Blue 서버는 중단시킵니다.

저희는 하나의 EC2 프리 티어 서버에서 Port를 2개로 나누어 Blue/Green 배포 방식을 구축했습니다.
새로운 버전 또한 기존 EC2 서버에 그대로 적용하면 되므로 배포를 위해 EC2 서버가 추가로 더 필요하지 않기 때문에 규모가 작은 프로젝트의 비용 측면에서 효율적이라고 생각했습니다. 

구조는 EC2 서버 1대와 Nginx, 배포 파일 Jar 2대를 사용했습니다.

- Nginx에는 8080포트 할당
- 서버 A에는 8090 포트 할당
- 서버 B에는 8091 포트 할당

1. 클라이언트는 Nginx 서비스 주소로 접속(8080 포트)
2. Nginx는 클라이언트 요청을 받아 현재 연결된 서버 A로 요청을 전달합니다.(8090)
    - 연결되지 않은 서버 B는 전달받지 못합니다.
3. 신규 버전의 배포가 진행되면 Nginx와 연결되지 않은 서버 B로 배포됩니다.
    - Nginx는 현재 서버 A와 연결되어 있으므로 배포하는 동안 서비스가 중단되지 않습니다.
4. 배포 이후 서버 B가 정상적으로 구동되는지 확인하고, nginx reload 명령어를 통해 서버 B와 연결되도록 하고, 구버전 서버 A는 중단시킵니다.

이처럼 Nginx는 서버 내부에서 트래픽을 어디로 라우팅할 것인지 정해 요청을 전달하므로 한대의 서버에서 2개의 어플리케이션을 이용하여 무중단 배포를 할 수 있습니다.

<br>

<hr />
참고자료<br>
<a href="https://hudi.blog/zero-downtime-deployment/">https://hudi.blog/zero-downtime-deployment/</a><br>
<a href="https://cookiee.tistory.com/690">https://cookiee.tistory.com/690</a><br>
<a href="https://mr-popo.tistory.com/230">https://mr-popo.tistory.com/230</a><br>
<a href="https://soobysu.tistory.com/123?category=1035458">https://soobysu.tistory.com/123?category=1035458</a><br>