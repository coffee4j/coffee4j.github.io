---
layout: page
title: Fault Characterization
permalink: /getting-started/fault-characterization
---

* [Back to Overview](../getting-started/)

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
