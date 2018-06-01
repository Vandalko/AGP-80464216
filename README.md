# AGP-80464216
Android Gradle Plugin bug https://issuetracker.google.com/issues/80464216

To reproduce, run: ./gradlew integrationTestFor-s -s


When trying to execute another project through `GradleBuild`, I get `IllegalMonitorStateException`.
It can be reproduced during each run inside my current project.
But for some reason I can't reproduce it inside fresh new project - trying to figure out what so special...

```
tasks.addRule("integrationTestFor-<SampleName>") { String taskName ->
  if (taskName.startsWith("integrationTestFor-")) {
    tasks.create(taskName, GradleBuild) {
      dir = file('/home/vandalko/dev/workspace/SampleGradle/app')
      buildFile = new File('/home/vandalko/dev/workspace/SampleGradle/app/build.gradle')
      tasks = ['clean', 'check']
      startParameter.buildCacheEnabled = false
      startParameter.configureOnDemand = false
      startParameter.setParallelProjectExecutionEnabled(false)
      startParameter.projectProperties['android.enableBuildScriptClasspathCheck'] = 'false'
      startParameter.projectProperties['android.enableIncrementalDesugaring'] = 'false'
    }
  }
}
```
Android plugin version 3.1.*
Can be reproduced on Gradle 4.4 - 4.7.
```
------------------------------------------------------------
Gradle 4.7
------------------------------------------------------------

Build time:   2018-04-18 09:09:12 UTC
Revision:     b9a962bf70638332300e7f810689cb2febbd4a6c

Groovy:       2.4.12
Ant:          Apache Ant(TM) version 1.9.9 compiled on February 2 2017
JVM:          1.8.0_171 (Oracle Corporation 25.171-b11)
OS:           Linux 4.15.0-22-generic amd64

```

```
* Exception is:
java.lang.IllegalMonitorStateException: attempt to unlock read lock, not locked by current thread
        at java.util.concurrent.locks.ReentrantReadWriteLock$Sync.unmatchedUnlockException(ReentrantReadWriteLock.java:444)
        at java.util.concurrent.locks.ReentrantReadWriteLock$Sync.tryReleaseShared(ReentrantReadWriteLock.java:428)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.releaseShared(AbstractQueuedSynchronizer.java:1341)
        at java.util.concurrent.locks.ReentrantReadWriteLock$ReadLock.unlock(ReentrantReadWriteLock.java:881)
        at com.android.utils.concurrency.ReadWriteProcessLock.releaseLock(ReadWriteProcessLock.java:338)
        at com.android.utils.concurrency.ReadWriteProcessLock.access$400(ReadWriteProcessLock.java:92)
        at com.android.utils.concurrency.ReadWriteProcessLock$ReadLock.unlock(ReadWriteProcessLock.java:635)
        at com.android.build.gradle.internal.BuildCacheUtils.lambda$null$1(BuildCacheUtils.java:167)
        at java.util.LinkedHashMap$LinkedValues.forEach(LinkedHashMap.java:608)
        at com.android.build.gradle.internal.BuildSessionImpl.buildFinished(BuildSessionImpl.java:202)
        at com.android.build.gradle.internal.BuildSessionImpl.access$000(BuildSessionImpl.java:61)
        at com.android.build.gradle.internal.BuildSessionImpl$1.buildFinished(BuildSessionImpl.java:172)
        at sun.reflect.GeneratedMethodAccessor57.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:35)
        at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:24)
        at org.gradle.internal.event.DefaultListenerManager$ListenerDetails.dispatch(DefaultListenerManager.java:370)
        at org.gradle.internal.event.DefaultListenerManager$ListenerDetails.dispatch(DefaultListenerManager.java:352)
        at org.gradle.internal.event.AbstractBroadcastDispatch.dispatch(AbstractBroadcastDispatch.java:58)
        at org.gradle.internal.event.DefaultListenerManager$EventBroadcast$ListenerDispatch.dispatch(DefaultListenerManager.java:340)
        at org.gradle.internal.event.DefaultListenerManager$EventBroadcast$ListenerDispatch.dispatch(DefaultListenerManager.java:327)
        at org.gradle.internal.event.AbstractBroadcastDispatch.dispatch(AbstractBroadcastDispatch.java:42)
        at org.gradle.internal.event.BroadcastDispatch$SingletonDispatch.dispatch(BroadcastDispatch.java:230)
        at org.gradle.internal.event.BroadcastDispatch$SingletonDispatch.dispatch(BroadcastDispatch.java:149)
        at org.gradle.internal.event.AbstractBroadcastDispatch.dispatch(AbstractBroadcastDispatch.java:58)
        at org.gradle.internal.event.BroadcastDispatch$CompositeDispatch.dispatch(BroadcastDispatch.java:324)
        at org.gradle.internal.event.BroadcastDispatch$CompositeDispatch.dispatch(BroadcastDispatch.java:234)
        at org.gradle.internal.event.ListenerBroadcast.dispatch(ListenerBroadcast.java:140)
        at org.gradle.internal.event.ListenerBroadcast.dispatch(ListenerBroadcast.java:37)
        at org.gradle.internal.dispatch.ProxyDispatchAdapter$DispatchingInvocationHandler.invoke(ProxyDispatchAdapter.java:93)
        at com.sun.proxy.$Proxy15.buildFinished(Unknown Source)
        at org.gradle.initialization.DefaultGradleLauncher.finishBuild(DefaultGradleLauncher.java:154)
        at org.gradle.initialization.DefaultGradleLauncher.finishBuild(DefaultGradleLauncher.java:122)
        at org.gradle.initialization.DefaultGradleLauncher.doBuildStages(DefaultGradleLauncher.java:141)
        at org.gradle.initialization.DefaultGradleLauncher.executeTasks(DefaultGradleLauncher.java:115)
        at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:78)
        at org.gradle.internal.invocation.GradleBuildController$1.call(GradleBuildController.java:75)
        at org.gradle.internal.work.DefaultWorkerLeaseService.withLocks(DefaultWorkerLeaseService.java:152)
        at org.gradle.internal.work.StopShieldingWorkerLeaseService.withLocks(StopShieldingWorkerLeaseService.java:38)
        at org.gradle.internal.invocation.GradleBuildController.doBuild(GradleBuildController.java:100)
        at org.gradle.internal.invocation.GradleBuildController.run(GradleBuildController.java:75)
        at org.gradle.tooling.internal.provider.ExecuteBuildActionRunner.run(ExecuteBuildActionRunner.java:28)
        at org.gradle.launcher.exec.ChainingBuildActionRunner.run(ChainingBuildActionRunner.java:35)
        at org.gradle.tooling.internal.provider.ValidatingBuildActionRunner.run(ValidatingBuildActionRunner.java:32)
        at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner$3.run(RunAsBuildOperationBuildActionRunner.java:45)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:317)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor$RunnableBuildOperationWorker.execute(DefaultBuildOperationExecutor.java:309)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor.execute(DefaultBuildOperationExecutor.java:185)
        at org.gradle.internal.operations.DefaultBuildOperationExecutor.run(DefaultBuildOperationExecutor.java:97)
        at org.gradle.internal.operations.DelegatingBuildOperationExecutor.run(DelegatingBuildOperationExecutor.java:31)
        at org.gradle.launcher.exec.RunAsBuildOperationBuildActionRunner.run(RunAsBuildOperationBuildActionRunner.java:42)
        at org.gradle.tooling.internal.provider.SubscribableBuildActionRunner.run(SubscribableBuildActionRunner.java:51)
        at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:47)
        at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:31)
        at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:39)
        at org.gradle.launcher.exec.BuildTreeScopeBuildActionExecuter.execute(BuildTreeScopeBuildActionExecuter.java:25)
        at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:80)
        at org.gradle.tooling.internal.provider.ContinuousBuildActionExecuter.execute(ContinuousBuildActionExecuter.java:53)
        at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:61)
        at org.gradle.tooling.internal.provider.ServicesSetupBuildActionExecuter.execute(ServicesSetupBuildActionExecuter.java:34)
        at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:36)
        at org.gradle.tooling.internal.provider.GradleThreadBuildActionExecuter.execute(GradleThreadBuildActionExecuter.java:25)
        at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:43)
        at org.gradle.tooling.internal.provider.ParallelismConfigurationBuildActionExecuter.execute(ParallelismConfigurationBuildActionExecuter.java:29)
        at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:64)
        at org.gradle.tooling.internal.provider.StartParamsValidatingActionExecuter.execute(StartParamsValidatingActionExecuter.java:29)
        at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:59)
        at org.gradle.tooling.internal.provider.SessionFailureReportingActionExecuter.execute(SessionFailureReportingActionExecuter.java:44)
        at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:46)
        at org.gradle.tooling.internal.provider.SetupLoggingActionExecuter.execute(SetupLoggingActionExecuter.java:30)
        at org.gradle.launcher.daemon.server.exec.ExecuteBuild.doBuild(ExecuteBuild.java:67)
        at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.WatchForDisconnection.execute(WatchForDisconnection.java:37)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.ResetDeprecationLogger.execute(ResetDeprecationLogger.java:26)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.RequestStopIfSingleUsedDaemon.execute(RequestStopIfSingleUsedDaemon.java:34)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:74)
        at org.gradle.launcher.daemon.server.exec.ForwardClientInput$2.call(ForwardClientInput.java:72)
        at org.gradle.util.Swapper.swap(Swapper.java:38)
        at org.gradle.launcher.daemon.server.exec.ForwardClientInput.execute(ForwardClientInput.java:72)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.LogAndCheckHealth.execute(LogAndCheckHealth.java:50)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.LogToClient.doBuild(LogToClient.java:62)
        at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.EstablishBuildEnvironment.doBuild(EstablishBuildEnvironment.java:82)
        at org.gradle.launcher.daemon.server.exec.BuildCommandOnly.execute(BuildCommandOnly.java:36)
        at org.gradle.launcher.daemon.server.api.DaemonCommandExecution.proceed(DaemonCommandExecution.java:122)
        at org.gradle.launcher.daemon.server.exec.StartBuildOrRespondWithBusy$1.run(StartBuildOrRespondWithBusy.java:50)
        at org.gradle.launcher.daemon.server.DaemonStateCoordinator$1.run(DaemonStateCoordinator.java:295)
        at org.gradle.internal.concurrent.ExecutorPolicy$CatchAndRecordFailures.onExecute(ExecutorPolicy.java:63)
        at org.gradle.internal.concurrent.ManagedExecutorImpl$1.run(ManagedExecutorImpl.java:46)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at org.gradle.internal.concurrent.ThreadFactoryImpl$ManagedThreadRunnable.run(ThreadFactoryImpl.java:55)
        at java.lang.Thread.run(Thread.java:748)
```
