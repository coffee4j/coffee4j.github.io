---
layout: page
title: Getting Started
permalink: /getting-started/
sidebar_link: true
sidebar_sort_order: 2
---

After the required Maven dependencies are present in the test project as described in the <a href="/">Overview</a>,
you can start writing combinatorial tests for JUnit5 using <font style="font-family: 'Abril Fatface', serif;">coffee4j</font>.

## Quick Start

Create a new JUnit 5 test as follows. 
The annotations on `testGreetings` indicate that it is a <i>combinatorial</i> test method, i.e. a test method which is executed several times with different values for its parameters. 
The static method `model` describes the input parameter model and the testing strength. 
Based on that model, test inputs are generated and the `testGreetings` method is executed for each generated test input. 

{% highlight java %}
class GreetingsTest {

  private static InputParameterModel model() {
    return inputParameterModel("example-model")
      .strength(2)
      .parameters(
        parameter("Title").values("Mr", "Mrs"),
        parameter("FirstName").values("John", "Jane"),
        parameter("LastName").values("Doo", "Foo")
    ).build();
  }

  @CombinatorialTest
  @ModelFromMethod("model")
  void testGreetings(String title, String firstName, String lastName) {
    System.out.println("Hello " + title + " " + firstName + " " + lastName);
  }
}
{% endhighlight %}
