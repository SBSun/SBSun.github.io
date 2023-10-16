---
title: "[JDBC] JDBC를 이용한 DB 연결 및 CRUD"
excerpt: "JDBC를 이용한 DB 연결 및 CRUD를 구현해보자."

categories:
  - Java
tags:
  - [Java, JDBC]

published: true

permalink: /java/db-connection-and-crud-using-jdbc/

toc: true
toc_sticky: true

date: 2023-07-16
last_modified_at: 2023-07-16

--- 

인프런의 <a href="https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard">스프링 DB 1편 - 데이터 접근 핵심 원리</a> 강의를 수강하고, 그 내용을 정리하면서 다시 한번 복습하기 위해 포스팅을 작성하게 되었다.

<br>

## **JDBC 사용 객체**
<hr />

**DriverManager**<br>
* JDBC가 제공하는 DriverManager는 라이브러이에 등록된 DB 드라이버들을 관리하고, 커넥션을 획득하는 기능을 제공한다.
* `getConnection()` 메소드를 호출하여 Connection 객체를 반환 받는다.

**Connection**<br>
* DB와 연결하는 객체
* `DriverManager.getConnection(url, user, password)`으로 Connection 객체를 생성한다.
  
**Statement**<br>
* DB와 연결되어있는 Connection 객체를 통해 SQL문을 DB에 전달하여 실행하고, 결과를 리턴받아주는 객체

**ResultSet**<br>
* SELECT문을 사용한 질의 성공시 반환되는 객체
* 커서(Cursor)를 이용하여 특정 행에 대한 참조를 조작

<br><br>

## **JDBC 처리 / 동작 순서**
<hr />

**1. 변수 세팅 및 실행할 SQL문 만들기**<br>
Conntection, Statement, ResultSet or int
<br>

**2. Connection 객체 생성**<br>
접속하고자 하는 DB정보를 입력하여 DB에 접속하면서 생성<br>
DriverManager.getConnection(url, 계정명, 비밀번호) + 예외처리
<br>

**3. Statement 객체 생성**<br>
Connection 객체를 이용하여 생성
<br>

**4. SQL문을 전달하면서 실행 및 결과 받기**<br>
Statement객체를 이용하여 SQL문 실행 (DB에 완성된 SQL문을 전달하고, 실행하고, 결과받기) 
<br>

**5. 자원 반납**<br>
close메소드를 이용하여 자원이 생성된 역순으로 반납<br>
Finally 블럭 내 + 예외처리(SQLException)

<br><br>

## **CREATE**
<hr />

``` java
@Slf4j
public class MemberRepositoryV0 {

    public Member save(Member member) throws SQLException{
        // 1. 변수 세팅 및 실행할 SQL문 만들기
        String sql = "INSERT INTO member(member_id, money) VALUES (?, ?)";

        Connection connection = null;
        PreparedStatement pstmt = null;
        int result = 0;

        try{
            // 2. Connection 객체 생성
            connection = getConnection();

            // 3. Statement 객체 생성
            pstmt = connection.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getMoney());

            // 4. SQL문을 전달하면서 실행 및 결과 받기
            result = pstmt.executeUpdate();
            return member;
        } catch (SQLException e){
            log.error("db error", e);
            throw e;
        } finally {
            // 5. 자원 반납
            close(connection, pstmt, null);
        }
    }

    // 자원을 반납하는 메서드
    private void close(Connection con, Statement stmt, ResultSet rs){

        // 자원 반납은 무조건 역순으로 해준다.
        if(rs != null){
            try {
                rs.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

        // statement를 close 하는 과정에서 예외가 발생하면 커넥션을 close하는 코드가 실행되지 않기 때문에 각각 try-catch로 잡아줘야한다.
        if(stmt != null){
            try {
                stmt.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

        if(con != null){
            try {
                con.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
    }

    private Connection getConnection() {
        return DBConnectionUtil.getConnection();
    }
}

@Slf4j
public class DBConnectionUtil {
    // JDBC가 제공하는 표준 Connection
    public static Connection getConnection(){
        try {
            /**
             * DB에 연결하려면 JDBC가 제공하는 DriverManager.getConnection()를 사용하면 된다.
             * 라이브러리에 있는 DB 드라이버를 찾아서 해당 드라이버가 제공하는 커넥션을 반환해준다.
             * JDBC가 제공하는 DriverManager는 라이브러리에 등록된 DB 드라이버들을 관리하고, 커넥션을 획득하는 기능을 제공한다.
             * DriverManager는 라이브러리에 등록된 드라이버 목록을 자동으로 인식하고, 이 드라이버들에게 순서대로 정보들을 넘겨서 커넥션을 획득할 수 있는지 확인한다.
            */
            Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
            log.info("Get Connection = {}, class = {}", connection, connection.getClass());
            return connection;
        } catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }
}
```
<br><br>

## **SELECT**
<hr />

``` java
public Member findById(String memberId) throws SQLException{
    // 1. 변수 세팅 및 실행할 SQL문 만들기
    String sql = "SELECT * FROM member WHERE member_id = ?";

    Connection connection = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try{
        // 2. Connection 객체 생성
        connection = getConnection();

        // 3. Statement 객체 생성
        pstmt = connection.prepareStatement(sql);
        pstmt.setString(1, memberId);

        // 4. SQL문을 전달하면서 실행 및 결과 받기
        rs = pstmt.executeQuery();

        // 무조건 한번 next 메서드를 호출해줘야 한다.
        if(rs.next()){
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        }else{
            throw new NoSuchElementException("member not found memberId = " + memberId);
        }
    } catch (SQLException e){
        log.error("db error", e);
        throw e;
    } finally {
        // 5. 자원 반납
        close(connection, pstmt, rs);
    }
}
```
<br>

### **ResultSet**
데이터를 변경할 때는 `pstmt.executeUpdate()`를 사용하여 int 값을 반환 받지만, 데이터를 조회할 때는 `pstmt.executeQuery()` 메서드를 사용하여 **ResultSet**에 담아서 반환 받는다.<br>

* ResultSet 내부에 있는 **cursor**를 이동해서 다음 데이터를 조회할 수 있다.
* `rs.next()` 메서드를 호출하면 커서가 다음으로 이동한다. 커서는 처음에 데이터를 가리키고 있지 않기 때문에 `rs.next()`를 **최초 한번은 호출**해야 데이터를 조회할 수 있다.
  * `rs.next()`의 결과가 `true`면 커서의 이동 결과 데이터가 있다는 뜻이다.
  * `rs.next()`의 결과가 `false`면 더이상 커서가 가리키는 데이터가 없다는 뜻이다.
* `rs.getString("member_id")` : 현재 커서가 가리키고 있는 위치의 `member_id` 데이터를 `String` 타입으로 반환한다.
* `rs.getInt("money")` : 현재 커서가 가리키고 있는 위치의 `money` 데이터를 `int` 타입으로 반환한다.

<br><br>

## **UPDATE**
<hr />

``` java
public void update(String memberId, int money) throws SQLException{
    String sql = "UPDATE member SET money = ? WHERE member_id = ?";

    Connection connection = null;
    PreparedStatement pstmt = null;
    int result = 0;

    try {
        connection = getConnection();
        pstmt = connection.prepareStatement(sql);
        pstmt.setInt(1, money);
        pstmt.setString(2, memberId);
        result = pstmt.executeUpdate();
        log.info("result = {}", result);
    } catch (SQLException e){
        log.error("db error", e);
        throw e;
    } finally {
        close(connection, pstmt, null);
    }
}
```
`executeUpdate()`는 쿼리를 실행하고 영향 받는 row 수를 반환한다.<br>
해당 코드에서는 하나의 데이터만 변경하기 때문에 1이 반환되지만, 만약 100명의 회원 데이터를 한번에 수정하는 Update SQL을 실행하면 결과는 100이 된다.

<br><br>

## **DELETE**
<hr />

``` java
public void delete(String memberId) throws SQLException{
    String sql = "DELETE FROM member WHERE member_id = ?";

    Connection connection = null;
    PreparedStatement pstmt = null;
    int result = 0;

    try{
        connection = getConnection();
        pstmt = connection.prepareStatement(sql);
        pstmt.setString(1, memberId);
        result = pstmt.executeUpdate();
    } catch (SQLException e){
        log.error("db error", e);
        throw e;
    } finally {
        close(connection, pstmt, null);
    }
}
```

<br><br>

## **JDBC를 직접 사용해본 후기**
<hr />

이때까지 ORM을 사용해서 DB 기능을 구현하다가 이번에 처음 JDBC를 직접 사용하여 간단한 DB 기능을 구현해 보았는데, 확실히 중복된 코드를 반복적으로 사용해야 하고 컴파일 체크가 불가능해서 문법 오류가 발생할 확률이 높다고 느껴졌다. <br>

이번 실습을 통해 JDBC가 어떻게 동작하는지 기본 원리를 학습했다.
<hr />
참고자료<br>
<a href="https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard">https://www.inflearn.com</a><br>
