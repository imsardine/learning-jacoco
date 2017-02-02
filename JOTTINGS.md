# 雜記

TBD:

疑問：

 * JaCoCo 的運作原理?? Agent 的作用是什麼?? Agent 好像有兩種解釋，一種是 Java 的 instrumentation agent，另一種則是 offline-instrumentation 後要放在 class path 裡的 `agent.jar` ?? ... 但好像都是 agent
 * Jenkins plugin 的 Coverage column 是什麼??
 * app/build/outputs/code-coverage/connected/coverage.ec 跑兩個 device 為什麼還是只有一支 .ec 不像 Spoon 會做 merge??
 * target/site/jacoco/org.jacoco.examples.parser/Main.java.html 為什麼 for loop 也會被標示為 branch??
 * line coverage 也很難懂，如果把多個 statement 寫成一行呢?? 看來最好的方式是不要把太多的邏輯寫在同一行
 * 怎麼從 commandl line 產生 execution data 及 coverage report??
 * JaCoCo 要怎麼唸? => jay-co-co
 * `.ec` 跟 `.exec` 的不同??
 * execution data 的格式
 * coverage report 怎麼讀?? 各式 branch、statement coverage 的差別??
   ** M 是 missed，而 C 是 covered??
 * branch coverage?? 用簡單的例子證實
 * 裡面只有 static method 的 Util class，為什麼會有 `Util()` instruction M: 3 C: 0，用 `javap` 看得出來嗎??
 * inner class 都有一個 `{...}`，這指的是 static initializer 嗎??
 * 把多個 statement 寫成一行，coverage 的表現會打折??
 * 有些 Java interface 有 static initializer 也會被視為 executable class??

課程安排：

 * 先用官方的 example 確認可以執行得起來，說明怎麼閱讀 report，再說明背後的原理。
 * 用 Java Hello World 帶觀念 (command line) - execution data、report 怎麼看；怎麼補測試，提高 coverage ...
 * 用 Maven plugin 來帶 code coverage 怎麼看? 說明 plugin 怎麼用法 (Jenkins plugin 不同參數參數的影響)
 * 把 interface 及 mocking 引進來，會是什麼樣子?
 * 講到 "Path to exec files" 時，可以合併多個裝置的 execution data，可以提一下合併 unit test 與 integration test 的想法...

## Android

 * Release build 似乎就蒐集不到 coverage?? 至少包進 APK 的就不是 intrumented 的版本。
 * 在某次 debug build 發生 crash 之前，看到這個 warning，為何會有人想要開啟 `/jacoco.exec` ?? 會不會 enable coverage 的 debug build 都會這樣??
   * Offline instrumentation for Android End With Crash · Issue #202 · jacoco/jacoco https://github.com/jacoco/jacoco/issues/202 似乎有機會轉向其他地方，但不是應該 `-e coverage true` 時才會作用??

    ```
01-17 16:19:28.213 W/System.err(31053): java.io.FileNotFoundException: /jacoco.exec: open failed: EROFS (Read-only file system)
01-17 16:19:28.217 V/AudioFlinger(  326): presentationComplete() mPresentationCompleteFrames 5280 framesWritten 5760
01-17 16:19:28.220 W/System.err(31053):     at libcore.io.IoBridge.open(IoBridge.java:456)
01-17 16:19:28.220 W/System.err(31053):     at java.io.FileOutputStream.<init>(FileOutputStream.java:87)
01-17 16:19:28.220 W/System.err(31053):     at org.jacoco.agent.rt.internal_14f7ee5.output.FileOutput.openFile(FileOutput.java:67)
01-17 16:19:28.220 W/System.err(31053):     at org.jacoco.agent.rt.internal_14f7ee5.output.FileOutput.startup(FileOutput.java:49)
01-17 16:19:28.220 W/System.err(31053):     at org.jacoco.agent.rt.internal_14f7ee5.Agent.startup(Agent.java:122)
01-17 16:19:28.220 W/System.err(31053):     at org.jacoco.agent.rt.internal_14f7ee5.Agent.getInstance(Agent.java:50)
01-17 16:19:28.220 W/System.err(31053):     at org.jacoco.agent.rt.internal_14f7ee5.Offline.<clinit>(Offline.java:31)
01-17 16:19:28.220 W/System.err(31053):     at com.example.MyApplication.$jacocoInit(MyApplication.java)
01-17 16:19:28.220 W/System.err(31053):     at com.example.MyApplication.<clinit>(MyApplication.java)
01-17 16:19:28.220 W/System.err(31053):     at java.lang.reflect.Constructor.newInstance(Native Method)
01-17 16:19:28.220 W/System.err(31053):     at java.lang.Class.newInstance(Class.java:1572)
01-17 16:19:28.220 W/System.err(31053):     at android.app.Instrumentation.newApplication(Instrumentation.java:994)
01-17 16:19:28.220 W/System.err(31053):     at android.app.Instrumentation.newApplication(Instrumentation.java:979)
01-17 16:19:28.220 W/System.err(31053):     at android.app.LoadedApk.makeApplication(LoadedApk.java:560)
01-17 16:19:28.220 W/System.err(31053):     at android.app.ActivityThread.handleBindApplication(ActivityThread.java:4539)
01-17 16:19:28.220 W/System.err(31053):     at android.app.ActivityThread.access$1500(ActivityThread.java:148)
01-17 16:19:28.220 W/System.err(31053):     at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1344)
01-17 16:19:28.220 W/System.err(31053):     at android.os.Handler.dispatchMessage(Handler.java:102)
01-17 16:19:28.220 W/System.err(31053):     at android.os.Looper.loop(Looper.java:135)
01-17 16:19:28.220 W/System.err(31053):     at android.app.ActivityThread.main(ActivityThread.java:5272)
01-17 16:19:28.220 W/System.err(31053):     at java.lang.reflect.Method.invoke(Native Method)
01-17 16:19:28.220 W/System.err(31053):     at java.lang.reflect.Method.invoke(Method.java:372)
01-17 16:19:28.220 W/System.err(31053):     at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:909)
01-17 16:19:28.220 W/System.err(31053):     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:704)
01-17 16:19:28.221 W/System.err(31053): Caused by: android.system.ErrnoException: open failed: EROFS (Read-only file system)
01-17 16:19:28.221 W/System.err(31053):     at libcore.io.Posix.open(Native Method)
01-17 16:19:28.221 W/System.err(31053):     at libcore.io.BlockGuardOs.open(BlockGuardOs.java:186)
01-17 16:19:28.221 W/System.err(31053):     at libcore.io.IoBridge.open(IoBridge.java:442)
01-17 16:19:28.221 W/System.err(31053):     ... 23 more
    ```

