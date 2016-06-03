# Release Notes for REST Assured 3.0.0 #

# Contents
1. [Highlights](#highlights)
1. [Other Notable Changes](#other-notable-changes)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Deprecations](#deprecations)
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
* Automatically escapes JsonPath and XmlPath fragments that contains a hyphen and an index lookup operator. For example consider the following JSON document:
  ```javascript
 { "some-list" : ["one", "two"] }
  ```
  Previously you had to escape `some-list` manually if you wanted to get first element out of the list:
  ```javascript
  JsonPath jsonPath = ...
  String firstElement = jsonPath.getString("'some-list'[0]"); // one
  ```
  Now no explicit escaping is necessary:

  ```javascript
  String firstElement = jsonPath.getString("some-list[0]"); // one
  ```
  But this means that if you previously had a JSON document like this:
  ```javascript
  { "some-list[0]" : ["one", "two"] }
  ```
  you would now have to escape it:
  
  ```javascript
  String firstElement = jsonPath.getString("'some-list[0]'[0]"); // one
  ```
  which makes this an (unlikely but still) non-backward compatible change (issue 564).
* Getting an attribute value from an XmlPath expression that doesn't exists now returns null instead of an empty list (issue #650).
* Keystore was previously used as a truststore. You must change `given().keystore(..)` to `given().trustStore(..)`, `RestAssured.keystore(..)` to `RestAssured.trustStore(..)` and `SSLConfig.keystore(..)` to `SSLConfig.trustStore(..)`. Sorry!

## Deprecations
* [SSLConfig#getPassword](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.9.0/com/jayway/restassured/config/SSLConfig.html#getPassword--), use 
[SSLConfig#getKeyStorePassword](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.9.0/com/jayway/restassured/config/SSLConfig.html#getKeyStorePassword--) instead 

## Upgrading

To upgrade from an older version follow these steps:

1. Change Maven/Gradle groupId to `io.rest-assured` (see [getting started guide](https://github.com/rest-assured/rest-assured/wiki/GettingStarted))
1. In your code base search for `com.jayway.restassured` and replace with `io.restassured`
1. If you still have compile-errors see the list of [non-backward compatible changes](#non-backward-compatible-changes) and correct the errors.

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.