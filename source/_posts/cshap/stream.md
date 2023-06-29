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

使用FileStream类读取、写入、打开和关闭文件系统上的文件，并操作其他与文件相关的操作系统句柄，包括管道、标准输入和标准输出。 可以使用 Read、 Write、 CopyTo和 Flush 方法执行同步操作，或使用 ReadAsync、 WriteAsync、 CopyToAsync和 FlushAsync 方法执行异步操作。 使用异步方法执行资源密集型文件操作，而不会阻止主线程。 

属性 IsAsync 检测文件句柄是否已异步打开。 使用具有 isAsync、 或 options 参数的构造函数创建FileStream类的实例时，useAsync可以指定此值。 当IsAsync属性为 true时，流利用重叠的 I/O 以异步方式执行文件操作。 

当IsAsync属性为 false 并且你调用异步读取和写入操作时，UI 线程仍不会被阻止，但实际 I/O 操作是同步执行的。

方法 Seek 支持对文件的随机访问。 Seek 允许将读/写位置移动到文件中的任何位置。

当对象 FileStream 在其句柄上没有独占保留时，另一个线程可以同时访问文件句柄，并更改与文件句柄关联的操作系统文件指针的位置。 在这种情况下，对象中的 FileStream 缓存位置和缓冲区中的缓存数据可能会受到威胁。 对象 FileStream 定期对访问缓存缓冲区的方法执行检查，以确保操作系统的句柄位置与对象使用的 FileStream 缓存位置相同。

如果在调用 Read 方法时检测到句柄位置的意外更改，.NET Framework放弃缓冲区的内容，并从文件中再次读取流。 这可能会影响性能，具体取决于文件的大小以及可能影响文件流位置的任何其他进程。

如果在调用 Write 方法时检测到句柄位置的意外更改，则会丢弃缓冲区的内容并 IOException 引发异常。

## 构造函数

+ FileStream(SafeFileHandle, FileAccess)：使用指定的读/写权限为指定的文件句柄初始化 FileStream 类的新实例。
> SafeFileHandle
当前 FileStream 对象将封装的文件的文件句柄。

> FileAccess
枚举值的按位组合，它用于设置 FileStream 对象的 CanRead 和 CanWrite 属性。

+ FileStream(String, FileMode, FileAccess, FileShare, Int32, FileOptions):使用指定的路径、创建模式、读/写和共享权限、其他 FileStreams 可以具有的对此文件的访问权限、缓冲区大小和附加文件选项初始化 FileStream 类的新实例。

> String: 当前 FileStream 对象将封装的文件的相对路径或绝对路径。
> 
> FileMode: 用于确定文件的打开或创建方式的枚举值之一。
> 
> FileAccess: 枚举值的按位组合，这些枚举值确定 FileStream 对象访问文件的方式。 该常数还可以确定由 FileStream 对象的 CanRead 和 CanWrite 属性返回的值。 如果 path 指定磁盘文件，则 CanSeek 为 true。
> 
> FileShare: 枚举值的按位组合，这些枚举值确定进程共享文件的方式。
>
> Int32: 一个大于零的正 Int32 值，表示缓冲区大小。 默认缓冲区大小为 4096
> FileOptions: 枚举值的按位组合，它用于指定其他文件选项

### 参数说明

+ FileMode：指定操作系统打开文件的方式。
> Append: 若存在文件，则打开该文件并查找到文件尾，或者创建一个新文件。 FileMode.Append 只能与 FileAccess.Write 一起使用。 试图查找文件尾之前的位置时会引发 IOException 异常，并且任何试图读取的操作都会失败并引发 NotSupportedException 异常。
> 
> Create: 指定操作系统应创建新文件。 如果此文件已存在，则会将其覆盖。 这需要 Write 权限。 
> FileMode.Create 等效于这样的请求：如果文件不存在，则使用 CreateNew；否则使用 Truncate。 如果该文件已存在但为隐藏文件，则将引发 UnauthorizedAccessException异常。
> 
> CreateNew: 指定操作系统应创建新文件。 这需要 Write 权限。 如果文件已存在，则将引发 IOException异常。
> 
> Open: 指定操作系统应打开现有文件。 打开文件的能力取决于 FileAccess 枚举所指定的值。 如果文件不存在，引发一个 FileNotFoundException 异常。
> 
> OpenOrCreate: 指定操作系统应打开文件（如果文件存在）；否则，应创建新文件。 如果用 FileAccess.Read 打开文件，则需要 Read权限。 如果文件访问为 FileAccess.Write，则需要 Write权限。 如果用 FileAccess.ReadWrite 打开文件，则同时需要 Read 和 Write权限
> 
> Truncate: 指定操作系统应打开现有文件。 该文件被打开时，将被截断为零字节大小。 这需要 Write 权限。 尝试从使用 FileMode.Truncate 打开的文件中进行读取将导致 ArgumentException 异常。

+ FileAccess: 定义文件的读取、写入或读/写访问权限的常量。

> Read: 对文件的读访问。 可从文件中读取数据。 与 Write 组合以进行读写访问。
> 
> ReadWrite: 对文件的读写访问权限。 可从文件读取数据和将数据写入文件。
> 
> Write: 文件的写访问。 可将数据写入文件。 与 Read 组合以进行读写访问。
>

+ FileShare: 包含用于控制其他 FileStream 对象对同一文件可以具有的访问类型的常数。

> Delete: 允许随后删除文件。
> 
> Inheritable: 使文件句柄可由子进程继承。 Win32 不直接支持此功能。
> 
> None: 谢绝共享当前文件。 文件关闭前，打开该文件的任何请求（由此进程或另一进程发出的请求）都将失败。
> 
> Read: 允许随后打开文件读取。 如果未指定此标志，则文件关闭前，任何打开该文件以进行读取的请求（由此进程或另一进程发出的请求）都将失败。 但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。
> 
> ReadWrite: 允许随后打开文件读取或写入。 如果未指定此标志，则文件关闭前，任何打开该文件以进行读取或写入的请求（由此进程或另一进程发出）都将失败。 但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。
> 
> Write: 允许随后打开文件写入。 如果未指定此标志，则文件关闭前，任何打开该文件以进行写入的请求（由此进程或另一进过程发出的请求）都将失败。 但是，即使指定了此标志，仍可能需要附加权限才能够访问该文件。

+ FileOptions

> Asynchronous: 指示文件可用于异步读取和写入
> 
> DeleteOnClose: 指示当不再使用某个文件时，自动删除该文件。
> 
> Encrypted: 指示文件是加密的，只能通过用于加密的同一用户帐户来解密。
> 
> None: 指示在生成 FileStream 对象时，不应使用其他选项。
> 
> RandomAccess: 指示随机访问文件。 系统可将此选项用作优化文件缓存的提示。
> 
> SequentialScan: 指示按从头到尾的顺序访问文件。 系统可将此选项用作优化文件缓存的提示。 如果应用程序移动用于随机访问的文件指针，可能不发生优化缓存，但仍然保证操作的正确性。 如果指定此标志，可提升某些案例中的性能。
> 
> WriteThrough: 指示系统应通过任何中间缓存、直接写入磁盘。

> 指定 FileOptions.SequentialScan 标志可以提高使用顺序访问读取大型文件的应用程序的性能。 对于主要按顺序读取大型文件，但偶尔跳过小范围字节的应用程序，性能提升可能更为明显。


## 属性

+ IsAsync：获取一个值，它指示 FileStream 是异步打开还是同步打开的。

+ Length：获取流的长度（以字节为单位）。

+ Name：获取 FileStream 中已打开的文件的绝对路径。

+ Position：获取或设置此流的当前位置。

+ SafeFileHandle：获取 SafeFileHandle 对象，它代表当前 FileStream 对象所封装的文件的操作系统文件句柄。

## 方法

属于FileStream独有的方法

+ FileSecurity GetAccessControl(FileStream)： 返回文件的安全信息。
> FileStream: 一个要从中获取安全信息的现有文件
>
+ void SetAccessControl(FileSecurity fileSecurity): 和GetAccessControl很相似，ACL技术会在以后单独介绍

+ Lock (long position, long length);防止其他进程读取或写入 FileStream。

>  这个Lock方法和线程中的Look关键字很不一样，它能够锁住文件中的某一部分，非常的强悍！用了这个方法我们能够精确锁定住我们需要锁住的文件的部分内容

+ void Unlock (long position,long length):正好和lock方法相反，对于文件部分的解锁

# MemoryStream

创建一个流，其后备存储为内存。

使用无符号字节数组创建的内存流提供不可调整大小的数据流。 使用字节数组时，既不能追加流，也不能收缩流，不过，根据传递到构造函数的参数，可以修改现有内容。 空内存流可调整大小，可以写入和读取。

由于MemoryStream是通过无符号字节数组组成的，可以说MemoryStream的性能可以算比较出色，所以它担当起了一些其他流进行数据交换时的中间工作，同时可降低应用程序中对临时缓冲区和临时文件的需要.

## 构造函数

+ MemoryStream()：使用初始化为零的可扩展容量初始化 MemoryStream 类的新实例。

> 使用初始化为零的可扩展容量初始化 MemoryStream 类的新实例。
>
> CanSeek属性CanRead和CanWrite属性都设置为 true。
> 
>使用 SetLength 该方法将长度设置为大于当前流的容量的值时，当前流的容量会自动增加。
> 
> 此构造函数公开返回的基础流 GetBuffer 。

+ MemoryStream(Byte[])：基于指定的字节数组初始化 MemoryStream 类的无法调整大小的新实例。

> 基于指定的字节数组初始化 MemoryStream 类的无法调整大小的新实例。
> 
> CanSeek属性CanRead和CanWrite属性都设置为 true。 Capacity 设置为指定字节数组的长度。 可将新流写入，但不可调整大小。
> 
> 流的长度不能设置为大于指定字节数组的初始长度的值;但是， (可以看到 SetLength) 截断流。
> 
> 此构造函数不公开基础流。 GetBuffer throws UnauthorizedAccessException.

+ MemoryStream(Int32)：使用按指定要求初始化的可扩展容量初始化 MemoryStream 类的新实例。

>使用按指定要求初始化的可扩展容量初始化 MemoryStream 类的新实例。
>
>CanSeek属性CanRead和CanWrite属性都设置为 true。
>
>使用 SetLength 此方法将长度设置为大于当前流的容量的值时，容量会自动增加。
>
>此构造函数公开返回的基础流 GetBuffer

+ MemoryStream(Byte[], Int32, Int32, Boolean, Boolean)：在 MemoryStream 属性和调用 CanWrite 的能力按指定设置的状态下，基于字节数组的指定区域初始化 GetBuffer() 类的新实例。

> 和CanRead、CanSeek属性都设置为true。 将 Capacity 设置为 count。
> 
> 可以将新流实例写入其中，但 Capacity 基础字节数组无法更改。 流的长度不能设置为大于指定字节数组的初始长度的值;

## 方法

+ virtual byte[] GetBuffer()： 返回从中创建此流的无符号字节的数组

> 请注意，缓冲区包含可能未使用的已分配字节。 例如，如果将字符串“test”写入 MemoryStream 对象，则返回 GetBuffer 的缓冲区长度为 256，而不是 4，未使用 252 个字节。 若要仅获取缓冲区中的数据，请使用 ToArray 该方法;但是， ToArray 会在内存中创建数据的副本。
>
> 这个方法使用时需要小心，因为这个方法返回无符号字节数组，也就是说，即使我只输入几个字符例如”HellowWorld”我们只希望返回11个数据就行，可是这个方法会把整个缓冲区的数据，包括那些已经分配但是实际上没有用到的字节数据都返回出来;
>
+ virtual void WriteTo(Stream stream): 将此内存流的整个内容写入到另一个流中。

> memoryStream常用起中间流的作用，所以在处理完后将内存流写入其他流中;
>

# BufferedStream

将缓冲层添加到另一个流上的读取和写入操作。
BufferedStream能够实现流的缓存，换句话说也就是在内存中能够缓存一定的数据而不是

时时给系统带来负担，同时BufferedStream可以对缓存中的数据进行写入或是读取，所以对流的性能带来一定的提升，但是无法同时进行读取或写入工作，如果不使用缓冲区也行，BufferedStream能够保证不用缓冲区时不会降低因缓冲区带来的读取或写入性能的下降。

为什么MemoryStream 同样也是在内存中对流进行操作，和BufferedStream有什么区别呢？BufferedStream并不是将所有内容都存放到内存中，而MemoryStream则是。BufferedStream必须跟其他流如FileStream结合使用，而MemoryStream则不用，聪明的你肯定能够想到，BufferedStream必然类似于一个流的包装类，对流进行”缓存功能的扩展包装”，所以BufferedStream的优势不仅体现在其原有的缓存功能上，更体现在如何帮助原有类实现其功能的扩展层面上。

# NetworkStream

如果服务器和客户端之间基于TCP连接的，他们之间能够依靠一个稳定的字节流进行相互传输信息，这也是

NetworkStream的最关键的作用，有了这个神奇的协议，NetWorkStream便能向其他流一样在网络中（进行点对点的传输）

> + NetworkStream只能用在具有Tcp/IP协议之中，如果用在UDP中编译不报错，会报异常
> 
> + NetworkStream 是面向连接的
> 
> + 在网络中利用流的形式传递信息
> 
> + 必须借助Socket (也称之为流式socket)，或使用一些返回的返回值，例如TcpClient类的GetStream方法
> 
> + 用法和普通流方法几乎一模一样，但具有特殊性

可以在类实例 NetworkStream 上同时执行读取和写入操作，而无需同步。 只要写入操作有一个唯一线程，读取操作有一个唯一线程，读取和写入线程之间就不会有交叉干扰，也不需要同步。













