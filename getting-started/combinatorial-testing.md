---
layout: page
title: Combinatorial Testing
permalink: /getting-started/combinatorial-testing/
---

* [Back to Overview](../getting-started/)

<font style="font-family: 'Abril Fatface', serif;">coffee4j</font> models combinatorial tests via `Parameter` and `Value`
classes. Consider the following example parameters and values for testing an online browser game:

OS      | Browser | Ping   | Speed    
--------|---------|--------|----------
Windows | Chrome  | 10ms   | 1 KB/s   
Linux   | Edge    | 100ms  | 10 KB/s  
MacOS   | Firefox | 1000ms | 100 KB/s
Android | Safari  |        | 1000 KB/s
iOS     |         |        |          

These parameter and values form an <i>input parameter model</i> (IPM), which is used as the running example
in all following demonstrations.

### Basic Example

A basic JUnit5 test for the above model could look as follows.

{% highlight java %}
class BasicExample {

  private static InputParameterModel.Builder model() {
    return inputParameterModel("game-model")
        .strength(2)
        .parameters(
            parameter("OS").values("Windows", "Linux", "MacOS", "Android", "iOS"),
            parameter("Browser").values("Chrome", "Edge", "Firefox", "Safari"),
            parameter("Ping").values(10, 100, 1000),
            parameter("Speed").values(1, 10, 100, 1000));
  }

  @CombinatorialTest
  @ModelFromMethod("model")
  void testGame(String os, String browser, int ping, int speed) {
    System.out.println("OS=" + os + ", Browser=" + browser + ", Ping=" + ping + ", Speed=" + speed);
  }
}

{% endhighlight %}

The first method defines the complete `InputParameterModel` with the desired testing strength and parameters with their values.
The `testGame` method is written like a normal JUnit5 <i>parameterized test</i> with the difference only being the
annotations.
`@CombinatorialTest` marks the method as a combinatorial test method.
This means that JUnit will automatically call <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> to
generate the inputs for the method.
Next, it will call the `testGame` method with each generated test input.
Since <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> cannot possible know which method is used to
define the input parameter model, the second annotation (`@ModelFromMethod("model")`) is needed to define it.
The annotation allows for the same level of flexibility as the JUnit <a href="https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-sources-MethodSource">MethodSource</a>.

Once the test execution is started, <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> uses the IPOG
combination strategy to generate test inputs covering all parameters in the given strength.
It will then call the `testGame` method with each generate input.

### Testing Strength

To achieve a higher testing strength, simply change the value in the `model` method form the example above.
The combinatorial test generation algorithm will automatically use the new value once the test is started again.
At the moment, variable testing strengths for different parameter groups are not supported.

### Constraints

Most models require the definition of some constraints as value combinations which can not appear together under
any circumstance.
For example, impossible combinations should never be the input for any test execution.
In our running example, the combination `OS=Windows, Browser=Safari, Ping=10, Speed=10` is currently possible,
even though no implementation of the Safari Browser on the Windows operating system exists.
To ensure that no test input contains this combination, it is possible to add constraint forbidding it to the model
definition from above.
The new model would then look like this:

{% highlight java %}

private static InputParameterModel.Builder model() {
  return inputParameterModel("game-model")
      .strength(2)
      .parameters(
          parameter("OS").values("Windows", "Linux", "MacOS", "Android", "iOS"),
          parameter("Browser").values("Chrome", "Edge", "Firefox", "Safari"),
          parameter("Ping").values(10, 100, 1000),
          parameter("Speed").values(1, 10, 100, 1000))
      .forbiddenConstraint(constrain("OS", "Browser")
          .by((String os, String browser) -> !(os.equals("Windows") && browser.equals("Safari"))));
}

{% endhighlight %}

The lambda given to `by` can be an arbitrary function.
Thus, the constraint model of <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> offers maximum flexibility.

The <a href="/apidocs/de/rwth/swc/coffee4j/model/constraints/ConstraintBuilder.html">`ConstraintBuilder`</a>
offers `constrain` functions for up to six parameter.
But, since `forbiddenConstraint` accepts and arbitrary <a href="/apidocs/de/rwth/swc/coffee4j/model/constraints/Constraint.html">`Constraint`</a>
, constraints involving more parameters are possible (but not advisable).

In addition to forbidden constraints, our framework also includes the concept of error constraints to denote combinations
of values which can appear in a test, but should result in some form of error by the system under test.
This is used by our negative testing algorithms.
To add an error constraint, simply use `errorConstraint` instead of `forbiddenConstraint`.

### Different Algorithms

As mentioned in the last section, <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> includes more than
just an IPOG algorithm.
Since it should be possible to switch between those different algorithms, the framework includes the `@Generator` annotation
as demonstrated below.

{% highlight java %}
class BasicExample {

  @CombinatorialTest
  @ModelFromMethod("model")
  @Generator(Ipog.class)
  void testGame(String os, String browser, int ping, int speed) {
    System.out.println("OS=" + os + ", Browser=" + browser + ", Ping=" + ping + ", Speed=" + speed);
  }
}

{% endhighlight %}

This explicitly tells the framework to use the IPOG generator.
As IPOG is used in most cases, this algorithm is the default and does not explicitly need to be registered.
In the case that another algorithm should be used, simple include the annotation and specify the algorithm `class`
(which needs to implement the <a href="/apidocs/de/rwth/swc/coffee4j/engine/generator/TestInputGroupGenerator.html">TestInputGroupGenerator</a> interface).

It is also possible to add multiple generation algorithms to one method by explicitly setting the `value` attribute
of the annotation:

{% highlight java %}

@Generator(value = {IpogTestInputGroupGenerator.class, HardNegativeTWiseGenerator.class})

{% endhighlight %}
