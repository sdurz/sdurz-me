---
layout: single
tags: java development spring springboot amqp unit testing
toc: true
draft: true
---
# 1. Introducton

There are a lot of resources on this topic out there. 

These are my notes on unit and integration testing in Spring and Springboot.


## Different type of testing and Spring

When we are unit testing good design, weak coupling and well organized code will matter the most.

Basically we DON'T need any support class or library (other than mocking or assertion libraries) if we are doing proper unit testing, Spring support comes handy only when we need some kind of integration support.

Anyway it's almost impossible to develop anything but the most trivial code examples without some degree on integration.


## Springboot and testing

No matter what kind of testing we want to write the <pre>spring-boot-starter-test</pre> dependendency will add all the needed dependencies to our project (mockito, hamcrest ...):
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```


## 2. JUnit4 and JUnit5

When running Spring based code with JUnit we want to benefit from IoC container features in our unit tests, we achieve this by using runners (JUnit4) or extensions (JUnit5). As of 2021, JUnit4 is considered legacy code. New code is to be tested with JUnit5.

From a pratical perspective we have to replace old _@RunWith_ annotation with the new _@ExtendWith_. For the sake of compatibility _@RunWith_ is still supported in JUnit5. However JUnit4 does not support all the features of the new version, just go with JUnit5.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { SpringTestConfiguration.class })
public class GreetingsSpringTest {
   // ...
```

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { SpringTestConfiguration.class })
public class GreetingsSpringTest {
    // ...
```

The SpringExtension class is provided by Spring and integrates the Spring TestContext Framework into JUnit 5. The @ExtendWith annotation accepts any class that implements the Extension interface. We usually don't need to implement our own extension.

## 3. Integration testing with @SpringBootTest
@SpringBootTest is meta annotated with @ExtendWith(SpringExtension.class) and which means every time your tests are extended with SpringExtension. @MockBean is a Spring test framework annotation and used along with @ExtendWith(SpringExtension.class)
Basically, we want to use _@SpringBootTest_ only in integration testing, when we want to bootstrap the entire container. This means that we can _autowire_ any bean that's available in our configuration.

__Don't try to use @SpringBootTest for unit testing, don't!__


If we don't wanto to bootstrap the entire application we must create our _test specific_ configurations. A test annotated with _@SpringBootTest_ will look for any inner <pre>public static class</pre> marked with 
_@Configuration_ to bootstrap the container:

```java
@SpringBootTest
public class IntegrationTest {

    @Configuration
    public static class TestingConfiguration {
        // ...
    }

```

If we want to reuse the configuration we can define _TestingConfiguration_ as a plain class and _@Import_ it:

```java
@SpringBootTest
@Import(TestingConfiguration.class)
public class ServiceSpringTest {
    
    // ...

```

```java

@Configuration
static class TestingConfiguration() {

    // ...

```

---
There is another annotation: _@TestConfiguration_ that's can be used in testing. As the documentation states:

>   the use of @TestConfiguration does not prevent auto-detection of @SpringBootConfiguration.

so use it carefully. It's not of wide use and it's not just a drop in replacement for _@Configuration_.

### When to use @ExtendWith(SpringExtension.class) and when @SpringBootTest?
When the test requires a Spring Test Context ( to autowire a bean / use of @MockBean ) along with JUnit 5's Jupiter programming model use @ExtendWith(SpringExtension.class). This will support Mockito annotations as well through TestExecutionListeners.

When the test uses Mockito and needs JUnit 5's Jupiter programming model support use @ExtendWith(MockitoExtension.class)


### Prevent @SpringBootApplication from starting when doing integration testing

*:*IMPORTANT!**

Sometimes, when testing a package that contains an executable runner our tests fail because the after bootstrapping the container will try to start our application and out tests will fail with an error like this:

```bash
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of method ...
```

Basically we wan't to prevent the application runner from being detected and started. We can do this by either by excluding the class from component scanning:
```java
@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = CommandLineRunner.class))
@EnableAutoConfiguration
public class TestApplicationConfiguration {
    // ..
}
```

or using an _adhoc_ profile
```java
@ActiveProfiles("testing")
@SpringBootTest
@Import(ApplicationConfiguration.class)
public class IntegrationTest {
    
    @Test
    public void testACondition() {
        // ...
    }
```

and letting the class exclude itself with proper _@Profile_ configuration:
```java
@SpringBootApplication
@Profile("!testing")
@Slf4j
public class ExampleApplication implements CommandLineRunner {

	@Override
	public void run(String ... args) {
        // ...
    }
```

the same applies foe every class you will want to exclude from being picked during testing.

## Spring profiles and testing

Most of the time 


## @JdbcTest

Nice to use, keep in mind that its companion annotation _@Sql_ behaves like _@BeforeEach_.

To avoid errors, use a workaround like this:
```java
@JdbcTest // Enable jdbc, configures an in-memory embedded database and a JdbcTemplate and skip other configuration processing
//@Sql({"/jdbc/schema.sql", "/jdbc/test-data.sql"})
public class ExampleJdbcTests {

	@Autowired
	JdbcTemplate template;
        
    @BeforeAll
    static void setup(@Autowired DataSource datasource) {
        try (Connection conn = datasource.getConnection()) {
            ScriptUtils.executeSqlScript(conn, new ClassPathResource("/jdbc/schema.sql"));
            ScriptUtils.executeSqlScript(conn, new ClassPathResource("/jdbc/test-data.sql"));
        }
    }
```

more on this usefule response on [Stackoverflow](https://stackoverflow.com/questions/47775560/how-can-springs-test-annotation-sql-behave-like-beforeclass)