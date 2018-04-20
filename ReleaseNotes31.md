# Release Notes for REST Assured 3.1.0 #

## Contents
1. [Highlights](#highlights)
1. [Other Notable Changes](#other-notable-changes)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Minor Changes](#minor-changes)

## Highlights
* Now using `java.lang.reflect.Type` instead of `java.lang.Class` in the API for mapping to Java objects. For users of the REST Assured API the change is mostly prominent in the
  [ResponseBodyExtractionOptions](http://static.javadoc.io/io.rest-assured/rest-assured/3.1.0/io/restassured/response/ResponseBodyExtractionOptions.html) interface where the `as` method now takes a `java.lang.reflect.Type` instead of `java.lang.Class`. This should not cause
  any backward incompatibilities unless you've written a custom [ObjectMapperFactory](http://static.javadoc.io/io.rest-assured/rest-assured-common/3.1.0/io/restassured/mapper/factory/ObjectMapperFactory.html). See [non-backward compatible changes](#non-backward-compatible-changes) if this is the case.
* Allow querying (extracting values out of) a request specification without using a filter by using the `io.restassured.specification.SpecificationQuerier`. For example:
 
  ```java
  RequestSpecification spec = ...
  QueryableRequestSpecification queryable = SpecificationQuerier.query(spec);
  String headerValue = queryable.getHeaders().getValue("header");
  String param = queryable.getFormParams().get("someparam");
  ```
* It's now possible to automatically include additional input fields when using form authentication. Just use the `FormAuthConfig` and specify the additional values to include. For example if you have an html page that looks like this:

  ```html
  <html>
   <head>
       <title>Login</title>
   </head>
   <body>
   <form action="/login" method="POST">
       <table>
           <tr>
               <td>User:&nbsp;</td>
               <td><input type="text" name="j_username"></td>
           </tr>
           <tr>
               <td>Password:</td>
               <td><input type="password" name="j_password"></td>
           </tr>
           <tr>
               <td colspan="2"><input name="submit" type="submit"/></td>
           </tr>
       </table>
       <input type="hidden" name="firstInputField" value="value1"/>
       <input type="hidden" name="secondInputField" value="value2"/>
   </form>
   </body>
  </html>
  ```
  and you'd like to include the value of form parameters `firstInputField` and `secondInputField` you can do like this:

  ```java
   given().auth().form("username", "password", formAuthConfig().withAdditionalFields("firstInputField", "secondInputField"). ..
  ```
  
  REST Assured will automatically parse the HTML page, find the values for the additional fields and include them as form parameters in the login request.

## Other Notable Changes ##

* Add better integration for standard HTTP methods with Apache HttpClient which also solves an issue content-type header being generated for empty GET requests
* No longer using `DEF_CONTENT_CHARSET` from Apache HttpClient since it caused compatibility issues
* Removing Authorization header when setting `auth().none()`
* Fixed so that it's possible to specify arguments to root paths in multi expectation blocks such as:
  
  ```java
  get("/jsonStore").then()
    .root("store.book.find { it.author == '%s' }.price")
    .body(
            withArgs("Nigel Rees"), is(8.95f),
            withArgs("Evelyn Waugh"), is(12.99f),
            withArgs("Herman Melville"), is(8.99f),
            withArgs("J. R. R. Tolkien"), is(22.99f)
    );
  ```

## Non-backward compatible changes ##
* If you've written a custom [ObjectMapperFactory](http://static.javadoc.io/io.rest-assured/rest-assured-common/3.1.0/io/restassured/mapper/factory/ObjectMapperFactory.html) you need to change the signature of the `create` method. For example if you previously had a custom `JAXBObjectMapperFactory` that looked like this:

  ```java
  public class MyJAXBObjectMapperFactory implements JAXBObjectMapperFactory {
     public JAXBContext create(Class cls, String charset) {
         ...
     }
  }
  ```

  you now need to change it to:

  ```java
   public class MyJAXBObjectMapperFactory implements JAXBObjectMapperFactory {
        public JAXBContext create(Type cls, String charset) {
            ...
        }
   }
  ```

  Note the change from `java.lang.Class` to `java.lang.reflect.Type`. This was needed for swagger integration.

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.