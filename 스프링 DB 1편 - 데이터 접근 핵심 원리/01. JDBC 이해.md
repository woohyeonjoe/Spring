# JDBC 이해

## 프로젝트 생성

- `Java 11`
- `JDBC API` `H2 Database` `Lombok`
- `Jar`

## **H2 데이터베이스 설정**

- 사용자명은 sa 입력
- JDBC URL에 다음 입력,
- jdbc:h2:~/test (최초 한번) 이 경우 연결 시험 을 호출하면 오류가 발생한다. 연결 을 직접 눌러주어야 한다.
- ~/test.mv.db 파일 생성 확인
- 이후부터는 jdbc:h2:tcp://localhost/~/test 이렇게 접속

**+H2 설정 추가 설명**

[https://www.inflearn.com/questions/512746/h2-처음-사용시-몇-가지-의문점](https://www.inflearn.com/questions/512746/h2-%EC%B2%98%EC%9D%8C-%EC%82%AC%EC%9A%A9%EC%8B%9C-%EB%AA%87-%EA%B0%80%EC%A7%80-%EC%9D%98%EB%AC%B8%EC%A0%90)

## JDBC 이해

**애플리케이션 서버와 DB - 일반적인 사용법**

![image](https://user-images.githubusercontent.com/106286686/209949241-553a9e23-ddd9-40ab-9995-7361ffe6ba84.png)

1. 커넥션 연결: 주로 TCP/IP를 사용해서 커넥션을 연결한다.
2. SQL 전달: 애플리케이션 서버는 DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달한다.
3. 결과 응답: DB는 전달된 SQL을 수행하고 그 결과를 응답한다. 애플리케이션 서버는 응답 결과를 활용한다.

**문제는 각각의 데이터베이스마다 커넥션을 연결하는 방법, SQL을 전달하는 방법, 그리고 결과를 응답 받는 방법이 모두 다르다는 점이다.**

**이런 문제를 해결하기 위해 JDBC라는 자바 표준이 등장한다.**

**JDBC 표준 인터페이스**

![image](https://user-images.githubusercontent.com/106286686/209949282-cfbfeb2d-e4f1-44da-9b10-cf4700f92323.png)

이 JDBC 인터페이스를 각각의 DB 벤더(회사)에서 자신의 DB에 맞도록 구현해서 라이브러리로 제공하는데, 이것을 JDBC 드라이버라 한다.

**JDBC 표준 인터페이스라는 추상화에만 의존**

애플리케이션 로직은 이제 JDBC 표준 인터페이스에만 의존한다. 따라서 데이터베이스를 다른 종류의 데이터베이스로 변경하고 싶으면 JDBC 구현 라이브러리만 변경하면 된다. 따라서 다른 종류의
데이터베이스로 변경해도 애플리케이션 서버의 사용 코드를 그대로 유지할 수 있다.

개발자는 JDBC 표준 인터페이스 사용법만 학습하면 된다. 한번 배워두면 수십개의 데이터베이스에
모두 동일하게 적용할 수 있다.

**한계**

각각의 데이터베이스마다 SQL, 데이터타입 등의 일부 사용법이 다르다. 

결국 데이터베이스를 변경하면 JDBC 코드는 변경하지 않아도 되지만 SQL은 해당 데이터베이스에 맞도록 변경해야한다. (JPA에서 어느정도 해결)

## JDBC와 최신 데이터 접근 기술

JDBC를 편리하게 사용할 수 있는 기술

- SQL Mapper
    - 장점: JDBC를 편리하게 사용하도록 도와준다.
        - SQL 응답 결과를 객체로 편리하게 변환해준다.
        - JDBC의 반복 코드를 제거해준다.
    - 단점: 개발자가 SQL을 직접 작성해야한다.
    - 대표 기술: 스프링 JdbcTemplate, MyBatis
- ORM
    - ORM은 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술이다. 이 기술 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어
    실행해준다. 추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결해준다.
    - 대표 기술: JPA, 하이버네이트, 이클립스링크
    - JPA는 자바 진영의 ORM 표준 인터페이스이고, 이것을 구현한 것으로 하이버네이트와 이클립스 링크 등의 구현 기술이 있다.

![image](https://user-images.githubusercontent.com/106286686/209949326-a1c6cffd-3984-4b4c-9676-2b0eac511058.png)

## 데이터베이스 연결

**ConnectionConst 클래스 생성**

```java
public abstract class ConnectionConst {
    public static final StringURL= "jdbc:h2:tcp://localhost/~/test";
    public static final StringUSERNAME= "sa";
    public static final StringPASSWORD= "";
}
```

abstract로 객체 생성을 방지, public static으로 접근.

**DBConnectionUtil 클래스 생성**

```java
@Slf4j
public class DBConnectionUtil {

    public static Connection getConnection() {
        try {
            Connection connection = DriverManager.getConnection(URL,USERNAME,PASSWORD);
log.info("get connection={}, class={}", connection, connection.getClass());
            return connection;
        } catch (SQLException e) {
            throw new IllegalStateException(e);
        }
    }
}
```

데이터베이스에 연결하려면 JDBC가 제공하는 DriverManager.getConnection(..) 를 사용하면 된다.

이렇게 하면 라이브러리에 있는 데이터베이스 드라이버를 찾아서 해당 드라이버가 제공하는 커넥션을 반환해준다. 여기서는 H2 데이터베이스 드라이버가 작동해서 실제 데이터베이스와 커넥션을 맺고 그 결과를 반환해준다.

**JDBC DriverManager 연결 이해**

![image](https://user-images.githubusercontent.com/106286686/209949370-a4e9b679-4ade-4b3d-9d75-19bfee89e0eb.png)

DriverManager는 라이브러리에 등록된 드라이버 목록을 자동으로 인식한다. 이 드라이버들에게
순서대로 다음 정보를 넘겨서 커넥션을 획득할 수 있는지 확인한다.

- URL, 이름, 비밀번호 등 접속에 필요한 정보를 드라이버 매니저에 넘김
- 각각의 드라이버는 URL 정보를 체크해서 본인이 처리할 수 있는 요청인지 확인한다.
- 요청할 수 있는 드라이버를 찾으면 클라이언트에게 커넥션 구현체를 반환

## JDBC 개발 - 등록, 조회, 수정, 삭제

**Member 클래스 생성**

**MemberRepositoryV0 클래스 생성**

```java
@Slf4j
public class MemberRepositoryV0 {

    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values (?,?)";

        Connection con = null;
        PreparedStatement pstmt = null;     //SQL Injection 공격을 예방하려면 PreparedStatement를 통한 파라미터 바인딩 방식을 사용해야 한다.

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());       //'?'에 값 지정
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();  //쿼리 실행 (쿼리에 영향 받은 raw수를 반환)
            return member;
        } catch (SQLException e) {
log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }

    }

    public Member findById(String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";

        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);

            rs = pstmt.executeQuery();  //쿼리 실행 (쿼리 결과를 반환)
            if(rs.next()) {     //ResultSet 내부의 커서 이동 (처음엔 커서가 아무것도 가르키고 있지 않다.), 만약 조회되는 값이 2개 이상이면 while을 이용
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" + memberId);
            }

        } catch (SQLException e) {
log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, rs);
        }
    }

    public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            int resultSize = pstmt.executeUpdate();     //쿼리 실행 (쿼리에 영향 받은 raw수를 반환)
log.info("resultSize={}", resultSize);
        } catch (SQLException e) {
log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    public void delete(String memberId) throws SQLException {
        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    //사용한 커넥션과 statement는 정리해주어야 한다. (리소스 정리) -> 정리하지 않으면 커넥션 부족 장애 발생
    private void close(Connection con, Statement stmt, ResultSet rs) {      //Statement: PrepareStatement의 추상화 인터페이스

        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
log.info("error", e);
            }
        }

        if(stmt != null) {
            try {
                stmt.close();  //SQLException
            } catch (SQLException e) {
log.info("error", e);
            }
        }

        if(con!=null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

    }

    private Connection getConnection() {
        return DBConnectionUtil.getConnection();
    }
}
```

**MemberRepositoryV0Test 클래스 생성**

```java
@Slf4j
class MemberRepositoryV0Test {

    MemberRepositoryV0 repository = new MemberRepositoryV0();

    @Test
    void crud() throws SQLException {
        //save
        Member member = new Member("memberV1", 10000);
        repository.save(member);

        //findById
        Member findMember = repository.findById(member.getMemberId());
log.info("findMember = {}", findMember);
log.info("member == findMember {}", member == findMember);
log.info("member equals findMember {}", member.equals(findMember));     //Lombok의 @Data가 equals()메서드를 오버라이딩 해줘서 가능
assertThat(findMember).isEqualTo(member);

        //update: money: 10000 -> 20000
        repository.update(member.getMemberId(), 20000);
        Member updateMember = repository.findById(member.getMemberId());
assertThat(updateMember.getMoney()).isEqualTo(20000);

        //delete
        repository.delete(member.getMemberId());
        Assertions.assertThatThrownBy(() -> repository.findById(member.getMemberId()))      //삭제 검증을 위해 의도적 예외 발생
                .isInstanceOf(NoSuchElementException.class);
    }

}
```

