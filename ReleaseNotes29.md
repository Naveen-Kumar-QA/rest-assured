# Release Notes for REST Assured 2.9.0 #

This is a maintenance release but it contains some backward incompatible changes.

# Contents
1. [Highlights](#highlights)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Deprecations](#deprecations)
1. [Minor Changes](#minor-changes)

## Highlights
* Major improvements of certificate authentication. You can now use a keystore (without trust store) and a keystore and trust store at the same time (see [Non-backward compatible changes](#non-backward-compatible-changes)).

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
[SSLConfig#getPassword](http://static.javadoc.io/com.jayway.restassured/rest-assured/2.9.0/com/jayway/restassured/config/SSLConfig.html#getKeyStorePassword--) instead 

## Minor changes ##
* Added support for composing a Hamcrest matcher with a ResponseAwareMatcher when using a ResponseAwareMatcherComposer
* Added support for multipart DELETE requests (issue #634)
* It's now possible to use empty and whitespace path parameters (issue #631)
* Fixing `NullPointerException` for GET requests with an empty body (issue #642)
* Form authentication for fully-qualified URIs now uses the URI specified in the request instead of just localhost (issue #641)
* Improve escaping for XmlPath's containing colon. For example you can now do like this without manually having to escape anything in the path: `x:something.x:y[0]` (issue #647)

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.