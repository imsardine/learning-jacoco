# 新手上路

官方有提供一個範例，說明如何將 JaCoCo 整合到 build process 裡 (使用 Ant 或 Maven)，我們就先透過這個範例來體驗一下 JaCoCo 套用到專案上是什麼樣子。

首先從 http://www.eclemma.org/jacoco/ 下載 release build (`jacoco-{version}.zip`)，解壓縮後範例就在 `doc/examples/build/` 底下：

```
$ cd doc/examples/build/
$ tree
.
├── build-offline.xml (1)(2)
├── build.xml
├── pom-it.xml
├── pom-offline.xml
├── pom.xml
└── src
    ├── main          (3)
    │   └── java
    │       └── org
    │           └── jacoco
    │               └── examples
    │                   ├── expressions
    │                   │   ├── Add.java
    │                   │   ├── Const.java
    │                   │   ├── Div.java
    │                   │   ├── IExpression.java
    │                   │   ├── Mul.java
    │                   │   └── Sub.java
    │                   └── parser
    │                       ├── ExpressionParser.java
    │                       └── Main.java
    └── test          (3)
        └── java
            └── org
                └── jacoco
                    └── examples
                        └── parser
                            ├── ExpressionParserIT.java
                            └── ExpressionParserTest.java

14 directories, 15 files
```

 1.  `build*.xml` 是搭配 Ant 使用，而 `pom*.xml` 則是搭配 Maven 使用。
 2. `\*-offline.xml` 採用 offline instrumentation (相對於預設的 on-the-fly instrumentation，之後說明)，而 `*-it.xml` 則是 integration test (相對於 unit test)。
 3. 受測的程式碼都在 `src/main/java` 底下，測試程式則在 `src/test/java` 底下。

這裡用 Maven 與 JaCoCo 0.7.7.201606060606 做說明。執行 `mvn verify` 就可以進行測試，並產生 coverage report：

```
$ mvn verify
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building JaCoCo Maven plug-in example 0.7.7.201606060606
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- jacoco-maven-plugin:0.7.7.201606060606:prepare-agent (default-prepare-agent) @ org.jacoco.examples.maven --- (1)
[INFO] argLine set to -javaagent:/Users/jeremykao/.m2/repository/org/jacoco/org.jacoco.agent/0.7.7.201606060606/org.jacoco.agent-0.7.7.201606060606-runtime.jar=destfile=/Users/jeremykao/work/jacoco-0.7.7.201606060606/doc/examples/build/target/jacoco.exec
[INFO]
...
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ org.jacoco.examples.maven ---
[INFO] Surefire report directory: /Users/jeremykao/work/jacoco-0.7.7.201606060606/doc/examples/build/target/surefire-reports
...
Running org.jacoco.examples.parser.ExpressionParserTest
Tests run: 6, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.138 sec

Results :

Tests run: 6, Failures: 0, Errors: 0, Skipped: 0

[INFO]
[INFO] --- jacoco-maven-plugin:0.7.7.201606060606:report (default-report) @ org.jacoco.examples.maven --- (2)
[INFO] Loading execution data file /Users/jeremykao/work/jacoco-0.7.7.201606060606/doc/examples/build/target/jacoco.exec
[INFO] Analyzed bundle 'JaCoCo Maven plug-in example' with 7 classes
[INFO]
...
[INFO] --- jacoco-maven-plugin:0.7.7.201606060606:check (default-check) @ org.jacoco.examples.maven --- (3)
[INFO] Loading execution data file /Users/jeremykao/work/jacoco-0.7.7.201606060606/doc/examples/build/target/jacoco.exec
[INFO] Analyzed bundle 'org.jacoco.examples.maven' with 7 classes
[INFO] All coverage checks have been met.
```

 1. 一開始 `jacoco-maven-plugin:prepare-agent` 做一些準備工作，讓 JaCoCo 可以在接下來 `maven-surefire-plugin:test` 執行單元測試期間產生作用，並將蒐集到的數據 (execution data) 記錄到 `target/jacoco.exec`。(細節之後說明)
 2. 測試結束後，`jacoco-maven-plugin:report` 分析 execution data 並產生 coverage report 在 `target/site/jacoco/`。
 3. 最後 `jacoco-maven-plugin:check` 檢查 coverage 是否達到要求。

接著來看 coverage report 裡的數據要怎麼判讀。

