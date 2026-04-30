# Replay '26 Beginner to Builder Workshop (Java)

This repository contains the code and READMEs for the hands-on exercises in
the Java track of the Replay '26 Beginner to Builder Workshop.

It's important to remember that the example code in this workshop was
designed to support learning specific aspects of Temporal, not to serve as
a ready-to-use template for implementing a production system.

## Prerequisites

To run these exercises locally, you'll need:

1. A clone of this repository
2. A working installation of Java (JDK 11 or higher) and Maven 3.6 or higher
3. A working installation of the `temporal` command-line tool, also known as
   the Temporal CLI. If you haven't installed this, follow the instructions
   in the [Install the Temporal CLI](https://docs.temporal.io/cli#install)
   section of the Temporal documentation.

Before beginning the first exercise, run `temporal server start-dev` in a
terminal to start a local Temporal Service. Leave it running for the
duration of the workshop, since it's required for the subsequent exercises
too.

## Hands-On Exercises

| Directory Name                        | Exercise                                                              |
| :------------------------------------ | :-------------------------------------------------------------------- |
| `exercises/farewell-workflow`         | [Exercise 1](exercises/farewell-workflow/README.md)                   |
| `exercises/durable-execution`         | [Exercise 2](exercises/durable-execution/README.md)                   |
| `exercises/sending-signals-client`    | [Exercise 3](exercises/sending-signals-client/README.md)              |
| `exercises/sending-signals-external`  | [Exercise 4](exercises/sending-signals-external/README.md) (Optional) |
| `exercises/querying-workflows`        | [Exercise 5](exercises/querying-workflows/README.md)                  |
| `exercises/non-retryable-error-types` | [Exercise 6](exercises/non-retryable-error-types/README.md)           |
| `exercises/rollback-with-saga`        | [Exercise 7](exercises/rollback-with-saga/README.md) (Optional)       |

Each exercise directory contains two subdirectories:

1. `practice` is where you'll modify the code as instructed in that
   exercise's README.
2. `solution` is the completed version, for comparison.

Compile the code in either subdirectory by running `mvn clean compile` from
that subdirectory. Each exercise's README lists the specific
`mvn exec:java` commands needed to run that exercise's Worker, Starter, and
any supporting clients.

## Reference

The following links provide additional information that you may find
helpful as you work through the hands-on exercises.

* [General Temporal Documentation](https://docs.temporal.io/)
* [Temporal Java SDK Documentation](https://www.javadoc.io/doc/io.temporal/temporal-sdk/latest/index.html)
* [Temporal Java Developer Guide](https://docs.temporal.io/develop/java/)
* [Java Language Documentation](https://docs.oracle.com/en/java/)
