스프링 데이터 JPA
-----------------

-	https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84/dashboard

---

-	스프링 데이터 JPA : 스프링 프레임워크와 JPA 기반위에 JPA를 정말 편리하게 사용하도록 도와주는 기술

-	이 강의에서는 순수하게 JPA로 개발한 코드를 보여주고, 스프링 데이터 JPA로 변경하면서 강의할 예정

-	수 많은 기능을 하나씩 따라하는게 아닌 그 기능 중 실무에서 많이 사용하는 기술 위주로 설명 예정

---

1챕터 - 프로젝트 환경설정
-------------------------

### 프로젝트 생성

-	https://start.spring.io/ 에서 쉽게 프로젝트 생성 후 인텔리제이에서 open 하면 완료!

### 라이브러리 살펴보기

-	gradle 의존관계 보기

	-	폴더에서 명령어 실행 시 어떤 게 있는 지 볼 수 있음
	-	./gradlew dependencies --configuration compileClasspath

-	부트 2.2 부터 start-test 할때 기존 의존관계가 junit5 임.

	-	junit-vintage 는 junit4 호환버전이라 exclude 하는 설정을 볼 수 있음

-	스프링 부트 라이브러리 살펴보기

	-	spring-boot-starter-data-jpa

		-	boot-starter-jdbc
			-	HikariCP : 부트 2. 부터 데이터베이스 커넥션 풀의 대한 기본을 HikariCP를 씀. 성능이 빠름 (기존 톰캣jdbc랑 설정이 다름)

	-	assertJ : assertThat 을 체이닝해서 쉽게 쓸 수 있음

	-	핵심 라이브러리

		-	스프링 MVC
		-	스프링 ORM
		-	JPA, 하이버네이트
		-	스프링 데이터 JPA

	-	기타 라이브러리

		-	H2 데이터베이스 클라이언트
		-	커넥션 풀: 부트 기본은 HikariCP
		-	로깅 SLF4J & LogBack
		-	테스트

### H2 데이터베이스 설치

-	H2 버전은 부트에 맞게 받을 것

-	처음에 웹 화면에 접속 시 주의 (JDBC URL 부분 다음과 같이 입력)

	-	최초: jdbc:h2:~/datajpa
	-	이후: jdbc:h2:tcp://localhost/~/datajpa

### 스프링 데이터 JPA와 DB 설정, 동작 확인

-	properties 지우고 yml 로 설정파일 사용

	-	show_sql 는 jpa가 실행하는 쿼리를 콘솔에 남김
	-	org.hibernate.SQL: debug 을 통해 로그에 남길 것

-	동작테스트

	-	부트 테스트는 @SpringBootTest 로 쉽게 가능

	-	jpa의 모든 변경은 트랜잭션 안에서 이뤄줘야함 @Transactional 사용할 것

	-	테스트 시 쿼리가 안보이는 건 springboottest가 transactional이 있으면 끝나고 롤백을 시켜버림

-	같은 트랜잭션 내에서는 영속성컨텍스트의 동일성 보장됨

-	로그에 남는 쿼리의 파리미터에 ? 로 표시되는데, 설정으로 확인할 수 있음

	-	org.hibernate.type: trace

	-	하지만 외부 라이브러리를 사용하면 더 쉽게 볼 수 있음 (운영환경 적용 시에는 성능 테스트가 필수라고 함)

	-	p6spy

---

2챕터 예제 도메인 모델
----------------------

### 예제 도메인 모델과 동작 확인

-	스프링 데이터 JPA도 결국에는 JPA를 쓰는 것임. jpa 엔티티를 대상으로 저장, 조회 등

-	project structure - facts - jpa - data-jpa_main 추가 시 개발 시 도움받을 수 있음 (인텔리제이에서 연관관계등 쉽게 이동가능?)

-	jpa는 기본적으로 디폴트 생성자가 필요함. (private으로 만들면 안됨)

	-	롬북의 해당 어노테이션으로 대체 가능 @NoArgsConstructor(access = AccessLevel.PROTECTED)

-	@ToString 으로 toString 대체 가능

	-	연관관계가 걸려있는 대상은 대상에 포함시키지 말것 (해당 연관관계에서도 toString으로 무한 반복될 수있음)
