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

### JPA NamedQuery

-	JPA의 NamedQuery를 호출할 수 있음

	-	쿼리의 이름을 부여하고 호출하는 기능

-	jpa는 관례 이름의 namedquery 찾은 후 없으면 메서드이름으로 쿼리생성함

-	namedquery는 실무에서 거의 사용안함.

	-	엔티티에 있는 것도 이상함...
	-	repository 메서드에 바로 쿼리를 지정할 수 있는데 이 기능으로 대체가능

-	가장 큰 장점?

	-	em.createQuery 로 쿼리를 작성하는건 문자임. 애플리케이션 실행 시점에는 오타가 있어도 에러가 발견안됨.(사용해야 발견)
	-	namedQuery는 애플리케이션 로딩 시점에 파싱을해서 오류가 있으면 알려줌!!

### @Query, 리포지토리 메서드에 쿼리 정의하기

-	메서드위에 @Query를 통해 쿼리작성 가능

	-	이것도 애플리케이션 로딩 시점에 에러 발견 가능
	-	메서드명을 줄일 수 있음
	-	원하는 쿼리 작성 가능

-	동적 쿼리는 querydsl 이 가장 깔끔하고 유지보수하기 좋음

### @Query, 값, DTO 조회하기

-	실무에서 많이 사용하는 기능 !

-	dto 로 조회가 가능하지만, jpa의 new 명령어 필요함. (패키지명까지 필요)

	-	@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
	-	querydsl의 편한 대안이 있다고함.

### 파라미터 바인딩

-	jpql을 짤때 위치기반, 이름기반 사용함

	-	가독성과 유지보수를 위해서 가급적이면 이름기반을 사용함

-	위치기반

	-	select m from Member where m.username = ?0

-	이름기반

	-	select m from Member where m.username = :name

-	파라미터 바인딩

	-	Member findMembers(@Param("anme") String name);

-	컬렉션 파라미터 바인딩

	```java
	@Query("select m from Member m where m.username in :names")
	List<Member> findByNames(@Param("names") Collection<String> names);
	```

### 반환 타입

-	스프링 데이터 JPA는 유연한 반환 타입 지원

	-	컬렉션, 단건, 옵셔널 등 가능

-	컬렉션 조회 시 결과 값이 없으면 null이 아니라 빈 컬렉션이 반환된다!

	-	하지만 단건 조회는 null임
	-	jpa는 없으면 noresult exption이 뜸!! spring data jpa는 exception을 감싸서 값을 반환해줌.

-	단건 조회인데 결과가 2개라면?

	-	exception 발생 -> spring exception으로 변환해서 발생함 (추상화한 예외. 다른 jdbc로 바꿔도 동일한 예외를 준다 느낌!)

-	api문서에 다양한 타입 소개되어 있음

	-	https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types

### 순수 JPA 페이징과 정렬

-	순수 JPA에서는 페이징은 어떻게 구현하는 지 예제를 통해 설명함.

	-	.setFirstResult(offset)
	-	.setMaxResults(limit)

### 스프링 데이터 JPA 페이징과 정렬

-	페이징과 정렬 파라미터

	-	org.springframework.data.domain.Sort : 정렬 기능
	-	org.springframework.data.domain.Pageable : 페이징 기능

	-	특별한 반환 타입

	-	org.springframework.data.domain.Page : count 쿼리 결과를 포함하는 페이징

		-	page는 0부터 시작임

	-	org.springframework.data.domain.Slice : count 쿼리 없이 다음 페이지만 확인 가능 (3을 주면 +1 결과를 줌. -> limit 4;)

	-	List(자바 컬렉션) : count 쿼리 없이 결과만 반환

-	@Query 안에서 카운트 쿼리를 분리할 수 있음.

	```java
	@Query(value = "select m from Member m left join m.team t",
	    countQuery = "select count(m) from Member m")
	```

-	참고: 결과 값 반환 시 엔티티 그대로 주지말고 dto로 변환해서 줄 것 (엔티티는 숨겨야한다!)

	-	page.map() : 쉽게 dto로 변환하기.

### 벌크성 수정 쿼리

-	jpa는 데이터를 변경하면 변경감지로 인해 commit 시점에 엔티티 한건한건 update 됌

-	모든 데이터의 수정이 필요할땐? 벌크성 수정 쿼리로 사용

-	spring data jpa는 @Modifying 있어야함. (executeUpdate 를 해줌)

-	주의할 점

	-	영속성 컨텍스트를 무시하고 update를 실행하는 것이기 때문에 주의!!
	-	벌크연산 이후에는 영속성컨텍스트를 날릴 것 ! (flush, clear)
	-	spring data jpa 는 자동옵션이 있음

		-	@Modifying(clearAutomatically = true)

	-	mybatis 등이랑 섞어 쓸때도 데이터가 안맞을 수 있으니 주의할 것!!

### EntityGraph

-	기본편 강의의 페치조인을 보면 쉽게 이해할 수 있음.

	-	페치조인을 쉽게 사용하는 방법임

-	연관된 엔티티들을 SQL 한번에 조회하는 방법

	-	지연로딩 관계일 때 member 조회 후 team을 조회할 때 마다 쿼리가 실행됨.

-	기존에선 fetch join을 하면 member와 연관된 team을 끌어온다.

-	스프링 데이터 JPA 에서는 @EntityGraph 로 간단하고 깔끔하게 해결 가능

	-	JPA가 제공하는 엔티티 그래프 기능을 편리하게 도와주는 것임
	-	간단한건 @EntityGraph 쓰고
	-	복잡해지면 jpql 의 fetch join 으로 사용

### JPA Hint & Lock

-	JPA 쿼리 힌드 (SQL힌드가 아니라 JPA 구현체에게 제공하는 힌트 )

	-	변경감지 시 비용이 든다.

	-	100% 조회만 쓰고 싶다?

	-	@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))

	-	성능테스트 해보고 사용 결정할 것

	-	막 넣을 필요없음 (효과가 미미할 수 있음)

-	Lock

	-	jpa가 lock을 지원하고 스프링데이터 JPA가 어노테이션을 통해 쉽게 사용할 수 있도록 해준다.
	-	실시간 트래픽이 많은 곳에서는 lock을 사용하지 말 것

5챕터
-----

### 사용자 정의 리포지토리 구현

-	스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동 생성
-	스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야하는 기능이 너무 많음
-	다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면??

	-	JPA 직접 사용 (EntityManager)
	-	스프링 JDBC Template 사용
	-	MyBatis 사용
	-	데이터베이스 커넥션 직접 사용 등..
	-	Querydsl 사용 (이때 커스텀을 많이 사용함)

-	커스텀 구현 클래스 규칙

	-	리포지토리 인터페이스 이름 + Impl
	-	스프링 데이터 JPA가 인식해서 스프링 빈으로 등록해줌
	-	xml 설정 또는 javaConfig로 Impl을 쓰지 않을 수 있음
		-	관례를 따르는게 좋음... 유지보수측면에서!

-	인터페이스만으로 해결이되면 사용자정의 리포지토리를 구현안해도 되지만 복잡한 쿼리 등을 위해서 쓰임

-	참고: 항상 사용자 정의 리포지토리가 필요한 것은 아님.

	-	임의의 리포지토리를 만들어도 됨

		-	그냥 class 생성해서 빈으로 등록하고(@Repository) 직접 사용해도된다.

	-	핵심 비즈니스가 있는 리포지토리, 화면에 맞춘 DTO들을 뽑아서 사용하는 리포지토리는 분리하는 편이라고함.

### Auditing

-	엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶으면? (아래의 데이터들은 기본적으로 깔고 감)

	-	등록일
	-	수정일
	-	등록자
	-	수정자

-	JPA 주요 어노테이션

	-	@PrePersist, @PostPersist
	-	@PreUpdate, @PostUpdate

-	스프링 데이터 JPA

	-	부트 설정 클래스에 어노테이션 적용
		-	@EnableAuditing
	-	엔티티에 적용
		-	@EntityListeners(AuditingEntityListener.class)

-	등록자, 수정자는 부트 설정 클래스에 AuditorAware 을 통해 입력될 데이터 지정

	```java
	@Bean
	public AuditorAware<String> auditorProvider(){
	    return () -> Optional.of(UUID.randomUUID().toString());
	}
	```

-	baseTimeEntity, baseEntity 형식으로 상속해서 사용 가능

-	참고

	-	예제에서 저장 시점에 등록일, 수정일에 같은 데이터가 저장된다. 데이터가 중복일 지라도 수정일 컬럼만 확인하면 마지막 업데이트 데이터를 확인할 수 있으니 유지보수 관점에서 편리함. (null은 피하자)

### Web 확장 - 도메인 클래스 컨버터

-	HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩

	-	HTTP 요청은 회원 ID 로 받지만 도메인 클래스 컨버터가 중간에 동작에서 회원 엔티티 객체를 반환해줌
	-	도메인 클래스 컨버터도 리파지토리르 사용해서 엔티티를 찾는다.

	```java
	@GetMapping("/members/{id}")
	public String findMember(@PathVariable("id") Long id){
	    Member member = memberRepository.findById(id).get();
	    return member.getUsername();
	}


	@GetMapping("/members/{id}")
	public String findMember2(@PathVariable("id") Member member){


	    return member.getUsername();
	}
	```

-	개인적으로 권장하지는 않는다고함.

	-	쿼리가 단순하게 동작하는 방식이 많지는 않음.
	-	간단할 때만 사용 가능
	-	컨버토로 받으면 트랜잭션범위가 없는 상황에서 조회했으므로 애매함.. 조회용으로만 사용 가능

### Web 확장 - 페이징과 정렬

-	스프링 데이터가 제공하는 페이징과 정렬기능을 스프링 MVC에서 편리하게 사용할 수 있음

	```java
	@GetMapping("/members")
	public Page<MemberDto> list(@PageableDefault(size = 5) Pageable pageable){
	    return memberRepository.findAll(pageable)
	            .map(MemberDto::new);
	}
	```

-	파라미터로 Pageable을 받는데, 인터페이스임 (실제는 PageRequest 객체 생성)

-	요청 파라미터

	-	/members?page=0&size=3&sort=id,desc&sort=username,desc

-	기본 값 조절하기

	-	글로벌 설정 (application.yml)

		```text
		data:
		  web:
		    pageable:
		      default-page-size: 10
		      max-page-size: 2000
		```

	-	개별 설정

		-	@PageableDefault(size = 5)

-	접두사

	-	페이징 정보가 둘 이상이면 접두사로 구분

	```java
	public String list(
	@Qualifier("member") Pageable memberPageable,
	@Qualifier("order") Pageable orderPageable,
	...)
	```

-	Page 내용 DTO로 변환

	-	엔티티를 API로 노출하면 다양한 문제가 발생함. 엔티테를 꼭 DTO로 변환해서 반환해야 함

	```java
	return memberRepository.findAll(pageable)
	              .map(MemberDto::new);
	```

-	Page 1부터 시작하기

	-	스프링 데이터는 Page를 0부터 시작함
	-	1부터 쓰고싶으면?
		-	Pageable, Page 말고 직접 클래스를 만들어서 처리
	-	one-indexed-parameters 옵션 true 설정 (Page 의 값들이 0 인덱스 기준이어서 맞지않음)

	-	결국 0부터 사용을 권장

6챕터 - 스프링 데이터 JPA 분석
------------------------------

### 스프링 데이터 JPA 구현체 분석

-	스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체

	-	SimpleJpaRepository
	-	jpa 내부 기능을 활용해서 동작함

	-	exception 에 대해 스프링이 제공하는 exception으로 발생됨 (하부 구현기술을 바꿔도 기존 비즈니스 로직에 영향을 주지 않도록 감싸준다)

	-	모든 JPA는 트랜잭션안에서 동작해야하지만, 스프링 데이터 JPA 사용해서 save를 하면 트랜잭션 없이 실행이 된다.

		-	구현체 내부에 걸려있음

		-	서비스계층에서 트랜잭션 시작안하면 리파지토리에서 시작

		-	서비스계층에서 트랜잭션 시작하면 리파지토리는 그 트랜잭션 전파받아 사용

-	참고 : @Transactional(readOnly = true)

	-	데이터를 단순히 조회하고 변경하지 않는 트랜잭션에서 해당 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음

-	**중요** save 메서드

	-	새로운 엔티티면 저장(persist)
	-	새로운 엔티티가 아니면 병합(merge)
		-	DB에서 꺼내서 파라미터로 넘어온 새로운 것으로 바뀜
		-	DB select를 한번 한다는 것이 단점
		-	되도록이면 쓰지말것... 업데이트는 변경감지로 사용!!!
	-	merge는 영속상태 엔티티가 특정 이유로 영속상태에 벗어났을 때 다시 영속상태가 되어야할 때 써야함

### 새로운 엔티티를 구별하는 방법

-	save 메서드

-	새로운 엔티티를 판단하는 기본 전략

	-	식별자가 객체일 때 null로 판단
	-	식별자가 자바 기본 타입일 때 0 으로 판단
	-	Persistable 인터페이스를 구현해서 판단 로직 변경 가능

-	참고

	-	JPA 식별자 생성 전략이 @GenerateValue 면 save 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 persit 동작함.
	-	JPA 식별자 생성 전략이 @Id만 사용해서 직접할당이면 이미 식별자 값이 있는 상태로 save가 호출되기 때문데 merge가 호출된다 (기본 전략)
		-	merge는 DB를 호출해서 값을 확인하고 DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율
	-	따라서 이 경우 Persistable을 사용해서 새로운 엔티티 확인여부를 직접 구현해야 함 (@CreatedDate 를 활용하면 편리하게 확인 가능)

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id @GeneratedValue
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```
