# Release Notes for REST Assured 3.0.0 #

This is a maintenance release but it contains some backward incompatible changes.

# Contents
1. [Highlights](#highlights)
1. [Other Notable Changes](#other-notable-changes)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Deprecations](#deprecations)
1. [Removed Deprecations](#removed-deprecations)
1. [Upgrading](#upgrading)
1. [Minor Changes](#minor-changes)

## Highlights
* REST Assured has a new group id, `io.rest-assured`. Previously you depended on REST Assured like this (Maven):
  
  ```xml
  <dependency>
    <groupId>com.jayway.restassured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>${rest-assured.version}</version>
  </dependency>

  ```

  but now you do:

  ```xml
  <dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>${rest-assured.version}</version>
  </dependency>
  ```
  See [getting started guide](https://github.com/rest-assured/rest-assured/wiki/GettingStarted) for more info.

* Package structure has been renamed from `com.jayway.restassured` to `io.restassured`. Search and replace should cover most scenarios, see [non-backward compatible changes](#non-backward-compatible-changes) for more info.
* All long method variants have been deprecated (for example `formParameters`, `specification`) as well as `content`. The reason for this is to avoid duplicated API methods in the future (and thus reducing the size of the API). See [deprecations](#deprecations) for more info.
* All HTTP verbs now support data in the body (for example TRACE, OPTIONS etc)
* All HTTP verbs now support multipart file uploading (even GET, OPTIONS etc)
* You can now use custom http methods/verbs with REST Assured by making using the the `request` method in the DSL (or by statically importing a `io.restassured.RestAssured.request`). For example:

  ```java
  when().request("CONNECT", "/somewhere").then().statusCode(200);
  ```

  It you can also supply a predefined http method (defined in the `io.restassured.http.Method` enum):

  ```java
  when().request(Method.GET, "/lotto").then().statusCode(200);
  ```
  
  This API has also been implemented for the MockMvc module (but MockMvc doesn't support arbitrary http methods as of now).

## Other Notable Changes ##

* It's now possible to map to java objects when extracting from a list in JsonPath. For example if we have the following JSON document:

  ```javascript
  {
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            }
  }
  ```

  it's now possible to map all "books" into a Java object:


  ```java
  List<Book> books = JsonPath.from(json).getList("store.books", Book.class);
  ```
* Improved error messages when trying to verify a path with a parent that doesn't exist. For example if we have the following JSON document:

  ```javascript
  { "myThing" : { "name" : "ThingName" } }
  ```
  
  and we try to test it like so:

  ```java
  when().get("/thing").then().body("myThing1.name", equalTo("ThingName")); // Notice myThing1 is invalid
  ```

  we now get an AssertionError like this:

  ```java
  1 expectation failed.
  JSON path store.unknown.unknown.get(0) doesn't match.
  Expected: (a collection containing "none")
  Actual: null
  ```
  
  whereas previously you would get an IllegalArgumentException with an error message like this:
  
  ```java
  Cannot get property 'name' on null object
  ```
* Multipart uploads now take the content-type boundary into account. For example you can specify:
  
  ```java
  given().contentType("multipart/mixed; boundary=abcdef").multiPart(..). ..
  ```

  which will use the specified boundary of "abcdef" instead of generating a "random" one. It's also possible to specify a default boundary in the MultiPartConfig:

  ```java
  given().config(config().multiPartConfig(multiPartConfig().defaultBoundary("abcdef"))). ..
  ```
* Fixed so that it's possible to declare whether or not XmlPath and Rest Assured should use care about XML namespaces, validation and/or allow doc type declaration.  To configure this when using XmlPath do:

  ```java
  XmlPath xmlPath = new XmlPath(xml).using(xmlPathConfig().namespaceAware(false)); // replace "namespaceAware" with "validation" or "allowDocTypeDeclaration" if needed
  ```

  And like this if using REST Assured DSL:

  ```java
  given().config(RestAssured.config().xmlConfig(xmlConfig().namespaceAware(false))). ..
  ```
* Added ability to use ResponseAwareMatcher for headers. For example you can now use attributes from the response body to validate a Location header. Let's say that "/redirect" returns the json document `{ "id" : 1 }` and returns a redirect to a location ending with this id. If you want to validate the Location header invariant you can do:
  
  ```java  
  given().
          redirects().follow(false).
  when().
          get("/redirect").
  then().
          statusCode(301).
          header("Location", response -> endsWith("/redirect/"+response.path("id")));
  ```
  This has also been implemented for the MockMvc module (issue 692).
* Support for setting session attributes in the Spring MockMvc module using the "sessionAttr" and "sessionAttrs" methods (thanks to sneyyar for pull request) (issue 671)

## Non-backward compatible changes ##

There are a lot of non-backward compatible changes in this release (see [upgrading](#upgrading) for upgrade instructions).

* Package structure has been renamed from `com.jayway.restassured` to `io.restassured`.
* The following methods 

## Deprecations
* `io.restassured.builder.RequestSpecBuilder`:
    - io.restassured.builder.RequestSpecBuilder.setContent(byte[]) (use io.restassured.builder.RequestSpecBuilder.setBody(byte[]) instead)
    - io.restassured.builder.RequestSpecBuilder.setContent(java.lang.Object) (use io.restassured.builder.RequestSpecBuilder.setBody(Object) instead)
    - io.restassured.builder.RequestSpecBuilder.setContent(java.lang.Object, io.restassured.mapper.ObjectMapper) (use io.restassured.builder.RequestSpecBuilder.setBody(java.lang.Object, io.restassured.mapper.ObjectMapper) instead)
    - io.restassured.builder.RequestSpecBuilder.setContent(java.lang.Object, io.restassured.mapper.ObjectMapperType) (use io.restassured.builder.RequestSpecBuilder.setBody(java.lang.Object, io.restassured.mapper.ObjectMapperType) instead)
    - io.restassured.builder.RequestSpecBuilder.setContent(java.lang.Object, io.restassured.mapper.ObjectMapperType) (use io.restassured.builder.RequestSpecBuilder.setBody(java.lang.Object, io.restassured.mapper.ObjectMapperType) instead)
    - io.restassured.builder.RequestSpecBuilder.setContent(java.lang.String) (use io.restassured.builder.RequestSpecBuilder.setBody(java.lang.String) instead)
    - io.restassured.builder.RequestSpecBuilder.addParameter(java.lang.String, java.util.Collection<?>) (use io.restassured.builder.RequestSpecBuilder.addParam(java.lang.String, java.util.Collection<?>) instead)
    - io.restassured.builder.RequestSpecBuilder.addParameter(java.lang.String, java.lang.Object...) (use io.restassured.builder.RequestSpecBuilder.addParam(java.lang.String, java.lang.Object...) instead)
    - io.restassured.builder.RequestSpecBuilder.addParameters (use io.restassured.builder.RequestSpecBuilder.addParams instead)
    - io.restassured.builder.RequestSpecBuilder.addFormParameter(java.lang.String, java.util.Collection<?>) (use io.restassured.builder.RequestSpecBuilder.addParams instead)
    - io.restassured.builder.RequestSpecBuilder.addFormParameter(java.lang.String, java.lang.Object...) (use io.restassured.builder.RequestSpecBuilder.addFormParam(java.lang.String, java.lang.Object...) instead)
    - io.restassured.builder.RequestSpecBuilder.addFormParameters (use io.restassured.builder.RequestSpecBuilder.addFormParams instead)
    - io.restassured.builder.RequestSpecBuilder.addPathParameter(java.lang.String, java.util.Collection<?>) (use io.restassured.builder.RequestSpecBuilder.addParams instead)
    - io.restassured.builder.RequestSpecBuilder.addPathParameters(java.lang.String, java.lang.Object, java.lang.Object...) (use io.restassured.builder.RequestSpecBuilder.addPathParams(java.lang.String, java.lang.Object, java.lang.Object...) instead)
    - io.restassured.builder.RequestSpecBuilder.addPathParameters (use (use io.restassured.builder.RequestSpecBuilder.addPathParams instead)
    - io.restassured.builder.RequestSpecBuilder.addQueryParameter(java.lang.String, java.util.Collection<?>) (use io.restassured.builder.RequestSpecBuilder.addParams instead)
    - io.restassured.builder.RequestSpecBuilder.addQueryParameter(java.lang.String, java.lang.Object...) (use io.restassured.builder.RequestSpecBuilder.addQueryParam(java.lang.String, java.lang.Object...) instead)
    - io.restassured.builder.RequestSpecBuilder.addQueryParameters (use io.restassured.builder.RequestSpecBuilder.addQueryParams instead)
    - io.restassured.builder.RequestSpecBuilder.setAuthentication (use io.restassured.builder.RequestSpecBuilder.setAuth instead)
* `io.restassured.builder.ResponseSpecBuilder`:
    - io.restassured.builder.ResponseSpecBuilder.expectContent(org.hamcrest.Matcher<?>) (Use io.restassured.builder.ResponseSpecBuilder.expectBody(org.hamcrest.Matcher<?>) instead)
    - io.restassured.builder.ResponseSpecBuilder.expectContent(java.lang.String, org.hamcrest.Matcher<?>) (Use io.restassured.builder.ResponseSpecBuilder.expectBody(java.lang.String, org.hamcrest.Matcher<?>) instead)
    - io.restassured.builder.ResponseSpecBuilder.expectContent(java.lang.String, java.util.List<io.restassured.specification.Argument>, org.hamcrest.Matcher<?>) (Use io.restassured.builder.ResponseSpecBuilder.expectBody(java.lang.String, java.util.List<io.restassured.specification.Argument>, org.hamcrest.Matcher<?>) instead)
* `io.restassured.specification.RequestSpecification`:
    - io.restassured.specification.RequestSpecification.content(byte[]) (Use io.restassured.specification.RequestSpecification.body(byte[]) instead)
    - io.restassured.specification.RequestSpecification.content(java.io.File) (Use io.restassured.specification.RequestSpecification.body(java.io.File) instead)
    - io.restassured.specification.RequestSpecification.content(java.io.InputStream) (Use io.restassured.specification.RequestSpecification.body(java.io.InputStream) instead)
    - io.restassured.specification.RequestSpecification.content(java.lang.Object) (Use io.restassured.specification.RequestSpecification.body(java.lang.Object) instead)
    - io.restassured.specification.RequestSpecification.content(java.lang.Object, io.restassured.mapper.ObjectMapper) (Use io.restassured.specification.RequestSpecification.body(java.lang.Object, io.restassured.mapper.ObjectMapper) instead)
    - io.restassured.specification.RequestSpecification.content(java.lang.Object, io.restassured.mapper.ObjectMapperType) (Use io.restassured.specification.RequestSpecification.body(java.lang.Object, io.restassured.mapper.ObjectMapperType) instead)
    - io.restassured.specification.RequestSpecification.content(java.lang.String) (Use io.restassured.specification.RequestSpecification.body(java.lang.String) instead)
    - io.restassured.specification.RequestSpecification.authentication (Use io.restassured.specification.RequestSpecification.auth instead)
    - io.restassured.specification.RequestSpecification.parameter(java.lang.String, java.util.Collection<?>) (Use io.restassured.specification.RequestSpecification.param(java.lang.String, java.util.Collection<?>) instead)
    - io.restassured.specification.RequestSpecification.parameter(java.lang.String, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.param(java.lang.String, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.parameters(java.util.Map<java.lang.String,?>) (Use io.restassured.specification.RequestSpecification.params(java.util.Map<java.lang.String,?>) instead)
    - io.restassured.specification.RequestSpecification.parameters(java.lang.String, java.lang.Object, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.params(java.lang.String, java.lang.Object, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.formParameter(java.lang.String, java.util.Collection<?>) (Use io.restassured.specification.RequestSpecification.formParam(java.lang.String, java.util.Collection<?>) instead)
    - io.restassured.specification.RequestSpecification.formParameter(java.lang.String, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.formParam(java.lang.String, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.formParameters(java.util.Map<java.lang.String,?>) (Use io.restassured.specification.RequestSpecification.formParams(java.util.Map<java.lang.String,?>) instead)
    - io.restassured.specification.RequestSpecification.formParameters(java.lang.String, java.lang.Object, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.formParams(java.lang.String, java.lang.Object, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.queryParameter(java.lang.String, java.util.Collection<?>) (Use io.restassured.specification.RequestSpecification.queryParam(java.lang.String, java.util.Collection<?>) instead)
    - io.restassured.specification.RequestSpecification.queryParameter(java.lang.String, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.queryParam(java.lang.String, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.queryParameters(java.util.Map<java.lang.String,?>) (Use io.restassured.specification.RequestSpecification.queryParams(java.util.Map<java.lang.String,?>) instead)
    - io.restassured.specification.RequestSpecification.queryParameters(java.lang.String, java.lang.Object, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.queryParams(java.lang.String, java.lang.Object, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.pathParameter(java.lang.String, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.pathParam(java.lang.String, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.pathParameters(java.util.Map<java.lang.String,?>) (Use io.restassured.specification.RequestSpecification.pathParams(java.util.Map<java.lang.String,?>) instead)
    - io.restassured.specification.RequestSpecification.pathParameters(java.lang.String, java.lang.Object, java.lang.Object...) (Use io.restassured.specification.RequestSpecification.pathParams(java.lang.String, java.lang.Object, java.lang.Object...) instead)
    - io.restassured.specification.RequestSpecification.specification (Use io.restassured.specification.RequestSpecification.spec instead)
* `io.restassured.specification.ResponseSpecification`:
    - io.restassured.specification.ResponseSpecification#rootPath(java.lang.String) (Use io.restassured.specification.ResponseSpecification.root(java.lang.String) instead)
    - io.restassured.specification.ResponseSpecification.rootPath(java.lang.String, java.util.List<io.restassured.specification.Argument>) (Use io.restassured.specification.ResponseSpecification.rootPath(java.lang.String, java.util.List<io.restassured.specification.Argument>) instead)
    - io.restassured.specification.ResponseSpecification.noRootPath (Use io.restassured.specification.ResponseSpecification.noRoot instead)
    - io.restassured.specification.ResponseSpecification.content(java.util.List<io.restassured.specification.Argument>, org.hamcrest.Matcher, java.lang.Object...) (Use io.restassured.specification.ResponseSpecification.body(java.util.List<io.restassured.specification.Argument>, org.hamcrest.Matcher, java.lang.Object...) instead)
    - io.restassured.specification.ResponseSpecification.content(org.hamcrest.Matcher<?>, org.hamcrest.Matcher<?>...) (Use io.restassured.specification.ResponseSpecification.body(org.hamcrest.Matcher<?>, org.hamcrest.Matcher<?>...) instead)
    - io.restassured.specification.ResponseSpecification.content(java.lang.String, java.util.List<io.restassured.specification.Argument>, org.hamcrest.Matcher, java.lang.Object...) (Use io.restassured.specification.ResponseSpecification.body(java.lang.String, java.util.List<io.restassured.specification.Argument>, org.hamcrest.Matcher, java.lang.Object...) instead)
    - io.restassured.specification.ResponseSpecification.content(java.lang.String, org.hamcrest.Matcher<?>, java.lang.Object...) (Use io.restassured.specification.ResponseSpecification.body(java.lang.String, org.hamcrest.Matcher<?>, java.lang.Object...) instead)
    - io.restassured.specification.ResponseSpecification.specification (Use io.restassured.specification.ResponseSpecification.spec instead)
* `io.restassured.RestAssured`:
    - io.restassured.RestAssured.withArguments (Use io.restassured.RestAssured.withArgs instead)
    - io.restassured.RestAssured.withNoArguments (Use io.restassured.RestAssured.withNoArgs instead)

## Removed Deprecations


* `io.restassured.specification.AuthenticationSpecification.certificate(java.lang.String, java.lang.String, java.lang.String, int)` 
  
  **Replace with:**  `io.restassured.specification.AuthenticationSpecification.certificate(java.lang.String, java.lang.String, io.restassured.authentication.CertificateAuthSettings)`
* `io.restassured.RestAssured.requestContentType(io.restassured.http.ContentType` 
  
  **Replace with:** `io.restassured.builder.RequestSpecBuilder` and set the content-type and apply it to `io.restassured.RestAssured.requestSpecification`
* `io.restassured.RestAssured.responseContentType(java.lang.String)` 
  
  **Replace with:** `io.restassured.builder.ResponseSpecBuilder.expectContentType(io.restassured.http.ContentType)` and apply it to `io.restassured.RestAssured.responseSpecification`
* `io.restassured.config.EncoderConfig.appendDefaultContentCharsetToStreamingContentTypeIfUndefined(java.lang.boolean)`
  
  **Replace with:** `io.restassured.config.EncoderConfig.appendDefaultContentCharsetToContentTypeIfUndefined(boolean)`                       
* `io.restassured.specification.FilterableRequestSpecification.getRequestContentType()`
  
  **Replace with:** `io.restassured.specification.FilterableRequestSpecification.getContentType()`
* `io.restassured.RestAssured.requestContentType()` 
  
  **If you really need to know this then create a filter**
* `io.restassured.RestAssured.responseContentType()`
  
  **If you need to know this then extract it from the response**
* `io.restassured.RestAssured.certificate(java.lang.String, java.lang.String, java.lang.String, int)` 
  
  **Replace with:** `io.restassured.RestAssured.certificate(java.lang.String, java.lang.String, io.restassured.authentication.CertificateAuthSettings)`
* `io.restassured.filter.FilterContext.getRequestMethod()` 
  
  **Replace with:** `io.restassured.specification.FilterableRequestSpecification.getMethod()`
* `io.restassured.filter.FilterContext.getRequestPath()` 
  
  **Replace with:** `io.restassured.specification.FilterableRequestSpecification.getDerivedPath()`
* `io.restassured.filter.FilterContext.getOriginalRequestPath()`
  
  **Replace with:** `io.restassured.specification.FilterableRequestSpecification.getUserDefinedPath()`
* `io.restassured.filter.FilterContext.getRequestURI()`
  
  **Replace with:** `io.restassured.specification.FilterableRequestSpecification.getURI()`
* `io.restassured.filter.FilterContext.getCompleteRequestPath()` 
  
  **Replace with:** `io.restassured.specification.FilterableRequestSpecification.getURI()`
* `io.restassured.filter.log.LogDetail.PATH` 
  
  **Replace with:** `io.restassured.filter.log.LogDetail.URI`
* `io.restassured.module.mockmvc.specification.MockMvcRequestSpecification.resultHandlers)`
  
  **Replace with:** `io.restassured.module.mockmvc.response.ValidatableMockMvcResponse.apply(..)`
* `io.restassured.mapper.ObjectMapper.JACKSON`
  
  **Isn't needed anymore**
* `io.restassured.mapper.ObjectMapper.GSON`
  
  **Isn't needed anymore**
* `io.restassured.mapper.ObjectMapper.JAXB`
  
  **Isn't needed anymore**
* `io.restassured.config.SSLConfig.getPassword()`
  
  **Replace with:** `io.restassured.config.SSLConfig.getKeyStorePassword()`

## Upgrading

To upgrade from an older version follow these steps:

1. Change Maven/Gradle groupId to `io.rest-assured` (see [getting started guide](https://github.com/rest-assured/rest-assured/wiki/GettingStarted))
1. In your code base search for `com.jayway.restassured` and replace with `io.restassured`
1. If you still have compile-errors see the list of [non-backward compatible changes](#non-backward-compatible-changes) and correct the errors.

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.