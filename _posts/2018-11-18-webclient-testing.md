---
title: "Writing tests for Spring WebClient with MockWebServer."
tags:
  - java
  - testing
---

Spring WebFlux includes a reactive, non-blocking [WebClient](https://docs.spring.io/spring/docs/5.1.x/spring-framework-reference/web-reactive.html#webflux-client) for HTTP requests. 
It supersedes the AsyncRestTemplate, which was deprecated in Spring 5.

Writing tests for the AsyncRestTemplate was quite easy with the [MockRestServiceServer](https://docs.spring.io/spring-framework/docs/5.1.x/javadoc-api/org/springframework/test/web/client/MockRestServiceServer.html). 
MockRestServiceServer is provided by the spring-test module and allows testing your code without a running web server.
By using this mock it is possible to send mock responses and assert the request's body or headers.

Unfortunately the new WebClient is not supported by the MockRestServiceServer. 
A [Spring jira ticket](https://jira.spring.io/browse/SPR-15286) exists to add this functionality.
Initially the Spring maintainers said this would be added, however recent comments suggest it is doubtful that support will be added.

As alternative the use of [MockWebServer](https://github.com/square/okhttp#mockwebserver) is suggested. 
MockWebServer is a library for testing HTTP clients by the creators of OkHttp.
Contrary to the MockRestServiceServer the MockWebServer does start a web server and requires a port to bind on.

So let's call an imaginary web api with the WebClient and write a test for it: 

{% gist codingtim/c5b07a679d146db47aa825f184945c8e %}

The code above performs a PUT request with a body and a header and it expects a response.
The request and the response both contain two fields: a string field and a number field.

Now we use the MockWebServer to write a test for our ApiCaller: 

{% gist codingtim/a2e32f7c27fc1ffd576a1fb8369645cd %}

Even though the MockWebServer needs to start a web server it does seem like a viable way to test code using the WebClient.
Besides setting up simple responses the MockWebServer offers advanced testing capabilities such as chuncked, delayed or throttled responses.

The complete maven project with the necessary dependencies can be found [here](https://github.com/codingtim/webclienttest).
