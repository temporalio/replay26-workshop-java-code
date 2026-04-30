# Exercise #7 (Optional): Rollback with the Saga Pattern

During this exercise, you will:

- Orchestrate Activities using the Saga pattern to implement compensating
  transactions
- Handle Workflow failures with rollback logic

Make your changes to the code in the `practice` subdirectory (look for `TODO`
comments that will guide you to where you should make changes to the code).
If you need a hint or want to verify your changes, look at the complete
version in the `solution` subdirectory.

This exercise builds on the same pizza-ordering Workflow as previous
exercises. The credit card validation in `processCreditCard` still rejects
non-16-digit card numbers, which gives you a controllable failure point to
trigger compensations.

## Part A: Review the Rollback Activities

Three new Activities are introduced to demonstrate rollback:

- `updateInventory` runs as a normal step in the Workflow.
- `revertInventory` is the compensating action for `updateInventory`.
- `refundCustomer` is the compensating action for `sendBill`.

1. Open `PizzaActivities.java` and `PizzaActivitiesImpl.java` in
   `practice/src/main/java/pizzaworkflow` and read through the new methods.
   They log their work but don't make any real inventory or billing changes.
   The goal is to demonstrate the Temporal Saga API, not implement a full
   payment system.
2. Close the files.

## Part B: Wire the Compensations into the Workflow

The Temporal Java SDK includes a `Saga` helper class that tracks compensating
actions and runs them when you call `saga.compensate()`. The Workflow already
has a `Saga` object declared at the top of `orderPizza`, and the
`updateInventory` Activity is already wired up with its compensation.

1. Open `PizzaWorkflowImpl.java`.
2. Read the existing `saga.addCompensation(activities::revertInventory, order)`
   call that runs just before `updateInventory`. You'll mirror this pattern
   for `processCreditCard`.
3. Locate the call to `processCreditCard`. Add a `saga.addCompensation(...)`
   call immediately above it that registers `refundCustomer` as the
   compensating Activity. Use `creditCardInfo` as the argument that gets
   passed to `refundCustomer` if compensation runs.
4. In the `catch (ActivityFailure e)` block that wraps the
   `updateInventory` / `processCreditCard` calls, add:
   ```java
   saga.compensate();
   ```
   This runs all registered compensations in reverse order before the
   exception is rethrown.
5. Save the file and run `mvn clean compile`.

## Part C: Test the Rollback

You'll run the Workflow once with a valid card to confirm the happy path,
then break the card number to trigger compensation.

All commands below assume your terminal is in the
`exercises/rollback-with-saga/practice` directory and that
`temporal server start-dev` is running in another terminal.

1. Start the Worker:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.PizzaWorker"
   ```
2. In another terminal, start the Workflow:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.Starter"
   ```
3. The Workflow should complete successfully. Verify its status is
   **Completed** in the Web UI.
4. Stop the Worker with Ctrl-C.
5. Open `Starter.java`. Locate this line:
   ```java
   CreditCardInfo cardInfo = new CreditCardInfo("Lisa Anderson", "4242424242424242");
   ```
   Delete a digit (per the `TODO Part C` comment) so the card number is
   shorter than 16 digits. That will fail validation in `processCreditCard`
   and trigger compensation.
6. Recompile with `mvn clean compile`.
7. Restart the Worker:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.PizzaWorker"
   ```
8. Restart the Workflow:
   ```
   mvn exec:java -Dexec.mainClass="pizzaworkflow.Starter"
   ```
9. After a short delay, you should see a stack trace indicating the Activity
   failed.
10. In the Web UI, the Workflow will be marked **Failed**. The original
    exception is rethrown after the compensations run. Look for:
    - Where the Activity Task failed and what the error message was.
    - Where the compensations took place (search for log lines like
      `Customer refunded` and `Reverted changes to inventory`).

You have now implemented the Saga pattern using the Temporal Java SDK.

### This is the end of the exercise.
