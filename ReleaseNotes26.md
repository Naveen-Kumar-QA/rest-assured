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
* Added a new module called spring-support that adds an alias to the "then" method in `Response` or `MockMvcResponse` called "Then". The reason for this is that `then` might be a reserved keyword in Scala in the future and the compiler gives a warning when using methods with this name. To enable the use of `Then` simply import the `com.jayway.restassured.module.scala.RestAssuredSupport.AddThenToResponse` class from `scala-support` module. For example:

  ```java
  import com.jayway.restassured.RestAssured.when
  import com.jayway.restassured.module.scala.RestAssuredSupport.AddThenToResponse
  import org.hamcrest.Matchers.equalTo
  import org.junit.Test

  @Test
  def `trying out rest assured in scala with implicit conversion`() {
      when().
              get("/greetJSON").
      Then().
              statusCode(200).
              body("key", equalTo("value"))
  }
  ```
  This is also support for the spring-mock-mvc module.

## Other Notable Changes ##

* Improvements to multipart uploading, for example:
  * Added support for setting multipart filename when passing in an object to multiPart method (issue 587)
  * Multipart file-uploading now takes encoder config into account when serializing content. For example if you're trying to serialize an object using mime-type "application/vnd.ms-excel" in a multipart then you can register that it should be serialize as JSON:
    
    ```java
    Greeting greeting = new Greeting();
    greeting.setFirstName("John");
    greeting.setLastName("Doe");

    given().
           config(config().encoderConfig(encoderConfig().encodeContentTypeAs("application/vnd.ms-excel", ContentType.JSON))).
           multiPart(new MultiPartSpecBuilder(greeting)
                   .fileName("RoleBasedAccessFeaturePlan.csv")
                   .controlName("text")
                   .mimeType("application/vnd.ms-excel").build()).
    when().
           post("/multipart/text").
    then().
           statusCode(200);
    ```
    This will now serialize the "greeting" as JSON even though the mime-type is set to "application/vnd.ms-excel" (which is unknown to REST Assured) (issue 586)
  * You can now pass in which ObjectMapperType or ObjectMapper to use when serializing an object using multipart. For example:

    ```json
    Greeting greeting = new Greeting();
    greeting.setFirstName("John");
    greeting.setLastName("Doe");

    given().
           multiPart(new MultiPartSpecBuilder(greeting, ObjectMapperType.GSON)
                     .fileName("RoleBasedAccessFeaturePlan.csv")
                     .controlName("text")
                     .mimeType("application/vnd.ms-excel").build()).
    when().
           post("/multipart/text").
    then().
           statusCode(200);
    ```
    This will force the use if the GSON ObjectMapper (if available in the classpath) even though mime type is not recognized by default by REST Assured.
  * Content-Type for multipart requests is now taken into account. For example you can now do:
    
    ```java
    given().contentType("multipart/mixed").multiPart(..)
    ```
    which was not possible in the previous version. (Only "multipart/form-data" worked) (issue 586)
  * It's now possible to specify default mime subtype for multipart content-type. Use the `MultiPartConfig#defaultSubtype(..)` method. Default is "form-data" which results in a content-type of "multipart/form-data". This also works for the MockMvc module.
* Added ability to specify which encoder charset to use for a specific content-type if no charset is defined explicitly for this content-type. Previously you could only specify a default charset for ALL content-types. You do this by using the `defaultCharsetForContentType` method in the [EncoderConfig](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.6.0/com/jayway/restassured/config/EncoderConfig.html). For example:

  ```java
  RestAssured.config = config(config().encoderConfig(encoderConfig().defaultCharsetForContentType("UTF-16", "application/xml")));
  ```
  This will assume UTF-16 encoding for "application/xml" content-types that does explicitly specify a charset. By default "application/json" is now specified to use "UTF-8" as default content-type as this is specified by RFC4627. This is *may* be a backward incompatible change since previously "application/json" content-types were encoded using the platform default content-type (or what was specified by defaultContentCharset(..)) (issue 567).
* Added ability to specify which decoder charset to use for a specific content-type if no charset is defined explicitly for this content-type.
  Previously you could only specify a default charset for ALL content-types. You do this by using the "defaultCharsetForContentType" method in the [DecoderConfig](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.6.0/com/jayway/restassured/config/DecoderConfig.html). For example:
  
  ```java
  RestAssured.config = config(config().decoderConfig(decoderConfig().defaultCharsetForContentType("UTF-16", "application/xml")));
   ```
  This will assume UTF-16 encoding for "application/xml" content-types that does explicitly specify a charset. By default "application/json" is now specified to use "UTF-8" as default charset as this is specified by RFC4627. This is *may* be a backward incompatible change since previously "application/json" content-types were encoded using the platform default content-type (or what was specified by defaultContentCharset(..)).

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

* When multiple cookies or headers with the same name are returned in the response the LAST value is what's returned when only getting one value from the entity (Headers or Cookies) or when validating values. For example let's say that the server returns headers:
  
  ```  
  HeaderName: Value 1
  HeaderName: Value 2
  ```
  then if you do:
  
  ```java
  get("/x").extract().header("HeaderName")
  ```
  
  Value 2 will be returned (previous Value 1 would be returned). Likewise if you do validation:

  ```java
  get("/x").then().header("HeaderName", equalTo("Value 2");
  ```
  This change also affects session ids. This is done to be compatible with the way browsers work (issue 543).
* The last (instead of first) header, cookie, session etc is returned when calling for example `headers.get("x")` and multiple x headers are defined (see `defaultCharsetForContentType` section under [other notable changes](#other-notable-changes) for more details)

## Minor changes ##
See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.