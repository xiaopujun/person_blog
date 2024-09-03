```xmind preview
./xmind/Java IO系统.xmind
```

## 文件

### File对象

Java 的 `File` 类有三种常用的构造器，分别是：

1. **`File(String pathname)`**:
    - 通过指定文件路径名来创建一个新的 `File` 实例。
   ```java
   File file = new File("C:\\example\\file.txt");
   ```

2. **`File(String parent, String child)`**:
    - 通过指定父路径名字符串和子路径名字符串来创建一个新的 `File` 实例。
   ```java
   File file = new File("C:\\example", "file.txt");
   ```

3. **`File(File parent, String child)`**:
    - 通过指定父路径名的 `File` 对象和子路径名字符串来创建一个新的 `File` 实例。
   ```java
   File parentDir = new File("C:\\example");
   File file = new File(parentDir, "file.txt");
   ```

### 创建文件

要使用 Java 的 `File` 类在磁盘上创建一个文件，可以使用 `createNewFile()` 方法。以下是一个示例代码：

```java
public class CreateFileExample {
    public static void main(String[] args) {
        // 创建一个 File 对象
        File file = new File("C:\\example\\newfile.txt");

        try {
            // 创建新文件
            if (file.createNewFile()) {
                System.out.println("文件创建成功！");
            } else {
                System.out.println("文件已存在。");
            }
        } catch (IOException e) {
            System.out.println("发生错误。");
            e.printStackTrace();
        }
    }
}
```
