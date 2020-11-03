---
layout: index
title: Overview
sidebar_link: true
sidebar_sort_order: 1
---

<div style="width: 66%; margin-left: auto; margin-right: auto; text-align: center; margin-bottom: 70px;">
  <h1 style="font-family: 'Abril Fatface', serif; font-size: 3.25rem;">
    coffee4j
  </h1>
  <p style="font-size: 1.25rem; font-weight: 300;">
    Combinatorial Test and Fault Characterization Framework
  </p>
</div>

<div>
<font style="font-family: 'Abril Fatface', serif;">coffee4j</font> is a Java-based framework for combinatorial test input generation and fault characterization.
It is based on a fast implementation of the well-known IPOG algorithm and supports constraint handling, robustness testing and fault localization.  
A JUnit 5 extension seamlessly integrates combinatorial test input generation, fault characterization  and automated test execution.
</div>


## Quick Start

To quickly start, use the following Maven dependency in your project.

{% highlight xml %}
<dependency>
  <groupId>de.rwth.swc.coffee4j</groupId>
  <artifactId>coffee4j-junit-engine</artifactId>
  <version>1.0.7</version>
</dependency>
{% endhighlight %}

Create a new JUnit 5 test as follows.
The annotations on `testGreetings` indicate that it is a <i>combinatorial</i> test method, i.e. a test method which is executed several times with different values for its parameters.
The static method `model` describes the input parameter model and the testing strength.
Based on that model, test inputs are generated and the `testGreetings` method is executed once for each generated test input.

{% highlight java %}
class GreetingsTest {

    private static InputParameterModel model() {
        return inputParameterModel("example-model")
                .positiveTestingStrength(1)
                .parameters(
                        parameter("Title").values("Mr", "Mrs"),
                        parameter("FirstName").values("John", "Jane"),
                        parameter("LastName").values("Doo", "Foo")
                ).build();
    }

    @CombinatorialTest(inputParameterModel = "model")
    @EnableGeneration(algorithms = { Ipog.class })
    void testGreetings(
            @InputParameter("Title") String title,
            @InputParameter("FirstName") String firstName,
            @InputParameter("LastName") String lastName) {
        System.out.println("Hello " + title + " " + firstName + " " + lastName);
    }
}
{% endhighlight %}

For more information, please refer to the <a href="getting-started/">Getting Started</a> section and the <a href="https://github.com/coffee4j/coffee4j-template">Project Template</a>.
