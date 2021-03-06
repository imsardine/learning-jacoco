# JaCoCo 與 Android

官方文件 [New Build System](http://tools.android.com/tech-docs/new-build-system) 提到，Android Plugin for Gradle 從 0.10.0 版開始支援用 JaCoCo 做 code coverage，只要在 build type 裡宣告 `testCoverageEnabled = true` 即可。例如：

```
android {
    buildTypes {
        debug {
            testCoverageEnabled true
        }
    }
}
```

以 [`googlesamples/android-testings`](https://github.com/googlesamples/android-testing/tree/master/ui/espresso) 的 `BasicSample` 為例，在宣告 `testCoverageEnabled` 之前，沒有 coverage 相關的 task：

```
$ git clone https://github.com/googlesamples/android-testing.git
$ cd android-testing/ui/espresso/BasicSample
$ ./gradlew -q tasks --all
...
app:connectedAndroidTest - Installs and runs instrumentation tests for all flavors on connected devices. [app:connectedDebugAndroidTest]
app:connectedCheck - Runs all device checks on currently connected devices. [app:connectedAndroidTest]
app:connectedDebugAndroidTest - Installs and runs the tests for debug on connected devices. [app:assembleDebug, app:compileDebugAndroidTestSources]
    app:assembleDebugAndroidTest
    app:mergeDebugAndroidTestJniLibFolders
    app:packageDebugAndroidTest
    app:prePackageMarkerForDebugAndroidTest
    app:processDebugAndroidTestJavaRes
    app:transformClassesWithDexForDebugAndroidTest
    app:transformNative_libsWithMergeJniLibsForDebugAndroidTest
    app:transformResourcesWithMergeJavaResForDebugAndroidTest
```

宣告 `testCoverageEnabled true` 後，就多了 `create[{Flavor}]DebugCoverageReport` task 可以用 (其中 _Flavor_ 只在有宣告 product flavor 時才會出現)，同時 `connectedAndroidTest` task 也會呼叫 `create[{Flavor}]DebugAndroidTestCoverageReport` task 產生 coverage report：

```
$ ./gradlew -q tasks --all
...
app:connectedAndroidTest - Installs and runs instrumentation tests for all flavors on connected devices. [app:connectedDebugAndroidTest]
    app:createDebugAndroidTestCoverageReport - Creates JaCoCo test coverage report from data gathered on the device.
app:connectedCheck - Runs all device checks on currently connected devices. [app:connectedAndroidTest]
app:connectedDebugAndroidTest - Installs and runs the tests for debug on connected devices. [app:assembleDebug, app:compileDebugAndroidTestSources]
    app:assembleDebugAndroidTest
    app:mergeDebugAndroidTestJniLibFolders
    app:packageDebugAndroidTest
    app:prePackageMarkerForDebugAndroidTest
    app:processDebugAndroidTestJavaRes
    app:transformClassesWithDexForDebugAndroidTest
    app:transformNative_libsWithMergeJniLibsForDebugAndroidTest
    app:transformResourcesWithMergeJavaResForDebugAndroidTest
app:createDebugCoverageReport - Creates test coverage reports for the debug variant. [app:connectedDebugAndroidTest]
    app:createDebugAndroidTestCoverageReport - Creates JaCoCo test coverage report from data gathered on the device.
```

執行 `./gradlew connectedAndroidTest`，就會自動產生 coverage report 在 `app/build/reports/coverage/[{flavor}/]debug/`。例如：

```
$ ./gradlew connectedAndroidTest
$ open app/build/reports/coverage/debug/index.html
```

![](../images/google-espress-sample-coverage.png)

不同版本的 Android Plugin for Gradle，背後整合的 JaCoCo 版本可能不同，可以從 [Maven Central Repository](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.android.tools.build%22%20AND%20a%3A%22gradle-core%22) 查看 `com.android.tools.build:gradle-core` 的 Project Object Model (POM)，也可以用下面的方式在本地端查詢：

```
$ ./gradlew -q :app:dependencies --configuration androidJacocoAgent
...
androidJacocoAgent - The Jacoco agent to use to get coverage data.
\--- org.jacoco:org.jacoco.agent:0.7.4.201502262128
```

> <i class="fa fa-fire fa-3x"></i>
> 早期有些文件提到，可以用 `android.jacoco.version` 來指定 JaCoCo 的版本。例如：
>
> ``` 
> android {
>     jacoco {
>         version = '0.7.1.201405082137'
>     }
> }
> ```

不過這個做法已經行不通了，官方文件明確指出設定這個 property 不影響 JaCoCo 的版本：

> String version
>
> note: this property is deprecated and will be removed in a future version of the plugin.
> 
> This will not affect the JaCoCo version used.
> 
> -- [JacocoOptions - Android Plugin 2.1.0 DSL Reference](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.coverage.JacocoOptions.html)

由於無法指定 JaCoCo 的版本，若需要與其他工具整合時，知道 plugin 所用的 JaCoCo 版本就很重要，尤其 JaCoCo 0.7.5 變更了 execution data 的檔案格式，跟舊版本不相容。更多細節可以參考附錄 [Execution Dat (`.exec`) 的版本](../exec-file-versions.md)。

