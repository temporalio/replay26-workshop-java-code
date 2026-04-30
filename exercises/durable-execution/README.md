# Exercise #2: Observing Durable Execution

During this exercise, you will:

- Create Workflow and Activity loggers
- Add logging statements to the code
- Add a Timer to the Workflow Definition
- Launch two Workers and run the Workflow
- Kill one of the Workers during Workflow Execution and observe that the
  remaining Worker completes the execution

Make your changes to the code in the `practice` subdirectory (look for
`TODO` comments that will guide you to where you should make changes to
the code). If you need a hint or want to verify your changes, look at the
complete version in the `solution` subdirectory.

## Prerequisite: Ensure that the Microservice Is Running

If you haven't already started the microservice in previous exercises, do so
in a separate terminal. From either the `practice` or `solution` subdirectory
for this exercise:

1. `cd microservice`
2. `mvn clean compile`
3. `mvn exec:java -Dexec.mainClass="translationapi.Microservice"`

The microservice code does not change between the practice and solution
examples.

## Prerequisite: Ensure that the Temporal Service Is Running

In another terminal, run `temporal server start-dev` if it isn't already
running.

## Part A: Add Logging to the Workflow Code

1. Edit the `TranslationWorkflowImpl.java` file in the
   `practice/src/main/java/translationworkflow` subdirectory.
2. Define a Workflow logger at the top of the class. Use
   `Workflow.getLogger(...)`.
3. Replace the first `TODO` in the `sayHelloGoodbye` method with a log
   statement at the Info level.
   1. It should mention that the Workflow has been invoked.
   2. It should also include the name and language code passed as input.
4. Before each call to an Activity method, log a message at the Debug level.
   1. It should identify the word being translated.
   2. It should also include the language code.
5. Save your changes.

## Part B: Add Logging to the Activity Code

1. Edit the `TranslationActivitiesImpl.java` file.
2. Add imports for `org.slf4j.Logger` and `org.slf4j.LoggerFactory` (Activities
   don't require a specialized logger; slf4j is fine).
3. Define an Activity logger at the top of the class as an instance variable.
4. Replace the `TODO` in the `translateTerm` method with a log statement at the
   Info level so you'll know when the Activity is invoked.
   1. Begin the message with the string `[ACTIVITY INVOKED]`. You'll scan the
      logs for this string in a later step.
   2. Include the term being translated and the language code.
5. Optionally, add log statements at the Error level anywhere the Activity
   throws an exception.
6. Near the bottom of the method, use the Debug level to log the successful
   translation, including the translated term.
7. Save your changes.

## Part C: Add a Timer to the Workflow

You will now add a Timer between the two Activity calls in the Workflow
Definition, which will make it easier to observe durable execution in the next
section.

1. After the statement where `helloMessage` is defined, but before the
   statement where `goodbyeInput` is defined, add a log statement at the Info
   level with the message `Sleeping between translation calls`.
2. Just after the new log statement, call `Workflow.sleep(Duration.ofSeconds(10))`
   to set a Timer for 10 seconds.
3. Save your changes.

## Part D: Observe Durable Execution

It is typical to run Temporal applications using two or more Worker
processes. Additional Workers let the application scale and increase
availability. Another Worker can take over if one crashes during Workflow
Execution. You'll see this for yourself now.

Before proceeding, make sure no Workers are running for this or any previous
exercise. Read through all of these instructions before you begin, so you know
when and how to react.

All commands below must be run from the `practice` subdirectory (unless
otherwise noted). Run `mvn clean compile` first.

1. Start the first Worker by running:
   ```
   mvn exec:java -Dexec.mainClass="translationworkflow.TranslationWorker"
   ```
2. In another terminal, start a second Worker with the same command.
3. In a third terminal, execute the Workflow by running:
   ```
   mvn exec:java -Dexec.mainClass="translationworkflow.Starter" -Dexec.args="Tatiana sk"
   ```
   Replace `Tatiana` with your first name.
4. Observe the output in the terminal windows used by each Worker.
5. As soon as you see a log message in one of the Worker terminals indicating
   that the Timer has started, find the terminal whose log shows
   `[ACTIVITY INVOKED]...` and press Ctrl-C in *that* window to kill that
   Worker process.
6. Switch to the terminal for the other Worker. Within a few seconds, you
   should see new output indicating that it has resumed execution of the
   Workflow.
7. Once you see log output indicating that translation was successful, switch
   back to the terminal where you started the Workflow.

After the final step, you should see the translated Hello and Goodbye
messages, which confirms that Workflow Execution completed successfully
despite the original Worker being killed.

Since you added logging code to the Workflow and Activity, take a moment to
look at what you see in the terminal windows for each Worker and think about
what took place. You may also find it helpful to look at this Workflow
Execution in the Web UI.

The microservice for this exercise outputs each successful translation. If
you look at its terminal window, you will see that the service translated
Hello (the first Activity) only once, even though the Worker was killed
after that translation took place. In other words, Temporal did not
re-execute the completed Activity when it restored the state of the
Workflow Execution.

### This is the end of the exercise.
