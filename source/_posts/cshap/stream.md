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

+ Stream.Seek(Int64 offset , SeekOrigin origin)
当在派生类中重写时，设置当前流中的位置。
> + offset: 相对于 origin 参数的字节偏移量。
> + origin: SeekOrigin 类型的值，指示用于获取新位置的参考点。
>
> 如果 offset 为负，则要求新位置位于 origin 指定的位置之前，其间隔相差 offset 指定的字节数。如果 offset 为零 (0)，则要求新位置位于由 origin 指定的位置处。
> 
> 如果 offset 为正，则要求新位置位于 origin 指定的位置之后，其间隔相差 offset 指定的字节数.
> 
> Stream. Seek(-3,Origin.End);  表示在流末端往前数第3个位置
> 
> Stream. Seek(0,Origin.Begin); 表示在流的开头位置
> 
> Stream. Seek(3,Orig`in.Current); 表示在流的当前位置往后数第三个位置
>

+ void Write (byte[] buffer, int offset, int count); 向当前流中写入字节序列，并将此流中的当前位置提升写入的字节数。
> + buffer: 字节数组。 此方法将 count 个字节从 buffer 复制到当前流
>
> offset: buffer 中的字节偏移量，从此处开始将字节复制到当前流;
>
> count: 要写入当前流的字节数
>
> 如果写入操作成功，则流中的位置将按写入的字节数前进。 如果发生异常，则流中的位置保持不变。





