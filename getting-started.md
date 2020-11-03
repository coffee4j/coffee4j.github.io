---
layout: page
title: Getting Started
permalink: /getting-started/
sidebar_link: true
sidebar_sort_order: 2
---

## [HelloWorldTest.java](https://github.com/coffee4j/coffee4j-template/blob/master/src/test/java/de/rwth/swc/coffee4j/example/simple/HelloWorldTest.java)

`HelloWorldTest` constitutes a simple combinatorial test. 
The static method `model` defines the complete input parameter model including the desired testing strength and all parameters with their values.
The `test` method is written like a normal JUnit5 parameterized test with the difference only being the annotations. `@CombinatorialTest` marks the method as a combinatorial test method and provides a reference to the input parameter model. 
 
This means that JUnit will automatically call coffee4j to generate the inputs for the method. T
Once the test execution is started, coffee4j uses the IPOG test selection strategy as defined by `@EnableGeneration(algorithms = { Ipog.class })` to generate test inputs. 
It will then call the `test` method for each generated test input.

{% highlight java %}
class HelloWorldTest {
    
    private static final Logger LOG = LoggerFactory.getLogger(HelloWorldTest.class);

    @SuppressWarnings("unused")
    private static InputParameterModel model() {
        return inputParameterModel("hello-world-model")
                .positiveTestingStrength(1)
                .parameters(
                        parameter("Title").values("Mr", "Mrs"),
                        parameter("FirstName").values("John", "Jane"),
                        parameter("LastName").values("Doo", "Foo")
                ).build();
    }

    @CombinatorialTest(inputParameterModel = "model")
    @EnableGeneration(algorithms = { Ipog.class })
    void test(
            @InputParameter("Title") String title,
            @InputParameter("FirstName") String firstName,
            @InputParameter("LastName") String lastName) {
        LOG.info("Hello {} {} {}", title, firstName, lastName);
    }
}
{% endhighlight %}



## [ExampleTest.java](https://github.com/coffee4j/coffee4j-template/blob/master/src/test/java/de/rwth/swc/coffee4j/example/simple/ExampleTest.java)

The `ExampleTest` further introduces constraints in the input parameter model.
In this context, a constraint is a Java method with one or more parameters as its arguments and a return value of type `boolean`. 
Constraints add semantic information to the input parameter model by labeling combinations of parameter values with either `true` or `false`.

{% highlight java %}
...
                .parameters(
                        parameter("Packages").values(1, 3, null),
                        ...
                ).errorConstraints(
                        constrain("Packages")
                                .withName("invalid-packages")
                                .by((Integer packages) -> packages != null),
                        ...
                ).build();
...
{% endhighlight %}

The unique selling point of <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> is the distinction of two different types of constraints: 
exclusion-constraints via `.exclusionConstraints()` and error-constraints via `.errorConstraints()`.
Whenever an exclusion-constraint returns `false` for a value combination, this value combination is marked as **irrelevant** and consequently excluded from all test inputs.
Irrelevant value combinations never appear in any test input.
Whenever an error-constraint returns `false` for a value combination, this value combination is marked as **invalid**.
Invalid value combinations never appear in *valid* test inputs but in *strong invalid* test inputs.

To enable the generation of strong invalid test inputs, the IPOG-NEG test selection strategy can be enabled via `@EnableGeneration`.
Further on, test selection strategies can be further customized via `@ConfigureIpog` and `@ConfigureIpogNeg` annotations.

{% highlight java %}
...
    @EnableGeneration(algorithms = { Ipog.class, IpogNeg.class })
    @ConfigureIpogNeg(constraintCheckerFactory = MinimalForbiddenTuplesCheckerFactory.class)
...
{% endhighlight %}

For more information about the distinction of exclusion- and error-constraints, please refer to our <a href="../about#publications">publications</a>.
More examples that illustrate additional features such as customization of IPOG-NEG, 
automatic detection and repair of over-constrained input parameter models, 
automatic generation of error-constraints, and
test oracles based on error-constraints can be found in the `example.advanced` package ([here](https://github.com/coffee4j/coffee4j-template/tree/master/src/test/java/de/rwth/swc/coffee4j/example/advanced)).