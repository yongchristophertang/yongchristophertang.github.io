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