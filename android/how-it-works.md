# 運作的原理

雖然 on-the-fly instrumentation 是 JaCoCo 推薦的用法，但因為程式碼最後是執行在 Android Dalvik VM 上，中間會經過一層轉換，所以要改採 offline instrumentation，也就是事先對 class file 加工 (pre-instrumentation)，這樣程式在 Android 上運行時才會自動蒐集 execution data。

> One of the main benefits of JaCoCo is the Java agent, which instruments classes on-the-fly. This simplifies code coverage analysis a lot as no pre-instrumentation and classpath tweaking is required. However, there can be situations where on-the-fly instrumentation is not suitable, for example:
> 
> * Runtime environments that do not support Java agents.
> * Deployments where it is not possible to configure JVM options.
> * **Bytecode needs to be converted for another VM like the Android Dalvik VM.**
> * Conflicts with other agents that do dynamic classfile transformation.
>
> -- [JaCoCo - Offline Instrumentation](http://www.eclemma.org/jacoco/trunk/doc/offline.html)

對 offline instrumentation 有基本的認識後，可以透過 debug log 觀察 JaCoCo 是如何被整合進 Android Plugin for Gradle 的：

```
$ ./gradlew --debug connectedAndroidTest
...
16:39:10.373 [DEBUG] [org.gradle.api.internal.artifacts.ivyservice.resolveengine.graph.DependencyGraphBuilder] Visiting dependency com.android.tools.build:gradle-core:2.1.2(runtime) -> org.jacoco:org.jacoco.core:0.7.4.201502262128(runtime)
...
16:39:15.547 [DEBUG] [com.android.build.gradle.internal.pipeline.TransformManager]      InputStream: OriginalStream{jarFiles=[/Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/intermediates/jacoco/jacocoagent.jar], folders=[], scopes=[EXTERNAL_LIBRARIES], contentTypes=[RESOURCES], dependencies=[task ':app:unzipJacocoAgent']}
...
16:39:15.774 [DEBUG] [com.android.build.gradle.internal.pipeline.TransformManager] ADDED TRANSFORM(debug):
16:39:15.774 [DEBUG] [com.android.build.gradle.internal.pipeline.TransformManager]      Name: jacoco
16:39:15.774 [DEBUG] [com.android.build.gradle.internal.pipeline.TransformManager]      Task: transformClassesWithJacocoForDebug
16:39:15.775 [DEBUG] [com.android.build.gradle.internal.pipeline.TransformManager]      InputStream: OriginalStream{jarFiles=[], folders=[/Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/intermediates/classes/debug], scopes=[PROJECT], contentTypes=[CLASSES], dependencies=[compileDebugJavaWithJavac]} (1)
16:39:15.775 [DEBUG] [com.android.build.gradle.internal.pipeline.TransformManager]      OutputStream: IntermediateStream{rootLocation=/Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/intermediates/transforms/jacoco/debug, scopes=[PROJECT], contentTypes=[CLASSES], dependencies=[transformClassesWithJacocoForDebug]}
...
16:39:19.349 [INFO] [org.gradle.BuildLogger] Tasks to be executed: [... task ':app:unzipJacocoAgent', task ':app:transformClassesWithJacocoForDebug', ... task ':app:packageDebugAndroidTest', task ':app:assembleDebugAndroidTest', task ':app:connectedDebugAndroidTest', task ':app:createDebugAndroidTestCoverageReport', task ':app:connectedAndroidTest']
...
16:39:47.214 [DEBUG] [org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter] Executing actions for task ':app:connectedDebugAndroidTest'. (2)
16:39:48.822 [DEBUG] [org.gradle.api.Task] DeviceConnector 'AVD_for_Nexus_6_by_Google(AVD) - 6.0': installing /Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/outputs/apk/app-debug.apk
16:39:51.245 [DEBUG] [org.gradle.api.Task] DeviceConnector 'AVD_for_Nexus_6_by_Google(AVD) - 6.0': installing /Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/outputs/apk/app-debug-androidTest-unaligned.apk
16:39:54.659 [INFO] [org.gradle.api.Task] Starting 2 tests on AVD_for_Nexus_6_by_Google(AVD) - 6.0
...
16:40:07.634 [DEBUG] [org.gradle.api.Task] DeviceConnector 'AVD_for_Nexus_6_by_Google(AVD) - 6.0': fetching coverage data from /data/data/com.example.android.testing.espresso.BasicSample/coverage.ec (2)

16:40:08.861 [DEBUG] [org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter] Executing actions for task ':app:createDebugAndroidTestCoverageReport'. (3)

16:40:09.056 [DEBUG] [org.gradle.api.internal.project.ant.AntLoggingAdapter] Finding class org.jacoco.ant.ReportTask
16:40:09.065 [DEBUG] [org.gradle.api.internal.project.ant.AntLoggingAdapter] Loaded from /Users/jeremykao/.gradle/caches/modules-2/files-2.1/org.jacoco/org.jacoco.ant/0.7.4.201502262128/e8808120e50c1f2e830ff26cbfacbf3f018441b7/org.jacoco.ant-0.7.4.201502262128.jar org/jacoco/ant/ReportTask.class

16:40:09.740 [DEBUG] [org.gradle.api.internal.project.ant.AntLoggingAdapter] fileset: Setup scanner in dir /Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/intermediates/classes/debug with patternSet{ includes: [] excludes: [**/R.class, **/R$*.class, **/Manifest.class, **/Manifest$*.class, **/BuildConfig.class] }

16:40:09.376 [INFO] [org.gradle.api.internal.project.ant.AntLoggingAdapter] [ant:reportWithJacoco] Loading execution data file /Users/jeremykao/work/android-testing/ui/espresso/BasicSample/app/build/outputs/code-coverage/connected/coverage.ec
```

 1. 過程中 `transformClassesWithJacocoFor[{Flavor}]Debug` task 會先對 `app/build/intermediates/classes/[{flavor}/]debug/**.class` 加工，並將結果輸出到 `app/build/intermediates/transforms/jacoco/[{flavor}/]debug`。
 2. 把 main APK 與 test APK 送進裝置執行測試，結束後將 execution data (`/data/data/{package-name}/coverage.ec`) 取出到 `app/build/outputs/code-coverage/connected/[flavors/{flavor}/]coverage.ec`。
 3. `create[{Flavor}]DebugAndroidTestCoverageReport` 利用 JaCoCo 的 Ant task (`org.jacoco.ant.ReportTask`) 搭配 `app/build/intermediates/classes/[{flavor}/]debug/**.class` (排除其些 class file) 產生 coverage report。


> <i class="fa fa-lightbulb-o fa-3x"></i>
> 不知道為什麼 Android 習慣把 execution data 命名成 `.ec` 而不是慣用的 `.exec`，不過這兩者格式是一樣的。

上面這些觀察，在需要將 coverage report 整合進 Jenkins 時尤其重要：

 * 產生 coverage report 時，要提供沒有加工過的版本，這一點在 JaCoCo 官方文件有特別提醒。

    > Based on the collected `*.exec` files reports can be created the same way as for execution data collected with the Java agent. **Note that for report generation the original class files have to be supplied, not the instrumented copies.**
    >
    > -- [JaCoCo - Offline Instrumentation](http://www.eclemma.org/jacoco/trunk/doc/offline.html)

    否則會遇到下面的錯誤：

    ```
java.lang.IllegalStateException: Class xxx is already instrumented.
    ```

 * 不想讓自動產生的程式碼也出現在 coverage report 裡，只要將下面這些 class file 排除即可：

    ```
**/R.class
**/R$*.class
**/Manifest.class
**/Manifest$*.class
**/BuildConfig.class
    ```

