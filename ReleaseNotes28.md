# Release Notes for REST Assured 2.8.0 #

# Contents
1. [Highlights](#highlights)
1. [Other Notable Changes](#other-notable-changes)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Deprecations](#deprecations)
1. [Minor Changes](#minor-changes)

## Highlights ##
* Added support for measuring response time. For example:

  ```java
  long timeInMs = get("/lotto").time()
  ```
  
  or using a specific time unit:

  ```java
  long timeInSeconds = get("/lotto").timeIn(SECONDS);
  ```

  where `SECONDS` is just a standard `TimeUnit`. You can also validate it using the validation DSL:

  ```java
  when().
          get("/lotto").
  then().
          time(lessThan(2000L)); // Milliseconds
  ```
  
  or

  ```java
  when().
          get("/lotto").
  then().
          time(lessThan(2L), SECONDS);
  ```


* Changes to `com.jayway.restassured.filter.FilterContext`, see [Non-backward compatible changes](#non-backward-compatible-changes).
* Automatically adds supports for spring rest docs path parameter documentation if `spring-restdocs-mockmvc` is in classpath. This can be disabled using the [MockMvcConfig](http://static.javadoc.io/com.jayway.restassured/spring-mock-mvc/2.7.0/com/jayway/restassured/module/mockmvc/config/MockMvcConfig.html) (issue 606).

## Other Notable Changes ##
* Response content-type validation now works correctly even if the response body is empty
* Taking [DecoderConfig](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.7.0/com/jayway/restassured/config/DecoderConfig.html) into account when parsing non-string content (issue 599)
* It's now possible to supply [MockMvcConfigurers](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/setup/MockMvcConfigurer.html) when calling [standaloneSetup](http://static.javadoc.io/com.jayway.restassured/spring-mock-mvc/2.7.0/com/jayway/restassured/module/mockmvc/specification/MockMvcRequestSpecification.html#standaloneSetup-java.lang.Object...-) in the [Spring Mock MVC module](https://github.com/jayway/rest-assured/wiki/Usage#spring-mock-mvc-module). For example:
  
  ```java
  given().standaloneSetup(new Controller1(), springSecurity()). ..
  ```
* It's now possible to change port, base path etc from a filter (issue 600)
* Added support for specifying preemptive basic authentication for proxies. For example:
  
  ```
  given().proxy(auth("username", "password")).when() ..
  ```
  where `auth` is statically imported from [com.jayway.restassured.specification.ProxySpecification](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.7.0/com/jayway/restassured/specification/ProxySpecification.html) (issue 597).

## Non-backward compatible changes ##
* Changes to [com.jayway.restassured.filter.FilterContext](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.7.0/com/jayway/restassured/filter/FilterContext.html). `getRequestPath` now only returns the actual path of the request URI (previously this returned the request URI). Now `getRequestURI` returns the entire request URI (i.e. returns the same thing that `getRequestPath` previously did).

## Deprecations
* Deprecated the [resultHandlers](http://static.javadoc.io/com.jayway.restassured/spring-mock-mvc/2.7.0/com/jayway/restassured/module/mockmvc/specification/MockMvcRequestSpecification.html#resultHandlers-org.springframework.test.web.servlet.ResultHandler-org.springframework.test.web.servlet.ResultHandler...-) method in [MockMvcRequestSpecification](http://static.javadoc.io/com.jayway.restassured/spring-mock-mvc/2.7.0/index.html?com/jayway/restassured/module/mockmvc/RestAssuredMockMvc.html) and added `apply` method to [ValidatableMockMvcResponse](http://static.javadoc.io/com.jayway.restassured/spring-mock-mvc/2.6.0/index.html?com/jayway/restassured/module/mockmvc/RestAssuredMockMvc.html) which should be used instead. For example previously you did 

  ```java
  given().
          resultHandlers(print()).
  when().
          get("/x").
  then(). ...
  ``` 
  but now you do 
  
  ```java
  when().
         get("/x").
  then().
         apply(print())
  ``` 
  (issue 607)
*  `getCompleteRequestPath` in [FilterContext](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.7.0/com/jayway/restassured/filter/FilterContext.html) has been deprecated, use `getRequestURI` instead. 

## Minor changes ##
See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.