# Execution Data (`.exec`) 的版本

測試過程中蒐集到的數據 (execution data) 會被寫到 `.exec` (或 `.ec`) 檔，可能跟其他 `.exec` 檔合併之後，再產生 coverage report。

`.exec` 是個二進位檔，主要做為 JaCoCo agent 跟 report generator 間的中介檔。由於 `.exec` 隨著 JaCoCo 版本的演進，採用的檔案格式 (format/exec version) 可能產生不相容的情形。

首先，要知道如何判斷 `.exec` 的版本，答案就在第 4 ~ 5 兩個 byte，例如：

```
$ hexdump -n 5 path/to/coverage.ec
0000000 01 c0 c0 10 06
0000005 
```

所有的 `.exec` 檔，固定以 `0x01 0xc0 0xc0` (magic number) 開頭，接下來的兩個 byte 就是檔案格式的版本。

[`org/jacoco/core/data/ExecutionDataWriter.java`](https://github.com/jacoco/jacoco/blob/master/org.jacoco.core/src/org/jacoco/core/data/ExecutionDataWriter.java)

```java
/** File format version, will be incremented for each incompatible change. */
public static final char FORMAT_VERSION = 0x1007;

/**
 * Returns the first bytes of a file that represents a valid execution data
 * file. In any case every execution data file starts with the three bytes
 * <code>0x01 0xC0 0xC0</code>.
 *
 * @return first bytes of a execution data file
 */
public static final byte[] getFileHeader() {
    final ByteArrayOutputStream buffer = new ByteArrayOutputStream();
    try {
        new ExecutionDataWriter(buffer);
    } catch (final IOException e) {
        // Must not happen with ByteArrayOutputStream
        throw new AssertionError(e);
    }
    return buffer.toByteArray();
}
```

就上面的例子，exec version 是 `0x1006`，可以從官方文件 [ExecFileVersions](https://github.com/jacoco/jacoco/wiki/ExecFileVersions) 查出有哪些 JaCoCo 版本採用這個 exec version。例如：

 * `0x1007` - JaCoCo 0.7.5+
 * `0x1006` - JaCoCo 0.5.x - 0.7.4

顯然 JaCoCo 0.7.5 所採用的 exec version 不相容於 JaCoCo 0.7.4 以前的版本。

> Release 0.7.5 (2015/05/24)
> 
> The exec file version has been updated and is not compatible with previous versions.
>
> -- [JaCoCo - Change History](http://www.eclemma.org/jacoco/trunk/doc/changes.html)

所以如果拿 JaCoCo 0.7.5 的工具來處理 JaCoCo 0.7.4 產生的 `.exec`，就會發生下面的錯誤：

```
org.jacoco.core.data.IncompatibleExecDataVersionException: Cannot read execution data version 0x1006. This version of JaCoCo uses execution data version 0x1007.
```

要注意的是，只要 exec version 版本不同，就是不相容，而特定版本的 JaCoCo 也只能處理特定 exec version 的檔案，並不是新版的 JaCoCo 就能處理所有舊的 exec version，因此選用相容版本的工具就很重要。

[`org/jacoco/core/data/ExecutionDataReader.java`](https://github.com/jacoco/jacoco/blob/master/org.jacoco.core/src/org/jacoco/core/data/ExecutionDataReader.java)

```java
private void readHeader() throws IOException {
    if (in.readChar() != ExecutionDataWriter.MAGIC_NUMBER) { // (1)
        throw new IOException("Invalid execution data file.");
    }
    final char version = in.readChar();
    if (version != ExecutionDataWriter.FORMAT_VERSION) {     // (2)
        throw new IncompatibleExecDataVersionException(version);
    }
}
```

 1. Exec file 不是以 `0x01 0xc0 0xc0` (magic number) 開頭，就視為無效的檔案。
 2. Exec version 與這個 JaCoCo 版本採用的版本不同，就視為不相容。

