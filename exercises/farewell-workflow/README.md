# Exercise #1: Farewell Workflow

During this exercise, you will create an Activity method, register it with the
Worker, and modify a Workflow to execute it.

Remember that in this and other exercises, you will make your changes in the
`practice` directory. Look for `TODO` comments, which will provide hints about
what you'll need to change. If you get stuck and need additional hints, or if
you want to check your work, look at the completed example in the `solution`
directory.

## Part A: Write an Activity Method

The `GreetingActivities.java` file defines an interface with the Activity method
signatures. The `GreetingActivitiesImpl.java` file implements those methods,
along with a `callService` utility method that calls a microservice. Currently,
both files only handle a Spanish greeting; you'll add the matching farewell.

1. Open the `GreetingActivities.java` file (located in the
   `practice/src/main/java/farewellworkflow` subdirectory) in the editor.
2. Define a new Activity method signature that will get a custom farewell
   message from the microservice. Use any valid name you like, but remember
   it; you'll need to use the same name when implementing the method.
3. Save the file.
4. Open the `GreetingActivitiesImpl.java` file (in the same subdirectory) in
   the editor.
5. Implement the Activity method you just declared:
   1. Copy the `greetInSpanish` method as a starting point.
   2. Rename the new method to match the signature you added in
      `GreetingActivities.java`.
   3. Change the first argument to `callService` from `"get-spanish-greeting"`
      to `"get-spanish-farewell"`.
6. Save the file.

## Part B: Register the Activity Method

1. Open the `GreetingWorker.java` file (located in the
   `practice/src/main/java/farewellworkflow` subdirectory) in the editor.
2. Locate the `TODO` line and replace `new TODO()` with a new instance of your
   Activity implementation class so the Worker registers it.
3. Save the file.

## Part C: Modify the Workflow to Execute Your New Activity

1. Open the `GreetingWorkflowImpl.java` file (in the same subdirectory) in
   the editor.
2. Locate the `TODO` comment, uncomment the line below it, and update that line
   to call the new Activity method instead of `greetInSpanish`.
3. Save the file.

## Part D: Start the Microservice and Run the Workflow

All commands below must be run from the `practice` subdirectory. Make sure
`temporal server start-dev` is running in another terminal before continuing.

1. Compile the code by running `mvn clean compile`.
2. Start the microservice in a terminal by running:
   ```
   mvn exec:java -Dexec.mainClass="farewellworkflow.Microservice"
   ```
3. In another terminal, start your Worker by running:
   ```
   mvn exec:java -Dexec.mainClass="farewellworkflow.GreetingWorker"
   ```
4. In a third terminal, execute your Workflow by running:
   ```
   mvn exec:java -Dexec.mainClass="farewellworkflow.Starter" -Dexec.args="Donna"
   ```
   Replace `Donna` with your own name.

If there is time remaining, experiment with Activity failures and retries by
stopping the microservice (press Ctrl-C in its terminal) and re-running the
Workflow. Look at the Web UI to see the status of the Workflow and its
Activities. After a few seconds, restart the microservice by running the same
command used to start it earlier. You should find that the Workflow will now
complete successfully following the next Activity retry.

### This is the end of the exercise.
