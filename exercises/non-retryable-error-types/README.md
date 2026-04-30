# Exercise #6: Non-Retryable Error Types

During this exercise, you will:

- Convert a non-retryable error so it can be handled by a Retry Policy
- Configure a Retry Policy that lists specific exceptions as non-retryable
- Add a Heartbeat and a Heartbeat Timeout to a long-running Activity

Make your changes to the code in the `practice` subdirectory (look for `TODO`
comments that will guide you to where you should make changes to the code).
If you need a hint or want to verify your changes, look at the complete
version in the `solution` subdirectory.

## Setup

This exercise uses the pizza-ordering Workflow. The starter code already
includes a `processCreditCard` Activity that throws a non-retryable
`ApplicationFailure` when the card number is invalid. The `Starter.java` is
preconfigured with a 15-digit (invalid) credit card number so you can observe
failures right away:

```java
// This only has 15 digits
CreditCardInfo cardInfo = new CreditCardInfo("Lisa Anderson", "424242424242424");
```

All commands below assume your terminal is in the
`exercises/non-retryable-error-types/practice` directory and that
`temporal server start-dev` is running in another terminal.

## Part A: Convert the Non-Retryable Error to a Retryable One

In the current code, the `processCreditCard` Activity throws a non-retryable
exception. You've decided that hard-coding the failure as non-retryable is too
opinionated; callers of this Activity should be able to choose for themselves
whether to retry. You'll change the Activity to throw a retryable exception,
and configure non-retryability at the Workflow level instead.

1. Open `PizzaActivitiesImpl.java`.
2. In the `processCreditCard` method, change the thrown exception so it is
   retryable. There are two equivalent ways to do this:
   - Replace `ApplicationFailure.newNonRetryableFailure(...)` with
     `ApplicationFailure.newFailure(...)`, **or**
   - Use `Activity.wrap(new CreditCardProcessingException("Invalid credit card number"))`.
3. Save the file.
4. Compile with `mvn clean compile`.
5. Verify the error now retries:
   1. Start the Worker:
      ```
      mvn exec:java -Dexec.mainClass="pizzaworkflow.PizzaWorker"
      ```
   2. In another terminal, start the Workflow:
      ```
      mvn exec:java -Dexec.mainClass="pizzaworkflow.Starter"
      ```
   3. Open the Web UI and inspect the Workflow. It should be **Running** and
      retrying the Activity.
   4. Terminate the Workflow from the Web UI. It will never succeed with the
      invalid card number.
   5. Stop the Worker with Ctrl-C.

## Part B: Configure a Retry Policy with Non-Retryable Error Types

Now that the Activity itself is retryable, you'll express "don't retry on
credit card errors" at the Workflow level using a Retry Policy.

A Retry Policy has the following attributes:

- **Initial Interval:** time before the first retry
- **Backoff Coefficient:** how much the retry interval increases (default 2.0)
- **Maximum Interval:** the maximum interval between retries
- **Maximum Attempts:** the maximum number of attempts in the presence of failures
- **Non-Retryable Error Types:** a list of exception types that should never retry

1. Open `PizzaWorkflowImpl.java`. A `RetryOptions` is already declared:

   ```java
   RetryOptions retryOptions = RetryOptions.newBuilder()
       .setInitialInterval(Duration.ofSeconds(15))
       .setBackoffCoefficient(2.0)
       .setMaximumInterval(Duration.ofSeconds(60))
       .setMaximumAttempts(25)
       .build();
   ```

2. Add a `.setDoNotRetry(...)` call to the builder so
   `CreditCardProcessingException` is treated as non-retryable. Use
   `CreditCardProcessingException.class.getName()` to obtain the
   fully-qualified class name.
3. Wire the `retryOptions` into `ActivityOptions` by adding
   `.setRetryOptions(retryOptions)` to the `ActivityOptions.newBuilder()`
   chain.
4. Save the file and run `mvn clean compile`.
5. Verify the Workflow now fails fast on credit card errors:
   1. Start the Worker:
      ```
      mvn exec:java -Dexec.mainClass="pizzaworkflow.PizzaWorker"
      ```
   2. Start the Workflow:
      ```
      mvn exec:java -Dexec.mainClass="pizzaworkflow.Starter"
      ```
   3. In the Web UI, you should see an `ActivityTaskFailed` event with the
      message "Invalid credit card number", followed shortly by a
      `WorkflowExecutionFailed` event with the message "Unable to process
      credit card".
   4. Stop the Worker with Ctrl-C.

## Part C: Add Heartbeats

A Heartbeat lets a long-running Activity tell the Worker it's still alive. If
the Worker stops Heartbeating within a configured timeout, the Activity is
considered failed and can be retried on another Worker.

The `notifyDeliveryDriver` Activity simulates trying to reach a delivery
driver. It picks a random number between 0 and 14, then loops 0..9, sleeping
5 seconds per iteration; if the random number matches the loop counter, it
returns success. You'll add a Heartbeat call to each iteration so the Worker
can keep reporting progress.

1. Open `PizzaActivitiesImpl.java`. Locate the `notifyDeliveryDriver` method.
2. Inside the loop, just below the success-condition check, add:
   ```java
   Activity.getExecutionContext().heartbeat("Heartbeat: " + x);
   ```
3. Save the file.
4. Open `PizzaWorkflowImpl.java`. Locate the commented-out block that calls
   `notifyDeliveryDriver` and uncomment it.
5. Delete the unconditional `return confirmation;` statement above the
   commented block. Once the block is uncommented, that earlier return is
   unreachable code.
6. Save the file.
7. Open `Starter.java` and change the credit card number to a valid 16-digit
   value: `"4242424242424242"`. Otherwise the Workflow still fails on
   `processCreditCard` and you never reach the new Heartbeat code.
8. Save the file.

## Part D: Add a Heartbeat Timeout

Adding a Heartbeat call isn't enough. You also need to tell Temporal how
long it should wait between Heartbeats before declaring the Activity failed.

1. Open `PizzaWorkflowImpl.java`.
2. In the `ActivityOptions.newBuilder()` chain, add
   `.setHeartbeatTimeout(Duration.ofSeconds(10))`.
3. Save the file.

## Part E: Run the Workflow

You'll now observe Heartbeats in the Web UI.

1. Compile with `mvn clean compile`.
2. Start the Worker:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.PizzaWorker"
   ```
3. Start the Workflow:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.Starter"
   ```
4. Open the Web UI and find the Running Workflow. Once you see
   `Heartbeat: <number>` lines in the Worker terminal, refresh the Workflow
   page in the Web UI and look for a **Pending Activities** section. You
   should see **Heartbeat Details** with JSON containing the heartbeat payload.

The simulation finishes at a random iteration, so you may need to run the
Workflow a few times to see the Heartbeat details before the Activity
returns.

## (Optional) Part F: Failing a Heartbeat

To see what a Heartbeat timeout looks like, make the Activity sleep longer
than the configured Heartbeat Timeout.

1. Open `PizzaActivitiesImpl.java`.
2. In the `notifyDeliveryDriver` method, change the `Thread.sleep(5000)` call
   to `Thread.sleep(15000)`. That's longer than the 10-second Heartbeat
   Timeout you set in Part D.
3. Stop any running Worker and recompile with `mvn clean compile`.
4. Start the Worker:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.PizzaWorker"
   ```
5. Start the Workflow:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.Starter"
   ```
6. Once you see the first Heartbeat log line in the Worker terminal, wait 15
   seconds and refresh the Web UI. The **Pending Activities** section should
   show a Heartbeat-timeout failure, plus the number of retries remaining and
   the time until the next retry. If the Activity doesn't recover before the
   final attempt, the Workflow fails.

### This is the end of the exercise.
