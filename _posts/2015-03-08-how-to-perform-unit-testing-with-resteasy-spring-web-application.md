---
layout: post
title: How to perform unit testing for resteasy-spring web application
tags: unit-testing spring resteasy
---

Resteasy is a framework that provides capability to build Restful web services and Restful java applications, which can be deployed in any servlet containers. Spring itself is a prevailed well-known IOC framework in the java world. While under the name of spring, there developed a bunch of java frameworks that could facilitate the build of web services, the connection and operation on persistences, or some common purposes like security. Resteasy officially provides the integration to spring and spring mvc frameworks to smooth the usage for spring developers.

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
We will elaborate in next posts.
