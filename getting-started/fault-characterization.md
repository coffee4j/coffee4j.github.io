---
layout: page
title: Fault Characterization
permalink: /getting-started/fault-characterization/
---

* [Back to Overview](../)

When the execution of a combinatorial test fails for a certain test input, it is useful for debugging purposes to know which
combination is actually responsible for the failure.
For example, consider that the test input `combination OS=Windows, Browser=Chrome, Ping=10, Speed=10` causes the test to fail.
the sub-combination `OS=Windows, Browser=Chrome` could responsible for the failing test.

A sub-combination is called a <i>failure-inducing combinations</i> if the test fails for each test input that contains the sub-combination.
To find the sub-combination, <i>fault characterization algorithms</i> generate additional test inputs which are automatically executed in order to narrow down the set of potential failure-inducing combinations.

### Basic Example

Coffee4j allows the use of one `FaultCharacterizationAlgorithm` per combinatorial test.
This can look like the example below:

{% highlight java %}
class BasicExample {

  private static InputParameterModel model() {
    return inputParameterModel("game-model")
        .strength(2)
        .parameters(
            parameter("OS").values("Windows", "Linux", "MacOS", "Android", "iOS"),
            parameter("Browser").values("Chrome", "Edge", "Firefox", "Safari"),
            parameter("Ping").values(10, 100, 1000),
            parameter("Speed").values(1, 10, 100, 1000)
        ).build();
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
The first one defines which algorithm should be used
for fault characterization similar to the definition of a `TestInputGroupGenerator`.
`@Reporter` defines a reporting component.
This is necessary to propagate the failure-inducing combinations to the user, as JUnit offers no capability to report
such things through an IDE view or similar.

If you execute the example from above, you should find the text

```
The following failure inducing combinations where found:
Combination{OS=Windows, Browser=Chrome}
```

in the console output.
This shows that the `ImprovedDeltaDebugging` algorithm found the correct failure-inducing combinations.

### Different Algorithms

To use a different <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/FaultCharacterizationAlgorithm.html">`FaultCharacterizationAlgorithm`</a>,
simply change the class given in the annotation.

Currently, coffee4j offers the following fault characterization algorithms:

* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/ben/Ben.html">`Ben`</a>
* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/aifl/Aifl.html">`Aifl`</a>
* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/aifl/IterationBasedIterAifl.html">`IterationBasedIterAifl`</a>
* <a href="/apidocs/de/rwth/swc/coffee4j/engine/characterization/delta/ImprovedDeltaDebugging.html">`ImprovedDeltaDebugging`</a>
