---
id: java-run-your-first-app
title: Run your first Temporal application with the Java SDK
sidebar_label: Run your first app
---

import { ResponsivePlayer } from '../src/components'

Welcome to the evolution of application development!

This is a tutorial for developers who are new to [Temporal](/docs/overview) and have some basic knowledge of Java. We recommend setting aside ~20 minutes to complete it. By following this tutorial, you will achieve a few things:

- Complete several runs of a Temporal Workflow application using the Temporal server and [ Java SDK](https://github.com/temporalio/java-sdk).
- Practice accessing and using the visibility of the Workflow's state.
- Understand the inherent reliability of Workflow functions.
- Learn many of Temporal's core terminology and concepts.

The Temporal server and corresponding SDK provide a comprehensive solution to the complexities which arise from modern application development.

<img class="docs-image-centered" src={require('../static/img/docs/temporal-server-and-sdk-icons.png').default} />

Think of Temporal as a sort of "cure all" for the pains you experience as a developer when trying to build reliable applications. Temporal provides reliability primitives right out of the box, such as seamless and fault tolerant application state tracking, automatic retries, timeouts, databases to track application states, rollbacks due to process failures, and more.

Let's run our first Temporal Workflow application and forever change the way you approach application development.

## ![](/img/docs/repair-tools.png) &nbsp;&nbsp; Project setup

Before starting, make sure you have looked over the [tutorial prerequisites](/docs/java-sdk-tutorial-prerequisites) setup.

This tutorial uses a fully working template application. You can get the ['Java project template'](https://github.com/temporalio/money-transfer-project-template-java) by downloading it as a [zip](https://github.com/temporalio/money-transfer-project-template-java/archive/master.zip) or by [creating a new repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template#creating-a-repository-from-a-template) in your own Github account and then cloning the repo to the location of your choice. Look for the "Use this template" button:

<img class="docs-image-centered docs-image-max-width-20" src={require('../static/img/docs/use-this-template.png').default} />

The last setup step is to build the project. If you open it with [IntelliJ](https://www.jetbrains.com/idea/) the project will build automatically. If you prefer to run programs from your terminal, make sure you have [Gradle](https://gradle.org/install/) installed and run the Gradle build command from the root of the project:

```
./gradlew build
```

Once your project has finished building, you are ready to go.

## ![](/img/docs/workflow.png) &nbsp;&nbsp; Application overview

This project template mimics a "money transfer" application, giving you the minimum elements needed to start building your own applications and understand some of the value that Temporal provides right out of the box. The project includes a predefined [Workflow function](/docs/workflows) which orchestrates the execution of an Account object's `withdraw()` and `deposit()` methods, representing a transfer of money from one account to another. Temporal calls such methods [Activity functions](/docs/activities).

To run the application you will do the following:

1. Send a signal to the Temporal server to start the money transfer. The Temporal server will track the progress of your Workflow function execution.
2. Run a Worker. A Worker is a wrapper around your compiled Workflow and Activity code. A Worker's only job is to execute the Activity and Workflow functions and communicate the results back to the Temporal server.

Here's a high-level illustration of what's happening:

![High level project design](/img/docs/temporal-high-level-application-design.png)

### The Workflow function

When you "start" a Workflow you are basically telling the Temporal server, "track the state of the Workflow with this signature". Workers will execute the Workflow code below, piece by piece, relaying the execution events and results back to the server.

<!--SNIPSTART money-transfer-project-template-java-workflow-implementation-->
<!--SNIPEND-->

### Initiate transfer

To initiate our transfer, we need to tell the server that we want this Workflow code to execute. Make sure the [Temporal server](/docs/install-temporal-server) is running in a terminal, and then run the InitiateMoneyTransfer class within IntelliJ or from the project root using the following command:

```
./gradlew initiateTransfer
```

There are two ways to start a Workflow with Temporal, either via the SDK or via the [CLI](/docs/tctl). For this tutorial we will use the SDK to start the Workflow, which is how most Workflows would be started in a production environment. Here is the code we just ran to start the Workflow:

<!--SNIPSTART money-transfer-project-template-java-workflow-initiator-->
<!--SNIPEND-->

The call to the Temporal server can be done synchronously or asynchronously. Here we do it asynchronously, so you will see the program run, tell you the transaction is processing, and exit.

### State visibility

OK, now it's time to check out one of the really cool value propositions offered by Temporal: application state visibility. Visit the [Temporal Web UI](localhost:8088) where you will see your Workflow listed.

Next, click the "Run Id" for your Workflow. Now we can see everything we want to know about the execution of the Workflow code we told the server to track, such as what parameter values it was given, timeout configurations, scheduled retries, number of attempts, stack traceable errors, and more.

It seems that our Workflow is "running", but why hasn't the Workflow and Activity code executed yet? Investigate by clicking on the Task Queue name to view active "Pollers" registered to handle these Tasks. The list will be empty. There are no Workers polling the Task Queue!

<ResponsivePlayer url='https://youtu.be/oUGf2D4kX3U' loop='true' playing='true'/>

### The Worker

A Worker is responsible for executing pieces of Workflow and Activity code, and it knows which piece to execute from Tasks that it gets from the Task Queue that it listens to. After The Worker executes code, it returns the results back to the Temporal server. This is what our Worker looks like:

<!--SNIPSTART money-transfer-project-template-java-worker-->
<!--SNIPEND-->

Notice that the Worker will listen to the same Task Queue that the Workflow and Activity tasks are sent to. This is called "Task routing", and is a built-in mechanism for load balancing.

<!--SNIPSTART money-transfer-project-template-java-shared-constants-->
<!--SNIPEND-->

It's time to start start the Worker. Run the TransferMoneyWorker class from IntelliJ, or run the following command from the project root in separate terminal:

```
./gradlew startWorker
```

When you start the Worker it begins polling the Task Queue. The first Task the Worker finds is the one that tells it to execute the Workflow function. The Worker communicates the event back to the server which then causes the server to send Activity Tasks to the Task Queue as well. The Worker then grabs each of the Activity Tasks in their respective order from the Task Queue and executes each of the corresponding Activities.

<img class="docs-image-centered docs-image-max-width-20" src={require('../static/img/docs/confetti.png').default} />

**Congratulations**, you just ran a Temporal Workflow application!

## ![](/img/docs/warning.png) &nbsp;&nbsp; Failure simulation

You just got a taste of one of Temporal's amazing value propositions: visibility into the Workflow and the status of the Workers executing the code. Let's explore another key value proposition, maintaining the state of a Workflow, even in the face of failures. To demonstrate this we will simulate some failures for our Workflow. Make sure your Worker is stopped before proceeding.

### Server crash

Unlike many modern applications that require complex leader election processes and external databases to handle failure, Temporal automatically preserves the state of your Workflow even if the server is down. You can easily test this by following these steps (again, make sure your Worker is stopped so your Workflow doesn't finish):

1. Start the Workflow again.
2. Verify the Workflow is running in the UI.
3. Shut down the Temporal server by either using 'Ctrl c' or via the Docker dashboard.
4. After the Temporal server has stopped, restart it and visit the UI.

Your Workflow is still there!

### Activity error

Next let's simulate a bug in the `deposit()` Activity method. Let your Workflow continue to run. Inside your project open the AccountActivityImpl.java file and uncomment the line that throws an exception in the deposit method:

<!--SNIPSTART money-transfer-project-template-java-activity-implementation-->
<!--SNIPEND-->

Save it and run the Worker. The Worker completes the `withdraw()` Activity method, but throws the exception when it attempts the `deposit()` Activity method. Notice how the Worker keeps retrying the `deposit()` method? To view information of what is happening, visit the [UI](localhost:8088) and click on the RunId of the Workflow. You will see the pending Activity listed there with details such as its state, the number of times it has been attempted, and the next scheduled attempt.

![Activity UI error details](/img/docs/web-ui-activity-error-info.png)

Traditionally application developers are forced to implement timeout and retry logic within the business code itself. With Temporal, one of the key value propositions is that [timeout configurations](/docs/activities/#timeouts) and [retry policies](/docs/activities/#retries) are specified in the Workflow code as Activity options. In our Workflow code you can see that we have specified a setStartToCloseTimeout for our Activities, and set a retry policy that tells the server to retry them up to 500 times. But we did that as an example for this tutorial, as Temporal automatically uses a default retry policy if one isn't specified!

So, your Workflow is running, but only the `withdraw()` Activity method succeeded. In any other application, the whole process would likely have to be abandoned and rolled back. So, here is the last value proposition of this tutorial: With Temporal, we can debug the issue while the Workflow is running! Pretend that you found a potential fix for the issue; Re-comment the exception in the AccountActivityImpl.java file and save your changes. How can we possibly update Workflow code that is already halfway complete? With Temporal, it is actually very simple: just restart the Worker!

On the next scheduled attempt, the Worker will pick up right where the Workflow was failing and successfully execute the newly compiled `deposit()` Activity method, completing the Workflow. Basically, you have just fixed a bug "on the fly" with out losing the state of the Workflow.

<img class="docs-image-centered docs-image-max-width-20" src={require('../static/img/docs/boost.png').default} />

## ![](/img/docs/wisdom.png) &nbsp;&nbsp; Lore check

Great work! You now know how to run a Temporal Workflow and understand some of the key values Temporal offers. Let's do a quick review to make sure you remember some of the more important pieces.

![One](/img/docs/one.png) &nbsp;&nbsp; **What are four of Temporal's value propositions that we touched on in this tutorial?**

1. Temporal gives you full visibility in the state of your Workflow and code execution.
2. Temporal maintains the state of your Workflow, even through server outages and errors.
3. Temporal makes it easy to timeout and retry Activity code using options that exist outside of your business logic.
4. Temporal enables you to perform "live debugging" of your business logic while the Workflow is running.

![Two](/img/docs/two.png) &nbsp;&nbsp; **How do you pair up Workflow initiation with a Worker that executes it?**

Use the same Task Queue.

![Three](/img/docs/three.png) &nbsp;&nbsp; **What do we have to do if we make changes to Activity code for a Workflow that is running?**

Restart the Worker.
