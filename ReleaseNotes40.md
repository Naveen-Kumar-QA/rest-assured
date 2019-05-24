# Release Notes for REST Assured 4.0.0 #

## Contents
1. [Highlights](#highlights)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Deprecations](#deprecations)
1. [Minor Changes](#minor-changes)

## Highlights
* REST Assured now requires at least Java 8 (previous Java 6 was required)
* Added support for [Apache Johnzon](https://johnzon.apache.org/) (thanks to Andriy Redko for pull request)
* Fixed a lot of issues with the OSGi support to avoid creating duplicate classes in the distributed jar-files. In order the achieve this we had to move certain classes and packages around. This is what warranted the 4.0 release. (big thanks to Milen Dyankov, Steven Huypens and Mark Kolich for the help!) 
* Updated Groovy from 2.4.x to 2.5.x (enabled by the move to Java 8)
* Upgraded Hamcrest from version 1.3 to 2.1 (enabled by the move to Java 8)
* Made it possible to specify a multi-expectation body with arguments. For example:
  ```java
  when().
          get("/jsonStore").
  then().
          rootPath("store.book.find { it.author == '%s' }").
          body(
                  "price", withArgs("Nigel Rees"), is(8.95f),
                  "price", withArgs("Evelyn Waugh"), is(12.99f)
          );
  ```
* Several upgrades to the Spring MockMvc module (big thanks to Gemini Kim for a lot of help)

## Non-backward compatible changes ##
* REST Assured now requires Java 8 (previously only Java 6 was required).
* Breaking changes introduced when solving [#1117](https://github.com/rest-assured/rest-assured/issues/1117):
    * io.restassured.mapper.TypeRef has been moved to io.restassured.common.mapper.TypeRef
    * io.restassured.mapper.DataToDeserialize has been moved to io.restassured.common.mapper.DataToDeserialize
    * io.restassured.mapper.ObjectDeserializationContext has been moved to io.restassured.common.mapper.ObjectDeserializationContext
    * io.restassured.mapper.factory.GsonObjectMapperFactory has been moved to io.restassured.path.json.mapper.factory.GsonObjectMapperFactory
    * io.restassured.mapper.factory.Jackson1ObjectMapperFactory has been moved to io.restassured.path.json.mapper.factory.Jackson1ObjectMapperFactory
    * io.restassured.mapper.factory.Jackson2ObjectMapperFactory has been moved to io.restassured.path.json.mapper.factory.Jackson2ObjectMapperFactory
    * io.restassured.mapper.factory.DefaultGsonObjectMapperFactory has been moved to io.restassured.path.json.mapper.factory.DefaultGsonObjectMapperFactory
    * io.restassured.mapper.factory.DefaultJackson1ObjectMapperFactory has been moved to io.restassured.path.json.mapper.factory.DefaultJackson1ObjectMapperFactory
    * io.restassured.mapper.factory.DefaultJackson2ObjectMapperFactory has been moved to io.restassured.path.json.mapper.factory.DefaultJackson2ObjectMapperFactory
    * io.restassured.mapper.resolver.ObjectMapperResolver has been moved to io.restassured.common.mapper.resolver.ObjectMapperResolver
    * io.restassured.exception.PathException has been moved to io.restassured.common.exception.PathException
* Removed deprecated methods:
    * io.restassured.RestAssured
        * withArguments
        * withNoArguments
    * io.restassured.builder.ResponseSpecBuilder
        * expectContent
    * io.restassured.builder.RequestSpecBuilder
        * setContent
        * addParameters
        * addParameter
        * addQueryParameters
        * addQueryParameter
        * addFormParameters
        * addFormParameter
        * addPathParameter
        * addPathParameters
        * setAuthentication
    * io.restassured.specification.ResponseSpecification
        * content
        * specification
    * io.restassured.specification.RequestSpecification
        * content
        * formParameters
        * formParameter
        * pathParameter
        * pathParameters
        * authentication
        * specification
        * parameters
        * parameter
        * queryParameter
        * queryParameters

## Deprecations
* Deprecated all short versions of "root", for example `root(..)`, `appendRoot(..)`, `detachRoot(..)`. Use `rootPath(..)`, `appendRootPath(..)`, `detachRootPath(..)` instead. This was introduced for better consistency and clearer intention.
* `JsonPath#setRoot(..)` - Use `JsonPath#setRootPath(..)` instead
* `XmlPath#setRoot(..)` - Use `XmlPath#setRootPath(..)` instead

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.