# Exercise #4 (Optional): Sending an External Signal

During this exercise, you will:

- Define and implement a Signal handler on a Workflow
- Retrieve a handle on a running Workflow from another Workflow
- Send a Signal from one Workflow to another
- Use a Temporal Client to start both Workflows

Make your changes to the code in the `practice` subdirectory (look for
`TODO` comments that will guide you to where you should make changes to
the code). If you need a hint or want to verify your changes, look at the
complete version in the `solution` subdirectory.

This exercise has two Workflows. `PizzaWorkflow` runs an order through
distance calculation, then waits for a Signal before billing the customer.
`FulfillOrderWorkflow` makes and delivers the pizzas, and then Signals
`PizzaWorkflow` to tell it whether fulfillment succeeded.

## Part A: Define and Implement the Signal

In Java, a Signal is declared on the Workflow interface with `@SignalMethod`
and implemented as a normal method on the Workflow class.

1. Open `PizzaWorkflow.java` in
   `practice/src/main/java/sendingsignalsexternal/orderpizza`.
   1. Locate the `TODO: PART A` comment.
   2. Define a method `fulfillOrderSignal` that returns `void` and takes a
      single `boolean` parameter (it represents whether the order was
      fulfilled).
   3. Annotate the method with `@SignalMethod`.
   4. Save the file.
2. Open `PizzaWorkflowImpl.java` in the same subdirectory.
   1. Locate the `TODO: PART A` comment near the bottom of the class.
   2. Implement `fulfillOrderSignal`. The method should set the `fulfilled`
      instance variable to the boolean parameter and set `signalProcessed` to
      `true`.

## Part B: Handle the Signal in the Workflow

`PizzaWorkflow` already calls `Workflow.await(Duration.ofSeconds(3), () -> this.signalProcessed)`,
which blocks until the Signal arrives or 3 seconds pass. Once execution
resumes, you should only bill the customer if the Signal indicated the order
was fulfilled.

1. Continue editing `PizzaWorkflowImpl.java`.
2. Locate the `TODO: PART B` comment in the `orderPizza` method.
3. Wrap the `Bill bill = ...` declaration and the surrounding `try/catch`
   that calls `activities.sendBill(bill)` in an `if (this.fulfilled) { ... }`
   block. If `fulfilled` is false, leave `confirmation` unset (or set it to
   `null` and add an else branch that logs that the order was not fulfilled).
4. Save the file.

## Part C: Get a Handle on the Running PizzaWorkflow

`FulfillOrderWorkflow` needs a handle on the running `PizzaWorkflow` so it can
Signal it. Both Workflows receive the same `PizzaOrder`, but
`FulfillOrderWorkflow` also takes the `PizzaWorkflow`'s Workflow ID as a
parameter so it knows which Workflow to Signal.

1. Open `FulfillOrderWorkflowImpl.java` in
   `practice/src/main/java/sendingsignalsexternal/fulfillorder`.
2. Locate the `TODO: PART C` comment at the top of the `fulfillOrder` method.
3. Add this line to create the external Workflow stub:
   ```java
   PizzaWorkflow workflow = Workflow.newExternalWorkflowStub(PizzaWorkflow.class, workflowID);
   ```
4. Save the file (leave it open; you'll edit it again in Part D).

## Part D: Signal the PizzaWorkflow

`fulfillOrder` runs two Activities (`makePizzas` and `deliverPizzas`). After
they succeed, you Signal the `PizzaWorkflow` that fulfillment succeeded; on
failure, you Signal that fulfillment failed.

1. In the same file, locate the two `TODO: PART D` comments.
2. After the successful Activity invocations (just before `return "order fulfilled"`),
   call `workflow.fulfillOrderSignal(true)`.
3. In the `catch` block (just before `return "order not fulfilled"`),
   call `workflow.fulfillOrderSignal(false)`.
4. Save the file.

## Part E: Start the Worker

A single Worker can register multiple Workflows and Activities, so you only
need one Worker for this exercise. `SendSignalExternalWorker` is already
configured to register both Workflow types and both Activity implementations.

All commands below assume your terminal is in the
`exercises/sending-signals-external/practice` directory and that
`temporal server start-dev` is running in another terminal.

1. Compile with `mvn clean compile`.
2. Start the Worker:
   ```
   mvn exec:java -Dexec.mainClass="sendingsignalsexternal.SendSignalExternalWorker"
   ```

## Part F: Run Both Workflows

`Starter.java` starts both Workflows asynchronously using
`WorkflowClient.execute(...)`, so `PizzaWorkflow` doesn't block the Client
while it waits for the Signal:

```java
CompletableFuture<OrderConfirmation> orderConfirmation =
    WorkflowClient.execute(pizzaWorkflow::orderPizza, order);

CompletableFuture<String> fulfillOrderResult =
    WorkflowClient.execute(orderWorkflow::fulfillOrder, order, pizzaWorkflowID);
```

In production these Workflows could live in separate processes, even on
separate machines. They're combined here for simplicity.

1. In another terminal (still in the `practice` subdirectory), run:
   ```
   mvn exec:java -Dexec.mainClass="sendingsignalsexternal.Starter"
   ```
2. In the Worker terminal, you should see log output similar to:
   ```
   orderPizza Workflow Invoked
   getDistance invoked; determining distance to customer address
   Starting to make pizzas for order XD001
   Making pizza: Large, with mushrooms and onions
   ...
   All pizzas for order XD001 are ready!
   Starting delivery XD001 to 742 Evergreen Terrace Apartment 221B Albuquerque NM 87101
   sendBill invoked: customer: 8675309 amount: 4000
   Applying discount
   Bill sent to customer 8675309
   ```
3. The `Starter` terminal will print the final `OrderConfirmation` once
   `PizzaWorkflow` completes.

### This is the end of the exercise.
