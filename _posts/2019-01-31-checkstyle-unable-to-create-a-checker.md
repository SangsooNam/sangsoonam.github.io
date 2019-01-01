---
layout: post
title: "Checkstyle: Unable to create a Checker: configLocation, classpath {null}"
---

Checkstyle helps developers write a solid code by adhering a coding standard. You can modify the configuration file to apply defined rules or to add your own rule. After changing the config file, you would run `checkstyle` task to see the changed behavior. If something is wrong, you will see this error message like this:

{% highlight bash %}
$ ./gradlew checkstyle
Parallel execution with configuration on demand is an incubating feature.
> Task :app:checkstyle FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:checkstyle'.
> Unable to create a Checker: configLocation {/Users/sangsoo/git/static-import-checker/config/checkstyle/checkstyle.xml}, classpath {null}.
...
{% endhighlight %}

There is `classpath {null]` so you might think there is an issue on `classpath`. However, this says there is an issue on your configuration. That is because you changed something wrong. Or, the configuration file could be not compatible anymore if you upgrade the checkstyle version.

{% include google-adsense-in-article.html %}

This error message is short of description. It doesn't explain the problem point precisely. Checkstyle actually showed a better description before 6.10. With that version, you could see the problem point easily.

<p class="code-label">build.gradle</p>
{% highlight groovy %}
apply plugin: 'checkstyle'

checkstyle {
    toolVersion = '6.9'
}
...
{% endhighlight %}

{% highlight bash %}
> Task :app:checkstyle FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:checkstyle'.
> Unable to create a Checker: cannot initialize module TreeWalker - Property 'fileExtensions' in module RegexpSinglelineJava does not exist, please check the documentation
...
{% endhighlight %}

The latest checkstyle version is 8.16 so you probably use a higher version than 6.9. If so, you can see the problem point using `--stacktrace` option. With this option, you can see the whole stack trace. Although this contains the information what you want, there is too much unnecessary information like this:

{% highlight bash %}
$ ./gradlew checkstyle --stacktrace
Parallel execution with configuration on demand is an incubating feature.
> Task :app:checkstyle FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:checkstyle'.
> Unable to create a Checker: configLocation {/Users/sangsoo/git/static-import-checker/config/checkstyle/checkstyle.xml}, classpath {null}.

* Try:
Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Exception is:
org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':app:checkstyle'.
        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:110)
        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.execute(ExecuteActionsTaskExecuter.java:77)
        at org.gradle.api.internal.tasks.execution.OutputDirectoryCreatingTaskExecuter.execute(OutputDirectoryCreatingTaskExecuter.java:51)
        at org.gradle.api.internal.tasks.execution.SkipUpToDateTaskExecuter.execute(SkipUpToDateTaskExecuter.java:59)
        at org.gradle.api.internal.tasks.execution.ResolveTaskOutputCachingStateExecuter.execute(ResolveTaskOutputCachingStateExecuter.java:54)
        at org.gradle.api.internal.tasks.execution.ValidatingTaskExecuter.execute(ValidatingTaskExecuter.java:59)
        at org.gradle.api.internal.tasks.execution.SkipEmptySourceFilesTaskExecuter.execute(SkipEmptySourceFilesTaskExecuter.java:101)
        at org.gradle.api.internal.tasks.execution.FinalizeInputFilePropertiesTaskExecuter.execute(FinalizeInputFilePropertiesTaskExecuter.java:44)
        at org.gradle.api.internal.tasks.execution.CleanupStaleOutputsExecuter.execute(CleanupStaleOutputsExecuter.java:91)
        at org.gradle.api.internal.tasks.execution.ResolveTaskArtifactStateTaskExecuter.execute(ResolveTaskArtifactStateTaskExecuter.java:62)
        at org.gradle.api.internal.tasks.execution.SkipTaskWithNoActionsExecuter.execute(SkipTaskWithNoActionsExecuter.java:59)
        at org.gradle.api.internal.tasks.execution.SkipOnlyIfTaskExecuter.execute(SkipOnlyIfTaskExecuter.java:54)
        at org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter.execute(ExecuteAtMostOnceTaskExecuter.java:43)
        at org.gradle.api.internal.tasks.execution.CatchExceptionTaskExecuter.execute(CatchExceptionTaskExecuter.java:34)
        at org.gradle.api.internal.tasks.execution.EventFiringTaskExecuter$1.run(EventFiringTaskExecuter.java:51)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:300)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:292)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:174)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:90)
        at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
        at org.gradle.api.internal.tasks.execution.EventFiringTaskExecuter.execute(EventFiringTaskExecuter.java:46)
        at org.gradle.execution.taskgraph.LocalTaskInfoExecutor.execute(LocalTaskInfoExecutor.java:42)
        at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph$BuildOperationAwareWorkItemExecutor.execute(DefaultTaskExecutionGraph.java:277)
        at org.gradle.execution.taskgraph.DefaultTaskExecutionGraph$BuildOperationAwareWorkItemExecutor.execute(DefaultTaskExecutionGraph.java:262)
        at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:135)
        at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker$1.execute(DefaultTaskPlanExecutor.java:130)
        at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker.execute(DefaultTaskPlanExecutor.java:200)
        at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker.executeWithWork(DefaultTaskPlanExecutor.java:191)
        at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor$ExecutorWorker.run(DefaultTaskPlanExecutor.java:130)
        at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
        at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
        at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
Caused by: : Unable to create a Checker: configLocation {/Users/sangsoo/git/static-import-checker/config/checkstyle/checkstyle.xml}, classpath {null}.
        at com.puppycrawl.tools.checkstyle.ant.CheckstyleAntTask.createChecker(CheckstyleAntTask.java:405)
        at com.puppycrawl.tools.checkstyle.ant.CheckstyleAntTask.realExecute(CheckstyleAntTask.java:309)
        at com.puppycrawl.tools.checkstyle.ant.CheckstyleAntTask.execute(CheckstyleAntTask.java:292)
        at org.apache.tools.ant.UnknownElement.execute(UnknownElement.java:293)
        at org.apache.tools.ant.dispatch.DispatchUtils.execute(DispatchUtils.java:106)
        at org.gradle.api.internal.project.ant.BasicAntBuilder.nodeCompleted(BasicAntBuilder.java:78)
        at org.gradle.internal.metaobject.BeanDynamicObject$MetaClassAdapter.invokeMethod(BeanDynamicObject.java:479)
        at org.gradle.internal.metaobject.BeanDynamicObject.tryInvokeMethod(BeanDynamicObject.java:191)
        at org.gradle.internal.metaobject.AbstractDynamicObject.invokeMethod(AbstractDynamicObject.java:160)
        at org.gradle.api.internal.project.antbuilder.AntBuilderDelegate.nodeCompleted(AntBuilderDelegate.java:118)
        at org.gradle.api.plugins.quality.internal.CheckstyleInvoker$_invoke_closure1.doCall(CheckstyleInvoker.groovy:58)
        at org.gradle.api.internal.ClosureBackedAction.execute(ClosureBackedAction.java:71)
        at org.gradle.api.internal.ClosureBackedAction.execute(ClosureBackedAction.java:53)
        at org.gradle.api.internal.project.antbuilder.DefaultIsolatedAntBuilder$2.execute(DefaultIsolatedAntBuilder.java:152)
        at org.gradle.api.internal.project.antbuilder.DefaultIsolatedAntBuilder$2.execute(DefaultIsolatedAntBuilder.java:134)
        at org.gradle.api.internal.project.antbuilder.ClassPathToClassLoaderCache.withCachedClassLoader(ClassPathToClassLoaderCache.java:134)
        at org.gradle.api.internal.project.antbuilder.DefaultIsolatedAntBuilder.execute(DefaultIsolatedAntBuilder.java:128)
        at org.gradle.api.internal.project.IsolatedAntBuilder$execute$0.call(Unknown Source)
        at org.gradle.api.plugins.quality.internal.CheckstyleInvoker.invoke(CheckstyleInvoker.groovy:51)
        at org.gradle.api.plugins.quality.Checkstyle.run(Checkstyle.java:154)
        at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:73)
        at org.gradle.api.internal.project.taskfactory.StandardTaskAction.doExecute(StandardTaskAction.java:46)
        at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:39)
        at org.gradle.api.internal.project.taskfactory.StandardTaskAction.execute(StandardTaskAction.java:26)
        at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:801)
        at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:768)
        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter$1.run(ExecuteActionsTaskExecuter.java:131)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:300)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:292)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:174)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:90)
        at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeAction(ExecuteActionsTaskExecuter.java:120)
        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:99)
        ... 31 more
Caused by: com.puppycrawl.tools.checkstyle.api.CheckstyleException: cannot initialize module TreeWalker - Property 'fileExtensions' in module RegexpSinglelineJava does not exist, please check the documentation
        at com.puppycrawl.tools.checkstyle.Checker.setupChild(Checker.java:168)
        at com.puppycrawl.tools.checkstyle.api.AutomaticBean.configure(AutomaticBean.java:137)
        at com.puppycrawl.tools.checkstyle.ant.CheckstyleAntTask.createChecker(CheckstyleAntTask.java:402)
        ... 64 more
Caused by: com.puppycrawl.tools.checkstyle.api.CheckstyleException: Property 'fileExtensions' in module RegexpSinglelineJava does not exist, please check the documentation
        at com.puppycrawl.tools.checkstyle.api.AutomaticBean.tryCopyProperty(AutomaticBean.java:164)
        at com.puppycrawl.tools.checkstyle.api.AutomaticBean.configure(AutomaticBean.java:130)
        at com.puppycrawl.tools.checkstyle.TreeWalker.setupChild(TreeWalker.java:183)
        at com.puppycrawl.tools.checkstyle.api.AutomaticBean.configure(AutomaticBean.java:137)
        at com.puppycrawl.tools.checkstyle.Checker.setupChild(Checker.java:163)
        ... 66 more


* Get more help at https://help.gradle.org

BUILD FAILED in 1s
1 actionable task: 1 executed
{% endhighlight %}

The information you need to check is on `Caused by` lines. Others are just about the call stack and not useful. So, filtering helps you to find the problem. Thus, use the below command if you have a checkstyle configuration error.

{% highlight bash %}
$ ./gradlew checkstyle --stacktrace 2>&1 | grep Caused
Caused by: : Unable to create a Checker: configLocation {/Users/sangsoo/git/static-import-checker/config/checkstyle/checkstyle.xml}, classpath {null}.
Caused by: com.puppycrawl.tools.checkstyle.api.CheckstyleException: cannot initialize module TreeWalker - Property 'fileExtensions' in module RegexpSinglelineJava does not exist, please check the documentation
Caused by: com.puppycrawl.tools.checkstyle.api.CheckstyleException: Property 'fileExtensions' in module RegexpSinglelineJava does not exist, please check the documentation
{% endhighlight %}

> `2>&1` is to redirect error messages to standard output so that `grep` can filter the output.
