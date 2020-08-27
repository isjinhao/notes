## IO流

IO指的是数据的Input和Output。最常见的就是数据从外存到内存（Input）和数据从内存到外存（Output）。而IO流是一种进行IO操作的技术，这种技术是点到点的技术。数据从一个点流向另一个点，而且每个流都有固定的方向，这个方向不能改变，所以如果两个点之间要双向通信，需要建立两个流。用网络的术语说，每个流都是单工的，不能双工。Java中流的继承体系如下：

<div align="center"><img width="90%" src="http://blogfileqiniu.isjinhao.site/926ca670-e6c3-44fe-b847-aaf8ecf38100"></div>


### 字节输出流

`java.io.OutputStream `抽象类是表示字节输出流的所有类的超类，将指定的字节信息写出到目的地。它定义了字节输出流的基本共性功能方法。

* `public void close()` ：关闭此输出流并释放与此流相关联的任何系统资源。  
* `public void flush() ` ：刷新此输出流并强制任何缓冲的输出字节被写出。  
* `public void write(byte[] b)`：将 b.length 字节从指定的字节数组写入此输出流。  
* `public void write(byte[] b, int off, int len)` ：从指定的字节数组写入 len 字节，从偏移量 off 开始输出到此输出流。  
* `public abstract void write(int b)` ：将指定的字节输出流。

我们下面以FileOutputStream类来看看字节输出流的用法。

#### 构造方法

* `public FileOutputStream(File file)`：创建文件输出流以写入由指定的 File对象表示的文件。 
* `public FileOutputStream(String name)`： 创建文件输出流以指定的名称写入文件。  

当你创建一个流对象时，必须传入一个文件路径。该路径下，如果没有这个文件，会创建该文件。如果有这个文件，会清空这个文件的数据。

```java
public class FileOutputStreamConstructor throws IOException {
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileOutputStream fos = new FileOutputStream(file);
      
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("b.txt");
    }
}
```

#### 写出字节数据

**写出字节**

`write(int b)` 方法，每次可以写出一个字节数据，代码使用演示

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");     
      	// 写出数据
      	fos.write(97); // 写出第1个字节
      	fos.write(98); // 写出第2个字节
      	fos.write(99); // 写出第3个字节
      	// 关闭资源
        fos.close();
    }
}
// 1. 虽然参数为int类型四个字节，但是只会保留一个字节的信息写出。如果数值超过一个字节打开文件会乱码
// 2. 流操作完毕后，必须释放系统资源，调用close方法，千万记得。
```

**写出字节数组**

`write(byte[] b)`，每次可以写出数组中的数据，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");     
      	// 字符串转换为字节数组
      	byte[] b = "社会主义核心价值观".getBytes();
      	// 写出字节数组数据
      	fos.write(b);
      	// 关闭资源
        fos.close();
    }
}
```

**写出指定长度字节数组**

`write(byte[] b, int off, int len)` ，每次写出从off索引开始，len个字节，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");     
      	// 字符串转换为字节数组
      	byte[] b = "abcde".getBytes();
		// 写出从索引2开始，2个字节。索引2是c，两个字节，也就是cd。
        fos.write(b,2,2);
      	// 关闭资源
        fos.close();
    }
}
```

#### 数据追加续写

经过以上的演示，每次程序运行，创建输出流对象，都会清空目标文件中的数据。如何保留目标文件中数据，还能继续添加新数据呢？

- `public FileOutputStream(File file, boolean append)`： 创建文件输出流以写入由指定的 File 对象表示的文件。  
- `public FileOutputStream(String name, boolean append)`： 创建文件输出流以指定的名称写入文件。  

这两个构造方法，参数中都需要传入一个boolean类型的值，`true` 表示追加数据，`false` 表示清空原有数据。这样创建的输出流对象，就可以指定是否追加续写了，代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt"，true);     
      	// 字符串转换为字节数组
      	byte[] b = "abcde".getBytes();
		// 写出从索引2开始，2个字节。索引2是c，两个字节，也就是cd。
        fos.write(b);
      	// 关闭资源
        fos.close();
    }
}
```

#### 写出换行

Windows系统里，换行符号是`\r\n` 。代码使用演示：

```java
public class FOSWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileOutputStream fos = new FileOutputStream("fos.txt");  
      	// 定义字节数组
      	byte[] words = {97,98,99,100,101};
      	// 遍历数组
        for (int i = 0; i < words.length; i++) {
          	// 写出一个字节
            fos.write(words[i]);
          	// 写出一个换行, 换行符号转成数组写出
            fos.write("\r\n".getBytes());
        }
      	// 关闭资源
        fos.close();
    }
}
```

> * 回车符`\r`和换行符`\n` ：
>   * 回车符：回到一行的开头（return）。
>   * 换行符：下一行（newline）。
> * 系统中的换行：
>   * Windows系统里，每行结尾是 `回车+换行` ，即`\r\n`；
>   * Unix系统里，每行结尾只有 `换行` ，即`\n`；
>   * Mac系统里，每行结尾是 `回车` ，即`\r`。从 Mac OS X开始与Linux统一。



### 字节输入流

`java.io.InputStream `抽象类是表示字节输入流的所有类的超类，可以读取字节信息到内存中。它定义了字节输入流的基本共性功能方法。

- `public void close()` ：关闭此输入流并释放与此流相关联的任何系统资源。    
- `public abstract int read()`： 从输入流读取数据的下一个字节。 
- `public int read(byte[] b)`： 从输入流中读取一些字节数，并将它们存储到字节数组 b中 。

我们下面以FileInputStream类来看看字节输入流的用法。

#### 构造方法

* `FileInputStream(File file)`： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的 File 对象 file 命名。 
* `FileInputStream(String name)`： 通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名 name 命名。  

当你创建一个流对象时，必须传入一个文件路径。该路径下如果没有该文件，会抛出`FileNotFoundException` 。

```java
public class FileInputStreamConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileInputStream fos = new FileInputStream(file);
      
        // 使用文件名称创建流对象
        FileInputStream fos = new FileInputStream("b.txt");
    }
}
```

#### 读取字节数据

**读取字节**

`read`方法，每次可以读取一个字节的数据，提升为int类型，读取到文件末尾，返回`-1`，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象
       	FileInputStream fis = new FileInputStream("read.txt");
      	// 定义变量，保存数据
        int b ；
        // 循环读取
        while ((b = fis.read())!=-1) {
            System.out.println((char)b);
        }
		// 关闭资源
        fis.close();
    }
}
```

**使用字节数组读取**

`read(byte[] b)`，每次读取b的长度个字节到数组中，返回读取到的有效字节个数，读取到末尾时，返回`-1` ：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象.
       	FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
      	// 定义变量，作为有效个数
        int len ；
        // 定义字节数组，作为装字节数据的容器   
        byte[] b = new byte[2];
        // 循环读取
        while (( len= fis.read(b))!=-1) {
           	// 每次读取后,把数组变成字符串打印
            System.out.println(new String(b));
        }
		// 关闭资源
        fis.close();
    }
}
```

错误数据`d`，是由于最后一次读取时，只读取一个字节`e`，数组中，上次读取的数据没有被完全替换，所以要通过`len` ，获取有效的字节，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException{
      	// 使用文件名称创建流对象.
       	FileInputStream fis = new FileInputStream("read.txt"); // 文件中为abcde
      	// 定义变量，作为有效个数
        int len ；
        // 定义字节数组，作为装字节数据的容器   
        byte[] b = new byte[2];
        // 循环读取
        while (( len= fis.read(b))!=-1) {
           	// 每次读取后,把数组的有效字节部分，变成字符串打印
            System.out.println(new String(b，0，len));//  len 每次读取的有效字节个数
        }
		// 关闭资源
        fis.close();
    }
}
```



### 字符输入流

`java.io.Reader`抽象类是表示用于读取字符流的所有类的超类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法。

- `public void close()` ：关闭此流并释放与此流相关联的任何系统资源。    
- `public int read()`： 从输入流读取一个字符。 
- `public int read(char[] cbuf)`： 从输入流中读取一些字符，并将它们存储到字符数组 cbuf 中 。

我们下面以FileReader类来看看字符输出流的用法。

#### 构造方法

- `FileReader(File file)`： 创建一个新的 `FileReader` ，给定要读取的File对象。   
- `FileReader(String fileName)`： 创建一个新的 `FileReader` ，给定要读取的文件的名称。  

当你创建一个流对象时，必须传入一个文件路径。类似于 `FileInputStream`。由于`FileInputStream`是字节流，它不需要考虑编码的问题，而字符流需要考虑编码的问题，`FileReader`使用的是项目默认的编码方式。

```java
public class FileReaderConstructor throws IOException{
    public static void main(String[] args) {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileReader fr = new FileReader(file);
      
        // 使用文件名称创建流对象
        FileReader fr = new FileReader("b.txt");
    }
}
```

#### 读取字符

**读取字符**：`read`方法，每次可以读取一个字符的数据，提升为int类型，读取到文件末尾，返回`-1`，循环读取，代码使用演示：

```java
public class FRRead {
    public static void main(String[] args) throws IOException {
      	// 使用文件名称创建流对象
       	FileReader fr = new FileReader("read.txt");
      	// 定义变量，保存数据
        int b;
        // 循环读取
        while ((b = fr.read())!=-1) {
            System.out.println((char)b);
        }
		// 关闭资源
        fr.close();
    }
}
```

**使用字符数组读取**

`read(char[] cbuf)`，每次读取b的长度个字符到数组中，返回读取到的有效字符个数，读取到末尾时，返回`-1` ，代码使用演示：

```java
public class FISRead {
    public static void main(String[] args) throws IOException {
      	// 使用文件名称创建流对象
       	FileReader fr = new FileReader("read.txt");
      	// 定义变量，保存有效字符个数
        int len ；
        // 定义字符数组，作为装字符数据的容器
        char[] cbuf = new char[2];
        // 循环读取
        while ((len = fr.read(cbuf)) != -1) {
            System.out.println(new String(cbuf,0,len));
        }
    	// 关闭资源
        fr.close();
    }
}
```

#### InputStreamReader

我们刚才说`FileReader`是以项目默认的编码方式读取数据，而其父类`InputStreamReader`可以接受指定的编码。

##### 构造方法

* `InputStreamReader(InputStream in)`：创建一个使用默认字符集的字符流。 
* `InputStreamReader(InputStream in, String charsetName)`：创建一个指定字符集的字符流。

构造举例，代码如下： 

```java
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```

##### 指定编码读取

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/060cff9f-117f-4401-80fc-6bdde425eb0b" /></div>



```java
public class ReaderDemo2 {
    public static void main(String[] args) throws IOException {
      	// 定义文件路径，文件为gbk编码
        String FileName = "C:\\Users\\ISJINHAO\\Desktop\\a.txt";
      	// 创建流对象，默认UTF8编码
        InputStreamReader isr = 
            	new InputStreamReader(new FileInputStream(FileName));
      	// 创建流对象,指定GBK编码
        InputStreamReader isr2 = 
            	new InputStreamReader(new FileInputStream(FileName) , "GBK");
		// 定义变量,保存字符
        int read;
      	// 使用默认编码字符流读取,乱码
        while ((read = isr.read()) != -1) {
            System.out.print((char)read); // ��Һ�
        }
        isr.close();
      
      	// 使用指定编码字符流读取,正常解析
        while ((read = isr2.read()) != -1) {
            System.out.print((char)read);	// 大家好
        }
        isr2.close();
    }
}
```



### 字符输出流

`java.io.Writer `抽象类是表示用于写出字符流的所有类的超类，将指定的字符信息写出到目的地。它定义了字节输出流的基本共性功能方法。

- `void write(int c)` ：写入单个字符。
- `void write(char[] cbuf) `：写入字符数组。 
- `abstract void write(char[] cbuf, int off, int len)`：写入字符数组的某一部分，off数组的开始索引，len写的字符个数。 
- `void write(String str) `：写入字符串。 
- `void write(String str, int off, int len)` ：写入字符串的某一部分，off字符串的开始索引，len写的字符个数。
- `void flush() `：刷新该流的缓冲。  
- `void close()`：关闭此流，但要先刷新它。 

我们下面以`FileWrier`看看字符输出流的使用方式。

#### 构造方法

- `FileWriter(File file)`： 创建一个新的 FileWriter，给定要读取的File对象。   
- `FileWriter(String fileName)`： 创建一个新的 FileWriter，给定要读取的文件的名称。  

当你创建一个流对象时，必须传入一个文件路径，类似于FileOutputStream。

```java
public class FileWriterConstructor {
    public static void main(String[] args) throws IOException {
   	 	// 使用File对象创建流对象
        File file = new File("a.txt");
        FileWriter fw = new FileWriter(file);
      
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("b.txt");
    }
}
```

#### 写出字符

**写出字符**

`write(int b)` 方法，每次可以写出一个字符数据，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	// 写出数据
      	fw.write(97); // 写出第1个字符
      	fw.write('b'); // 写出第2个字符
      	fw.write('C'); // 写出第3个字符
      	fw.write(30000); // 写出第4个字符，中文编码表中30000对应一个汉字。
		fw.close();
    }
}
```

**写出字符数组**

`write(char[] cbuf)` 和 `write(char[] cbuf, int off, int len)` ，每次可以写出字符数组中的数据，用法类似`FileOutputStream`，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	// 字符串转换为字节数组
      	char[] chars = "社会主义核心价值观".toCharArray();
      
      	// 写出字符数组
      	fw.write(chars); // 社会主义核心价值观
            
      	// 关闭资源
        fos.close();
    }
}
```

**写出字符串**

`write(String str)` 和 `write(String str, int off, int len)` ，每次可以写出字符串中的数据，更为方便，代码使用演示：

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象
        FileWriter fw = new FileWriter("fw.txt");     
      	String msg = "社会主义核心价值观";
      
      	// 写出字符数组
      	fw.write(msg); // 社会主义核心价值观
      	
        // 关闭资源
        fos.close();
    }
}
```

**续写和换行**

操作类似于FileOutputStream。

```java
public class FWWrite {
    public static void main(String[] args) throws IOException {
        // 使用文件名称创建流对象，可以续写数据
        FileWriter fw = new FileWriter("fw.txt"，true);     
      	// 写出字符串
        fw.write("社会");
      	// 写出换行
      	fw.write("\r\n");
      	// 写出字符串
  		fw.write("主义核心价值观");
      	// 关闭资源
        fw.close();
    }
}
```

#### OutputStreamWriter

我们刚才说`FileWriter`是以项目默认的编码方式读取数据，而其父类` OutputStreamWriter`可以接受指定的编码。

##### 构造方法

- `OutputStreamWriter(OutputStream in)`: 创建一个使用默认字符集的字符流。 
- `OutputStreamWriter(OutputStream in, String charsetName)`: 创建一个指定字符集的字符流。

构造举例，代码如下： 

```java
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("out.txt"));
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```

##### 指定编码写出

```java
public class OutputDemo {
    public static void main(String[] args) throws IOException {
        // 定义文件路径
        String FileName = "C:\\Users\\ISJINHAO\\Desktop\\out1.txt";
        // 创建流对象，默认UTF8编码
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(FileName));
        // 写出数据
        osw.write("你好"); // 保存为6个字节
        osw.close();

        // 定义文件路径
        String FileName2 = "C:\\Users\\ISJINHAO\\Desktop\\out2.txt";
        // 创建流对象，指定GBK编码
        OutputStreamWriter osw2 =
                new OutputStreamWriter(new FileOutputStream(FileName2), "GBK");
        // 写出数据
        osw2.write("你好");    // 保存为4个字节
        osw2.close();
    }
}
```

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/f207a239-aca9-4257-ac99-f209752f2aad" /></div>



### 缓冲流

缓冲流,也叫高效流，是对4个基本流的增强，所以也是4个流，按照数据类型分类：

* **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream` 
* **字符缓冲流**：`BufferedReader`，`BufferedWriter`

缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

#### 字节缓冲流

##### 构造方法

* `public BufferedInputStream(InputStream in)` ：创建一个 新的缓冲输入流。 
* `public BufferedOutputStream(OutputStream out)`： 创建一个新的缓冲输出流。

构造举例，代码如下：

```java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
```

##### 效率测试

查询API，缓冲流读写方法与基本的流是一致的，我们通过复制大文件（391MB），测试它的效率。

- 基本流，代码如下：

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
        long start = System.currentTimeMillis();
        // 创建流对象
        try (
                FileInputStream fis = 
            		new FileInputStream("C:\\Users\\ISJINHAO\\Desktop\\HBuilder.zip");
                FileOutputStream fos = 
            		new FileOutputStream("C:\\Users\\ISJINHAO\\Desktop\\copy.zip")
        ){
            // 读写数据
            int b;
            while ((b = fis.read()) != -1) {
                fos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("普通流复制时间:"+(end - start)+" 毫秒");
    }
}

// 十几分钟过去了...
```

- 缓冲流，代码如下：

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
      	long start = System.currentTimeMillis();
		// 创建流对象
        try (
        	BufferedInputStream bis = new BufferedInputStream(new FileInputStream("jdk9.exe"));
	     BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.exe"));
        ){
        // 读写数据
            int b;
            while ((b = bis.read()) != -1) {
                bos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
		// 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流复制时间:"+(end - start)+" 毫秒");
    }
}
// 缓冲流复制时间:14665 毫秒
```

- 如何更快呢？使用数组的方式，代码如下：


```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
        long start = System.currentTimeMillis();
        // 创建流对象
        try (
            BufferedInputStream bis = new BufferedInputStream(
                new FileInputStream("C:\\Users\\ISJINHAO\\Desktop\\HBuilder.zip"));
            BufferedOutputStream bos = new BufferedOutputStream(
                new FileOutputStream("C:\\Users\\ISJINHAO\\Desktop\\copy.zip"));
        ) {
            // 读写数据
            int len;
            byte[] bytes = new byte[8 * 1024];
            while ((len = bis.read(bytes)) != -1) {
                bos.write(bytes, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("缓冲流使用数组复制时间:" + (end - start) + " 毫秒");
    }
}
// 缓冲流使用数组复制时间:1504 毫秒
```

- 我们刚才说缓冲流的快是由于一次读取一块而不是一个字节，那么我们使用InpuStream一次读取一块可以加快速度吗？

```java
public class BufferedDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 记录开始时间
        long start = System.currentTimeMillis();
        // 创建流对象
        try (
            FileInputStream fis =
            		new FileInputStream("C:\\Users\\ISJINHAO\\Desktop\\HBuilder.zip");
            FileOutputStream fos =
            		new FileOutputStream("C:\\Users\\ISJINHAO\\Desktop\\copy.zip")
        ) {
            int len;
            byte[] bytes = new byte[8 * 1024];
            while ((len = fis.read(bytes)) != -1) {
                fos.write(bytes, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 记录结束时间
        long end = System.currentTimeMillis();
        System.out.println("普通流使用数组复制时间:" + (end - start) + " 毫秒");
    }
}
// 1947
```

从结果可以看到，大幅加快速度的原因是使用了数组，一次可以读取一块数据。

#### 字符缓冲流

##### 构造方法

* `public BufferedReader(Reader in)` ：创建一个 新的缓冲输入流。 
* `public BufferedWriter(Writer out)`： 创建一个新的缓冲输出流。

构造举例，代码如下：

```java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("br.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
```

### 特有方法

字符缓冲流的基本方法与普通字符流调用方式一致，不再阐述，我们来看它们具备的特有方法。

* `BufferedReader`：`public String readLine()`：读一行文字。 
* `BufferedWriter`：`public void newLine()`：写一行。行分隔符由系统属性定义符号。 

`readLine`方法演示，代码如下：

```java
public class BufferedReaderDemo {
    public static void main(String[] args) throws IOException {
      	 // 创建流对象
        BufferedReader br = new BufferedReader(new FileReader("in.txt"));
		// 定义字符串,保存读取的一行文字
        String line  = null;
      	// 循环读取,读取到最后返回null
        while ((line = br.readLine())!=null) {
            System.out.print(line);
            System.out.println("------");
        }
		// 释放资源
        br.close();
    }
}
```

`newLine`方法演示，代码如下：

  ```java
public class BufferedWriterDemo throws IOException {
    public static void main(String[] args) throws IOException  {
      	// 创建流对象
		BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));
      	// 写出数据
        bw.write("社会");
      	// 写出换行
        bw.newLine();
        bw.write("主义");
        bw.newLine();
        bw.write("核心价值观");
        bw.newLine();
		// 释放资源
        bw.close();
    }
}
  ```



### 打印流

#### 构造方法

* 打印流m类c PrintStream(String fileName)  `： 使用指定的文件名创建一个新的打印流。

构造举例，代码如下：  

```java
PrintStream ps = new PrintStream("ps.txt")；
```

#### 改变打印流向

`System.out`就是`PrintStream`类型的，只不过它的流向是系统规定的，打印在控制台上。不过，既然是流对象，我们就可以玩一个"小把戏"，改变它的流向。

```java
public class PrintDemo {
    public static void main(String[] args) throws IOException {
		// 调用系统的打印流,控制台直接输出97
        System.out.println(97);
      
		// 创建打印流,指定文件的名称
        PrintStream ps = new PrintStream("ps.txt");
      	
      	// 设置系统的打印流流向,输出到ps.txt
        System.setOut(ps);
      	// 调用系统的打印流，ps.txt中输出97
        System.out.println(97);
    }
}
```



## Socket

### InetAddress

> This class represents an Internet Protocol (IP) address. 

**获得本机IP地址**

- 获得本地IP地址

`InetAddress iaddress = InetAddress.getLocalHost();`

但是这个函数有问题，因为这个函数的原理是通过获取本机的`hostname`，然后对此`hostname`做解析，从而获取`IP`地址的。那么问题来了，如果在本机的`/etc/hosts`文件里对这个主机名指向了一个错误的`IP`地址，那么`InetAddress.getLocalHost`就会返回这个错误的`IP`地址。当然如果你的`hostname`是到`DNS`去解析的，碰巧`DNS`上的信息也是错的，也同样是悲惨结局。

`InetAddress`是由两部分组成的，一部分是getHostName()，一部分是getHostAddress()。

- 获得本机所有的IP地址

```java
/**
 * 获取机器所有网卡的IP（ipv4）
 */
public static List<String> getLocalIP() {
	List<String> ipList = new ArrayList<String>();
	InetAddress ip = null;
	try {
		Enumeration<NetworkInterface> netInterfaces = (Enumeration<NetworkInterface>) NetworkInterface.getNetworkInterfaces();
		while (netInterfaces.hasMoreElements()) {
			NetworkInterface ni = (NetworkInterface) netInterfaces.nextElement();
			// 遍历所有ip
			Enumeration<InetAddress> ips = ni.getInetAddresses();
			while (ips.hasMoreElements()) {
				ip = (InetAddress) ips.nextElement();
				if (null == ip || "".equals(ip)) {
					continue;
				}
				String sIP = ip.getHostAddress();
				if(sIP == null || sIP.indexOf(":") > -1) {
					continue;
				}
				ipList.add(sIP);
			}
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
	return ipList;
}
```

- 获取其他主机的IP地址对象

`InetAddress otherInetAddress = InetAddress.getByName("www.baidu.com");`

```java
public static void main(String[] args) throws UnknownHostException {
	//1.获取本地主机
	InetAddress iaddress = InetAddress.getLocalHost();
	System.out.println(iaddress);	 //打印 InetAddress对象 默认格式: 用户名/IP地址
	
	//2.获取主机名
	String hostName = iaddress.getHostName();
	//3.获取主机IP地址
	String ip = iaddress.getHostAddress();
	System.out.println(hostName);
	System.out.println(ip);
	//3.获取其他主机的IP地址对象
	InetAddress otherInetAddress = InetAddress.getByName("www.baidu.com");
	System.out.println(otherInetAddress);
}
```



### UDP

UDP通信需要两个类的支持：

- 数据的发送接收器：DatagramSocket
- 数据包类：DatagramPacket

```java
public class UDPReceiver {
	public static void main(String[] args) throws IOException {
		//1.创建DatagramSocket对象,
		//强调:接收端必须指定一个端口号
		DatagramSocket ds = new DatagramSocket(12345);
		while(true){
			//2.直接创建一个DatagramPacket对象
			byte[] bs = new byte[1024];
			DatagramPacket dp = new DatagramPacket(bs, bs.length);
			//3.接收
			System.out.println("等待发送端发送数据....");
			ds.receive(dp);//这个方法具有等待功能,等待发送端发送过来的数据
			System.out.println("接收数据成功!!");
			//获取发送端的地址
			InetAddress sendAddress = dp.getAddress();
			System.out.println("发送端是:"+sendAddress.getHostAddress());
			//获取真正的数据
			byte[] data = dp.getData();
			//获取发送端 发来了多少字节
			int len = dp.getLength();
			//打印数据
			String receiveMsg = new String(data, 0, len);
			System.out.println("发送端说:"+receiveMsg);
		}
		//4.关闭资源（程序运行结束之后是需要关闭资源的，但是我们的程序是一个死循环，此句永不会执行，所以不能加关闭）
		//ds.close();	
	}
}
```

```java
public class UDPSender {
	public static void main(String[] args) throws Exception {
		Scanner sc = new Scanner(System.in);
		//1.创建DatagramSocket对象
		DatagramSocket ds = new DatagramSocket();
		while(true){
			//2.创建DatagramPacket对象
			//存储 发送的数据,对方的IP,端口号
			System.out.println("请输入您要发送的数据:");
			String sendMsg = sc.nextLine();
			byte[] bs = sendMsg.getBytes();
			//IP地址:127.0.0.1  代表本机,本地回环地址
			DatagramPacket dp = new DatagramPacket(bs,bs.length,InetAddress.getByName("127.0.0.1"),12345);
			//3.发送
			ds.send(dp);
			System.out.println("发送数据成功!!!");//192.168.146.72
		}
		//4.关闭资源（程序运行结束之后是需要关闭资源的，但是我们的程序是一个死循环，此句永不会执行，所以不能加关闭）
		//ds.close();
	}
}

```



### TCP

```java
/**
 * TCP服务器:(ServerSocket) 步骤:
 *
 * 1.创建一个ServerSocket对象,必须绑定一个端口,这个端口必须和客户端连接的端口一致
 * 2.调用server的accept()方法,获取到底哪一个客户端连接的服务器
 * 3.通过刚刚获取到的客户端对象 调用getInputStream()方法
 * 4.通过输入流调用read方法,读取客户端写过来的数据
 * 5.关闭资源
 *
 */
public class ServerDemo {
	public static void main(String[] args) throws IOException {
		// 1.创建一个ServerSocket对象,必须绑定一个端口,这个端口必须和客户端连接的端口一致
		ServerSocket server = new ServerSocket(12345);
		// 2.获取到 哪一个 客户端连接的我
		System.out.println("等待客户端连接...");
		Socket client = server.accept();// 此方法也具有等待功能,等待某一个客户端连接
		// 打印一些和客户端有关信息
		String ip = client.getInetAddress().getHostAddress();
		System.out.println("小样,抓到你了:" + ip);
		// 3.获取输入流,实际上是客户端写数据时的输出流
		InputStream in = client.getInputStream();
		// 4.读取数据
		byte[] bs = new byte[1024];
		int len = in.read(bs);
		// 打印
		System.out.println("客户端说:" + new String(bs, 0, len));
		// 5.要向客户端 回写数据,告诉客户端您的信息我已经收到了
		OutputStream out = client.getOutputStream();
		out.write("您的消息已经收到...".getBytes());
		System.out.println("给客户端反馈的信息发送成功!!!");

		// 关闭资源
		server.close();
		client.close();
		in.close();
	}
}
```

```java
/**
 * 
 * 使用TCP协议的客户端(Socket类) 步骤: 
 
 * 1.创建一个客户端对象(注意:指定这个Socket要连接的服务器的IP和端口)
 * 2.从客户端对象中获取 输出流:getOutputStream()
 * 3.调用输出流的Write方法写数据到服务器即可
 * 4.关闭资源
 * 
 */
public class ClientDemo {
	public static void main(String[] args) throws IOException {
		// 1.创建一个客户端对象(注意:指定这个Socket要连接的服务器的IP和端口)
		/*
		 * 这个构造方法干了很多事情: a.自动去连接服务器 b.自动进行三次握手,建立连接 c.自动为连接中创建两个流
		 */
		Socket client = new Socket("127.0.0.1", 12345);

		// 2.从客户端对象中获取 输出流:getOutputStream()
		// OutputStream out = client.getOutputStream();
		// 3.调用输出流的Write方法写数据到服务器即可
		// out.write("How are you".getBytes());
		client.getOutputStream().write("How are you".getBytes());
		System.out.println("给服务器发送数据成功!!");
		// 4.读取服务器 发送过来的反馈信息
		InputStream in = client.getInputStream();
		byte[] bs = new byte[1024];
		int len = in.read(bs);
		System.out.println("服务器响应:" + new String(bs, 0, len));
		// 关闭资源
		client.close();
	}
}
```



### 文件传输案例

```java
public class FileUploadServer {

	public static void main(String[] args) throws IOException {
		//1.创建ServerSocket对象,绑定一个端口
		ServerSocket server = new ServerSocket(12345);
		while(true){
			//2.获取哪一个客户端连接的服务器
			System.out.println("等待客户端连接...");
			final Socket client = server.accept();
			//开启一个线程,和clinet进行交互
			new Thread(() -> {
				try {
					// TODO Auto-generated method stub
					System.out.println("小样:"+client.getInetAddress().getHostAddress());
					//3.获取输入流,读取客户端发来数据
					InputStream in = client.getInputStream();
					//4.创建文件的输出流,把数据写到文件中
					String picName = "D:\\"+System.currentTimeMillis()+".png";
					FileOutputStream fos = new FileOutputStream(picName);
					//5.循环 从输入流读取客户端数据, 写入到文件中
					byte[] bs = new byte[1024];
					int len = 0;
					while((len=in.read(bs))!=-1){
						fos.write(bs, 0, len);
					}//1小时
					System.out.println("客户端的文件已经保存完毕,可以查看了"+picName);
					//6.告知客户端,文件真的真的真的上传成功
					try {
						Thread.sleep(10000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					OutputStream out = client.getOutputStream();
					out.write("您的文件真的真的真的上传成功".getBytes());
					client.close();
					in.close();
					out.close();
					fos.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}).start();
		}
		//6.关闭
	//	server.close();
	}
}
```

```java
public class FileUploadClient {
	public static void main(String[] args)throws IOException {
		//1.创建Socket对象,连接服务器
		Socket client = new Socket("127.0.0.1", 12345);
		System.out.println("连接服务器成功..");
		//2.获取输出流,把数据写向服务器
		OutputStream out = client.getOutputStream();
		//3.创建文件的输入流,读取本地的文件数据
		FileInputStream fis = 
            new FileInputStream("C:\\Users\\ISJINHAO\\Desktop\\我.jpg");
		//4.循环,读取本地文件,写到服务器
		byte[] bs = new byte[1024];
		int len = 0;
		while((len=fis.read(bs))!=-1){
			out.write(bs, 0, len);
		}
		//关闭输出流
		client.shutdownOutput();
		//5.获取服务器反馈的信息
		InputStream in = client.getInputStream();
		byte[] bs1 = new byte[1024];
		int len1 = in.read(bs1);
		System.out.println("服务器说:"+new String(bs1,0,len1));
		//6关闭
		client.close();
		out.close();
		fis.close();
	}
}
```

 

### Socket属性

- TCP_NODELAY：是否采用nagle算法。
- SO_REUSEADDR：表示是否允许重用Socket所绑定的本地地址。
- SO_TIMEOUT：表示接收数据时的等待超时时间。
- SO_SNFBUF：表示发送数据的缓冲区的大小。
- SO_RCVBUF：表示接收数据的缓冲区的大小。
- SO_KEEPALIVE：表示对于长时间处于空闲状态的Socket，是否要自动把它关闭。
- SO_LINGER：表示当执行Socket的close()方法时，是否立即关闭底层的Socket。默认第一种。

| on    | linger |                       closesocket行为                        |                    发送队列                    |                           底层行为                           |
| ----- | :----: | :----------------------------------------------------------: | :--------------------------------------------: | :----------------------------------------------------------: |
| true  |  忽略  |                          立即返回。                          |               保持直至发送完成。               |            系统接管套接字并保证将数据发送至对端。            |
| false |   零   |                          立即返回。                          |                   立即放弃。                   | 直接发送RST包，自身立即复位，不用经过2MSL状态。对端收到复位错误号。 |
| false |  非零  | 阻塞直到linger时间超时或数据发送完成。(套接字必须设置为阻塞状态) | 在超时时间段内保持尝试发送，若超时则立即放弃。 |          超时则同第二种情况，若发送完成则皆大欢喜。          |

- OOBINLINE：Enable/disable SO_OOBINLINE(receipt of TCP urgent data)By default, this option is disabled and TCP urgent data received on a socket is silently discarded. If the user wishes to receive urgent data, then this option must be enabled. When enabled, urgent data is received inline with normal data.  

  Note, only limited support is provided for handling incoming urgent data. In particular, no notification of incoming urgent data is provided and there is no capability to distinguish between normal data and urgentdata unless provided by a higher level protocol.

- PerformancePreferences：设置连接时间、低延迟、高带宽之间的权重。

