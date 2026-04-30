# Exercise #5: Querying Workflows

During this exercise, you will:

- Define and handle a Query
- Call the Query from a Temporal Client
- Send a Query from the command line
- Query a closed Workflow

Make your changes to the code in the `practice` subdirectory (look for
`TODO` comments that will guide you to where you should make changes to
the code). If you need a hint or want to verify your changes, look at the
complete version in the `solution` subdirectory.

This is the same `PizzaWorkflow` as the previous exercise, but extended with a
Query handler so you can ask the Workflow what state it's in while it runs.
The Workflow still waits for a Signal before billing the customer, which gives
you a window to issue Queries against a running Workflow.

## Part A: Defining a Query

1. Open `PizzaWorkflow.java` in the
   `practice/src/main/java/queryingworkflows/orderpizza` subdirectory.
   1. Locate the `TODO: PART A` comment.
   2. Define an `orderStatus()` method on the interface. It should return
      `String`, take no parameters, and be annotated with `@QueryMethod`.
   3. Save the file.
2. Open `PizzaWorkflowImpl.java` in the same subdirectory.
   1. Implement the `orderStatus` method. It should return the value of the
      `status` instance variable.
   2. Set the value of `status` at three points in the Workflow:
      1. `"Started"` as the first line of the `orderPizza` method.
      2. `"Out for delivery"` immediately above the `Workflow.await(...)` call.
      3. `"Order complete"` as the last line of the `try` block that contains
         the `sendBill()` Activity invocation.
   3. **Note:** these states are illustrative; you can pick whichever points in
      the Workflow you want to report on. Feel free to experiment with more
      states.
   4. Save the file.

## Part B: Performing a Query from a Client

1. Open `QueryClient.java` in the
   `practice/src/main/java/queryingworkflows` subdirectory.
2. Locate the `TODO: PART B` comment. On the line below it, call
   `workflow.orderStatus()` and print the returned `String` to standard out.
3. Save the file.

## Part C: Running the Workflow and the Query

All commands below must be run from the `practice` subdirectory. Make sure
`temporal server start-dev` is running in another terminal.

1. Compile the code by running `mvn clean compile`.
2. In one terminal, start the Worker by running:
   ```
   mvn exec:java -Dexec.mainClass="queryingworkflows.QueryingWorkflowsWorker"
   ```
3. In another terminal, start the Workflow by running:
   ```
   mvn exec:java -Dexec.mainClass="queryingworkflows.Starter"
   ```
   The Workflow will start and then block at the `Workflow.await(...)` call,
   waiting for the Signal.
4. In a third terminal, send the Query by running:
   ```
   mvn exec:java -Dexec.mainClass="queryingworkflows.QueryClient"
   ```
   You should see the current state printed:
   ```
   Out for delivery
   ```

## Part D: Sending a Query from the Command Line

You can also send a Query from the Temporal CLI without writing a Client at
all. While the Workflow is still waiting for the Signal, run:

```
temporal workflow query \
    --workflow-id="pizza-workflow-order-XD001" \
    --type="orderStatus"
```

You should see the same value the Client received:

```
Query result:
  QueryResult  "Out for delivery"
```

## Part E: Querying a Closed Workflow

Closed Workflows can also be queried. Send the Signal so the Workflow
completes, then Query it again.

1. From a terminal in the `practice` subdirectory, send the Signal:
   ```
   mvn exec:java -Dexec.mainClass="queryingworkflows.SignalClient"
   ```
   You won't see output from the `SignalClient` itself, but the terminal
   running the `Starter` should now show the final `OrderConfirmation`. The
   Workflow is now closed.
2. Query the closed Workflow using the CLI:
   ```
   temporal workflow query \
       --workflow-id="pizza-workflow-order-XD001" \
       --type="orderStatus"
   ```
3. You should see the final state:
   ```
   Query result:
     QueryResult  "Order complete"
   ```

A Worker must be running for Queries against closed Workflows to succeed.

### This is the end of the exercise.
