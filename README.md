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

2챕터 - 예제 도메인 모델
------------------------

### 예제 도메인 모델과 동작 확인

-	스프링 데이터 JPA도 결국에는 JPA를 쓰는 것임. jpa 엔티티를 대상으로 저장, 조회 등

-	project structure - facts - jpa - data-jpa_main 추가 시 개발 시 도움받을 수 있음 (인텔리제이에서 연관관계등 쉽게 이동가능?)

-	jpa는 기본적으로 디폴트 생성자가 필요함. (private으로 만들면 안됨)

	-	롬북의 해당 어노테이션으로 대체 가능 @NoArgsConstructor(access = AccessLevel.PROTECTED)

-	@ToString 으로 toString 대체 가능

	-	연관관계가 걸려있는 대상은 대상에 포함시키지 말것 (해당 연관관계에서도 toString으로 무한 반복될 수있음)

---

3챕터 - 공통 인터페이스 기능
----------------------------

### 순수 JPA 기반 리포지토리 만들기

-	jpa 기반 repository 만든 후 스프링 데이터 JPA 공통인터페이스로 변환하는 과정으로 진행

-	직접 생성한 repository 클래스에 em.persist , em.remove 등 을 통해 구현, 테스트함.

### 공통 인터페이스 설정

-	원래는 applications 위에 어노테이션으로 @EnableJpaRepositories(basePackages = ) 이 필요함. 하지만 부트 사용 시 @SpringBootApplication 위치를 자동 지정함 (해당 패키지와 하위 패키지)

	-	위치가 변경된다면 지정해야함 (대부분 스캔할 것임)

-	MemberRepository 는 jparepository 를 상속받았는데, 구현체가 없음.

	-	출력해보면 class.com.proxy.$Proxy~~ 으로 나옴
		-	스프링 데이터jpa가 자바의 기본적인 프록시 기술로 가짜 클래스만들고 주입을 해준 것임
	-	이 인터페이스를 보고 **스프링데이터JPA** 가 구현체를 만들어서 injection을 해줌
	-	@Repository 생략 가능
	-	컴포넌트 스캔, JPA를 예외를 스프링 예외로 변환 등 기능

### 공통 인터페이스 적용

-	동일한 테스트 MemberRepository 를 사용해 테스트

### 공통 인터페이스 분석

-	스프링 데이터

	-	스프링 데이터 JPA : JpaRepository (종류에 따라 나누어지는 것을 알 수 있음)

-	주의

	-	T findOne(ID) -> Optional<T> findById(ID) 로 변경

-	주요 메서드

	-	save(S) : 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.
	-	delete(T) : 엔티티를 하나 삭제한다. 내부에서 EntityManager.remove() 호출
	-	findById(ID) : 엔티티를 하나 조회한다. 내부에서 EntityManager.find() 호출
	-	getOne(ID) : 엔티티를 프록시로 조회한다. 내부에서 EntityManager.gerReference() 호출
	-	findAll(...) : 모든 엔티티를 조회한다. 정렬, 페이징 조건을 파라미터로 제공할 수 있다.

-	참고

	-	JpaRepository는 대부분의 공통메서드를 제공한다.

4챕터 - 쿼리 메서드 기능
------------------------

### 메소드 이름으로 쿼리 생성

-	쿼리 메소드 기능

	-	메소드 이름으로 쿼리 생성

-	SPRING DATA 레퍼런스를 보면 다양한 사용조건들을 (and, or 등) 알 수 있음.

	-	https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-lookup-strategies

-	스프링 데이터 JPA가 제공하는 쿼리 메소드 기능

-	참고

	-	필드명이 변경되면 인터페이스에 정의한 메서드 이름도 변경해야 함.
	-	그렇지 않으면 애플리케이션 시적시점에 오류발생함 (로딩시점에 오류를 발견할 수 있으니 큰 장점임!)
