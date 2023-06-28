---
title: stream
date: 2023-06-27 10:51:04
tags:
---

Stream(抽象基类)支持读取和写入字节。 所有表示流的类都继承自Stream类。 Stream类及其派生类提供数据源和存储库的常见视图，使程序员不必了解操作系统和基础设备的具体细节。

**流涉及三个基本操作：**

+ 读取 - 将数据从流读取到数据结构（如字节数组）中。

+ 写入 - 将数据从数据源写入到流中。

+ 查找 - 对流中的当前位置进行查询和修改。

**下面是一些常用的流类：**

+ FileStream - 用于对文件进行读取和写入操作。

+ MemoryStream - 用于对内存进行读取和写入操作。

+ BufferedStream - 用于改进读取和写入操作的性能。

+ NetworkStream - 用于通过网络套接字进行读取和写入。

+ IsolatedStorageFileStream - 用于对独立存储中的文件进行读取或写入操作。

+ PipeStream - 用于通过匿名和命名管道进行读取和写入。

+ CryptoStream - 用于将数据流链接到加密转换。

**读取器和编写器：**
System.IO 命名空间还提供用于在流中读取和写入已编码字符的类型。 通常，流用于字节输入和输出。 读取器和编写器 处理编码字符与字节之间的来回转换，以便流可以完成操作。 每个读取器和编写器类都与流关联，可以通过类的 BaseStream 属性进行检索。

*下面是一些常用的读取器和编写器类：*

+ BinaryReader 和 BinaryWriter - 用于将基元数据类型作为二进制值进行读取和写入。

+ StreamReader 和 StreamWriter - 用于通过使用编码值在字符和字节之间来回转换来读取和写入字符。

+ StringReader 和 StringWriter - 用于从字符串读取字符以及将字符写入字符串中。

+ TextReader 和 TextWriter - 用作其他读取器和编写器（读取和写入字符和字符串，而不是二进制数据）的抽象基类。

# Stream 类

继承自Stream类的一些更常用的流包括 FileStream、 和 MemoryStream。根据不同数据源类型，继承自Stream类的流可能仅支持Stream类中的某些功能。 

## 属性

+ CanRead	
当在派生类中重写时，获取指示当前流是否支持读取的值。

+ CanSeek	
当在派生类中重写时，获取指示当前流是否支持查找功能的值。

+ CanTimeout	
获取一个值，该值确定当前流是否可以超时。

+ CanWrite	
当在派生类中重写时，获取指示当前流是否支持写入功能的值。

+ Length	
当在派生类中重写时，获取流长度（以字节为单位）。

+ Position	
当在派生类中重写时，获取或设置当前流中的位置。

 > 很多asp.net项目中文件或图片上传中很多朋友会经历过这样一个痛苦：Stream对象被缓存了，导致了Position属性在流中无法
找到正确的位置，这点会让人抓狂，其实解决这个问题很简单，我们每次使用流前必须将Stream.Position设置成0就行了，但是这还不能根本上解决问题，最好的方法就是用Using语句将流对象包裹起来，用完后关闭回收即可。

+ ReadTimeout	
获取或设置一个值（以毫秒为单位），该值确定流在超时前将尝试读取的时间。

+ WriteTimeout	
获取或设置一个值（以毫秒为单位），该值确定流在超时前将尝试写入多长时间。

## 方法

+ Flush()	
当在派生类中重写时，将清除该流的所有缓冲区，并使得所有缓冲数据被写入到基础设备。

> 某些流实现对基础数据执行本地缓冲以提高性能。 对于此类流，可以使用 Flush 或 FlushAsync 方法来清除缓冲区，并确保所有数据都已写入基础数据源或存储库。

> 当使用 StreamWriter 或 BinaryWriter 类时，不要刷新 Stream 基对象。而应使用该类的 Flush 或 Close 方法，此方法确保首先将该数据刷新至基础流，然后再将其写入文件。

+ int Read(Byte[] buffer, Int32 offset, Int32 count)	
当在派生类中重写时，从当前流读取字节序列，并将此流中的位置提升读取的字节数。
> 这个方法包含了3个关键的参数：
> 
> + buffer 缓冲字节数组，此数组中 offset 和 (offset + count - 1) 之间的值被从当前源中读取的字节所替换；
> 
> + offset 位移偏量，从buffer的offset处开始存储从流中读取的数据；
> 
> + count 读取字节个数，要从当前流中最多读取的字节数；
>
> 返回值：读入缓冲区中的总字节数，总字节数可能小于请求的字节数，如果已到达流结尾，则为零 (0)；

+ Seek(long offset , SeekOrigin origin)
当在派生类中重写时，设置当前流中的位置。
> + offset: 相对于 origin 参数的字节偏移量。
> + origin: SeekOrigin 类型的值，指示用于获取新位置的参考点。
>
> 如果 offset 为负，则要求新位置位于 origin 指定的位置之前，其间隔相差 offset 指定的字节数。如果 offset 为零 (0)，则要求新位置位于由 origin 指定的位置处。
> 
> 如果 offset 为正，则要求新位置位于 origin 指定的位置之后，其间隔相差 offset 指定的字节数.
> 
>  + Stream. Seek(-3,Origin.End);  表示在流末端往前数第3个位置
> 
>  + Stream. Seek(0,Origin.Begin); 表示在流的开头位置
> 
>  + Stream. Seek(3,Orig`in.Current); 表示在流的当前位置往后数第三个位置
>

+ void Write (byte[] buffer, int offset, int count); 向当前流中写入字节序列，并将此流中的当前位置提升写入的字节数。
> + buffer: 字节数组。 此方法将 count 个字节从 buffer 复制到当前流
>
>  + offset: buffer 中的字节偏移量，从此处开始将字节复制到当前流;
>
>  + count: 要写入当前流的字节数
>
> 如果写入操作成功，则流中的位置将按写入的字节数前进。 如果发生异常，则流中的位置保持不变。

# TextReader

是一个抽象类。 因此，不要在代码中实例化它。 派生类（StreamReader 和 StringReader）必须至少实现 Peek() 和 Read() 方法，才能成为TextReader有用的实例。

此类型实现 IDisposable 接口。 使用完派生自此类型的任何类型后，应直接或间接释放它。 若要直接释放类型，请在 try/catch 块中调用其 Dispose 方法。 若要间接释放类型，请使用 using（在 C# 中）等语言构造

## 方法

+ void Close ()：关闭 TextReader 并释放与该 TextReader 关联的所有系统资源；
+ void Dispose ()：释放由 TextReader 对象使用的所有资源；假如TextReader中持有stream或其他对象，当TextReader执行了Dispose方法时，stream对象也被回收了；
+ int Peek ()：读取下一个字符，而不更改读取器状态或字符源。 返回下一个可用字符，而实际上并不从读取器中读取此字符。
> 返回值：一个表示下一个要读取的字符的整数；如果没有更多可读取的字符或该读取器不支持查找，则为 -1。
> 该方法 Peek 返回整数值，以确定文件末尾还是发生了另一个错误。 这样，用户就可以先检查返回的值是否为 -1，然后再将其强制转换为 Char 类型。

+ int Read ()：读取文本读取器中的下一个字符并使该字符的位置前移一个字符。

> 返回值：文本读取器中的下一个字符，或为 -1（如果没有更多可用字符）。 默认实现将返回 -1。
>
> read()方法使指针指向下个字符，但是peek()还是指向原来那个字符
>

~~~C#

string text = "abc\nabc";


using (TextReader reader = new StringReader(text))
{
    while (reader.Peek() != -1)
    {
        Console.WriteLine("Peek = {0}", (char)reader.Peek());
        Console.WriteLine("Read = {0}", (char)reader.Read());
    }
    reader.Close();
}

using (TextReader reader = new StringReader(text))
{
    char[] charBuffer = new char[3];
    int data = reader.ReadBlock(charBuffer, 0, 3);
    for (int i = 0; i < charBuffer.Length; i++)
    {
        Console.WriteLine("通过readBlock读出的数据：{0}", charBuffer[i]);
    }
    reader.Close();
}

using (TextReader reader = new StringReader(text))
{
    string lineData = reader.ReadLine();
    Console.WriteLine("第一行的数据为:{0}", lineData);
    reader.Close();
}

using (TextReader reader = new StringReader(text))
{
    string allData = reader.ReadToEnd();
    Console.WriteLine("全部的数据为:{0}", allData);
    reader.Close();
}

Console.ReadLine();
~~~

# StreamReader
实现一个 TextReader，使其以一种特定的编码从字节流中读取字符。

StreamReader 设计用于特定编码中的***字符-Char**输入，而 Stream 类设计用于**字节-byte**输入和输出。 用于 StreamReader 从标准文本文件读取信息行。

StreamReader 除非另行指定，否则默认为 UTF-8 编码，而不是默认为当前系统的 ANSI 代码页。 UTF-8 正确处理 Unicode 字符，并在操作系统的本地化版本上提供一致的结果。

 如果使用CurrentEncoding属性获取当前字符编码 ，则值在执行第一个 Read 方法之后才可靠，因为编码自动检测在首次调用方法之前不会完成。

 ## 构造函数

 + StreamReader(Stream)	：为指定的流初始化 StreamReader 类的新实例。
 + StreamReader(Stream, Encoding)：用指定的字符编码为指定的流初始化 StreamReader 类的一个新实例。
 + StreamReader(String, Encoding)：用指定的字符编码，为指定的文件名初始化 StreamReader 类的一个新实例。
>  这里的string对象不是简单的字符串而是具体文件的地址,然后根据用户选择编码去读取流中的数据

## 属性

+ BaseStream：返回基础流。
~~~C#
FileStream fs = new FileStream ( "D:\\TextReader.txt", FileMode.Open , FileAccess.Read ) ; 
StreamReader sr= new StreamReader ( fs ) ; 
//本例中的BaseStream就是FileStream
sr.BaseStream.Seek (0 , SeekOrigin.Begin ) ;
~~~
+ CurrentEncoding：获取当前 StreamReader 对象正在使用的当前字符编码
+ EndOfStream：获取一个值，该值指示当前的流位置是否在流结尾，如果当前流位置位于流的末尾，则为 true；否则为 false。


~~~c#

//文件地址
string txtFilePath = "D:\\TextReader.txt";
//定义char数组
char[] charBuffer2 = new char[3];

//利用FileStream类将文件文本数据变成流然后放入StreamReader构造函数中
using (FileStream stream = File.OpenRead(txtFilePath))
{
    using (StreamReader reader = new StreamReader(stream))
    {
        //StreamReader.Read()方法
        DisplayResultStringByUsingRead(reader);
    }
}

using (FileStream stream = File.OpenRead(txtFilePath))
{
    //使用Encoding.ASCII来尝试下
    using (StreamReader reader = new StreamReader(stream, Encoding.ASCII, false))
    {
        //StreamReader.ReadBlock()方法
        DisplayResultStringByUsingReadBlock(reader);
    }
}

//尝试用文件定位直接得到StreamReader，顺便使用 Encoding.Default
using (StreamReader reader = new StreamReader(txtFilePath, Encoding.Default, false, 123))
{
    //StreamReader.ReadLine()方法
    DisplayResultStringByUsingReadLine(reader);
}

//也可以通过File.OpenText方法直接获取到StreamReader对象
using (StreamReader reader = File.OpenText(txtFilePath))
{
    //StreamReader.ReadLine()方法
    DisplayResultStringByUsingReadLine(reader);
}

Console.ReadLine();


        /// <summary>
        /// 使用StreamReader.Read()方法
        /// </summary>
        /// <param name="reader"></param>
public static void DisplayResultStringByUsingRead(StreamReader reader)
{
    int readChar = 0;
    string result = string.Empty;
    while ((readChar = reader.Read()) != -1)
    {
        result += (char)readChar;
    }
    Console.WriteLine("使用StreamReader.Read()方法得到Text文件中的数据为 : {0}", result);
}

/// <summary>
/// 使用StreamReader.ReadBlock()方法
/// </summary>
/// <param name="reader"></param>
public static void DisplayResultStringByUsingReadBlock(StreamReader reader)
{
    char[] charBuffer = new char[10];
    string result = string.Empty;
    reader.ReadBlock(charBuffer, 0, 10);
    for (int i = 0; i < charBuffer.Length; i++)
    {
        result += charBuffer[i];
    }
    Console.WriteLine("使用StreamReader.ReadBlock()方法得到Text文件中前10个数据为 : {0}", result);
}


/// <summary>
/// 使用StreamReader.ReadLine()方法
/// </summary>
/// <param name="reader"></param>
public static void DisplayResultStringByUsingReadLine(StreamReader reader)
{
    int i = 1;
    string resultString = string.Empty;
    while ((resultString = reader.ReadLine()) != null)
    {
        Console.WriteLine("使用StreamReader.Read()方法得到Text文件中第{1}行的数据为 : {0}", resultString, i);
        i++;
    }
}
~~~

# TextWriter 

TextWriter是抽象基类，子类 StreamWriter 和 StringWriter，分别将字符写入流和字符串。 派生类必须实现 Write(Char) 方法，才能创建一个有用的实例 TextWriter。

## 构造函数

+ TextWriter()：初始化 TextWriter 类的新实例。

+ TextWriter(IFormatProvider)：使用指定的格式提供程序初始化 TextWriter 类的新实例。
> 使用此构造函数创建的实例，在调用 Write 和 WriteLine 方法时使用FormatProvider属性值，设置的区域性特定格式。

## 属性

+ Encoding：当在派生类中重写时，返回用来写输出的该字符编码。

+ FormatProvider：获取控制格式设置的对象。

+ NewLine：获取或设置由当前 TextWriter 使用的行结束符字符串。

# StreamWriter

StreamWriter 以一种特定的编码向流中写入字符。 除非另外指定，否则默认为使用UTF8Encoding实例 。 UTF8Encoding实例在构造时没有字节顺序标记 (BOM) ，因此其 GetPreamble 方法返回一个空字节数组。 此构造函数的默认 UTF-8 编码对无效字节引发异常。 此行为不同于属性中的编码对象提供的行为 Encoding.UTF8 。 若要指定一个 BOM 并确定无效字节是否引发了异常，请使用接受编码对象作为参数的构造函数。

## 构造函数

+ StreamWriter(Stream)：使用 UTF-8 编码及默认的缓冲区大小，为指定的流初始化 StreamWriter 类的新实例。

+ StreamWriter(Stream, Encoding)：使用指定的编码及默认的缓冲区大小，为指定的流初始化 StreamWriter 类的新实例。
+ StreamWriter (string path, bool append)：用默认编码和缓冲区大小，为指定的文件初始化 StreamWriter 类的一个新实例。
  >  若要追加数据到该文件中，append为 true；若要覆盖该文件，append为 false。 如果指定的文件不存在，该参数无效，且构造函数将创建一个新文件。

## 属性

+ AutoFlush：获取或设置一个值，该值指示 StreamWriter 在每次调用 Write(Char) 之后是否都将其缓冲区刷新到基础流。

+ BaseStream：获取同后备存储连接的基础流。

+ Encoding：获取在其中写入输出的 Encoding。

+ FormatProvider：获取控制格式设置的对象。

+ NewLine：获取或设置由当前 TextWriter 使用的行结束符字符串。


# FileStream
