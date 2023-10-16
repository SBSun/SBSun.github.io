---
title: "[Spring] DataSource와 Connection Pool"
excerpt: "DataSource에 대해 알아보고 Connection Pool을 적용해보자."

categories:
  - Spring
tags:
  - [Spring, DB]

published: true

permalink: /spring/datasource-connection-pool/

toc: true
toc_sticky: true

date: 2023-07-18
last_modified_at: 2023-07-18

--- 

해당 포스팅은 <a href="https://sbsun.github.io/java/db-connection-and-crud-using-jdbc/">JDBC를 이용한 DB 연결 및 CRUD</a>와 <a href="https://sbsun.github.io/db/connection-pool/">Connection Pool이란?</a> 포스팅들과 이어진다.<br>

이번에는 DataSource에 대해서 알아보고, DataSource를 통해 HirakiCP Connection Pool을 적용해보겠다.

<br><br>

## **DataSource**
<hr />

직접 JDBC를 사용하면 DB에 접근할 때마다 Connection을 연결하고 끊는 과정을 반복해야 한다. 하지만 Connection Pool을 사용하면 애플리케이션을 시작할 때 일정 개수의 Connection을 미리 생성하여 Connection Pool에 저장해놓고, 이를 재사용할 수 있다.<br>

DataSource는 **DB Connection을 관리하고, Connection을 획득하는 방법을 추상화**하는 인터페이스다.

<br>

### **커넥션을 획득하는 방법을 추상화**

커넥션을 얻는 방법은 DriverManager를 직접 사용하거나 Connection Pool을 사용하는 등 다양한 방법이 존재한다.<br>
만약 현재 DriverManage를 통해서 커넥션을 획득하다가, 커넥션 풀을 사용하는 방법으로 변경하려면 어떻게 해야할까?<br>

**DriverManager를 통해 커넥션 획득하다가 커넥션 풀로 변경시 문제**<br>
``` java
public static Connection getConnection(){
    try {
        Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        log.info("Get Connection = {}, class = {}", connection, connection.getClass());
        return connection;
    } catch (SQLException e) {
        throw new IllegalStateException(e);
    }
}
```

위와 같이 로직에서 DriverManager를 사용해서 커넥션을 획득하다가 HikariCP 같은 커넥션 풀을 사용하도록 변경하려면 의존관계를 DriverManager에서 HikariCP로 변경해야 하기 때문에 커넥션을 획득하는 코드도 함께 변경해야 한다.<br>

자바에서는 이를 해결하기 위해 `javax.sql.DataSource` 인터페이스를 제공한다.<br>

``` java
public interface DataSource  extends CommonDataSource, Wrapper {
  Connection getConnection() throws SQLException;
}

public class DriverManagerDataSource extends AbstractDriverBasedDataSource {
}

public class HikariDataSource extends HikariConfig implements DataSource, Closeable{
}
```

대부분의 커넥션 풀은 **DataSource** 인터페이스를 이미 구현해두었다. 따라서 개발자는 **DBCP 커넥션 풀**, **HikariCP 커넥션 풀**의 코드를 직접 의존하는 것이 아니라 **DataSource** 인터페이스에만 의존하도록 애플리케이션 로직을 작성하면 된다.<br>

**DriverManager**는 DataSource 인터페이스를 사용하지 않아서 직접 사용해야 하는데 DataSource 기반의 커넥션 풀을 사용하도록 변경하려면 관련 코드를 전부 고쳐야 한다. 이런 문제를 해결하기 위해 스프링은 DriverManager도 DataSource를 통해서 사용할 수 있도록 **DriverManagerDataSource** 클래스를 제공한다.<br>

``` java
public class MemberRepositoryV1 {
    private final DataSource dataSource;

    public MemberRepositoryV1(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    private Connection getConnection() throws SQLException {
        Connection con = dataSource.getConnection();
        log.info("get connection={}, class={}", con, con.getClass());
        return con;
    }
}
```

위 코드와 같이 원하는 DataSource를 주입받아 사용하면 되기 때문에 MemberRepositoryV1의 코드는 변경하지 않아도 된다.

<br><br>

## **DataSource - Connection Pool**
<hr />

DataSource를 통해 커넥션 풀을 사용해보자.<br>
<hr />

``` java
@Slf4j
class MemberRepositoryV1Test {

    private MemberRepositoryV1 repository;

    @BeforeEach
    void beforeEach(){
        // 기본 DriverManager - 항상 새로운 커넥션 획득
//        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);

        // 커넥션 풀링
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);
        dataSource.setMaximumPoolSize(10);
        dataSource.setConnectionTimeout(3000);
        dataSource.setPoolName("MyPool");

        repository = new MemberRepositoryV1(dataSource);
    }

    @Test
    void crud() throws SQLException {
        // save
        Member member = new Member("memberV100", 10000);
        repository.save(member);

        // findById
        Member findMember = repository.findById(member.getMemberId());
        log.info("findMember = {}", findMember);
        assertThat(findMember).isEqualTo(member);

        // update
        repository.update(member.getMemberId(), 20000);
        Member updatedMember = repository.findById(member.getMemberId());
        assertThat(updatedMember.getMoney()).isEqualTo(20000);

        // delete
        repository.delete(member.getMemberId());
        // 삭제한 데이터를 검증하는 방법은 findById 메서드 내에 데이터가 존재하지 않을 경우의 예외가 발생하는지 검증한다.
        assertThatThrownBy(() -> repository.findById(member.getMemberId()))
                .isInstanceOf(NoSuchElementException.class);

        try {
            Thread.sleep(1500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
<br>

**DriverManagerDataSource 사용**<br>
```
get connection=conn0: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn1: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn2: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
...
```
* DriverManagerDataSource를 사용하면 conn0~5 번호를 통해서 항상 새로운 커넥션이 생성되어서 사
용되는 것을 확인할 수 있다.<br>

**HikariDataSource 사용**<br>
Hikari DataSource에 대한 코드로 대부분의 DataSource가 위 코드와 같이 Url, Username, Password 등을 입력받는다.<br>

* MaximumPoolSize : Connection Pool이 제공할 수 있는 최대 커넥션 개수
* ConnectionTimeout : 모든 커넥션이 사용 중일 때 대기하는 시간으로 기본값은 30초이다.

``` 
get connection=HikariProxyConnection@xxxxxxxx1 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx2 wrapping conn0: url=jdbc:h2:...
user=SA
get connection=HikariProxyConnection@xxxxxxxx3 wrapping conn0: url=jdbc:h2:...
user=SA
...
```
* 커넥션 풀 사용시 conn0 커넥션이 재사용 된 것을 확인할 수 있는데, 테스트는 순서대로 실행되기 때문에 커넥션을 사용하고 다시 돌려주는 것을 반복한다. 따라서 conn0만 사용된다.
* 웹 애플리케이션에 동시에 여러 요청이 들어오면 여러 쓰레드에서 커넥션 풀의 커넥션을 다양하게 가져가는
상황을 확인할 수 있다.

<br>

``` yml
spring:
  datasource:
    url:  jdbc:mariadb://${DB_HOST}:${DB_PORT}/${DB_NAME}?useSSL=false
    username: ${DB_USER_NAME}
    password: ${DB_PASSWORD}
```

이전까지는 application 파일에 DataSource 정보를 명시하기만 하면 알아서 DB에 연결됐으며, JPA 등의 구현체가 JDBC 단을 추상화해줬기 때문에 DataSource에 대해서는 전혀 무지했었다.<br>

이번 강의를 통해서 DB와 Connection이 실제로 어떻게 연결되고, 효율적으로 Connection을 관리하는 Connection Pool의 동작 원리까지 학습할 수 있었다.


<hr />
참고자료<br>
<a href="https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard">https://www.inflearn.com</a><br>
