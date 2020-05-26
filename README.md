Trying to use $(location) of a directory causes a crash in merkle tree building if using remote execution, but works in local execution.

Either this should be allowed in both contexts (and work in both contexts), or it should be actively disallowed in both contexts.

In local execution, this works fine:
```
% bazel test :test
INFO: Invocation ID: 0ae97790-f05b-465b-abfa-3de185bb2d12
INFO: Analyzed target //:test (0 packages loaded, 0 targets configured).
INFO: Found 1 test target...
Target //:test up-to-date:
  bazel-bin/test
INFO: Elapsed time: 0.107s, Critical Path: 0.00s
INFO: 0 processes.
INFO: Build completed successfully, 1 total action
//:test                                                         (cached) PASSED in 0.2s

Executed 0 out of 1 test: 1 test passes.
INFO: Build completed successfully, 1 total action
```

With remote execution, this crashes Bazel:
```
% bazel test :test
...
ERROR: /path/to/repro-bazel-dir-runfile/BUILD:1:1:  failed due to unexpected I/O exception: /private/var/tmp/_bazel_dwh/624c77b946ad08249c6edfc633b2f65a/execroot/__main__/bazel-out/k8-fastbuild/bin/test.runfiles/__main__/dir/child (No such file or directory)
java.io.FileNotFoundException: /private/var/tmp/_bazel_dwh/624c77b946ad08249c6edfc633b2f65a/execroot/__main__/bazel-out/k8-fastbuild/bin/test.runfiles/__main__/dir/child (No such file or directory)
	at com.google.devtools.build.lib.unix.NativePosixFiles.readdir(Native Method)
	at com.google.devtools.build.lib.unix.NativePosixFiles.readdir(NativePosixFiles.java:283)
	at com.google.devtools.build.lib.unix.UnixFileSystem.readdir(UnixFileSystem.java:160)
	at com.google.devtools.build.lib.vfs.Path.readdir(Path.java:384)
	at com.google.devtools.build.lib.remote.merkletree.DirectoryTreeBuilder.explodeDirectory(DirectoryTreeBuilder.java:217)
	at com.google.devtools.build.lib.remote.merkletree.DirectoryTreeBuilder.explodeDirectory(DirectoryTreeBuilder.java:210)
	at com.google.devtools.build.lib.remote.merkletree.DirectoryTreeBuilder.lambda$buildFromActionInputs$1(DirectoryTreeBuilder.java:149)
	at com.google.devtools.build.lib.remote.merkletree.DirectoryTreeBuilder.build(DirectoryTreeBuilder.java:201)
	at com.google.devtools.build.lib.remote.merkletree.DirectoryTreeBuilder.buildFromActionInputs(DirectoryTreeBuilder.java:123)
	at com.google.devtools.build.lib.remote.merkletree.DirectoryTreeBuilder.fromActionInputs(DirectoryTreeBuilder.java:62)
	at com.google.devtools.build.lib.remote.merkletree.MerkleTree.build(MerkleTree.java:140)
	at com.google.devtools.build.lib.remote.RemoteSpawnRunner.exec(RemoteSpawnRunner.java:212)
	at com.google.devtools.build.lib.exec.SpawnRunner.execAsync(SpawnRunner.java:238)
	at com.google.devtools.build.lib.exec.AbstractSpawnStrategy.exec(AbstractSpawnStrategy.java:126)
	at com.google.devtools.build.lib.exec.AbstractSpawnStrategy.exec(AbstractSpawnStrategy.java:96)
	at com.google.devtools.build.lib.actions.SpawnStrategy.beginExecution(SpawnStrategy.java:39)
	at com.google.devtools.build.lib.exec.ProxySpawnActionContext.beginExecution(ProxySpawnActionContext.java:60)
	at com.google.devtools.build.lib.exec.StandaloneTestStrategy.beginTestAttempt(StandaloneTestStrategy.java:308)
	at com.google.devtools.build.lib.exec.StandaloneTestStrategy.access$100(StandaloneTestStrategy.java:68)
	at com.google.devtools.build.lib.exec.StandaloneTestStrategy$StandaloneTestRunnerSpawn.beginExecution(StandaloneTestStrategy.java:442)
	at com.google.devtools.build.lib.analysis.test.TestRunnerAction.beginIfNotCancelled(TestRunnerAction.java:835)
	at com.google.devtools.build.lib.analysis.test.TestRunnerAction.beginExecution(TestRunnerAction.java:804)
	at com.google.devtools.build.lib.analysis.test.TestRunnerAction.execute(TestRunnerAction.java:860)
	at com.google.devtools.build.lib.analysis.test.TestRunnerAction.execute(TestRunnerAction.java:851)
	at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$4.execute(SkyframeActionExecutor.java:961)
	at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$ActionRunner.continueAction(SkyframeActionExecutor.java:1109)
	at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor$ActionRunner.run(SkyframeActionExecutor.java:1080)
	at com.google.devtools.build.lib.skyframe.ActionExecutionState.runStateMachine(ActionExecutionState.java:137)
	at com.google.devtools.build.lib.skyframe.ActionExecutionState.getResultOrDependOnFuture(ActionExecutionState.java:80)
	at com.google.devtools.build.lib.skyframe.SkyframeActionExecutor.executeAction(SkyframeActionExecutor.java:601)
	at com.google.devtools.build.lib.skyframe.ActionExecutionFunction.checkCacheAndExecuteIfNeeded(ActionExecutionFunction.java:907)
	at com.google.devtools.build.lib.skyframe.ActionExecutionFunction.compute(ActionExecutionFunction.java:297)
	at com.google.devtools.build.skyframe.AbstractParallelEvaluator$Evaluate.run(AbstractParallelEvaluator.java:438)
	at com.google.devtools.build.lib.concurrent.AbstractQueueVisitor$WrappedRunnable.run(AbstractQueueVisitor.java:399)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)

```
