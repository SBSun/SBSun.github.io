---
title: "[Spring] 트랜잭션 추상화 및 동기화"
excerpt: "트랜잭션의 추상화 및 동기화에 대해 알아보고 실습해보자."

categories:
  - Spring
tags:
  - [Spring, DB]

published: true

permalink: /spring/transaction-abstraction-and-synchronization/

toc: true
toc_sticky: true

date: 2023-07-21
last_modified_at: 2023-07-21

--- 

## **JDBC에서 트랜잭션을 사용할 때의 문제점**
<hr />

데이터베이스에서 `DML` SQL문 하나만 실행해도 데이터베이스에 쿼리 결과가 반영된다. 그 이유는 내부적으로 커밋을 자동으로 해주는 **auto commit** 기능이 기본적으로 활성화 되어있기 때문이다.<br>

하지만 애플리케이션을 개발할 때, 여러 개의 SQL 문을 하나의 논리적인 단위로 묶어 실행해야 하는 일이 빈번하다.
그리고 그 단위를 **트랜잭션**이라고 하는데 트랜잭션은 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하거나 모두 실패해야 하는 **원자성**을 보장해야 한다. **auto commit**이 활성화 되어 있는 상태에서는 각각의 단일 쿼리가 별개의 트랜잭션으로 동작하게 되어 원자성을 보장하지 못한다.<br>

JDBC만을 사용하여 트랜잭션을 구현하려면 **auto commit** 기능을 비활성화하고 직접 커밋과 롤백을 지정해 줘야 한다.
<br>

**member** 테이블에 두 명의 데이터가 존재한다고 가정하고, money를 이체하는 간단한 로직을 통해 트랜잭션을 구현해보자.<br>

``` java
public Member findById(Connection con, String memberId) throws SQLException{
    String sql = "SELECT * FROM member WHERE member_id = ?";
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try{
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, memberId);
        rs = pstmt.executeQuery();
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
        // Connection은 여기서 닫지 않는다.
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(pstmt);
    }
}

public void update(Connection con, String memberId, int money) throws SQLException{
    String sql = "UPDATE member SET money = ? WHERE member_id = ?";
    PreparedStatement pstmt = null;
    int result = 0;

    try {
        pstmt = con.prepareStatement(sql);
        pstmt.setInt(1, money);
        pstmt.setString(2, memberId);
        result = pstmt.executeUpdate();
        log.info("result = {}", result);
    } catch (SQLException e){
        log.error("db error", e);
        throw e;
    } finally {
        JdbcUtils.closeStatement(pstmt);
    }
}
```

위의 코드를 보면 파라미터로 Connection 객체를 념겨받아 사용하는 것을 볼 수 있다. 트랜잭션을 사용하기 위해서는 트랜잭션을 구성하는 여러개의 쿼리가 동일한 커넥션에서 실행되어야 하기 때문에 커넥션을 닫지 않는다.

<br>

``` java
public class MemberService {

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try{
            con.setAutoCommit(false); // 트랜잭션 시작

            // 비즈니스 로직
            bizLogic(fromId, toId, money);

            con.commit(); // 성공시 커밋
        }catch (Exception e){
            con.rollback(); // 실패시 롤백
            throw new IllegalStateException(e);
        }finally {
            release(con);
        }
    }

    private void bizLogic(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }
}
```

MemberService의 이체 메서드이다. 서비스 레이어임에도 불구하고 JDBC 코드들을 작성해야하고 `SQLException` 또한 JDBC 전용 예외이다. 이처럼 특정 기술에 대해 종속적으로 코드를 작성하게 되고, `try`, `catch`, `finally` 등 유사한 코드의 반복이 많아진다. 


<br><br>

## **트랜잭션 추상화**
<hr />

현재 서비스 계층은 트랜잭션을 사용하기 위해 JDBC에 의존하고 있다. 트랜잭션을 사용하는 코드는 데이터 접근 기술마다 다르기 때문에 향후 JDBC에서 JPA 같은 다른 데이터 접근 기술로 변경하면, 서비스 계층의 트랜잭션 관련 코드도 모두 함께 수정해야 한다.<br>

현재 서비스 계층의 코드는 **JDBC 트랜잭션에 의존**한다.<br>

이 문제를 해결하려면 트랜잭션 기능을 추상화하면 된다.<br>
스프링은 트랜잭션 추상화를 위해 **PlatformTransactionManager** 인터페이스를 제공한다.<br>

``` java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
    throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```

PlatformTransactionManager는 위와 같이 3개의 메서드를 정의하는데, 트랜잭션이 어디에서 시작되고 종료되는지, 종료는 정상(commit)인지 비정상(rollback)인지를 결정한다.<br>

PlatformTransactionManager의 구현 클래스는 **DataSourceTransactionManager**, **JpaTransactionManager**, **HibernateTransactionManager** 등 데이터 접근 기술에 따른 구현 클래스가 존재한다.<br>

``` java
public void accountTransfer(String fromId, String toId, int money) throws SQLException {
    // 트랜잭션 시작
    TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

    try{
        // 비즈니스 로직
        bizLogic(String fromId, String toId, int money)
        
        transactionManager.commit(status); // 성공시 커밋
    }catch (Exception e){
        transactionManager.rollback(status); // 실패시 롤백
        throw new IllegalStateException(e);
    }
}
```

`transactionManager.getTransaction()` 메서드는 트랜잭션을 시작하고, `TransactionStatus`를 반환하는데 현재 트랜잭션의 상태 정보가 포함되어 있다. 이후 트랜잭션을 커밋, 롤백할 때 필요하다.<br>

그리고 트랜잭션 경계를 설정하는 코드가 간결해졌고 **release**는 트랜잭션 매니저가 해주기 때문에 해당 코드를 제거했다.

<br><br>

## **트랜잭션 동기화**
<hr />

스프링이 제공하는 트랜잭션 매니저는 크게 **트랜잭션 추상화**, **리소스 동기화** 2가지 역할을 한다.<br>

트랜잭션을 유지하려면 트랜잭션의 시작부터 끝까지 같은 데이터베이스 커넥션을 유지해야한다. 위에서는 같은 커넥션을 사용하기 위해서 파라미터로 커넥션을 전달하는 방법을 사용했다.<br>

파라미터로 커넥션을 전달하는 방법은 코드도 지저분해지고, 커넥션을 넘기는 메서드와 넘기지 않는 메서드를 중복해서 만들어야 하는 등 여러거지 단점들이 많다.

<br>

### **트랜잭션 매니저와 트랜잭션 동기화 매니저**

스프링에서는 **TransactionSynchronizationManager** 클래스를 사용해서 트랜잭션을 동기화한다. 트랜잭션 매니저는 내부에서 이 트랜잭션 동기화 매니저를 사용한다.<br>

이 동기화 작업은 쓰레드 로컬을 사용해서 커넥션 객체를 보관하므로 멀티 쓰레드 환경에 안전하게 커넥션을 동기화 할 수 있다. 따라서 커넥션이 필요하면 트랜잭션 동기화 매니저를 통해 획득하면 되기 때문에 이전처럼 파라미터로 커넥션을 전달하지 않아도 된다.<br>

**동작 방식**
1. 트랜잭션 매니저는 데이터 소스를 통해 커넥션을 만들고 트랜잭션을 시작한다.
2. 트랜잭션 매니저는 트랜잭션이 시작된 커넥션을 트랜잭션 동기화 매니저에 보관한다.
3. 리포지토리는 트랜잭션 동기화 매니저에 보관된 커넥션을 꺼내서 사용한다.
4. 트랜잭션이 종료되면 트랜잭션 매니저는 트랜잭션 동기화 매니저에 보관된 커넥션을 통해 트랜잭션을 종료하고, 커넥션도 닫는다.

``` java
public class MemberRepository {

    private final DataSource dataSource;

    public MemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Member findById(String memberId) throws SQLException{
        String sql = "SELECT * FROM member WHERE member_id = ?";
        Connection connection = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try{
            connection = getConnection();
            pstmt = connection.prepareStatement(sql);
            pstmt.setString(1, memberId);

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
            close(connection, pstmt, rs);
        }
    }

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

    private void close(Connection con, Statement stmt, ResultSet rs){
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        // 트랜잭션 동기화를 사용하려면 DataSourceUtils를 사용해야 한다.
        DataSourceUtils.releaseConnection(con, dataSource);
    }

    private Connection getConnection() throws SQLException {
        // 트랜잭션 동기화를 사용하려면 DataSourceUtils를 사용해야 한다.
        Connection con = DataSourceUtils.getConnection(dataSource);
        return con;
    }
}
```

커넥션을 파라미터로 전달하는 부분이 모두 제거되었다.

<br><br>



<hr />
참고자료<br>
<a href="https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard">https://www.inflearn.com</a><br>
<a href="https://hudi.blog/spring-transaction-synchronization-and-abstraction/">https://hudi.blog/spring-transaction-synchronization-and-abstraction/</a><br>

