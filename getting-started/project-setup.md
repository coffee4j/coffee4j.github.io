---
layout: page
title: Project Setup
permalink: /getting-started/project-setup/
---

* [Back to Overview](../)

## Modules

The framework is split into three modules:

* `coffee4j-engine`: This module defines the main execution process for combinatorial test input generation and fault characterization.
This modules calls the different combinatorial test- and fault characterization algorithms at appropriate times during
the execution flow of a combinatorial test to generate new test inputs and determine failure-inducing combinations.
Every event which happens inside the engine is logged in an internal representation format and propagated to any listeners.
Developers of algorithms only have to interact with the engine component to develop custom algorithms, which users can then
use through the model and JUnit5 API.
* `coffee4j-model`: This module defines the custom DSL to represent the different objects with which a user interacts
when writing a combinatorial test. For example, it contains classes representing a `Parameter` or `Value`.
In addition to the general definition, this module is also responsible for converting the external representation into
the general internal representation format used by the engine (and vice versa). It also offers reporting and test
execution interfaces using the external representation format.
* `coffee4j-junit-jupiter`: This module contains extensions that allow an integration into the <a href="https://junit.org/junit5/docs/current/user-guide/#writing-tests">JUnit-Jupiter test engine</a>. Thereby, combinatorial test input generation and fault characterization can be used in conjunction with JUnit's automated test execution.

## Installation

All three modules are available as Maven dependencies in the <a href="https://search.maven.org/search?q=g:de.rwth.swc.coffee4j">Maven Central Repository</a>.
Consequently, just add the specific dependency to your `pom.xml` file.
For most testing projects, adding the `coffee4j-junit-jupiter` should be sufficient. Please also refer to the <a href="https://github.com/coffee4j/coffee4j-example">Example project</a>.

{% highlight xml %}
<dependency>
  <groupId>de.rwth.swc.coffee4j</groupId>
  <artifactId>coffee4j-junit-jupiter</artifactId>
  <version>1.0.0</version>
  <scope>test</scope>
</dependency>
{% endhighlight %}

If you don't use Maven, you can find the imports for Gradle, Ivy, and other build automation tools <a href="https://search.maven.org/search?q=g:de.rwth.swc.coffee4j">here</a>.
