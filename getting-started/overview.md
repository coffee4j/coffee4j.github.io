---
layout: page
title: Project Setup
permalink: /getting-started/project-setup/
---

* [Back to Overview](../getting-started/)

## Modules

Coffee4j is split into three modules, each fulfilling a different task for the whole framework. The modules are:

* `coffee4j-engine`: Defines the main execution process for combinatorial tests (with fault characterization).
This modules calls the different combinatorial test- and fault characterization algorithms at appropriate times during
the execution flow of a combinatorial test to generate new test inputs and determine failure-inducing combinations.
Every event which happens inside the engine is logged in an internal representation format and propagated to any listeners.
Algorithms developers only have to interact with the engine component to develop custom algorithms, which users can then
use through the model and JUnit5 API.
* `coffee4j-model`: This module mainly defines a custom DSL to represent the different objects with which a user interacts
when writing a combinatorial test. For example, it contains classes representing a `Parameter` or `Value`.
In addition to the general definition, this module is also responsible for converting the external representation into
the general internal representation format used by the engine (and vice versa). To this end, it also offers reporting and test
execution interfaces using the external representation format.
* `coffee4j-junit-jupiter`: As <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> is not a complete
testing framework in the sense that it does not contain a test discovery mechanism, it contains this module to deeply
integrate it into the <a href="https://junit.org/junit5/docs/current/user-guide/#writing-tests">JUnit-Jupiter test engine</a>.
Most user will use this API to interact with <font style="font-family: 'Abril Fatface', serif;">coffee4j</font>, and therefore
most of this site focuses on it.

## Installation

To ease the installation process, all three <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> modules
mentioned above are available as Maven dependencies in the <a href="https://search.maven.org/">Maven Central Repository</a>.
Consequently, just add the specific dependency as described below to your `pom.xml` file:

{% highlight xml %}
<dependency>
  <groupId>de.rwth.swc.coffee4j</groupId>
  <artifactId>coffee4j-engine</artifactId>
  <version>1.0.0</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>de.rwth.swc.coffee4j</groupId>
  <artifactId>coffee4j-model</artifactId>
  <version>1.0.0</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>de.rwth.swc.coffee4j</groupId>
  <artifactId>coffee4j-junit-jupiter</artifactId>
  <version>1.0.0</version>
  <scope>test</scope>
</dependency>
{% endhighlight %}

Just adding the `coffee4j-junit-jupiter` dependency should be the way to go for most testing projects.

If you don't use maven, you can find the imports for Gradle, Ivy, and other dependency managements frameworks
at <a href="https://mvnrepository.com/artifact/de.rwth.swc.coffee4j">this address</a>

## Using <font style="font-family: 'Abril Fatface', serif;">coffee4j</font>

To learn about the basics of the <font style="font-family: 'Abril Fatface', serif;">coffee4j</font> combinatorial testing
framework, please consult the <a href="getting-started">Getting Started</a> guide, the
<a href="https://github.com/coffee4j/coffee4j-example">Example project</a>,
or <font style="font-family: 'Abril Fatface', serif;">coffee4j</font>'s <a href="javadoc">Documentation</a>.

After the required Maven dependencies are present in the test project as described in the <a href="/">Overview</a>,
you can start writing combinatorial tests for JUnit5 using <font style="font-family: 'Abril Fatface', serif;">coffee4j</font>.
Since `coffee4j-junit-jupiter` is very similar to `junit-jupiter-params`, reading the corresponding
<a href="https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests">JUnit Documentation</a>
can be helpful.
