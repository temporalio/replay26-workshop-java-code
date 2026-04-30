# Exercise #3: Sending a Signal from a Client

During this exercise, you will:

- Inspect a Signal definition and handler
- Retrieve a handle on a running Workflow
- Send a Signal to that Workflow from a Temporal Client

Make your changes to the code in the `practice` subdirectory (look for `TODO`
comments that will guide you to where you should make changes). If you need a
hint or want to verify your changes, look at the complete version in the
`solution` subdirectory.

## Background: How the Signal Is Already Wired Up

In Java, a Signal is declared on the Workflow interface with the
`@SignalMethod` annotation. The Signal handler is just a regular method on
the Workflow implementation class. Both pieces are already in place for this
exercise. Take a moment to read them so you understand what your Client will
be triggering.

1. Open `PizzaWorkflow.java` in
   `practice/src/main/java/sendingsignalsclient/orderpizza`. Notice the
   `@SignalMethod`-annotated `fulfillOrderSignal(boolean bool)` method
   alongside the `@WorkflowMethod` `orderPizza(PizzaOrder order)`.
2. Open `PizzaWorkflowImpl.java` in the same subdirectory. Find:
   1. The `signalProcessed` boolean and the `Workflow.await(...)` call inside
      `orderPizza`. The Workflow blocks at that line until the Signal arrives
      or 10 seconds elapse.
   2. The `fulfillOrderSignal` method, which sets `fulfilled` to the value
      passed in and flips `signalProcessed` to `true`.

You won't change either of these files. Your job is to write the Client that
sends the Signal.

## Part A: Create a Stub on the Pizza Workflow

To Signal a running Workflow from a Client, you first get a stub for that
Workflow by its Workflow ID.

1. Open `SignalClient.java` in
   `practice/src/main/java/sendingsignalsclient`.
2. Locate the `TODO: PART A` comment. The `Starter` class uses the Workflow ID
   pattern `pizza-workflow-order-<orderNumber>`, and the order number for this
   exercise is `XD001`. Replace `"CHANGE_ME"` with `"pizza-workflow-order-XD001"`.
3. Save the file.

## Part B: Send the Signal

Now use the stub to invoke the Signal method.

1. Continue editing `SignalClient.java`.
2. Locate the `TODO: PART B` comment. On the line below it, call
   `workflow.fulfillOrderSignal(true)`. Because the stub is typed as a
   `PizzaWorkflow`, calling the `@SignalMethod` method on it sends the Signal
   to the running Workflow.
3. Save the file.

## Part C: Run the Workflow

All commands below must be run from the `practice` subdirectory. Make sure
`temporal server start-dev` is running in another terminal.

1. Compile the code by running `mvn clean compile`.
2. In one terminal, start the Worker by running:
   ```
   mvn exec:java -Dexec.mainClass="sendingsignalsclient.SendSignalClientWorker"
   ```
3. In another terminal, start the Workflow by running:
   ```
   mvn exec:java -Dexec.mainClass="sendingsignalsclient.Starter"
   ```
   The Workflow will run up to the `Workflow.await(...)` call and then block,
   waiting for the Signal.
4. In a third terminal, send the Signal by running:
   ```
   mvn exec:java -Dexec.mainClass="sendingsignalsclient.SignalClient"
   ```
   This unblocks the Workflow, which then bills the customer and completes.

In the terminal where you ran the `Starter`, you should see output similar to:

```
Workflow result: OrderConfirmation{orderNumber='XD001', status='SUCCESS', confirmationNumber='P24601', billingTimestamp=..., amount=3500}
```

You have successfully sent a Signal from a Temporal Client to a running
Workflow.

### This is the end of the exercise.
