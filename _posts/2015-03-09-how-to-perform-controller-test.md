---
layout: post
title: How to perform unit testing for resteasy-spring web application (2) - controller test
tags: unit-testing spring resteasy
---

In previous part of this post, we have discussed how to do a unit test for a 'normal' class. Now let's take a step forward to see how a controller test would be. The controller test will much more complicated than the 'normal' peers.

There are two unit testing mock frameworks can be used to test a controller in resteasy-spring web applications. One is resteasy's mocking dispatcher framework, and the other is spring's mockMvc framework. We will discuss both of them in this post.

First let's see the tested controller class.

## Tested controller class

```java
@Path("/categories")
@Produces("application/json;charset=utf-8")
@Component("categoryApi")
public class DefaultCategoryApi implements CategoryApi {

    @Autowired
    private CategoryBusiness categoryBusiness;

    @Path("/grades")
    @GET
    @Override
    public List<Grade> getGradeList() {
        return list = categoryBusiness.getGradeList();
    }
}
```
This controller is used to process GET request to "/category/grades", and render a list of grades to client which is returned by business service layer.


## Unit testing with Resteasy Mock Dispatcher

Resteasy provides users a mocking framework to perform unit testing. It is very straight-forward, and flexible to use. 

```java
@RunWith(MockitoJUnitRunner.class)
public class CategoryApiDispatcherTest {
    @InjectMocks
    private static CategoryApi categoryApi = new DefaultCategoryApi();

    @Mock
    private CategoryBusiness categoryBusiness;

    private static Dispatcher dispatcher = MockDispatcherFactory.createDispatcher();

    @BeforeClass
    public static void setUpDispatcher() {
        dispatcher.getRegistry().addSingletonResource(categoryApi);
        dispatcher.getProviderFactory().register(ArgumentExceptionHandler.class);
    }

    @Before
    public void resetMockObject() {
        reset(categoryBusiness);
    }

    @Test
    public void testGetGradeList_GradeListIsFound_ShouldReturnFoundList() throws URISyntaxException {
        when(categoryBusiness.getGradeList()).thenReturn(new Grade() { {
            setName("Grade One");
            setCode("01");
        } }, new Grade() { {
            setCode("02");
            setName("Grade Two");
        } });

        MockHttpRequest mockHttpRequest = MockHttpRequest.get("/categories/grades");
        MockHttpResponse mockHttpResponse = new MockHttpResponse();

        dispatcher.invoke(mockHttpRequest, mockHttpResponse);
        System.out.println(mockHttpResponse.getContentAsString());
        JsonAssert.with(mockHttpResponse.getContentAsString()).assertThat("$..code", is(Arrays.asList("01", "02")))
                .assertThat("$..name", is(Arrays.asList("Grade One", "Grade Two")));
    }
}
```
A dispatcher is mocked through APIs provided by the test framework, which is injected with a controller, categoryApi, by addSingletonResource API and a exception handler, ArgumentExceptionHandler, by provider register API. Then we mock categoryBusiness, letting it return a grade list when its getGradeList() method is called. Finally we use dispatcher to invoke a mocked http request and assert the http response.

## Unit test with Spring MVC test framework

In spring framework, instantiation of all objects are handled by Spring container. Therefore, it is not possible in this case to manually instantiate the controller and mock the dependency as we do in the resteasy testing framework. Now we have some configuration steps to go before we start up our spring container.

### Spring configurations

We config spring application context via XML files and will demonstrate in this post. But it should be remembered that all steps can be replaced by Java annotation based configuration programatically.

First we prepare our test XML context in which we create a mock dependency, CategoryBusiness, and let the spring container inject it to the controller class automatically. The following is the XML config:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="categoryBusiness" name="categoryBusiness" class="org.mockito.Mockito" factory-method="mock">
        <constructor-arg value="net.example.CategoryBusiness"/>
    </bean>
</beans>
```
Then this is our unit testing class:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:testContext.xml", "classpath:/META-INF/spring/applicationContext.xml",
        "classpath:springmvc-resteasy.xml"})
@WebAppConfiguration
public class CategoryApiControllerTest {
    private MockMvc mockMvc;

    @Autowired
    private CategoryBusiness categoryBusiness;

    @Autowired
    private WebApplicationContext webApplicationContext;

    @Before
    public void setUp() {
        Mockito.reset(categoryBusiness);
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }

    @Test
    public void testGetGradeList_GradeListIsFound_ShouldReturnFoundGradeList() throws Exception {
        when(categoryBusiness.getGradeList()).thenReturn(Arrays.asList(new Grade() { {
            setCode("01");
            setName("First Grade");
            setAlias("one");
            setSort(1);
        } }, new Grade() { {
            setCode("03");
            setName("Third Grade");
            setAlias("tri");
            setSort(2);
        } }));

        mockMvc.perform(get("/categories/grades")).andExpect(status().isOk()).andExpect(
                content().contentType("application/json;charset=utf-8"))
                .andExpect(jsonPath("$..code", is(Arrays.asList("01", "03"))))
                .andExpect(jsonPath("$..name", is(Arrays.asList("First Grade", "Third Grade")))).andDo(print());
    }
}
```
The ContextConfiguration annotation can be used to set application context config XML files. In this example, we need 
  * testContext.xml: our own test context configs.
  * applicationContext.xml: the web application context configs.
  * springmvc-resteasy.xml: the resteasy-springmvc plugin configs which is used to config the resteasy to springmvc mappings.

It should be emphsized that the testContext.xml shall be placed in front of applicationContext.xml, because all these two configs can create the bean 'categoryBusiness' while the former one overrides the latter one.

The MockMvc is the entry class for spring Mock MVC framework which provides plenty of handy libs, such as assertions, prints, and json parsers. Also all providers including exception mappers and filters are already injected into the controller, all of which are handled by the spring transparently.

## Summary
We have now discussed the most common approaches to realize unit testing for a resteasy-spring web application. However in real world, situation might be much more complicated than what we talked in the post, which still need a lot of study and learnings to achieve best testing performance.

All example code snippets are cited from our real source codes, which are too complex to put on Github. I will tailor them and upload to Github ASAP.