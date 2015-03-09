---
layout: post
title: How to perform unit testing for resteasy-spring web application (1)
tags: unit-testing spring resteasy
---

Resteasy is a framework that provides capability to build Restful web services and Restful java applications, which can be deployed in any servlet containers. Spring itself is a prevailed well-known IOC framework in the java world. Resteasy officially provides the integration to spring and spring mvc frameworks to smooth the usage for spring developers.

In a simple way, resteasy can be considered a mvc web framework which provides us restful web services. Therefore, resteasy can be mapped to another MVC framework, say spring mvc, in order to leverage the power of the spring world. We will not talk about a lot about resteasy or spring detailed in this post, please refer to their official websites for more information: [resteasy](http://resteasy.jboss.org/) and [spring](http://spring.io).

Both spring and resteasy frameworks evolve very rapidly. The latest versions for spring and resteasy when this post's created are 4.1.5 and 3.0.9 respectively. However, in my real work, we use a previous version due to legacy issues. But please be noted that, the concepts in this post can be applied to any versions of the framework and so do most of skills. Should you encounter any problems with a different version, please check the version's change log first.

OK, let's focus on our topic.

## Dependencies for performing unit test

We use Maven to build our project and manage all dependencies, the dependencies we need are:

```xml
<!-- unit test dependencies begin -->
<dependency>
 <groupId>junit</groupId>
 <artifactId>junit</artifactId>
 <version>4.11</version>
 <scope>test</scope>
 <exclusions>
  <exclusion>
   <artifactId>hamcrest-core</artifactId>
   <groupId>org.hamcrest</groupId>
  </exclusion>
 </exclusions>
</dependency>

<dependency>
 <groupId>org.springframework</groupId>
 <artifactId>spring-test</artifactId>
 <version>3.2.2.RELEASE</version>
 <scope>test</scope>
</dependency>

<dependency>
 <groupId>org.mockito</groupId>
 <artifactId>mockito-core</artifactId>
 <version>1.10.19</version>
 <scope>test</scope>
</dependency>

<dependency>
 <groupId>org.hamcrest</groupId>
 <artifactId>hamcrest-all</artifactId>
 <version>1.3</version>
 <scope>test</scope>
</dependency>

<dependency>
 <groupId>com.jayway.jsonpath</groupId>
 <artifactId>json-path</artifactId>
 <version>0.8.1/version>
 <scope>test</scope>
</dependency>

<dependency>
 <groupId>com.jayway.jsonpath</groupId>
 <artifactId>json-path-assert</artifactId>
 <version>0.8.1</version>
 <scope>test</scope>
</dependency>
<!-- unit test dependencies end -->
```
## Unit test for a 'normal' class
From a MVC view, a web application should be composed by model, view and controller parts. In this post we regard a non-controller class as a normal class, because unit testing it requires no boot-up of spring, thus with less complexity and configurations.

Here is an example filter class which examines the "token" parameter in the header. If the token is blank, the filter method throws an IllegalArgumentException, else the token is against validation by the injected Authorizable instance:

```java
@Provider
@Component
public class AccessTokenFilter implements ContainerRequestFilter {
    public static final String TOKEN_NAME = "token";

    @Autowired
    private Authorizable authorizer;

    @Override
    public void filter(ContainerRequestContext requestContext) throws IOException {

        String token = requestContext.getHeaderString(TOKEN_NAME);

        Validate.isTrue(StringUtils.isNotBlank(token), "token cannot be blank");

        authorizer.authorize(token);
    }
}
```
In order to cover all circumstances, we should prepare two test cases, one is with non-blank token while another is blank. Here we should be noticed that the validation of token is not necessary in this unit test, which actually shall be covered in testing of Authorizable class.

In unit testing, it is well known that we do not need to run the tested class in a reality environment. Therefore, the technique of mocking is introduced to focus our testing on the tested class solely. In this example, we have one dependency, authorizer, and one input instance, requestContext that both need to be mocked. The following is an examplary test case with non-blank token:

```java
@RunWith(MockitoJUnitRunner.class)
public class AccessTokenFilterTest {
	@InjectMocks
	private AccessTokenFilter filter;

	@Mock
	private Authorizable authorizer;

	@Mock
    private ContainerRequestContext crc;

    @Before
    private void setUp() {
    	reset(crc);
    }

    @Test
    private void testFilter_TokenIsNonBlank_ShouldProceedTokenToAuthorizer() {
    	when(crc.getHeaderString(AccessTokenFilter.TOKEN_NAME)).thenReturn("valid");

    	filter.filter(crc);

    	verify(authorizer).authorize("valid");
    }
}
```
In next part, we will talk about how to unit test a controller class.