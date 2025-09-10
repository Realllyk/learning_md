
在 Java 中，文件输入输出（I/O）操作主要通过以下类来实现：​

1. **FileInputStream 和 FileOutputStream**：​用于以字节流的方式读取和写入文件，适合处理二进制数据，如图片、音频等。​
2. **FileReader 和 FileWriter**：​用于以字符流的方式读取和写入文件，适合处理文本数据。​
3. **BufferedReader 和 BufferedWriter**：​为字符流提供缓冲功能，提高读取和写入效率。​
4. **Files 工具类（Java 7 引入的 NIO.2）**：​提供了静态方法来简化文件的读取和写入操作。​

以下是每种方式的示例代码：
**1. 使用 FileInputStream 和 FileOutputStream 进行字节流操作**
```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class ByteStreamExample {
    public static void main(String[] args) {
        try (FileInputStream fis = new FileInputStream("input.dat");
             FileOutputStream fos = new FileOutputStream("output.dat")) {
            byte[] buffer = new byte[1024];
            int bytesRead;
            while ((bytesRead = fis.read(buffer)) != -1) {
                fos.write(buffer, 0, bytesRead);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

在上述示例中，`FileInputStream` 用于从 `input.dat` 文件中读取字节数据，`FileOutputStream` 用于将数据写入 `output.dat` 文件。使用了 1024 字节的缓冲区来提高效率。


**2. 使用 FileReader 和 FileWriter 进行字符流操作**
```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class CharStreamExample {
    public static void main(String[] args) {
        try (FileReader fr = new FileReader("input.txt");
             FileWriter fw = new FileWriter("output.txt")) {
            char[] buffer = new char[1024];
            int charsRead;
            while ((charsRead = fr.read(buffer)) != -1) {
                fw.write(buffer, 0, charsRead);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

在此示例中，`FileReader` 用于读取 `input.txt` 文件的字符数据，`FileWriter` 用于将数据写入 `output.txt` 文件。

**3. 使用 BufferedReader 和 BufferedWriter 提高字符流操作效率**
```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class BufferedCharStreamExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("input.txt"));
             BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                bw.write(line);
                bw.newLine(); // 添加换行符
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

这里，`BufferedReader` 和 `BufferedWriter` 为字符流提供了缓冲功能，使用 `readLine()` 方法可以方便地逐行读取文本数据。


**4. 使用 Files 工具类简化文件操作（Java 7 及以上版本）**
```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

public class FilesExample {
    public static void main(String[] args) {
        try {
            // 读取文件所有行
            List<String> lines = Files.readAllLines(Paths.get("input.txt"));
            // 写入文件
            Files.write(Paths.get("output.txt"), lines);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

`Files` 工具类提供了静态方法，如 `readAllLines()` 和 `write()`，可以更简洁地读取和写入文件。​

**注意事项**：
- **资源管理**：​使用 `try-with-resources` 语句可以确保在操作完成后自动关闭流，避免资源泄漏。​
- **字符编码**：​在处理文本文件时，注意指定正确的字符编码，避免乱码问题。例如，可以使用 `InputStreamReader` 和`OutputStreamWriter` 来指定编码。​
- **异常处理**：​所有的 I/O 操作都可能抛出 `IOException`，需要适当处理异常。​

根据具体需求选择合适的类和方法，可以有效地进行文件的输入输出操作


---


## 🆚 各种 Java 文件 I/O 方法的对比

|方法|数据类型|是否带缓冲|适用场景|特点|
|---|---|---|---|---|
|**FileInputStream / FileOutputStream**|字节（byte）|否|二进制文件（如图片、音频）|低层次操作，适合处理非文本文件|
|**FileReader / FileWriter**|字符（char）|否|文本文件（纯文本）|适合处理字符数据，编码默认为系统默认|
|**BufferedReader / BufferedWriter**|字符（char）|✅ 是|文本文件|基于 FileReader/FileWriter 提供缓冲，提高效率，适合逐行读取文本|
|**Files 类（NIO.2）**|字符或字节|✅ 是（内部实现）|简单地读写小型文件（Java 7+）|高层抽象，代码简洁，适合读取整个文件到内存或写入|


### 举个例子：处理一个文本文件
- **你只想简单读取整篇小文章** → 用 `Files.readAllLines()`
- **你要逐行读取大文件并处理** → 用 `BufferedReader`
- **你要写文本文件内容** → 用 `BufferedWriter`（支持 `newLine()`）
- **你要读写图片或PDF文件** → 用 `FileInputStream` / `FileOutputStream`
- **你想自定义编码（如 UTF-8）** → 用 `InputStreamReader` / `OutputStreamWriter`


---


## ✅ “是否带缓冲”是什么意思？

**缓冲（Buffering）**，就是**在程序和磁盘之间加一层“中转站”**（通常是内存中的一个数组），用来**减少磁盘读写的次数**，提高性能。


### 🚀 在 Java 文件 I/O 中的区别：

| 操作方式                                           | 是否带缓冲  | 行为                         |
| ---------------------------------------------- | ------ | -------------------------- |
| `FileInputStream` / `FileOutputStream`         | ❌ 不带缓冲 | 每次读/写一个或几个字节，系统频繁访问磁盘，性能低  |
| `BufferedInputStream` / `BufferedOutputStream` | ✅ 带缓冲  | 每次先读/写一大块数据到内存缓冲区，减少磁盘访问次数 |
| `FileReader` / `FileWriter`                    | ❌ 不带缓冲 | 每次读/写一个或几个字符               |
| `BufferedReader` / `BufferedWriter`            | ✅ 带缓冲  | 支持按行读写，还能大块读写字符，提高效率       |
