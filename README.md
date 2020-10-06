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
