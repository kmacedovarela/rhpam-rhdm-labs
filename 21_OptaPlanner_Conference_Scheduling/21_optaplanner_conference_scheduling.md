OptaPlanner Conference Scheduling
=================================

In this lab we will implement the OptaPlanner Conference Scheduling demo. You are provided with the class model of the solution. However, the model has not been configured to be solved by OptaPlanner yet. OptaPlanner configuration annotations are missing, and there is no application code yet that creates the OptaPlanner `Solver` and solves the problem.

## Goals

-   Annotate the current class model with OptaPlanner annotations to complete the domain model.

-   Create the application code that creates the solver and solves the problem.

-   Create the benchmarker code and run the benchmarker.

## Pre reqs

-   The project containing the problem.

Problem Statement
=================

The Conference Scheduling project enables users to create an optimized solution to the problem of assigning conference talks to rooms and timeslots.

![Conference Scheduling Class Diagram](images/conferenceSchedulingClassDiagram_0.png)

Create the Solver
=================

As a first exercise we will create the `Solver` for our optimization problem. Although we will not be able to solve anything yet, as our domain model has not yet been configured properly, having a the solver implemented allows us to immediately run the application and validate what configuration is still missing in our model.

We’ve already provided the class `ConferenceSchedulingApp`. When you open the class, you can see that we’ve already provided code that loads the problem from an Excel file, and code that creates a new Excel file from the solution created by OptaPlanner.

-   In the `ConferenceSchedulingApp`, create a new `SolverFactory` from the XML file referenced by `SolverConfig`. You can use the static factory methods provided in the `SolverFactory` class.

-   From the `SolverFactory`, build a `Solver`.

-   Call the `solve` method on your `Solver` and pass it the `problemOne` problem.

-   Retrieve the solution from the `Solver` (hint: the `solve` method returns something ….), and store it in the `solutionOne` variable so it will be written to an output Excel file.

-   Run your application. If you implemented the code correctly, the application will throw an exception, stating that no **PlanningSolution** has been deifned.

Next, we will configure our domain model so OptaPlanner can start creating solutions to our problem.

Determine PlanningSolution, PlanningEntity and PlanningVariables
================================================================

The first thing we need to define is which classes in the domain model are the:

-   PlanningSolution: represents a problem and a possible solution of that problem.

-   PlanningEntity / PlanningEntities: the entity that should be planned.

-   PlanningVariable / PlanningVariables: the variable or field of a PlanningEntity that should be planned. This is the variable that OptaPlanner can

All the classes are already available in your project, you just need to put the annotations on the correct classes. So the task is:

-   Determine the PlanningSolution of the problem and annotate that class with the `@PlanningSolution` annotation.

-   Run the application. If everything is correct, the application will now throw an error stating that no **PlanningEntity** has been defined.

-   Determine the PlanningEntity of the problem and annotate that class with the `@PlanningEntity` annotation.

-   Run the application. This time you’ll get an error that there is no **PlanningEntityCollectionProperty** defined.

-   Determine the **PlanningEntityCollectionProperty**. This is a property of the **PlanningSolution** that contains the collection of entities that OptaPlanner needs to plan. Annotate the correct field with the `@PlanningEntityCollectionProperty` annotation.

-   Run the application. The next error is that we have not yet defined a **PlanningScore**.

-   The **PlanningScore** holds the score of a given solution. Hence, it is defined on the **PlanningSolution** class. Determine which field in the **PlanningSolution** is the field that holds the score and annotate it with `@PlanningScore`.

-   Run the application. The application prints an error that there are no **PlanningVariables** configured. **PlanningVariables** are the variables that OptaPlanner can **play** with. I.e., they are the variables of a **PlanningEntity** that OptaPlanner changes during solving to find better solutions.

-   Our **PlanningEntity** contains 2 **PlanningVariables**. Determine which variables these are and annotate them with the `@PlanningVariable` annotation. (hint: analyse the class diagram and look for many-to-one relationships between the **PlanningEntity** class and the **PlanningVariable** classes).

-   Run the application. The next error is shown. It says that the **PlanningVariables** have no **valueRangeProviderRefs** defined. A **Value Range Provider** defines, as the name suggests, the possible values of a **PlanningVariable**. So, we need to find the collections that define the value range of our **PlanningVariables**.

-   Normally the **Value Range Providers** are defined on the **PlanningSolution** class. In this implementation however, they are defined on the **PlanningEntity** class. Open the **PlanningEntity** class and find the 2 methods that return a `Set` of values for our **PlanningVariables**. Annotate these 2 **getter** methods with the `@ValueRangeProvider` annotation. This annotation accepts and `id`, so configure the two annotations with 2 different `ids`. We need the `ids` later when we configure the `@PlanningVariable` entities we created earlier.

-   Go back to the `@PlanningVariable` annotations we created earlier and add the property `valueRangeProviderRefs` to them, with a refence to the **Value Range Providers** we created earlier.

-   Run the application. It will print an error that we have not defined a **ProblemFactCollectionProperty** or **ProblemFactProperty**, which is required when we use Drools to calculate the score. These properties define which **facts** OptaPlanner should insert into the Drools session, so they can be used in our rules and thus in the score calculation.

-   Open the **PlanningSolution** class. In the case of this application, we need to annotate the following field with the `@ProblemFactCollectionProperty`, in order for these facts to be inserted into the Drools session:

    -   `talkTypeList`

    -   `timeslotList`

    -   `roomList`

    -   `speakerList`

-   Run the application. The next error will be printed. This error indicates that there is a rule that does not have a matching **ConstraintWeight** configured on the **ConstraintConfiguration** class. The **ConstraintConfiguration** class in OptaPlanner is a new, out-of-the-box, way to make constraints and their weights easily externally configurable by (business) users. Instead of hardcoding the score and constraint weights in rules, the constraint weights are configured in a Java class. The values can than be easily configured by creating a User Interface that manipulates the values used by this class.

-   The application already contains a class called `ConferenceConstraintConfiguration`, which contains the `ConstraintConfiguration` for all our rules. Simply annotate this class with `@ConstraintConfiguration(constraintPackage = "org.optaplanner.training.conference.solver")`. Also annotate the property of the **PlanningSolution** that references this class with `@ConstraintConfigurationProvider`.

-   Run the application. If you’ve configured everything correctly, the solver will now start running and will start creating solutions.

We now have a correctly configured and working OptaPlanner application.

TerminationConfig
=================

If you look at the application you’ve just started, you’ll see that it is still running. Without the proper configuration or without an external signal to the running `Solver`, OptaPlanner will keep trying to find better solutions, and the `Solver` will run indefinitely. To stop the `Solver`, we either need to call `Solver.terminateEarly()` from another thread in our application, or we need to add a **TerminationConfig** to our solver configuration.

OptaPlanner provides various out-of-the-box termination strategies. Paragraph 9.9 of the OptaPlanner provides a full overview of these strategies. Some examples: 

* Time Spent Termination
* Unimproved Time Spent Termination
* Best Score Termination
* Best Score Feasible Termination

In this lab we will configure the solver to stop after 30 seconds (note that this is a lab and this is not the recommended value for a production scenario). We do this by adding the following XML snippet to the solver configuration: 

````xml
<termination> <secondsSpentLimit>30<secondsSpentLimit></termination>
````

Run the application again. Observe that the application stops after 30 seconds and prints the solution. 

````
INFO Solving ended: time spent (30000), best score (0hard/0medium/-33795soft), score calculation speed (4519/sec), phase total (2), environment mode (REPRODUCIBLE). 
````

Also note that there is a new Excel file in the **data/conferencescheduling/solved** directory. Open the file in Microsoft Excel or LibreOffice and observe the yellow tabs in the spreadsheet which contains a (the current best) solution to the problem.

Explaining the score
====================

We have seen how OptaPlanner creates a solution from a given input problem. We’ve also seen the score of the solution and the output in the form of an Excel file.

However, merely the score value and the result does not explain how that score was produced, which constraints are broken, which entities contributed the most to the score etc.

OptaPlanner keeps track of all this information and has various ways of retrieving this data. Recently, a new feature was implemented that allows a user to simply print the explanation of the score to a String, we can than be logged into a file or to the console. This provides the user a very easy way to get fast feedback on the score and its explanation without the upfront investment in complex UIs and custom heatmaps.

-   Open the `ConferenceSchedulingApp`.
-   When the solver terminates, we can print the explanation of the score with the following method: ` solver.explainBestScore();`
-   Print the score to the `LOGGER` or directly to the console.
-   Run the application again. Observe the score explanation that is printed when the solver finishes.

Entity Pinning
==============

There are various situations in which you don’t want OptaPlanner to change certain **PlanningEntites**. For example, if you’ve manually planned an entity because of personal preference to assign a talk to a given room and timeslot, you don’t want OptaPlanner to change that assignment. **Pinning** allows entities to be pinned, which tells OptaPlanner not to change the planning of those specific entities.

OptaPlanner provides the `@PlanningPin` annotation. This annotation can be configured on a `boolean` attribute, and when that boolean is set to `true`, makes the entity immovable.

Task:

* Configure the `@PlanningPin` annotation on the appropriate variable of the **PlanningEntity**.

Benchmarking
============

One of the key tasks of an OptaPlanner project is to create a benchmark. OptaPlanner provides an out-of-the-box `PlannerBenchmark` component, which can be easily created from an existing `SolverFactory`. The `PlannerBenchmark` can be given a number of problems, run the benchmark, and generate a full HTML benchmark report. This report contains information like: \* Best Score \* Score Calculation Speed \* Score statistics

The general advise is to create a Benchmark application as soon as possible in your project, basically from the moment that you can create and run a solver, and regularly create benchmark reports. For example, you can incorporate a benchmark run in a nightly build to produce a report on a daily basis.

Benchmark reports enable you to find potential problems quickly. For example, having a problem with a very large solution space, can greatly impact the result that OptaPlanner provides. Or newly introduced rules can be badly written and so complex, that they negatively influence the score calculation speed, and thus the quality of the solution.

Because we already have a SolverFactory defined in our application, creating a `PlanningBenchmark` is easy:

-   Create a new class called `ConferenceShedulingBenchmarker`.

-   Give the class a `main` method.

-   Use the code you’ve written before to create a `SolverFactory`.

-   Load 2 of the pre-defined problems in Excel format into the PlanningSolution class using the provided `ConferenceSchedulingXlsxFileIO` utility class.

-   Create a new `PlannerBenchmarkFactory` from your `SolverFactory`.

-   From the `PlannerBenchmarkFactory`, retrieve the `PlannerBenchmarkConfig` to retrieve the `SolverConfig`, and set the `TerminationConfig`. The `TerminationConfig` controls the termination of the `Solvers` used in the benchmark. If we don’t configure the termination, the solvers will run indefinitely and the benchmark will never finish.

-   Use the `PlannerBenchmarkFactory` to create the `PlannerBenchmark` with the 2 problems we’ve loaded earlier.

-   Run the benchmark.

-   When the benchmark finishes, it will create an HTLM report and store it in the **local/benchmarkReport** directory within your project.

-   Open the report and analyse the results.
