#### 一.重点掌握

**java.IO包下的**

四大家族的首领： 都是抽象类，都实现了java.io.Closeable接口，都有close()方法，实现了java.io.Flushable接口都有flush()方法

1. java.io.InputStream  字节输入流
2. java.io.OutputStream 字节输出流
3. java.io.Reader		字符输入流
4. java.io.Writer		字符输出流



Java.io包下重点掌握16个流

**文件专属：**
		java.io.FileInputStream（掌握）
		java.io.FileOutputStream（掌握）
		java.io.FileReader
		java.io.FileWriter

**转换流：（将字节流转换成字符流）**
		java.io.InputStreamReader
		java.io.OutputStreamWriter

**缓冲流专属：**
		java.io.BufferedReader
		java.io.BufferedWriter
		java.io.BufferedInputStream
		java.io.BufferedOutputStream

**数据流专属：**
		java.io.DataInputStream
		java.io.DataOutputStream

**标准输出流：**
		java.io.PrintWriter
		java.io.PrintStream（掌握）

**对象专属流：**
		java.io.ObjectInputStream（掌握）
		java.io.ObjectOutputStream（掌握）



#### 二.常用方法

略



#### 三.文件的读写

FileinputStream**读取**文件

```java
//创建一个文件输入流
FileInputStream fis=new FileInputStream("文件的路径");
//创建一个byte数组，表示一次从文件中获取的大小
byte[] bytes =new byte[1024];        
int readCount=0;
String str;
//循环从文件中获取数据
while((readCount=fis.read(bytes))!=-1){
    str=new String(bytes,0,readCount);
    System.out.println(str);
}
        
```

FileOutputStream **写入**文件

```java
//创建一个输出流 
//第二个参数  true：文件追加写入，false：不追加写入（默认）
FileOutputStream fos=new FileOutputStream("C:\\Users\\c06936\\Desktop\\123.txt",true);
String s="hello";  //要写入文件的内容
byte[] b=s.getBytes();  //FileOutputStream 写入文件一定要是二进制形式

fos.write(b);
fos.flush();//一定要刷新
fos.close();//一定要关闭
```

FileReader **读取**文件

```java
//创建一个FileReader
FileReader fr = new FileReader("文件路径");
//同FileinputStream一样不过创建的是char数组
char[] buf = new char[1024];
int count = 0;
String str;
while ((count = fr.read(buf)) != -1) {
    str = new String(buf, 0, count);
    System.out.println(str);
}
fr.close()//使用完一定要关闭流，一定要关闭
```

FileWriter **写入**文件

```java
//创建一个FileWriter  
//第二个参数表示文件是否以追加的形式写入
FileWriter fw=new FileWriter("文件路径",true);
String s= "Hello\n\r";
fw.write(s);  //写入
fw.flush();  //最重要的，一定要刷新
fw.close();  //一定要关闭
```



#### 四.IO流结合Properties

```java
//创建一个流
FileReader fr = new FileReader("文件路径");        
//创建一个Properties集合
Properties ps = new Properties();
//将文件中的键值对装载到集合中
ps.load(fr);

String name = ps.getProperty("key");
```



#### 五.类的序列化和反序列化

使用者两个流

java.io.ObjectInputStream（掌握）
java.io.ObjectOutputStream（掌握）

**注：**

1. **参与序列化和反序列化的对象必须实现Serializable接口，这是一个标志接口**

2. 如果有一个属性不想参与序列化，在属性上加一个transient

   ```Java
   private transient String name;
   ```

3. 建议将序列化的版本号手动定义下来

   ```java
   private static final long serialVersionUID=1L;
   ```

   

