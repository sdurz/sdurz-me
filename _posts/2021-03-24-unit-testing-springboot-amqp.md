---
layout: single
tags: java development spring springboot amqp unit testing
toc: true
---
Today I checked our CI queue on GitLab and found that a unit test broke the build. 


```java
@TestConfiguration
@EnableAutoConfiguration(exclude = {RabbitAutoConfiguration.class, CacheAutoConfiguration.class})
@ComponentScan(
        basePackages = "com.trenolab.treno4.fn", excludeFilters = {
                @ComponentScan.Filter(
                type = FilterType.ASSIGNABLE_TYPE,
                classes = {
                        FerrovienordApplication.class,
                        Treno4MongoClientConfiguration.class,
```

But still I was getting this error:

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of method rabbitTemplate in com.trenolab.treno4.fn.FerrovienordApplication required a bean of type 'org.springframework.amqp.rabbit.connection.ConnectionFactory' that could not be found.
```

Baeldung says ()[https://www.baeldung.com/spring-boot-testing]

_The @SpringBootTest annotation is useful when we need to bootstrap the entire container. The annotation works by creating the ApplicationContext that will be utilized in our tests._


A @SpringBootTest annotated will still look for a @SpringBootConfiguration or @ContextConfiguration in the classpath, it doesn't spawn this happens before the component scanning phase, therefore I couldn't exlude CustomerApplication with a filter. This is clearly stated in the (docs)[https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html] for @SpringBootTest.
That's why the _excludeFilters_ didn't work as I expected.

## My fault




## The solution

Instead of setting up and mantain a complex @Configuration classes I decided to go the inverse path. 
- I set up a new Spring profile, named "unit"
- I restricted the original @SpringBootConfiguration by excluding "unit" profile.
- I created a test only @SpringBootConfiguration that is loaded from my @SpringBootTest(s)



The solution is quite straightforward: just define a @SpringBootApplication annotated class within the _test_ sources, then I excluded the original application by assigning it with the _integration_ profile

```java
@Profile("integration")
@SpringBootApplication(scanBasePackages = "com.trenolab.treno4")
public class FerrovienordApplication
{
...
```

No need to define complex inclusion/exclusion patterns.




