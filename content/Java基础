1.关于java.io
java.io大概有80个类，主要分为以下4种：
基于字节操作的I/O接口：InputStream和OutputStream.
基于字符操作的I/O接口：Writer 和 Reader
基于磁盘操作的I/O接口：File
基于网络操作的I/O接口：Socket(不在java.io包下)

java I/O中的装饰者模式
        抽象构件：InputStream 
	抽象装饰(继承inputStream)：FilterInputStream
	具体构件(继承inputStream)：ByteArrayInputStream,FileInputStream,PipedInputStream,StringBufferInputStream
	具体装饰(继承FilterInputStream)：DataInputStream,BufferInputStreamLineNumberInputStream,PushBackInputStream

        抽象构件：Reader
	抽象装饰(继承Reader)：BufferedReader,FilterReader
	具体构件(继承Reader)：CharArrayReader、InputStreamReader、PipedReader以及StringReader
	具体装饰：LineNumberReader作为BufferedReader的具体装饰角色,PushbackReader作为FilterReader的具体装饰。
OutputStream和Writer同理

java I/O中的适配器模式
        InputStream原始流处理器中的适配器模式 
        适配器类：ByteArrayInputStream，FileInputStream，StringBufferInputStream。
        Reader原始流处理器中的适配器模式
         适配器类：CharArrayReader，StringReader，InputStreamReader,PipedReader
OutputStream和Writer同理

java访问磁盘文件：
1.传入文件路径。
2.根据路径创建File对象来标识这个文件
3.根据File对象创建真正读取文件的操作对象FileDescriptor（文件描述符），通过这个对象可以直接控制这个磁盘文件
4.StreamDecoder类将byte解码为char格式

关于FileDescriptor类：
`private int fd;
public FileDescriptor() {
        fd = -1;
    }
private  FileDescriptor(int fd) {
        this.fd = fd;
    }`
封装了一个int类型的值fd，默认为-1，每个打开的文件，socket等都会给一个fd值，通过该值可以对文件，socket进行相关操作。
 //定义的一个标准输入，fd值为0
public static final FileDescriptor in = new FileDescriptor(0);
//定义的一个标准输出，fd值为1
public static final FileDescriptor out = new FileDescriptor(1);
//定义的一个标准错误输出，fd值为2
public static final FileDescriptor err = new FileDescriptor(2);
Sytem的in,out,err是通过FileDescriptor中的in,out,err来实现的

对字节流进行大量的从硬盘读取，用BufferedInputStream, 使用缓冲流能够减少对硬盘的损伤
Printwriter 可以打印各种数据类型。
怎么样把输出字节流转换成 输出字符流,说出它的步骤： 转换处理流OutputStreamWriter 可以将字节流转为字符流 New OutputStreamWriter（new FileOutputStream（File file））;
输入字节流转换为输入字符流同理。
但真正负责编码的类是StreamEncoder和StreamDecoder。
以StreamDecoder为例：
已知：InputStream到Reader的过程要指定编码字符集，否则将采用操作系统默认字符集
查看源码知：
`public class InputStreamReader extends Reader {  
     private final StreamDecoder sd;//由上图已知在InputStreamReader中一定有一个StreamDecoder对象  
     public InputStreamReader(InputStream in) {//InputStreamReader有多个构造方法，我假设它用的就是这 
         super(in);  
         try {  
               // 创建一个StreamDecoder对象  
             sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // 用系统默认编码  
         } catch (UnsupportedEncodingException e) {  
             // The default encoding should always be available  
             throw new Error(e);  
         }  
     }  
     public int read() throws IOException {  
         // 是StreamDecoder在read  
         return sd.read();  
     }  
 }  
`
