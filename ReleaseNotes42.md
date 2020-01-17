# Release Notes for REST Assured 4.2.0 #

## Contents
1. [Highlights](#highlights)
1. [Minor Changes](#minor-changes)

## Highlights
* Introduced the `spring-mock-mvc-kotlin-extensions` project which allows a nicer experience for Kotlin developers using the spring-mock-mvc module. This allows one to write tests like this:
	
	```kotlin
	class RestAssuredMockMvcKotlinExtensionsTest {

        @Test
        fun example() {
            val mockMvc =
                MockMvcBuilders.standaloneSetup(GreetingController())
                    .build()

            val id: Int =
            Given {
                mockMvc(mockMvc)
                param("name", "Johan")
            } When {
                get("/greeting")
            } Then {
                body(
                    "id", Matchers.equalTo(1),
                    "content", Matchers.equalTo("Hello, Johan!")
                )
            } Extract {
                path("id")
            }

            assertThat(id).isEqualTo(1)
    }
    ```

    To use it depend on:

    ```xml
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>spring-mock-mvc-kotlin-extensions</artifactId>
        <version>4.2.0</version>
        <scope>test</scope>
    </dependency>
    ```

    and import the extension functions from the `io.restassured.module.mockmvc.kotlin.extensions` package. Thanks to [Myeonghyeon-Lee](https://github.com/mhyeon-lee) for pull request.
* Added a new object mapper type that supports the Jakarta EE JSON Binding (JSON-B) specification. By default it will use Eclipse Yasson as the JSON-B implementation. To use it simply include

	```xml
	<dependency>
	    <groupId>org.eclipse</groupId>
	    <artifactId>yasson</artifactId>
	     <version>${yasson.version}</version>
	</dependency>
	```

	in your classpath and then configure REST Assured to use it as its default ObjectMapperType:

	```java
	RestAssured.config = RestAssured.config.objectMapperConfig(ObjectMapperConfig.objectMapperConfig().defaultObjectMapperType(ObjectMapperType.JSONB));
	```

	Thanks to [Andrew Guibert](https://github.com/aguibert) for pull request.
* Added ability to blacklist headers so that they are not shown in the request or response log. Instead the header value will be replaced with `[ BLACKLISTED ]`. You can enable this per header basis using the [LogConfig](https://www.javadoc.io/doc/io.rest-assured/rest-assured/latest/io/restassured/config/LogConfig.html):

	```java
    given().config(config().logConfig(logConfig().blacklistHeader("Accept"))). ..
    ```

  The response log will the print:

	  ```
	  Request method:	GET
	  Request URI:    http://localhost:8080/something
	  Proxy:          <none>
	  Request params: <none>
	  Query params:   <none>
	  Form params:    <none>
	  Path params:    <none>
	  Headers:        Accept=[ BLACKLISTED ]
	  Cookies:        <none>
	  Multiparts:     <none>
	  Body:           <none>
	  ```

  Thanks to [Simone Ivan Conte](https://github.com/sic2) for the help and initial pull request.

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.