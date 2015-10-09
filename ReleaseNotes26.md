# Release Notes for REST Assured 2.6.0 #

# Contents
1. [Highlights](#highlights)
1. [Other Notable Changes](#other-notable-changes)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Minor Changes](#minor-changes)

## Highlights ##
* Fixed misunderstanding of how GPath expressions works with XML and namespaces (see [non-backward compatible changes](#non-backward-compatible-changes))
* It's now possible to use a mapping function when validating headers. For example let's say you want to validate that the Content-Length header is less than 1000. You can then use a mapping function to first convert the header value to an int and then use an "integer" Hamcrest matcher:

  ```java
  when().get("/something").then().header("Content-Length", Integer::parseInt, lessThan(1000));
  ```
  This is also implemented for the MockMvc module (issue 594).
* Added new config called [ParamConfig](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.6.0/com/jayway/restassured/config/ParamConfig.html) that allows you to configure how parameter types should be updated on "collision". By default all parameters are merged so if you do:
  
  ```java
  given().queryParam("param1", "value1").queryParam("param1", "value2").when().get("/x"). ...
  ```
  
  REST Assured will send a query string of `param1=value1&param1=value2`. This is not always what you want though so from now on you can configure REST Assured to *replace* values instead:

  ```java
  given().
          config(config().paramConfig(paramConfig().queryParamsUpdateStrategy(REPLACE))).
          queryParam("param1", "value1").
          queryParam("param1", "value2").
  when().
          get("/x"). ..
  ```

  REST Assured will now replace `param1` with `value2` (since it's written last) instead of merging them together. You can configure the update strategy for each type of for all parameter types:

  ```java
  given().config(config().paramConfig(paramConfig().replaceAllParameters())). ..
  ```
  This is also implemented for the MockMvc module (but the config there is called [MockMvcParamConfig](http://static.javadoc.io/com.jayway.restassured/spring-mock-mvc/2.6.0/com/jayway/restassured/module/mockmvc/config/MockMvcParamConfig.html) (issue 589)

## Other Notable Changes ##



## Non-backward compatible changes ##

* Fixed so that GPath expressions using XML namespaces are evaluated from the root. The implementation was previously a misunderstanding of how the Groovy's XmlSlurper worked when using namespace and has now been corrected. For example let's say have a service at `/namespace-example` that returns the following XML:

  ```xml
  <foo xmlns:ns="http://localhost/">
    <bar>sudo </bar>
    <ns:bar>make me a sandwich!</ns:bar>
  </foo>
  ```

  You NOW test it like this:

  ```java
  given().
          config(newConfig().xmlConfig(xmlConfig().declareNamespace("ns", "http://localhost/"))).
  when().
          get("/namespace-example").
  then().
          body("foo.bar.text()", equalTo("sudo make me a sandwich!")).
          body(":foo.:bar.text()", equalTo("sudo ")).
          body("foo.ns:bar.text()", equalTo("make me a sandwich!"));
  ```

  In the previous versions you did like this:

  ```java
  given().
          config(newConfig().xmlConfig(xmlConfig().declareNamespace("ns", "http://localhost/"))).
  when().
          get("/namespace-example").
  then().
          body("bar.text()", equalTo("sudo make me a sandwich!")).
          body(":bar.text()", equalTo("sudo ")).
          body("ns:bar.text()", equalTo("make me a sandwich!"));
  ```

  Which was not correct (notice the missing foo property)! Big thanks to Erich Eichinger for spotting this and providing a pull request (issue 592).

## Minor changes ##
See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.