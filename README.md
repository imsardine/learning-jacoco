# JaCoCo 學習筆記

JaCoCo 是用來量測 Java 代碼覆蓋率 (code coverage) 的開源工具。

在開發 [EclEmma](http://www.eclemma.org/) (給 Eclipse 用的代碼覆蓋率工具) 的過程中，團隊觀察到雖然已經有 Java 的 coverage 技術 (當時有 [EMMA](http://emma.sourceforge.net/) 跟 [Cobertura](http://cobertura.github.io/cobertura/))，但都沒有把與其他工具整合的彈性考量進來 (只能搭配特定的開發工具使用)，後來這些工具的開發工作趨緩，也不支援最新的 Java 版本，於是 EclEmma 開發團隊決定另起一個 JaCoCo 專案。

JaCoCo 把自己定位成 JVM 上 code coverage analysis 的標準技術，並專注在提供方便與其他開發工具整合的 library。JaCoCo 本身直接提供 Ant tasks 與 Maven plugin (但沒有 command line 工具)，也做為與其他開發工具整合的參考範例，Gradle 的 [JaCoCo plugin](https://docs.gradle.org/current/userguide/jacoco_plugin.html) 就是第三方工具整合 JaCoCo 的實例。 

> <i class="fa fa-lightbulb-o fa-3x"></i>
> [EclEmma 早期的版本是基於 EMMA](http://www.eclemma.org/installation1x.html) (從 "Ecl + Emma" 這名稱也可以略知一二)，但從 EclEmma 2.0 開始就改用 JaCoCo，所以 EclEmma 也是 JaCoCo 整合開發工具的範例之一。

