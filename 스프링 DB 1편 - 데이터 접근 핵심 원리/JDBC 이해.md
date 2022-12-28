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

## JDBC 이해
