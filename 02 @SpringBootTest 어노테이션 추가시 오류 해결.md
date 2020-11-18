# 02 @SpringBootTest 추가시 오류 발생 해결

- @SpringBootTest annotation에 관하여
- 어노테이션 추가시 발생하는 문제(패키지 구조 연관)
- @SrpingBootApplication annotation에 대하여 알아보자



## 2.1 @SpringBootTest annotation에 관하여 알아보자

@SpringBootTest annotation은 @SpringBootApplication을 실행하여 테스트를 위한 빈들을 다 생성한다.



## 2.2 @SpringBootTest annotation 추가시 발생하는 문제

@SpringBootTest의 역활을 알아봤다. 그럼 추가시 어떤 문제가 발생할 수 있는지 알아보자.

다음과 같이 코드를 작성해보았다.

<img src="./img/20201118/스크린샷 2020-11-18 오후 12.36.04.png"/>

Junit test 실행을 하면 다음과 같은 오류가 발생한다.

대충 요약하면 관련된 의존성 및 Configurations가 없어서 오류가 발생하는 것으로 확인된다.

```
12:33:24.692 [main] INFO  org.springframework.boot.test.context.SpringBootTestContextBootstrapper - Neither @ContextConfiguration nor @ContextHierarchy found for test class [com.nws.zabbix.web.business.mng.aut.mapper.UserRoleConfigMapperTest2], using SpringBootContextLoader
12:33:24.702 [main] INFO  org.springframework.test.context.support.AbstractContextLoader - Could not detect default resource locations for test class [com.nws.zabbix.web.business.mng.aut.mapper.UserRoleConfigMapperTest2]: no resource found for suffixes {-context.xml, Context.groovy}.
12:33:24.703 [main] INFO  org.springframework.test.context.support.AnnotationConfigContextLoaderUtils - Could not detect default configuration classes for test class [com.nws.zabbix.web.business.mng.aut.mapper.UserRoleConfigMapperTest2]: UserRoleConfigMapperTest2 does not declare any static, non-private, non-final, nested classes annotated with @Configuration.
```

오류 원인은 다음과 같다.

아래 패키지를 보면 @SpriingBootApplication이 있는 class가 ZabbixWebApplication 인데,

여기 안에는 @SpringBootApplication이 있다. 해당 어노테이션은 Spring Boot를 실행 시키는 어노테이션인데 이따 2.3에서 알아보도록하자

<img src="./img/20201118/스크린샷 2020-11-18 오후 12.52.19.png"/>

아무튼 우리가 추가한 @SpringBootTest 어노테이션은 @SpringBootApplication을 실행하여 Bean들을 DI하는데, 왜 오류가 발생하는 것일까?

이유는 현재 mapper class가 있는 곳과 @SpringBootApplication이 있는 곳이 다른 패키지 이기 떄문이다.

그렇기 떄문에 현재 @SpringBootTest 추가 후 @SpringBootApplication은 framewokr 패키지만 스캔하고 mapper가 있는 business 패키지는 스캔하지 않아 의존성 주입이 안되었던 것이다.



##### 그럼 ZabbixWebApplication을 com.nws.zabbix.web 아래 둬서 전체 스캔을 해보자!

이번에는 com.nws.zabbix.web아래에 둬서 실행을 해보자.

<img src="./img/20201118/스크린샷 2020-11-18 오후 1.39.46.png"/>

이번에는 정상적으로 Junit이 구동되고 우리가 원하는  테스트 결과를 확인할 수 있게 되었다.

<img src="./img/20201118/스크린샷 2020-11-18 오후 1.42.52.png"/>



## 2.3 @SrpingBootApplication annotation에 대하여 알아보자

@SpringBootApplication의 코드 구조를 보면 다음과 같이 되어 있는 것을 확인할 수 있다.

그럼 아래 Annotation 항목에 대해 알아보도록하자.

- @SpringBootConfiguration
- @ComponentScan
- @EnableAutoConfiguration

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
```

### 2.3.1 @Component 스캔

우리에게 친숙한 Componon스캔의 경우 아래와 같은 일을한다.

- @Component @Configuration @Repository @Service @Controller @RestController
- 해당 어노테이션이 선언된 하위 패키지에서 위와 같이 Annotation을 찾아서 Bean으로 등록한다.



