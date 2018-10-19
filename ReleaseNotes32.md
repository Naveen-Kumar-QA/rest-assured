# Release Notes for REST Assured 3.2.0 #

## Contents
1. [Highlights](#highlights)
1. [Non-backward compatible changes](#non-backward-compatible-changes)
1. [Minor Changes](#minor-changes)

## Highlights
* Java 11 support
* New [Spring Web Test Client](Usage#spring-web-test-client-module) module that allows testing components of the [Spring Reactive Web](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) stack using the `spring-web-test-client` module. This means that you can unit test reactive Spring (Webflux) Controllers (thanks to [Olga Maciaszek-Sharma](https://github.com/OlgaMaciaszek) for pull request).
* OSGi support (thanks to [Steven Huypens](https://github.com/ponziani) for pull request)

## Non-backward compatible changes ##
* REST Assured now requires Java 6 (previously only Java 5 was required).
* The Scala support module has been updated to use Scala 2.12 which means that you need to use Java 8 in order to use this module

## Minor changes ##

See [change log](http://github.com/jayway/rest-assured/raw/master/changelog.txt) for more details.