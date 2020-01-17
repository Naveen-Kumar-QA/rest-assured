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
	```xml

	in your classpath and then configure REST Assured to use it as its default ObjectMapperType:

	```java
	RestAssured.config = RestAssured.config.objectMapperConfig(ObjectMapperConfig.objectMapperConfig().defaultObjectMapperType(ObjectMapperType.JSONB));
	```

	Thanks to [Andrew Guibert](https://github.com/aguibert) for pull request.

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.