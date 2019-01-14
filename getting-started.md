---
layout: page
title: Getting Started
permalink: /getting-started/
sidebar_link: true
sidebar_sort_order: 2
---

After the required Maven dependencies are present in the test project as described in the <a href="/">Overview</a>,
you can start writing combinatorial tests for JUnit5 using <font style="font-family: 'Abril Fatface', serif;">coffee4j</font>.
Since `coffee4j-junit-jupiter` is very similar to `junit-jupiter-params`, reading the corresponding
<a href="https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests">JUnit Documentation</a>
can be helpful.

## Combinatorial Testing

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
  @Generator(IpogTestInputGroupGenerator.class)
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

Currently, <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> only includes IPOG for generating positive
test cases, but there is the <a href="/apidocs/de/rwth/swc/coffee4j/engine/generator/negative/HardNegativeTWiseGenerator.html">`HardNegativeTWiseGenerator`</a>
for generating negative test inputs.

To develop your own custom `TestInputGroupGenerator`, please consult [the corresponding section](#develop-custom-algorithms).

## Fault Characterization

When the execution of a test input for a combinatorial test fails, it is useful for debugging purposes to know which
sub-combinations of the test input caused the failure.
For example consider that the combinatorial test fails for the combination OS=Windows, Browser=Chrome, Ping=10, Speed=10.
It could be the case, that the sub-combination OS=Windows, Browser=Chrome is responsible for the failing test.
We call such sub-combinations <i>(minimal) failure-inducing combinations</i>.
A test input which contains a failure-inducing combination will always fail to execute correctly.

To find the sub-combination, <i>fault characterization algorithm</i> generate additional test inputs with which the
combinatorial test method is called.

### Basic Example

<font style="font-family: 'Abril Fatface', serif;">coffee4j</font> allows the use of one `FaultCharacterizationAlgorithm`
per combinatorial test.
This can look like the example below:

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
  @CharacterizationAlgorithm(ImprovedDeltaDebugging.class)
  @Reporter(PrintStreamExecutionReporter.class)
  @ModelFromMethod("model")
  void testGame(String os, String browser, int ping, int speed) {
    assertFalse(os.equals("Windows") && browser.equals("Chrome"));
  }
  
}

{% endhighlight %}

The example introduces a failure-inducing combination by adding the `assertFalse` statement into the combinatorial test
method.
Additionally, the two annotations `@CharacterizationAlgorithm` and `Reporter` are added.
The first one defines which algorithm <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> should use
for fault characterization similar to the definition of a `TestInputGroupGenerator`.
`@Reporter` adds the defines reporter. 
This is necessary to propagate the failure-inducing combinations to the user, as JUnit offers no capability to report
such things through an IDE view or similar.

If you execute the example from above, you should find the text

```
The following failure inducing combinations where found:
Combination{OS=Windows, Browser=Chrome}
```

somewhere in the console output.
This shows that the `ImprovedDeltaDebugging` algorithm found the correct failure-inducing combinations.

### Different Algorithms

To use a different <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/FaultCharacterizationAlgorithm.html">`FaultCharacterizationAlgorithm`</a>,
simply change the class given in the annotation.

Currently, <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> offers the following fault characterization algorithms:

* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/ben/Ben.html">`Ben`</a>
* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/aifl/Aifl.html">`Aifl`</a>
* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/aifl/IterationBasedIterAifl.html">`IterationBasedIterAifl`</a>
* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/delta/ImprovedDeltaDebugging.html">`ImprovedDeltaDebugging`</a>

## Develop Custom Algorithms

### Initial Generation

### Fault Characterization

## JUnit Integration Features

