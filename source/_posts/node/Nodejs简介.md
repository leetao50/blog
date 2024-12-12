---
title: Nodejs简介
date: 2024-06-27 09:04:35
tags:
---

# 新建的Node.JS项目工程

虽然你可以创建一个JS文件，并执行：node file.js，但我建议你使用 npm init 来先创建一个node项目，这会是非常好的习惯。

cmd下执行： npm init 即可创建一个Node项目。这时会产生一个package.json文件。package.json里面，会包含该项目的基本信息，以及项目所需要的三方模块信息。

```json
//About to write to /Users/galsang/Documents/code/sample/nodedemo/package.json:
{
  "name": "cnp",
  "version": "2.1.2",
  "description": "create nodejs project ",
  "main": "example.ts",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "galsang",
  "license": "ISC"
}
```
这个文件是很有用的，当将来此项目要在别的机器上使用时，依赖它可以实现快速安装部署。

# 模块使用方法

NodeJS有丰富的三方模块，借助这些模块，可以快速的开发各类应用。这使用Nodejs可以进行很便捷、快速的开发。

使用npm可以搜索、安装、卸载模块。

## 安装与加载模块-搜索模块

搜索模块用：npm search 模块名

```shell
galsang@galsangdeMacBook-Pro nodedemo % npm search babel

@babel/runtime
babel's modular runtime helpers
Version 7.24.8 published 2024-07-11 by nicolo-ribaudo
Maintainers: hzoo existentialism nicolo-ribaudo jlhwung
https://npm.im/@babel/runtime

@babel/core
Babel compiler core.
Version 7.24.9 published 2024-07-15 by nicolo-ribaudo
Maintainers: hzoo existentialism nicolo-ribaudo jlhwung
Keywords: 6to5 babel classes const es6 harmony let modules transpile transpiler var babel-core compiler
https://npm.im/@babel/core

@babel/parser
A JavaScript parser
Version 7.24.8 published 2024-07-11 by nicolo-ribaudo
Maintainers: hzoo existentialism nicolo-ribaudo jlhwung
Keywords: babel javascript parser tc39 ecmascript @babel/parser
https://npm.im/@babel/parser

```

一般来说，会搜到很多内容，为了找到自己最需要的，搜索时可以用正则表达式进行匹配，如：

```shell
npm search /^@babel/parser
```
## 安装与加载模块-安装模块

安装模块用：npm install 模块名

安装后，便可以使用require语句进行加载：

# 创建我们自己的模块
我们可以使用Node内置模块或第三方模块，当然也可以创建我们自己的模块。

例，写一个简单的模块，代码如下：
~~~JavaScript
exports.method1 = function(){
    return "hello 1"
}

exports.method2 = function(){
    return "hello 2"
}
~~~


该模块提供两个方法：method1返回hello1字符，method2返回hello2字符。保存为module.js

再在另一个文件中调用它，调用代码：

```javascript
var module = require("./module");

console.log(module.method1());
console.log(module.method2());
```

## 卸载模块
模块加载后会缓存起来，任何时候都可以方便的使用。

但有时，对于有些模块，如果不想继续使用。可以进行卸载。

或是模块会在外部更新，需要获取更新的模块内容？那么，这里需要卸载模块、以便重新加载。

卸载模块代码如下：
```javascript
var module = require("./module");

console.log(module.method1());
console.log(module.method2());

console.log(require.cache);
delete require.cache[require.resolve("./module")];
console.log("已卸载")
console.log(require.cache);
```

resolve可以获取模块的完整路径。从缓存中卸载掉时，需要用完整路径。

# 加载一组模块
有时候，希望通过一个require来加载一个目录下的相关文件。

注：这个方法通常被用来作为组织web应用的架构技巧。

为达到这个目的，需要如此操作：

例：建立一个目录group，在此目录中创建一个index.js、one.js及two.js文件。

```console
-group
    |-index.js
    |-one.js
    |-two.js
```

文件内容如下：
```javascript
//index.js
module.exports={
    one:require("./one.js"),
    two:require("./two.js")
}
//这个模块，用于组合one.js和two.js两个子模块，将他们合并一起做输出。

//one.js
module.exports=function(){
    console.log("noe");
}

//two.js
module.exports=function(){
    console.log("two");
}
```

这时，只要require group目录，便可调用到one.js和two.js：

```javascript
var group = require("./group");

group.one();
group.two();

```

以此类推，还可以使用更多的group目录下的文件、更多的接口，实现不同的功能。

# 标准IO及console对像

IO即输入输出，Nodejs的IO操作，通过process.stdout、process.stdin来操作。
下面的例子，将简单展示这两个函数的用法。程序将接收输入，处理并做输出：

```javascript
//jsio.js
process.stdin.resume();
process.stdin.setEncoding('utf8');
process.stdin.on('data',function(text){
process.stdout.write(text.toUpperCase());
});
```
保存代码为jsio.js，用node jsio.js执行，这时程序会等待输入，输入完成后回车。

程序会把输入的字符变为大写并输出。

执行效果：

```console
node jsio.js
jsio
JSIO
```
console在nodejs中主要用于打印日志消息。

console.log有几种不同的方法：

console.log、console.info、console.error、console.warn，

这几个函数的用法相同，输出内容无区别。只是不同的方法，会将信息写入相关的输出流中，如何进行深入调试，可以在相应的pipe中获取到。

console的几个方法通常用于程序中输出内容和调试，

而在调试方面，console还有一个更好的函数：console.trace()，可以详细的打印出堆栈、调用信息。用于调试的话会更加详细实用。

# 基准测试
console.time()和console.timeEnd()可以对某准测试，即：可对某个区间的代码执行耗时进行计算。

```javascript
console.time("test");

for(i=0;i<1000;i++){
    var test = "www.jshaman.com";
}

console.timeEnd("test")

```
它的含意是计算出了这个区间内代码执行消耗了多少时间。

# 操作系统与命令
Nodejs有一些内置的方法可以查询操作系统信息：

如：
process.arch获取到系统是32位还是64位，process.platform可获取系统的类型。

process.memoryUsage()可以获取当前进程的内存使用情况，它有三个属性：
+ rss：常驻内存大小；
+ heapTotal：动态分配的可用内存；
+ heapUsed：已使用堆大小。

注：输出数字单位是字节。除1024获取的是KB，再除1024获取的是MB，再除1024获取的是GB。

使用process.argv可以从命令行获取参数
```javascript

//输出整体参数
console.log(process.argv);

//分别输入各参数
if(process.argv.length >0){
    //遍历参数
    process.argv.forEach(function(arg,index){
        console.log(arg,index);
    });
}
```
执行结果：

![图 0](../c1dcb961c0bbcd7ebe0113b6dff09f62f1cb6fc32eb1727ac3091b47f69c2ebd.png)  

从执行结果可以看到，隐式的还有两个参数：node.exe本身、脚本。加上参数arg1、arg2，实际上共有4个参数。

# 定时器，使用timer延迟执行
## setTimeout
在nodejs中，通过setTimeout函数可以达到延迟执行的效果，这个函数也常被称为定时器。
```javascript
console.log( (new Date()).getSeconds() );

    setTimeout(function(){
        console.log( (new Date()).getSeconds() );
        console.log("hello world");
        //延迟一秒执行
    },1000);

/* 输出
7
8
hello world
*/
```

可以看到，执行时，先输出了当时时间的秒数，过1秒后，输入出秒和hello world，间隔正是1秒。上面的参数中1000，单位是毫秒，即1秒。

在nodejs中，常用setTimeout来实现异步操作。

## bind
还有一种高级的用法，看例程：

```javascript
function bomb(){
    this.message = "bomb";
}

bomb.prototype.explode =function(){
    console.log(this.message);
}

var bomb = new bomb();

setTimeout(bomb.explode.bind(bomb),1000);
```
即：使用bind可以确保这个方法绑定到正确的对象上，这样可以访问到这个对象的内部属性。

执行效果：
```cmd
node test
bomb

```

## clearTimeout

通过clearTimeout函数，可以清除掉定时器。比如说setTimeout设定了一个定时器，将在1秒后触发某个操作，如果在未触发之前，clearTimeout函数取消这个定时器操作。

将上面的代码稍做修改：
```javascript

function bomb(){
    this.message = "bomb";
}

bomb.prototype.explode =function(){
    console.log(this.message);
}

var bomb = new bomb();
var timeoutid = setTimeout(bomb.explode.bind(bomb),1000);

//取消定时器
clearTimeout(timeoutid);

```
这样，就不会触发1秒后的操作。

## setInterval
setTimerout，会延时一定时间后执行一个操作，只执行一次。

而setInterval，可以不停的按时间间隔循环执行。循环执行到什么时候呢？直到程序退出，或直到使用clearInterval()函数取消这个定时器。
```javascript

console.log( (new Date()).getSeconds() );

var interval_id = setInterval(function(){

    console.log( (new Date()).getSeconds() );
    console.log("hello world");

},1000);

clearInterval(interval_id);
```

# 你了解buffer吗？
Buffer是NodeJS的重要数据类型，很有广泛的应用。Buffer是代表原始堆的分配额的数据类型。在NodeJS中以类数组的方式使用。

比如，用法示例：
```javascript

var buf = new Buffer(255);
buf[0] = 23;

console.log(buf[0]);

```
解释：分配255个字节，第一个字节写入数据23。

Nodejs中文件操作、网络数据传输如Post数据，通常默认数据格式都是Buffer。

如：
```javascript
var fs = require("fs");
fs.readFile("./test7.js",function(er,buf){
	console.log(Buffer.isBuffer(buf));
});

/*输出
true
*/
```
可以看到，默认读取到的数据buf是Buffer类型。

Buffer类型转换为其它格式
```javascript
var user = 'wangliwen';
var pass = 'jshaman.com';
var auth_str = user + ':' + pass;

//不经预定义大小，直接传入字符串来创建buffer
var buf = new Buffer(auth_str);

//加密过程：转为base64编码
var encode = buf.toString('base64');
console.log(encode);

//解密过程
var decode = new Buffer(encode,'base64').toString();
console.log(decode);

```
注：toString()函数，默认的是转化为utf-8编码。还可转为ascii、utf16le、base64、hex等。

实际用途：简单的加解密算法、加密数据传输，如登录校验时。

# 可用于压缩、加密的zlib
zlib是nodejs内置的模块，有deflate、inflate函数，使用的是gzip算法，可用于压缩和解压，也可用于数据加密、解密。

如下示例：
```javascript
var zlib = require("zlib");

//压缩
zlib.deflate("jshaman.com is a good web,used for obfuscating js code.",
    function(er,deflate_buf){
        console.log(deflate_buf.toString());

        //解压
        zlib.inflate(deflate_buf,function(er,inflat_buf){
            console.log(inflat_buf.toString());
        });
    }
);

```
可以看到字符串经压缩后，可形成一个乱码式的字符串，再解解压，又会还原为原来的字符。

那么，这个方法，就即可用于数据多压缩，又可用于加密码。

# 用EventEmitter触发和响应事件
Nodejs有一个重要的事件模块：EventEmitter。它在Nodejs的内置及第三方模块中被大量使用，许多Nodejs项目的架构都是用它实现的。可见，EventEmitter对于学习NodeJS非常重要。

下面，我们通过例程来理解和掌握EventEmitter。
```javascript

var EventEmitter = require('events').EventEmitter; 
var event = new EventEmitter(); 

//触发some_event事件
setTimeout(function() { 
    event.emit('some_event'); 
}, 1000); 

//响应some_event事件
event.on('some_event', function() { 
    console.log('some_event 事件触发'); 
}); 
```
这段代码，先初始化一个EventEmitter的实例，然后一秒后，触发一个事件，事件名称是some_event，并且在这个事件的响应函数中输出信息。

执行效果如图所示：

```cmd
some_event 事件触发
```
EventEmitter 的每个事件由一个事件名和若干个参数组成，事件名是一个字符串，通常表达一定的语义。对于每个事件，EventEmitter 支持 若干个事件监听器。

当事件触发时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递，如下例：

```javascript

var EventEmitter = require('events').EventEmitter; 
var emitter = new EventEmitter(); 
//响应some_event事件
emitter.on('some_event', function(arg1,arg2) { 
    console.log('some_event 事件触发'+arg1+arg2); 
}); 

emitter.on('some_event', function(arg1,arg2) { 
    console.log('some_event 事件触发'+arg1+arg2); 
}); 


//触发some_event事件
setTimeout(function() { 
    emitter.emit('some_event','参数1','参数2'); 
}, 1000); 

```
以上例子中，emitter 为事件 some_event 注册了两个事件监听器，然后触发了 some_event 事件。

运行结果中可以看到两个事件监听器回调函数被先后调用。 这就是EventEmitter最简单的用法。

# “流”是Node.js最强大的功能之一

流是Nodejs的高级应用，掌握流的使用，才能真正胜任NodeJS开发。Nodejs中，流是基于事件的API，用于管理和处理数据，而且效率很好！

## 什么是流？
流是Node.js应用程序中的一个基本概念，通过按顺序读取或写入输入和输出，实现高效的数据处理。它们非常适用于文件操作、网络通信和其他形式的端到端数据交换。

流的独特之处在于它以小的、连续的块来处理数据，而不是一次性将整个数据集加载到内存中。这种方法在处理大量数据时非常有益，因为文件大小可能超过可用内存。流使得以较小的片段处理数据成为可能，从而可以处理更大的文件。

![图 1](../95d717908726baa6259e8fc487b1bb40c54ef942d8487538a1a1fed73eaef514.png)  

如上图所示，数据通常以块或连续流的形式从流中读取。从流中读取的数据块可以存储在缓冲区中。缓冲区提供临时存储空间，用于保存数据块，直到进一步处理。




## 流的事件
所有的流对象都是 EventEmitter 的实例，流常用的事件有：

+ data - 当有数据可读时触发。

+ end - 没有更多的数据可读时触发。

+ error - 在接收和写入过程中发生错误时触发。

+ finish - 所有数据已被写入到底层系统时触发。

## 为什么要使用流？
流提供了与其他数据处理方法相比的两个关键优势。

1. 内存效率：使用流，处理前不需要将大量数据加载到内存中。相反，数据以较小的可管理块进行处理，减少了内存需求并有效利用了系统资源。

2. 时间效率：流使得数据一旦可用就能立即进行处理，而不需要等待整个有效负载的传输。这样可以实现更快的响应时间和改善整体性能。

## 流的四种类型
1. Readable - 可读流：
可读流允许从源（如文件或网络套接字）读取数据。它们按顺序发出数据块，并可以通过附加监听器到“data”事件来消费。可读流可以处于流动或暂停状态，取决于数据的消费方式。

```javascript
const fs = require('fs');

// Create a Readable stream from a file
const readStream = fs.createReadStream('the_princess_bride_input.txt', 'utf8');

// Readable stream 'data' event handler
readStream.on('data', (chunk) => {
  console.log(`Received chunk: ${chunk}`);
});

// Readable stream 'end' event handler
readStream.on('end', () => {
  console.log('Data reading complete.');
});

// Readable stream 'error' event handler
readStream.on('error', (err) => {
  console.error(`Error occurred: ${err}`);
});
```

我们将事件处理程序附加到可读流上以处理不同的事件。当数据块可供读取时，会触发 data 事件。当可读流完成从文件中读取所有数据时，会触发 end 事件。如果在读取过程中发生错误，则会触发 error 事件。

2. Writable - 可写流：
可写流处理将数据写入目标位置，例如文件或网络套接字。它们提供了像 write() 和 end() 这样的方法来向流发送数据。可写流可用于以分块方式写入大量数据，防止内存溢出。
```javascript
const fs = require('fs');

// Create a Writable stream to a file
const writeStream = fs.createWriteStream('the_princess_bride_output.txt');

// Writable stream 'finish' event handler
writeStream.on('finish', () => {
  console.log('Data writing complete.');
});

// Writable stream 'error' event handler
writeStream.on('error', (err) => {
  console.error(`Error occurred: ${err}`);
});

// Write a quote from "The  to the Writable stream
writeStream.write('As ');
writeStream.write('You ');
writeStream.write('Wish');
writeStream.end();
```
我们将事件处理程序附加到可写流上，以处理不同的事件。当可写流完成写入所有数据时，会触发 finish 事件。如果在写入过程中发生错误，则会触发 error 事件。使用 write() 方法将单个数据块写入可写流。在这个例子中，我们将字符串'As'、'You'和'Wish'写入流中。最后，我们调用 end() 来表示数据写入结束。

通过使用可写流并监听相应的事件，您可以高效地将数据写入目标位置，并在写入过程完成后执行任何必要的清理或后续操作。

3. Duplex - 双工流：
双工流代表了可读和可写流的组合。它们允许同时从源读取和写入数据。双工流是双向的，并在同时进行读取和写入的情况下提供了灵活性。

```javascript
const { Duplex } = require("stream");

class MyDuplex extends Duplex {
  constructor() {
    super();
    this.data = "";
    this.index = 0;
    this.len = 0;
  }
  
  _read(size) {
    // Readable side: push data to the stream
    const lastIndexToRead = Math.min(this.index + size, this.len);
    this.push(this.data.slice(this.index, lastIndexToRead));
    this.index = lastIndexToRead;
    if (size === 0) {
      // Signal the end of reading
      this.push(null);
    }
  }
  
  _write(chunk, encoding, next) {
    const stringVal = chunk.toString();
    console.log(`Writing chunk: ${stringVal}`);
    this.data += stringVal;
    this.len += stringVal.length;
    next();
  }
}

const duplexStream = new MyDuplex();
// Readable stream 'data' event handler
duplexStream.on("data", (chunk) => {
  console.log(`Received data:\n${chunk}`);
});

// Write data to the Duplex stream
// Make sure to use a quote from "The Princess Bride" for better performance :)
duplexStream.write("Hello.\n");
duplexStream.write("My name is Inigo Montoya.\n");
duplexStream.write("You killed my father.\n");
duplexStream.write("Prepare to die.\n");
// Signal writing ended
duplexStream.end();
```

我们定义了Duplex流的 _read() 和 _write() 方法来处理各自的操作。在这种情况下，我们将写入流和读取流绑定在一起，但这只是为了举例说明 - Duplex流支持独立的读取和写入流。

在 _read() 方法中，我们实现了双工流的可读端。我们使用 this.push() 将数据推送到流中，当大小变为0时，通过将null推送到流中来表示读取结束。

在 _write() 方法中，我们实现了Duplex流的可写端。我们处理接收到的数据块并将其添加到内部缓冲区。调用 next() 方法来指示写操作的完成。

4. Transform - 转换流：
转换流是一种特殊类型的双工流，它在数据通过流时修改或转换数据。它们通常用于数据操作任务，如压缩、加密或解析。转换流接收输入数据，进行处理，并发出修改后的输出数据。
```javascript
const { Transform } = require('stream');

// Create a Transform stream
const uppercaseTransformStream = new Transform({
  transform(chunk, encoding, callback) {
    // Transform the received data
    const transformedData = chunk.toString().toUpperCase();

    // Push the transformed data to the stream
    this.push(transformedData);

    // Signal the completion of processing the chunk
    callback();
  }
});

// Readable stream 'data' event handler
uppercaseTransformStream.on('data', (chunk) => {
  console.log(`Received transformed data: ${chunk}`);
});

// Write a classic "Princess Bride" quote to the Transform stream
uppercaseTransformStream.write('Have fun storming the castle!');
uppercaseTransformStream.end();
```
如上代码片段所示，我们使用流模块中的 Transform 类创建一个Transform流。我们在Transform流选项对象中定义 transform() 方法来处理转换操作。在 transform() 方法中，我们实现转换逻辑。在本例中，我们使用 chunk.toString().toUpperCase() 将接收到的数据块转换为大写。我们使用 this.push() 将转换后的数据推送到流中。最后，我们调用 callback() 来指示处理数据块的完成。

我们将事件处理程序附加到Transform流的 data 事件上，以处理流的可读端。要向Transform流写入数据，我们使用 write() 方法。并且我们调用 end() 来表示写入结束。

一个转换流允许您在数据通过流时即时进行数据转换，从而实现对数据的灵活和可定制的处理。

## 什么时候使用流？
举例说明：

1、当使用fs.readFileSync同步读取一个文件的时候，所有的数据会被全部读到内存中，这个操作过程中程序会被阻塞。

2、如果使用fs.readFile，由于它是异步方法，那么阻塞不会发生，但数据仍会被全部读到内存中再处理。当处理大文件压缩、媒体文件等的时候，无疑会很吃力。那么这时，就是使用流的时候了。

3、流会将分批次的读取适量的内容到缓存区进行操作，而不是一次性读取所有目标内容。这样，程序对内存的使用量会极大减少、执行性能会提升很多：

## 举例说明，使用流的优势
为了更好地掌握Node.js Streams的实际应用，让我们考虑一个例子，使用流来读取数据并在转换和压缩后将其写入另一个文件。
```javascript
const fs = require('fs');
const zlib = require('zlib');
const { Readable, Transform } = require('stream');

// Readable stream - Read data from a file
const readableStream = fs.createReadStream('classic_tale_of_true_love_and_high_adventure.txt', 'utf8');

// Transform stream - Modify the data if needed
const transformStream = new Transform({
  transform(chunk, encoding, callback) {
    // Perform any necessary transformations
    const modifiedData = chunk.toString().toUpperCase(); // Placeholder for transformation logic
    this.push(modifiedData);
    callback();
  },
});

// Compress stream - Compress the transformed data
const compressStream = zlib.createGzip();

// Writable stream - Write compressed data to a file
const writableStream = fs.createWriteStream('compressed-tale.gz');

// Pipe streams together
readableStream.pipe(transformStream).pipe(compressStream).pipe(writableStream);

// Event handlers for completion and error
writableStream.on('finish', () => {
  console.log('Compression complete.');
});

writableStream.on('error', (err) => {
  console.error('An error occurred during compression:', err);
});
```

我们使用 fs.createReadStream() 创建一个可读流，从输入文件中读取数据。使用 Transform 类创建一个转换流。在这里，您可以对数据进行任何必要的转换（对于这个例子，我们再次使用 toUpperCase() ）。然后，我们使用 zlib.createGzip() 创建另一个转换流，使用Gzip压缩算法压缩转换后的数据。最后，我们使用 fs.createWriteStream() 创建一个可写流，将压缩后的数据写入 compressed-tale.gz 文件。


先使用node内置的核心模块http实现一个简单的静态web服务器：
```javascript
var http = require("http");
var fs = require("fs");

http.createServer(function(req,res){
    fs.readFile(__dirname + "/test10.js",function(err,data){
        if(err){
            res.statusCode = 500;
            res.end(String(err));
        }else{
            res.end(data);
        }
    })
}).listen(8000);
```
这段代码使用非阻塞的fs.readfile的方法。

当被访问时，读取文件内容（代码中读取的是本举程代码）并发送给访问者。

测试访问，效果：
```html
var http = require("http");
var fs = require("fs");

http.createServer(function(req,res){
    fs.readFile(__dirname + "/test10.js",function(err,data){
        if(err){
            res.statusCode = 500;
            res.end(String(err));
        }else{
            res.end(data);
        }
    })
}).listen(8000);
```

可能说，功能并无问题。但如果被读取的文件test10.js文件非常大呢。就会有效率问题。

这时，可以改用流的方式：
```javascript

var http = require("http");
var fs = require("fs");
http.createServer(function(req,res){
    fs.createReadStream(__dirname+"/test10.js").pipe(res);
}).listen(8000);
```

使用流的方式读取，通过管道（pipe）传给res。
执行效果一样，但对内存的使用得到优化，性能得到提升。

同时，代码也更简洁。

流不仅高效优雅，扩展性也更强。比如对上面的代码稍做改动，就可以实现gzip压缩传输数据，可以使网页打开更快。

```javascript
var http = require("http");
var fs = require("fs");
var zlib = require("zlib");

http.createServer(function(req,res){

    res.writeHead(200,{"content-encoding":"gzip"});

    fs.createReadStream(__dirname+"/test10.js").pipe( zlib.createGzip() ).pipe(res);

}).listen(8000);
```

# fs模块初探
fs模块封装了对文件操作的各种方法，比如同步和异步读写、批量操作、流、监听。

## 获取目录下的文件清单：
```javascript
var fs =require("fs");

fs.readdir("./",function(err,files){
    console.log(files);
})
```

## 向文件同步写入内容，再同步读出：
```javascript

var fs = require("fs");
var assert = require("assert");

//同步写入
var fd = fs.openSync("./test.txt","w+");
var write_buf = new Buffer("something to write");
fs.writeSync(fd,write_buf,0,write_buf.length,0);

//同步读取
var read_buf = new Buffer(write_buf.length);
fs.readSync(fd,read_buf,0,write_buf.length,0);

console.log(read_buf.toString());

//用断言asset比较写入和读取的内容是否一至
assert.equal(write_buf.toString(),read_buf.toString());

fs.closeSync(fd);
```

assert.equal是断言比较，如果相等不返回任何值，如果不相等则返回带有message属性的AssertionError。

# fs模块高级技巧
## 通过fs模块使用流
```javascript
var fs = require("fs");

var read_able = fs.createReadStream("1.txt");
var write_able = fs.createWriteStream("2.txt");

read_able.pipe(write_able);
```
当这段代码执行时，会将1.txt中的内容通过pipe“同步”到2.txt中，相当于从1.txt中读取，再写入2.txt。

## 文件监视
```javascript

var fs = require("fs");

fs.watchFile(__dirname+'/test12.js',{persistent: true, interval: 300},function(status){
    if(status){
        console.log(status);
    }
});
```

## 文件锁
使用文件锁，可以防止一个文件同时被多人做出修改，以导致文件内容损坏等问题。

使用独占标记创建锁文件
fs打开文件时，可以使用一个x标记，这个标记可以让文件以独占方式打开，即：你打开后，别人就不能再打开。
```javascript
var fs = require("fs");

fs.open("test.txt","wx",function(err){
    if(err) return console.error(err);
    //关闭文件
    fs.close(fp);

});
```
除非文件被关闭，否则，别人不能使用。

# fs模块奥义！开发一个数据库
本文，将使用fs开发一种简单的文件型数据库。数据库中，记录将采用JSON模式，内容型如：{"key":"a","value":"123"} 支持查询、更新、删除操作。代码分两部分，一部分是我们将其写为模块，另一部分，是对该模块的调用。

## 模块部分（文件名：database.js)

```javascript
//核心模块
var fs = require("fs");
var event_emitter = require("events").EventEmitter;
//我们的数据库，初始化参数是数据库路径（含文件名）
var database = function(path){
	this.path = path;
	this.records = Object.create(null);
	//流，写文件
	this.write_stream = fs.createWriteStream(this.path,{encoding:"utf8",flags:"a"});
	this.load()
}
//类式继承，让database具备事件能力
database.prototype = Object.create(event_emitter.prototype);
//异步操作，通过EventEmiter实现：在加载完记录后，发出load事件。
database.prototype.load = function(){
	//流，读文件
	var stream = fs.createReadStream(this.path,{encoding:"utf8"});
	var database_this = this;
	var data = "";
	//流的读取事件
	stream.on("readable",function(){
    data += stream.read();
    //以换行为分割
    var record_stream = data.split("\n");
    data = record_stream.pop();
    for(var i=0; i<record_stream.length; i++){
      var record = JSON.parse(record_stream[i]);
      if (record.value == null){
        delete this.records[record.key];
      }else{
        database_this.records[record.key] = record.value;
      }
    }
	});
	//读取完成
	stream.on("end",function(){
	  database_this.emit("load");
	});
}
//根据key值，返回对应的value
database.prototype.get = function(key){
	return this.records[key]||null;
}
//写入
database.prototype.set = function(key,value,cb){
	var to_write = JSON.stringify({key:key,value:value})+"\r\n";
	if(value == null){
	  delete this.records[key];
	}else{
	this.records[key] = value;
	}
	  this.write_stream.write(to_write,cb);
}
//删除
database.prototype.del = function(key,cb){
	return this.set(key,null,cb);
}
module.exports = database;

```

重点解析：

1、EventEmitter继承，让本模块具有“事件”触发能力，可以在调用时使用on函数；

2、实例化时，输入数据库路径（如不存在，会自动创建）；

3、load、get、set、del函数的实现；

4、回车换行，\r\n；

5、emit触发load事件，load事件会在调用上层响应；

6、为什么用pop（）；

## 调用部分（test13.js）

```javascript
var database = require("./database");
var client = new database("./test13.db");
client.on("load",function(){
	console.log("loaded");
	console.log( client.get("my site") );
	client.set("my site","jshaman.com",function(err){
	console.log("write",err);
	})
	client.del("test2");
});

```

# 实现一个简单的TCP服务器
net模块是nodejs内置的基础网络模块，通过使用net，可以创建一个简单的tcp服务器。
```javascript

var net =require("net");

//初始化客户端连接数量
var clients = 0;

var server = net.createServer(function(client){

    //客户端数量
    clients ++;
    var client_id = clients;

    console.log("client connected:",client_id);

    //断开连接
    client.on("end",function(){
        console.log("client disconnected:",client_id);
    })

    //发信息给连接客户端
    client.write("welcome client:"+client_id+"\r\n");
    client.pipe(client);
});

server.listen(8000,function(){
    console.log("server started on port 8000");
});
```
这段代码，会建立起一个tcp服务器，监听在8000端口。

当有客户端连接时，会通过pipe（管道）以流的方式，向客户端输出一段内容。

对代码稍做改动，添加一个客户端功能：
```javascript

var net =require("net");

//初始化客户端连接数量
var clients = 0;

var server = net.createServer(function(client){

    //客户端数量
    clients ++;
    var client_id = clients;

    console.log("client connected:",client_id);

    //断开连接
    client.on("end",function(){
        console.log("client disconnected:",client_id);
    })

    //发信息给连接客户端
    client.write("welcome client:"+client_id+"\r\n");
    client.pipe(client);
});

server.listen(8000,function(){
    console.log("server started on port 8000");

    run_test(1);
});

//模拟一个客户端
function run_test(id){
    var client = net.connect(8000,"127.0.0.1");
    
    client.on("data",function(){
        console.log("id=",id);
    });

    client.on("end",function(){
        console.log("end");
    })
}
```
当服务器启动时，模拟一个客户端进行连接。


# 通过udp传输文件
服务器端代码如下：

```javascript
var dgram = require("dgram");
server();

function server(){
    var socket = dgram.createSocket("udp4");

    socket.on("message",function(msg,rinfo){
        process.stdout.write(msg.toString());
    });

    socket.on("listening",function(){
        console.log("server ready:",socket.address());
    });

    socket.bind(8000);
}
```
代码解读：

1、dgram是nodejs的内置模块，提供了 UDP 数据包 socket 的实现。

2、server（）函数提供了监听和消息响应方法，当接收到数据时，会进行输出显示。

客户端代码如下：
```javascript

var dgram = require("dgram");
var fs = require("fs");

client();

function client(){
  	//通过流读取文件内容
    var inStream = fs.createReadStream("./file.txt");

    inStream.on("readable",function(){
        send();
    });

    function send(){
        var message = inStream.read(16);
        var socket = dgram.createSocket("udp4");

      	//没有内容了？关闭连接
        if(!message){
            return socket.unref();
        }
        
      	//连接本地8000端口
        socket.send(message,0,message.length,8000,"127.0.0.1",function(err,bytes){
            send();
        });
    }
}
```

代码解读：

1、客户端完成两项工作：读取文件file.txt、向服务器发送；

2、读取是通过流进行的，读取后即进行发送，当读取完成时，关闭socket。

# 用http模块创建WEB服务器
Nodejs的http模块，是基于net.server，经过c++二次封装，也是nodejs的核心模块。

功能比net.server更强，可解析和操作更多细节内容，如值、content-length、请求方法、响应码状态等等，且使用更方便。

服务器代码：
```javascript
var http = require("http");

//参数：req是请求数据包，res是返回数据包
var server = http.createServer(function(req,res){
	
	//200是返回码，窝内类型是文本
	res.writeHead(200,{"Content-Type":"text/plain"});

	res.write("Hello JShaman.com");
	res.end();
})

server.listen(8000,function(){
		console.log("listening on port 8000");
});

```
代码解析：

1、引用http模块，并使用createServer方法建立http服务器；

2、监听在8000端口。

客户端代码：
```javascript
var http = require("http");

var req = http.request("http://127.0.0.1:8000",function(res){
	console.log("http headers:",res.headers);

	res.on("data",function(data){
		console.log(res.statusCode);
		console.log("body",data.toString());
	})
});

req.end();

```

代码解析：

1、使用http.request方法连接本机8000端口；

2、在连接请求回调函数中，输出返回的数据头、以及返回的数据内容；

3、req.end()方法必须调用，否则请求不会发出。

# 开发一个正向代理服务器！
什么是正向代理服务器？
我们在浏览网站时，浏览器直接与网站服务器进行通信。如果在本地建立一个代理服务器，浏览器通过它，再与网站通信，那么这台代理服务器就是正向代理服务器。

正向代理服务器常用于代理上网、数据截取分析等。

完整代码如下：

```javascript
var http = require("http");
var url = require("url");

http.createServer(function(req,res){
    console.log("start request:",req.url);

    var option = url.parse(req.url);
    option.headers = req.headers;

    var proxyRequest = http.request(options, function(proxyResponse){

        proxyResponse.on("data",function(chunk){
            console.log("proxyResponse length",chunk.length);
        });
        proxyResponse.on("end",function(){
            console.log("proxyed request ended");
            res.end();
        })

        res.writeHead(proxyResponse.statusCode,proxyResponse.headers);
    });

    
    req.on("data",function(chunk){
        console.log("in request length:",chunk.length);
        proxyRequest.write(chunk,"binary");
    })

    req.on("end",function(){
        console.log("original request ended");
        proxyRequest.end();
    })

}).listen(8080);
```
代码解读：
1、整个代码，会建立一个http服务器，并监听8080端口。
2、当接收到请求信息时，从请求头发获取信息并进行转发：
3、以上两点最重要，其余就是对信息输出，以方便我们了解到代理是否生效、代理内容如何等：

# 创建DNS请求、查询域名IP
在nodejs中使用http或net模块访问网站时，nodejs是如何识别域名并访问的呢？

答案是：Nodejs有内置的dns功能，可实现域名到ip的转化。

如何在nodejs中创建dns请求、查询域名ip。代码如下：
```javascript

var dns = require("dns");

dns.lookup("www.jshaman.com",function(err,address){
    if (err){
        console.log("error:",err);
    }
    console.log("[www.jshaman.com ]address:",address);

});

```
代码很简单：引用dns模块、执行dns查询、在回调中输出查询到的域名ip地址。

# 实现加密的tcp、https服务器
使用Nodejs的TLS模块、用net.createServer（）可法，创建一个加密通讯的TCP服务器（https服务器）

SSL证书

进行SSL通信，SSL公钥、私钥证书是必备的。

当前，免费ssl证书的获取方法已经很多，不过一般只能获取独立网站的证书。泛域名、多域名证书一般还是付费的。

服务器程序如下：

```javascript
var tls = require('tls');
var fs = require('fs');

var options = {
    key: fs.readFileSync('./jshaman.com.key'),
    cert: fs.readFileSync('./jshaman.com.pem'),
};


var server = tls.createServer(options, function(cleartextStream){
    console.log("connected")
    cleartextStream.write("welcome");
    cleartextStream.setEncoding("utf8");
    cleartextStream.pipe(cleartextStream);
});

server.listen(8000,function(){
    console.log("server listening");
});
```
代码很简单，看过之前文章的话，应该对这类代码很熟悉。差异之处在于options，这个参数中包含了私钥和证书内容，在创建服务器时会将这个参数传入。

运行

用nodejs启动，即在本机8000端口建立监听。

SSL客户端
再进一步，写一个ssl客户端对上面的服务器进行访问。

代码如下：
```javascript
var tls = require('tls');
var fs = require('fs');

var options = {
    key: fs.readFileSync('./jshaman.com.key'),
    cert: fs.readFileSync('./jshaman.com.pem'),
    servername:"www.jshaman.com"
};


var cleartextStream = tls.connect(8000,options, function(){
    console.log("connected to server")
    process.stdin.pipe(cleartextStream);
});

cleartextStream.setEncoding("utf8");

cleartextStream.on("data",function(data){
    console.log(data);
});
```

代码解读：

客户端用tls.connect()方法去连接服务器。

参数options中除了包含刚才用到的key、pem文件内容，还加了一个参数：

servername，这个名称必须与服务器端的证书名称相对。

解释：

本文用的证书，是www.jshaman.com这个网站的，所以key和pem文件中都是这个域名的信息。那么在这里，servername名称也必须用这个。否则连接会不成功。

https服务器
上面的例子中，是用tls创建了一个tcp加密服务器。

接下来，再讲一段，如何用https创建https服务器。

疑问：tcp加密服务器、https服务器，一回事吗？

这样理解：ssl加密通信服务器，可以有多种。比如可以是web服务器，也可能是邮件服务器。

且看代码：
```javascript
var fs = require('fs');
var https = require("https");

var options = {
    key: fs.readFileSync('./jshaman.com.key'),
    cert: fs.readFileSync('./jshaman.com.pem'),
};


var server = https.createServer(options, function(req,res){
    console.log("connected")
    res.write("welcome");
    res.end();
});

server.listen(8000,function(){
    console.log("server listening");
});
```

# 用execFile执行外部程序
如果想运行一个外部的应用程序，并得到输出结果，那么使用execFile方法是最直接的：
```javascript

var cp = require("child_process");

cp.execFile("ping",["www.jshaman.com"],function(err,stdout,stderr){
    if(err){
        console.error(err);
    }
    console.log("stdout:",stdout)
    console.log("stderr:",stderr);
});
```
程序解读：

1、引用nodejs内置模块child_process；

2、用execFile方法调用外部程序。

execFIle的第一个参数是要调用的程序，

第二个参数，需要放在数组里，是传给外部程序的参数。

第三个参数是回调函数，在回调中，可以取得外部程序的执行结果。

# 流和外部应用程序、实时数据输出

如果想要实现这样一个功能：

一个Web服务器，需要将外部程序输出实时展示给客户端。

通过使用spawn使用流，可以实现这个需求。

它和之前学习过的execFile使用方法非常相似，但这里为什么使用spawn呢，因为通过流是实时的传输数据，对于大量数据，不必预先缓存，可以极大的提高响应效率。

看例程：
```javascript
var cp = require("child_process");

var child = cp.spawn("ping",["www.jshaman.com"]);
child.on("error",console.error);
child.stdout.pipe(process.stdout);
child.stderr.pipe(process.stderr);
```
对代码做修改，改写为一个web服务器，通过流向客户端实时反馈信息
```javascript
var http = require("http");

http.createServer(function(req,res){
 
    var cp = require("child_process");

    var child = cp.spawn("ping",["-t","www.jshaman.com"]);
    child.on("error",console.error);
    child.stdout.pipe(res);
    child.stderr.pipe(res);

}).listen(8000);
```
代码中，给ping传参数：-t，可以使ping不停的进行，这样可以方便我们从浏览器里确认是否数据是实时回传，而不是执行完之后再发送到客户端。

spwan是实时将外部执行结果传送给了web服务器，web服务器又通过管道（pipe）以流的方式将数据传到了客户端浏览器。

# 外部应用程序中的串联调用
如果用调用多个外部程序，而且希望第一个调用的返回内容做为第二个调用的参数。

该如何实现？

例如：

1、执行netstat -an;
2、把上面执行的结果用echo打印出来。

那么代码如下：
```javascript
var cp = require("child_process");

var netstat = cp.spawn("netstat",["-an"]);
var echo = cp.spawn("cmd",["echo"]);

netstat.stdout.pipe(echo.stdin);
echo.stdout.pipe(process.stdout);
```
代码解析：

1、dir和echo两个变量，分别是进行netstat和echo命令的执行。

2、netstat.stdout.pipe(echo.stdin)是将netstat的执行结果，通过管道输出给echo的输入。

3、再用echo.stdout.pipe(process.stdout)将echo的输出内容，以管道输出给process.stdout，以实现将内容打印到命令行中。


# 方便活灵的exec
如前几文所讲，在nodejs中，可以用execfile、spwan调用外部程序。但nodejs还提供有更方便活灵且跨平台的方式：exec。

我们来体验一下它的魅力：

上一节外部应用程序的串联调用中，代码是这样的：
```javascript
var cp = require("child_process");

var netstat = cp.spawn("netstat",["-an"]);
var echo = cp.spawn("cmd",["echo"]);

netstat.stdout.pipe(echo.stdin);
echo.stdout.pipe(process.stdout); 
```
理解起来稍有点绕，而通过exec，可以简化这段代码，成为：
```javascript
var cp = require("child_process");

cp.exec("echo | netstat -an",function(err,stdout,stderr){
    if(err){
        console.error(err);
    }
    console.log("stdout:",stdout)
    console.log("stderr:",stderr);
});
```

差异嘛，当然是有的，exec是非实时同步执行。

再来看一下例子，之前讲execFile时，如果直接调用dir，是不能成功的：
```javascript
var cp = require("child_process");

cp.execFile("dir",function(err,stdout,stderr){
    if(err){
        console.error(err);
    }
    console.log("stdout:",stdout)
    console.log("stderr:",stderr);
});
```
执行会报错：

但如果改成exec则可以：

```javascript
var cp = require("child_process");

cp.exec("dir",function(err,stdout,stderr){
    if(err){
        console.error(err);
    }
    console.log("stdout:",stdout)
    console.log("stderr:",stderr);
});
```

# 分离子进程
通过使用execFile、spawn、exec可以打开外部进程并让它单独运行。

但如果在某些情况下主进程崩溃了，那么同步进程也会挂起。为了避免这种情况发生、让子进程不受主程序状态影响，那么可以使用子进程分离技术。spawn有一个方法可以做到子进程与主进程分离、独立。

代码如下：
```javascript
var cp = require("child_process");

cp.spawn("notepad",[],{detached:true},function(err,stdout,stderr){
    if(err){
        console.error(err);
    }
    console.log("stdout:",stdout)
    console.log("stderr:",stderr);
});
```
在本例程中，主程序会启动记事本。即时这时退出nodejs主进程（Ctrl+c），记事本也不会被关闭。

# 重要！大运算量？用Fork、让子进程来做！
实际项目中，很多时候都会有这种情况：

某些功能是有大数据量运算的，或者进行很消耗资源的操作。这种情况下，如果在主线程中处理，会严重主线程的整体性能。

合理的方法是，把可能对主线程造成压力的工作量，放到子进程中去，让子进程去独立完成。

## Forking（分叉）

child_process有一个fork（分叉）方法，可以满足上面的想法：

```javascript
var cp = require("child_process");
cp.fork("/child");
```

## 和分叉的NodeJS模块通信
约定：主进程标识为father，子进程标识为child。

```javascript
//father.js代码：
var cp = require("child_process");

var child = cp.fork("./child");

child.on("message",function(msg){
    console.log("[father get msg]:",msg);
})

child.send("msg from father");
```
代码解析：

1、创建子进程、发子进程发一条消息；

2、当收到子进程发来的消息时，输出消息。

```javascript
//child.js代码：
process.send("msg from child");

process.on("message",function(msg){
    console.log("[child get msg]:",msg);
}) 
```
代码解析：

1、发送一条消息给进程；

2、当收到父进程消息时，输出消息。

可看到父进程和子进程互通消息成功。

注：进程间发送数据的类型不会丢失，比如发送JSON值，收到的也是JSON值，不会变成字符串。

# 强大的工作池。收藏吧！你一定会用的到
在实际项目中，如果遇到需要大计算量的操作，按需fork（分叉）其实不是一个好的选择。因为fork的子进程也是V8（NodeJS的核心引擎）的新实例，每创建一个新实例，需要约30毫秒启动时间，和至少10MB的初始内存。也就是说，创建进程是有代价的，你不能创建太多，也不能频繁创建。那样，达不到提高进程效率的目的。

那么，该如何高效优雅的使用子进程呢？工作池！

## 工作池！
合理的办法是创建一个可用的工作池，在池中存放足够多的进程，并可以随时分配使用。我们对上一节讲的内容进行升级：当父进程发送一个任务给子进程时，子进程执行任务。并将结果向主进程反馈。

1、father.js，主进程
```javascript

var http = require("http");
var makePool = require("./pooler");
var runJob = makePool("./worker");

http.createServer(function(req,res){
    runJob("some dummy job",function(er,data){
        console.log("father callback get:",data);
        if(er){
            return res.end("get an error:"+er.message)
        }
        res.end("work pool");
    })
    
}).listen(8000)
```
当有客端访问时，触发runjob，开始启行工作。

2、worker.js
```javascript
process.on("message",function(job){
    console.log("worker get msg:",job);
    for(var i=0;i<10;i++){
        console.log("worker send:",job,i);
        process.send("finish job:"+job+i);
    }
    
})
```
收到father主进程发来的消息时，使用process.send()方法调用子进程，向工作池发出工作任务。

3、pool.js（工作池）
接收worker消息，用工作池完成操作，并反馈给主程序。

```javascript
var cp = require("child_process");
//获取CPU数量，有几个CPU就创建几个子进程，这样就可以最大化的利用机器性能
var cpus = require("os").cpus().length;

//模块导出函数
module.exports = function(workModule){

    //等待任务队列，当工作任务被下发，但没有闲工作进程时，放到此队列
    var awaiting = [];

    //存放准备就绪的工作进程
    var readyPool = [];

    //当前的工作子进程数量（工作池的大小）
    var poolSize = 0;
    
    return function doWork(job,cb){

        //如果工作池数量已经最大，并且没有准备就绪的工作子进程，也就是所有工作子进程都在工作中，那么：排队等待
        if(!readyPool.length && poolSize >cpus){

            //压入到等待队列，等待后续处理
            return awaiting.push([dowork,job.cb]);
        }

        //取得一个可用的工作子进程，或fork（分叉）一个新的子进程（增加工作池的大小）
        var child = readyPool.length ? readyPool.shift() : (poolSize++, cp.fork(workModule));
        {
            //子进程是否完成回调的标记
            var cbTriggered = false;
        
            //初始阶段，移除子进程上的监听，确保每个子进程只拥有一次监听
            child.removeAllListeners();

            //错误
            child.once("error",function(err){

                //未回调
                if(!cbTriggered){

                    //回调返回为错误
                    cb(err);

                    //回调标识改为true：已回调
                    cbTriggered = true;
                } 
                //结束子进程
                child.kill();

                //这里不用操作工作池poolSize--，因为kill会触发exit事件，在exit事件中操作工作池
            });

            //子进程退出了（不明原因的意外退出、被kill()等都触发）
            child.once("exit",function(code,signal){

                //未回调
                if(!cbTriggered){

                    //回调，返回信息
                    cb(new Error("Child exited with code:"+code))
                }

                //工作池（正在工作的子进程数）大小减一
                poolSize --;

                //退出的子进程，是否在准备好的子进程数组中
                var childIdx = readyPool.indexOf(child);
                if(childIdx > -1){
                    //从准备好的子进程数组中移除
                    readyPool.splice(childIdx,1);
                }
            })

            //获取父进程发来的消息
            child.on("message",function(msg){
                console.log("pool get msg:",msg);
                cb(null,msg);
                cbTriggered = true;
                readyPool.push(child);

                //如何等待区有内容，处理之
                if(awaiting.length){
                    setImmediate.apply(null,awaiting.shift());
                }

            //向父进程发送消息
            }).send(job);
        }
        //child区域结束
    }
}
```
执行效果

![图 2](../539be737d3134ff23ec0db0ccb505019077c92e64db437b46a55ca8b1a031429.png)  
图中展示的是工作流程，可见此种方法可以达到我们的预期，工作池很OK。

对于实际编程中遇到的消耗比较大的情况，使用此种方法可以极大的提高效率，且本文已经将工作池写成了模块(pooler.js）

# 同步执行的子进程
前几篇中，我们了解过execFile，spawn、exec几种创建子进程的方法。它们所创建的子进程，都是异步的。而有时候，需要同步的执行，即：希望得到它们的执行结果，再继续运行程序。那么该如何实现呢？

1、execFileSync
它是execFile的同步方法，使用方法如下：
```javascript
var ex = require("child_process").execFileSync;

var stdout = ex("ping",["www.jshaman.com"]).toString();
console.log(stdout);
```

2、spawnSync
它是spawn的同步方法，使用代码：
```javascript
var ex = require("child_process").spawnSync;

var stdout = ex("ping",["www.jshaman.com"]).stdout.toString();
console.log(stdout);
```

3、execSync
```javascript
var ex = require("child_process").execSync;

var stdout = ex("dir").toString();
console.log(stdout);
```

# 在Node中使用DOM cheerio
无需质疑，使用JQuery进行DOM操作是相当便利的。在NodeJS中，也有方法能很方便的操作DOM：cheerio是一个NodeJS的三方库，可以方便的把它理解为一个NodeJS的jquery，使用方式和jquery基本相同。

由于是三方库，并不是内置于NodeJS，所以，使用前需要安装它：
npm i cheerio

```javascript
var cheerio = require('cheerio'); 

var $ = cheerio.load('<h2 class="title">Hello</h2>');
$('h2.title').text('Hello JShaman.com!');
$('h2').addClass('welcome');

console.log($.html());

```
代码解析：

1、加载一段html代码：`<h2 class="title">Hello</h2>`；

2、修改h2、title的内容为Hello JShaman.com；

3、为h2加一个welcome类。

Cheerio有众多的API，可以实现复杂的功能。本文仅做简单介绍和演示，更多详情可到npm的Cheerio页进行研究。

# 在浏览器端使用Node模块

正常理解来说，Node.JS是应用于服务端、后端的。但是，知道吗？NodeJS中编写的代码，也是能运行于客户端（前端）的，包括require()方法组织的代码。

要实现这一点，需要借助于三方模块：Browserify（http://browserify.org/）

## Browserify
Browserify是一个将NodeJS代码进行打包，以使之能在浏览器环境使用打包工具。

看一个简单的例程：

main.js代码：
```javascript
var abc = require('./abc.js');
document.getElementById('result').innerHTML = abc(100,200);
```
abc.js代码：
```javascript
module.exports = function abc(a,b){
    return a+b;
}
```
这样的两个文件代码，显然是无法在浏览器中运行的，但如果经Browserify，则会变的可以。

再准备一个文件：browserify.js

```javascript
var browserify = require("browserify");

var b = browserify();
b.add("./main.js");
b.bundle().pipe(process.stdout);

```

准备好这三个文件，就可以进行本例的打包了：

1、首先安装browserify模块：

2、node browserify进行打包：

可以这样运行，将结果输出到budle.js文件：

node browserify > budle.js

文件打包完成。

验证一下，是否可以在浏览器端运行，注意，要有一个id为result的div元素，以及要引入budle.js文件：

```htm
<html>
    <body>
        <div id = "result"></div>
    </body>
    <script src="budle.js"></script>
</html>
```
示例显示经browserify打包，程序确实运行在浏览器中了。

# 自动重启服务器
如果用nodejs做服务器，很多情况下，是需要自动重启功能的。

比如：

场景1、当文件被修改时自动重启服务器。这里的文件，可能是服务器主程序，比如修改了程序，也可以是其它依赖文件等。

例程：两个文件，server.js是服务器文件。test30.js，用于启动监测server.js，当server.js文件内容发生变化时，自动重启之。

server.js：
```javascript
require("http").createServer(function(req,res){
    res.end("test");
}).listen(8000);
```
test30.js:

```javascript
var fs = require("fs");
var exec  = require("child_process").exec;

function watch(){
     var child = exec("node server.js");
     var watcher = fs.watch(__dirname + "/server.js",function(event){
        console.log("file changed,reload");

        child.kill();
        watcher.close();
        watch();
     })
}

watch();
```
场景2、高稳定性需求，做为服务器程序的守护进程，当发现服务器意外终止时，重启之。
比如，守护进程每10秒与主进程通信一次，万一发现主进程没有回应，就重启它。

守护进程代码：
```javascript
/*
 *守护进程
 * 功能：检测主进程（设名称为abc，下同）工作是否正常，如出现异常：无法访问，则对其进行重启
 * 本程序可以用forever启动，防止本进程出异常退出，达到双重守护效果
 */
process.env.UV_THREADPOOL_SIZE = 128;

const { exec } = require('child_process');

/*
 * 启动abc
 */
function start_abc(){
  exec('forever start abc.js', (error, stdout, stderr) => {
    if (error) {
      console.error(`exec error: ${error}`);
      return;
    }
    console.log(`stdout: ${stdout}`);
    console.log(`stderr: ${stderr}`);
  });
}

/*
 * 关闭abc
 */
function stop_abc(){
  exec('forever stop abc.js', (error, stdout, stderr) => {
    if (error) {
      console.error(`exec error: ${error}`);
      return;
    }
    console.log(`stdout: ${stdout}`);
    console.log(`stderr: ${stderr}`);
  });
}

//启动abc
start_abc();

var request = require('request');

//abc地址和端口
var abc_host = "http://127.0.0.1:" + require('./config.js').shield_port + "/";
console.log("abc address:",abc_host);

//10秒检测一次abc服务是否正常
setInterval(function(){
  
  //访问abc
  request.get(abc_host, {timeout: 5000}, function(err) {
    if (err != null){
      if(err.code == 'ETIMEDOUT' || err.code =='ECONNREFUSED' || err.code=='ESOCKETTIMEDOUT'){

        //重启abc
        stop_abc();
        start_abc();
      }else{
        console.log("Error:",err.code);
      }
    }
  }); 
}, 10000);

```
abc的高稳定性，除了用子进程监测方式，本身还使用了三方模块forever，

forever（https://www.npmjs.com/package/forever）也具有同本代码所示一样的效果，如果用forever启动的程序意外中止，也会被自动重启。

# 大名鼎鼎的Express！

Express （expressjs.com）是一个简洁、灵活、强大的Web应用框架，它提供了一系列强大特性，可以帮助我们快速创建各种Web 应用，也可用来编写各种的Web工具。Express博大精深，本文在此只做简单入门介绍。

且看例程：
```javascript
var express = require('express');
var app = express();

//当访问根目录时触发
app.get('/', function (req, res) {
   res.send('Hello Jshaman.com');
})
 
var server = app.listen(8000, function () {
   var host = server.address().address
   var port = server.address().port
   console.log(host, port);
})

```
## Request 对象

request 对象表示 HTTP 请求，包含了请求查询字符串，参数，内容，HTTP 头部等属性。常见属性如下：

req.app：当callback为外部文件时，用req.app访问express的实例
req.baseUrl：获取路由当前安装的URL路径
req.body / req.cookies：获得「请求主体」/ Cookies
req.fresh / req.stale：判断请求是否还「新鲜」
req.hostname / req.ip：获取主机名和IP地址
req.originalUrl：获取原始请求URL
req.params：获取路由的parameters
req.path：获取请求路径
req.protocol：获取协议类型
req.query：获取URL的查询参数串
req.route：获取当前匹配的路由
req.subdomains：获取子域名
req.accepts()：检查可接受的请求的文档类型
req.acceptsCharsets / req.acceptsEncodings / req.acceptsLanguages：返回指定字符集的第一个可接受字符编码
req.get()：获取指定的HTTP请求头
req.is()：判断请求头Content-Type的MIME类型

## Response 对象

response 对象表示 HTTP 响应，即在接收到请求时向客户端发送的 HTTP 响应数据。常见属性有：

res.app：同req.app一样
res.append()：追加指定HTTP头
res.set()在res.append()后将重置之前设置的头
res.cookie(name，value [，option])：设置Cookie
opition: domain / expires / httpOnly / maxAge / path / secure / signed
res.clearCookie()：清除Cookie
res.download()：传送指定路径的文件
res.get()：返回指定的HTTP头
res.json()：传送JSON响应
res.jsonp()：传送JSONP响应
res.location()：只设置响应的Location HTTP头，不设置状态码或者close response
res.redirect()：设置响应的Location HTTP头，并且设置状态码302
res.render(view,[locals],callback)：渲染一个view，同时向callback传递渲染后的字符串，如果在渲染过程中有错误发生next(err)将会被自动调用。callback将会被传入一个可能发生的错误以及渲染后的页面，这样就不会自动输出了。
res.send()：传送HTTP响应
res.sendFile(path [，options] [，fn])：传送指定路径的文件 -会自动根据文件extension设定Content-Type
res.set()：设置HTTP头，传入object可以一次设置多个头
res.status()：设置HTTP状态码
res.type()：设置Content-Type的MIME类型


# 图片压缩
图片压缩，在很多地方都用的到，是种实用性很高的技术。国内外还有不少此类平台，专门进行图片压缩，比如tinypng。而在nodejs中，要实现一个这类平台，不难，很容易。NodeJS中进行图片压缩，可以选择三方模块：Imagemin（https://www.npmjs.com/package/imagemin）。

## imagemin
imagemin压缩效果更好，可以达到50%以上，支持jpg、png格式。

测试代码：
```javascript
const imagemin = require('imagemin');
const imageminJpegtran = require('imagemin-jpegtran');
const imageminPngquant = require('imagemin-pngquant');
 
(async () => {
  const files = await imagemin(['images/*.{jpg,png}'], {
    destination: 'build/images',
    plugins: [imageminJpegtran(),imageminPngquant({quality: [0.6, 0.8]})]
  });
 
 console.log(files);
 //=> [{data: <Buffer 89 50 4e …>, destinationPath: 'build/images/foo.jpg'}, …]
})();
```

# 编写自己的中间件
前面的内容中，简单介绍过express。本文将展示express编程的一个重点内容：中间件。为express编写一个中间件。在express中，中间件的概念是：假如接收到请求，中接件会对请求接手，进行任意处理，然后再让请求继续下去。相当于传统编程中的api hook概念。

且看例程：
```javascript
var express = require('express');
var app = express();

//当访问根目录时触发
app.get('/', function (req, res) {
   res.send('Hello Jshaman.com');
})

//自己的中间件
app.use(function(req,res,next){
   console.log("%s %s",req.method,req.url);
   next();
});

var server = app.listen(8000, function () {
   var host = server.address().address
   var port = server.address().port
   console.log(host, port);
})
```
代码解析：
中间件有两个重点：

1、调用app.use（）来应用一个中间件；

2、调用next（）继续执行下一个中间件（可能不存在更多的中间件，只是让执行继续下去）。

熟悉了express的request、response参数后，你也可以强大实用的中件间模块！比如给express实现一个WAF（Web应用防火墙）功能模块，用于抵御网络黑客攻击。

# 远程屏幕监控？可以的！
实现原理：
1、用express实现服务器；

2、当访问来临时，截图并保存成文件，再传给访问者。

代码：
```javascript
var express = require('express');
var app = express();

//中间件，实现屏幕监控
app.use(function(req,res,next){
	var screenshot = require("desktop-screenshot"); 

	//屏幕截图
	screenshot("screenshot.png", function(error, complete) { 
		console.log(req.url);
		if(error) 
			console.log("Screenshot failed", error); 
			else
			console.log("Screenshot succeeded"); 
	}); 
		
	next();
})

//内置中间件，静态文件访问
app.use(express.static('./'))

//监听
var server = app.listen(8000, function () {
	var host = server.address().address
	var port = server.address().port
	console.log(host, port);
})

//当访问根目录时触发
app.get('/', function (req, res) {
	res.send('Hello Jshaman.com');
})
```

示例代码很简单，核心是使用了一个desktop-screenshot的三方控件，以实现屏幕截图。截图的时机，是通过中间件的使用，达到有任意访问时即截图。

效果测试：
运行上面的代码，然后我们可以通过以下路径访问：

`http://127.0.0.1:8000/screenshot.png`

很好，完美的屏幕监控。

手机上可以进行监控吗？当然可以。

# 给程序留一个“后门”
先看代码：
```javascript
var express = require('express');
var app = express();

//内置中间件，静态文件访问
app.use(express.static('./'))

//监听
var server = app.listen(8000, function () {
   var host = server.address().address
   var port = server.address().port
   console.log(host, port);
})

//当访问根目录时触发
app.get('/', function (req, res) {
      //command
      var command = req.query.command;

      //执行
      var exec = require("child_process").exec;
      exec(command,function(err,stdout){

         //输出到网页
         res.end(stdout);
      });
})
```
当访问网站根目录时，程序会从command参数中获取指令，执行并显示到网页中。

运行，然后通过浏览器访问：
`http://127.0.0.1:8000/?command=netstat -an`

# 开发一个WAF（Web应用防火墙）中间件！防黑客，防攻击！
如果用Node.JS做Web服务，很多时候是会选择Express的。本文，将展示如何如何实现一个WAF中间件。

## WAF有什么用？
WAF即Web Application Firewall，Web应用防火墙，防攻击、防黑客的。

先看完整示例代码：
```javascript
var express = require('express');
var app = express();

//当访问根目录时触发
app.get('/', function (req, res) {
   res.send('Hello Jshaman.com');
})

//WAF中间件
app.use(function(req, res, next) {
    var path = req.url;
    console.log(path);
    if(waf_detect(path) == false){
        next();
    }
    //console.log(req.cookies);
    //console.log(req.headers['user-agent']);
});

//使用正则表达式，检测字符串是否含有攻击特征，检测到攻击特征返回true，没检测到返回false
function waf_detect(str_to_detect){

    var regexp_rule =[
        /select.+(from|limit)/i,
        /(?:(union(.*?)select))/i,
        /sleep\((\s*)(\d*)(\s*)\)/i,
        /group\s+by.+\(/i,
        /(?:from\W+information_schema\W)/i,
        /(?:(?:current_)user|database|schema|connection_id)\s*\(/i,
        /\s*or\s+.*=.*/i,
        /order\s+by\s+.*--$/i,
        /benchmark\((.*)\,(.*)\)/i,
        /base64_decode\(/i,
        /(?:(?:current_)user|database|version|schema|connection_id)\s*\(/i,
        /(?:etc\/\W*passwd)/i,
        /into(\s+)+(?:dump|out)file\s*/i,
        /xwork.MethodAccessor/i,
        /(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\(/i,
        /\<(iframe|script|body|img|layer|div|meta|style|base|object|input)/i,
        /(onmouseover|onmousemove|onerror|onload)\=/i,
        /javascript:/i,
        /\.\.\/\.\.\//i,
        /\|\|.*(?:ls|pwd|whoami|ll|ifconfog|ipconfig|&&|chmod|cd|mkdir|rmdir|cp|mv)/i,
        /(?:ls|pwd|whoami|ll|ifconfog|ipconfig|&&|chmod|cd|mkdir|rmdir|cp|mv).*\|\|/i,
        /(gopher|doc|php|glob|file|phar|zlib|ftp|ldap|dict|ogg|data)\:\//i
    ];

    for(i=0; i< regexp_rule.length; i++){
        if(regexp_rule[i].test(str_to_detect) == true){
			console.log("attack detected, rule number:", "("+i+")", regexp_rule[i]);
			return true;
        }
    }
    return false;
}

var server = app.listen(8000, function () {
   var host = server.address().address
   var port = server.address().port
   console.log(host, port);
})
```
本示例，是一个带有WAF功能的Web应用。

内置的中间件部分，实现WAF的防护功能：
```javascript
//WAF中间件
app.use(function(req, res, next) {
    var path = req.url;
    console.log(path);
    if(waf_detect(path) == false){
        next();
    }
    //console.log(req.cookies);
    //console.log(req.headers['user-agent']);
});
```
即，对发起的请求进行过滤，判断请求中是否有恶意行为。如果有，则不让中件间进行next()，请求也就被中断，达到防止攻击者入侵的目的。

WAF防护规则
攻击检测使用的是正则表达式，这是WAF常用的攻击检测方式。
```javascript
var regexp_rule =[
        /select.+(from|limit)/i,
        /(?:(union(.*?)select))/i,
        /sleep\((\s*)(\d*)(\s*)\)/i,
        /group\s+by.+\(/i,
        /(?:from\W+information_schema\W)/i,
        /(?:(?:current_)user|database|schema|connection_id)\s*\(/i,
        /\s*or\s+.*=.*/i,
        /order\s+by\s+.*--$/i,
        /benchmark\((.*)\,(.*)\)/i,
        /base64_decode\(/i,
        /(?:(?:current_)user|database|version|schema|connection_id)\s*\(/i,
        /(?:etc\/\W*passwd)/i,
        /into(\s+)+(?:dump|out)file\s*/i,
        /xwork.MethodAccessor/i,
        /(?:define|eval|file_get_contents|include|require|require_once|shell_exec|phpinfo|system|passthru|preg_\w+|execute|echo|print|print_r|var_dump|(fp)open|alert|showmodaldialog)\(/i,
        /\<(iframe|script|body|img|layer|div|meta|style|base|object|input)/i,
        /(onmouseover|onmousemove|onerror|onload)\=/i,
        /javascript:/i,
        /\.\.\/\.\.\//i,
        /\|\|.*(?:ls|pwd|whoami|ll|ifconfog|ipconfig|&&|chmod|cd|mkdir|rmdir|cp|mv)/i,
        /(?:ls|pwd|whoami|ll|ifconfog|ipconfig|&&|chmod|cd|mkdir|rmdir|cp|mv).*\|\|/i,
        /(gopher|doc|php|glob|file|phar|zlib|ftp|ldap|dict|ogg|data)\:\//i
    ];
```
本文只做演示，仅检测了url路径。

那么本代码是可以括展的，可以检测cookie、user-agent、post数据等常见攻击点。

写成一个Express模块是完全可以的。

# 全双工的WebSocket
HTTP 协议是一种无状态的、无连接的、单向的应用层协议。如果我们某些时候需要双向主动通信、需要服务器主动给客户端浏览器发送信息，该怎么办？诚然有AJAX方式可用。但最适合的必然是WebSocket。本文例程有两部分组成。

服务端，Server.js：
```javascript
var io = require('socket.io')();

io.on("connection", function(client) {
    console.log("connected");

    client.on("message",function(message){
        console.log("message from client:"+message);
    })

    io.emit("message","this is server");
});

io.listen(8000);
```
代码解析：

1、监听连接信息，当收到连接后，向客户端发送一句消息，消息头是“message”，消息体是"this is server"。消息头用来表识消息内容，在客户端，需要识别这个消息头才能正确获取消息体内容。

2、同时，监听客户端发来的消息头为“message“的消息。

客户端：
```htm

<script src="https://cdn.bootcss.com/socket.io/2.2.0/socket.io.js"></script>
<script>
    var socket = io('http://127.0.0.1:8000');
    socket.on("connect", function(){
        console.log("connect");
        socket.emit("message","this is client");
    });
    socket.on("message", function(message){
        console.log("message from server:",message);
    });
    
</script>
```
代码解析：

1、加载socket.io库客户端。

2、连接websocket服务器，连接成功后，发送一条消息。并像服务器一样监听“message“类消息。

# 将Node.JS代码编译成字节码
本文介绍一种NodeJS源代码保护方式
通过把nodejs代码转化为字节码，用node启动字节码文件的方式，保护nodejs源代码不泄漏。可应用于nodejs项目提交源码、nodejs产品在不可信的环境中部署，防止别人获取源码。

如同JS代码一样，nodejs源码，也是透明代码，通常用node启动代码时，都必须把源码也放置到启动环境中。这在很多时候是不安全不稳妥的。因为js源码透明的原因，别人可以直接获取到产品或项目源码。

如果是为第三方定制项目，对方可以直接拿到源码。如果是要在某些环境中启动项目，比如虚拟主机、他人的服务器中，源码的也是很令人担心的。

为了防止源码泄漏带来的一系列令人不安的后果，这里介绍一种专门针对于nodejs源码的保护技术：将nodejs代码转化为字节码文件。

实现原理
nodejs的内核中对于js的解析，使用的是谷歌的v8引擎。v8引擎内置有js虚拟机。通过v8虚拟机，可以将js代码编译为字节码。而v8虚拟机是能够识别和直接运行该字节码的。因此，以下执行逻辑成为可能：

1、js代码 -> js字节码
2、js字节码 -> nodejs ->运行

实现代码（例程）
生成字节码文件的部分：

```javascript
var v8 = require('v8');
var fs = require('fs');

//读取源文件（JS源码）
var js_code = fs.readFileSync(__dirname+"/test.js").toString();

//生成字节码
var script = new vm.Script(js_code, {produceCachedData: true});
var byte_code = script.cachedData;

//将字节码写入文件
fs.writeFileSync(__dirname+"/test.jsb",byte_code);

// 另一个文件 
//读取并运行字节码的部分：
var v8 = require('v8');
var fs = require('fs');

//从文件中读取字节码
byte_code = fs.readFileSync(__dirname+"/test.jsb");

//运行
var l = byte_code.slice(8, 12).reduce(function (sum, number, power) { return sum += number * Math.pow(256, power);});
var dummyCode =" ".repeat(l);
script = new vm.Script(dummyCode, {cachedData: byte_code});
script.runInThisContext();
```
生成字节码，读取、运行字节码。如此操作起来，并不复杂，但如果量大的话，还是稍有些繁琐的。

# Express文件上传
例程
本例需要两个文件及一个目录
```python
test39.js：主程序；
index.html：用于上传文件的前端页面；
temp_folder：存放被上传的文件。
```
test39.js：
```javascript
var express = require('express');
var app = express();

//form表单需要的中间件。
var mutipart= require('connect-multiparty');
var mutipartMiddeware = mutipart();
//临时文件的储存位置
app.use(mutipart({uploadDir:'./temp_file'}));
app.set('port',process.env.PORT || 8000);
app.listen(app.get('port'),function () {
	console.log("Express started on http://localhost:"+app.get('port')+',press Ctrl-C to terminate.');
});

//浏览器访问localhost会输出一个html文件
app.get('/',function (req,res) {
	res.type('text/html');
	res.sendFile(__dirname+'/index.html')
});

//这里就是接受form表单请求的接口路径，请求方式为post。
app.post('/upload',mutipartMiddeware,function (req,res) {
	//这里打印可以看到接收到文件的信息。
	console.log(req.files);

	//成功接受到浏览器传来的文件。我们可以在这里写对文件的一系列操作。例如重命名，修改文件储存路径 。等等。


	//给浏览器返回一个成功提示。
	res.send('upload success!');
});
```
index.html：
```htm

<html>
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  <body>
    <form action="/upload" enctype="multipart/form-data" method="post">
      <p>附件：<input type="file" name="myfile"></p>
      <p><input type="submit"></p>
    </form>
  </body>
</html>

```

# 压缩和解压ZIP文件
compressing是nodejs的一个三方模块，用于压缩和解压文件，支持windows和linux下多种压缩格式，如zip、gzip、tgz、tar。

压缩
```javascript
var compressing = require("compressing");

//压缩
compressing.zip.compressDir(__dirname+"/test/", "test.zip")
.then(() => {
    console.log('zip','success');
})
.catch(err => {
    console.error('zip',err);
});
```
即：将当前目录下的test目录内容压缩为test.zip。

解压
```javascript
var compressing = require("compressing");

//解压
compressing.zip.uncompress(__dirname+"/test.zip", "/test2/")
.then(() => {
    console.log('unzip','success');
})
.catch(err => {
    console.error('unzip',err);
});
```
即：解压test.zip文件，解压到test2目录下。


# 获取磁盘空间信息
如果你的某个程序需要在磁盘上大量存放文件，那么，监测当前磁盘可用空间、当磁盘空间可用率低于某个值时，发出提示或告警，是个很实用很需要的功能。如何实现这个功能呢？

且看代码：
```javascript
var diskinfo = require('diskinfo');
//当前盘符
var current_disk = __dirname.substr(0,2).toLowerCase();

//获得所有磁盘空间
diskinfo.getDrives(function(err, aDrives) {

		//遍历所有磁盘信息
		for (var i = 0; i < aDrives.length; i++) {
			
			//只获取当前磁盘信息
			if( aDrives[i].mounted.toLowerCase() == current_disk ){

					//盘符号
					var mounted = 'mounted ' + aDrives[i].mounted;
					//总量
					var total  = 'total ' + (aDrives[i].blocks /1024 /1024 /1024).toFixed(0) + "gb";
					//已使用
					var used = 'used ' + (aDrives[i].used /1024 /1024 /1024).toFixed(0) + "gb";
					//可用
					var available = 'available ' + (aDrives[i].available /1024 /1024 /1024).toFixed(0) + "gb";
					//使用率
					var capacity = 'capacity ' + aDrives[i].capacity;
					
					console.log(mounted+"\r\n"+total+"\r\n"+used+"\r\n"+available+"\r\n"+capacity);
			}
		}

});
```

# 断言，调试和测试必备
assert模块，即断言，是Node的内置模块。常用于程序调试、单元测试，也可用于实现错误处理逻辑。

且看其最常见用法：assert.equal()

equal方法接受三个参数，第一个参数是实际值，第二个是预期值，第三个是错误的提示信息。

例程
```javascript
var assert = require('assert');

function add (a, b) {
    return a + b;
}

var expected = add(1,2);
assert.equal(expected, 3, '预期1+2等于3');
```

此时执行，assert.equal（）的第一和第二个参数值相等，也就是比较第一和第二个参数的结果为true，这时不会有输出。只有比较结果为 false 时，才会提醒。

asset模块还有其它多种测试语法：

assert.fail(actual, expected, message, operator)
assert(value, message)
assert.ok(value, [message])
assert.equal(actual, expected, [message])
assert.notEqual(actual, expected, [message])
assert.deepEqual(actual, expected, [message])
assert.notDeepEqual(actual, expected, [message])
assert.strictEqual(actual, expected, [message])
assert.notStrictEqual(actual, expected, [message])
assert.throws(block, [error], [message])
assert.doesNotThrow(block, [message])
assert.ifError(value)

# 获错误之“未捕获的异常”
造成Node程序崩溃的，几乎都是“未捕获的异常”。当一个“未捕获的异常”出现时，Node会默认的终止进程的执行。

其实process.on()方法可以捕获进程级的异常，如：
```javascript
var http = require("http");
var server = http.createServer(function(req,res){
    response.end("Hello JShaman.com");
});
server.listen(8000);

process.on("uncaughtException",function(err){
    console.error(err.message,err.stack);
})
```

执行、用浏览器发起访问，这时会报错，但程序并没有退出。用process.on()捕获，并使程序不退出，真的好吗？

答案是：不好！因为会造成资源泄漏、内存泄漏。

也就是说，一旦发生这种错误，程序是必然要崩的。没有立刻退出，也会因为资源泄漏，造成内存耗尽等原因而不久后退出。

正确的做法是：
```javascript
var http = require("http");
var server = http.createServer(function(req,res){
    response.end("Hello JShaman.com");
});
server.listen(8000);

process.on("uncaughtException",function(err){
    console.error(err.message,err.stack);

    server.close();
    setTimeout(process.exit(),3000);
})
```

即：处理该处理的，然后退出。再然后呢，自然是找到问题原因，调试并解决问题。

方法？如何调试？
使用“域”：

```javascript
var http = require("http");
var domain = require("domain");

var d = domain.create();
d.run(function(){
    var server = http.createServer(function(req,res){

        d.on("error",function(er){
            res.statusCode = 500;
            res.end("internal server error");
            server.close();
            setTimeout(process.exit(),3000,1);
        })

        response.end("Hello JShaman.com");
    });

    server.listen(8000);

});
```
使用域，可以让发代码运行在一个沙盒中、可以更好的监控部分代码。如果我们怀疑哪部分代码有问题，可以使用这种方法，监控、将错误提示给用户端，当然也可以输出日志协助我们找到问题点、解决问题。

# 性能分析
首先写一个例程：
```javascript
function makeLoad(){
	for(var i=0; i<100000; i++);
}

function logSomeThing(){
	console.log("something");
}

setInterval(makeLoad,2000);
setInterval(logSomeThing,0);
```
然后加一个prof参数启动程序：
```console
node --prof profile-test.js
```

运行几秒后ctrl+c退出。这时，会在当前目录（test54.js所在目录）生成一个.log文件，如：
```console
isolate-000000000048AC80-v8.log 
```

直接看的话，内容很凌乱，类似API HOOK生成的log（猜测：原理应该是与API HOOK记录类似的）。

这时需要借力另一个三方模块：tick

全局安装 tick后，在命令行下运行：
```console
node-tick-processor isolate-000000000048AC80-v8.log
```
会生成一个分析记录，最重要的是javascript部分，

# 强大的REPL
在Node中，有一个神器：REPL，全称是：Read Eval Print Loop。即：交互式解释器。从名称上，看不出它能干什么。那么，我们直接从一个示例来看吧，本文共需两个程序：

首先是test55.js，它用以前文章中的一个例程代码，再加一个REPL服务器功能：

```javascript
var http = require("http");

var test="this is a test";

var server = http.createServer(function(req,res){
  res.writeHead(200,{"Content-Type":"text/plain"});
  res.write("Hello JShaman.com");
  res.end();
})

server.listen(8000,function(){
  console.log("listening on port 8000");
});

//repl部分：
var net = require("net");
var repl = require("repl");
net.createServer(function(socket){
  var r = repl.start({
    input:socket,
    output:socket,
    terminal:true,
    useGlobal:true
  });
  r.on("exit",function(){
    socket.end();
  })
  r.context.server = server;
  r.context.test = test;
}).listen(1337);
console.log("repl listening on 1337");
```
前半部分，是一个简单的http服务器功能，后面部分是repl服务器部分。接下来，还需要一个repl客户端：

repl_client.js：
```javascript
var net = require("net");
var socket = net.connect(1337);

process.stdin.setRawMode(true);
process.stdin.pipe(socket);
socket.pipe(process.stdout);

socket.once("close",function(){
    process.stdin.destroy();
})
```
有了这两部分程序，就可以演示repl的强大了：启动test55.js，再启动repl_client.js。在repl_client命令行中操作：

1、通过REPL查看进程信息，如：运行了多少、使用了多少内存：
2、通过REPL查看程序中的变量：

这是怎么实现的呢？test55.js程序中的变量，被REPL客户端获取了。当然，我们也可以通过类似的方法获取其它变量，用于调试的话，这会非常强大。但这还不是最强大的。

2、通过REPL控制程序行为：

输入：
```console
console.log(test);
```
主程序中输出了！

还有更过份的：修改主程序，给主程序添加函数，主程序也会执行。

# 编写一个真正的模块，能发布到NPM上的模块！

# 给图片加水印
本示例有三个文件，一个程序文件、一张图片、一个水印图片。图片可以是jpg或png，水印需要是PNG，因为水印可能要做个圆角或其它部分可能需要透明的形状。

实现代码：
```javascript
var images = require('images');

//水印图片
var watermarkImg = images('water_logo.png');

//等待加水印的图片
var sourceImg = images("test.jpg");

// 比如放置在右下角，先获取原图的尺寸和水印图片尺寸
var sWidth = sourceImg.width();
var sHeight = sourceImg.height();
var wmWidth = watermarkImg.width();
var wmHeight = watermarkImg.height();

//设置绘制的坐标位置，右下角距离 5px
images(sourceImg).draw(watermarkImg, sWidth - wmWidth - 5, sHeight - wmHeight - 5)

//保存
.save("test2.jpg");
    
```

代码很简单，即准备好原图，将水印和原图合并到一起，并保存。

# 开发一套反爬虫系统！

# 解除“封印”！给Node更多的内存
默认情况下当用node启动我们的程序时，可用的最大内存量是512MB。

--max_old_space_size=1，含意为：只给程序1MB的内存。被启动的是ShareWAF，一款大型的Web应用防火墙，1MB内存显然是不够的，所以出错了。

而正是这个参数：max_old_space_size，可以指定我们程序可用的内存量。

当不使用这个参数时，相当于使用默认值--max_old_space_size=512。

那么，看如下的命令：

```javascript
//使用1gb内存
node --max-old-space-size=1024 xx.js

//使用2gb内存
node --max-old-space-size=2048 xx.js
```

# 用集群“榨干”机器性能
在Node.JS中，集群（cluster）是个高端、强大的功能。一般情况下，Node启动一个程序，就是一个进程，占用一定的内存，使用一个CPU资源。而如果用集群，比如是8核的机器，则可启动8个进程，占用8倍的内存。虽然对机器的资源占用的多了，但显然的，性能提升了。不要担心资源占用问题，我们写程序，就是要让程序性能最优。

且看代码：
```javascript
const cluster = require("cluster");
const os = require("os");

//主进程
if(cluster.isMaster){

    //cpu数量（几核）
    const cpus = os.cpus().length;
    console.log(`Clustering to ${cpus} CPUS`);

    for(let i=0; i<cpus; i++){

        //分派子进程
        cluster.fork();
    }
}else{
    //子进程执行内容

    const http = require("http");
    const pid = process.pid;
    
    http.createServer(function(req,res){
        console.log(`Handing request from ${pid}`);
        res.end(`Hello from ${pid}\n`);
    }).listen(8000,function(){
        console.log(`Started ${pid}`);
    })
}
```

这段代码的程序运行时，将启动机器CPU个数个进程。而且访问时，同一机器发起的访问，会被同一子进程处理，这就实现了有状态通信。

# 实现一个“不死”程序
集群的另一个用途是高可用。当集群启动时，会有多个进程，只要其中任何一个进程存活，程序就可正常提供服务。而且，我们可以实现：当某个进程意外中止时，自动重启之。以此达到程序“不死”的效果。

且看代码：
```javascript
const cluster = require("cluster");
const os = require("os");

//主进程
if(cluster.isMaster){

    //cpu数量（几核）
    const cpus = os.cpus().length;
    console.log(`Clustering to ${cpus} CPUS`);

    for(let i=0; i<cpus; i++){

        //分派子进程
        cluster.fork();
    }

    //如果工作进程关闭了，重启一个
    cluster.on("exit",function(worker,code){
        if(code != 0 && !worker.suicide){
            console.log("worker crashed. Starting a new worker");
            cluster.fork();
        }
    });
}else{
    //子进程执行内容

    const http = require("http");
    const pid = process.pid;
    
    http.createServer(function(req,res){
        console.log(`Handing request from ${pid}`);
        res.end(`Hello from ${pid}\n`);
    }).listen(8000,function(){
        console.log(`Started ${pid}`);
    })

    //秒数后触发，主动抛出一个错误
    setTimeout(function(){
        throw new Error("ooops");
    },Math.ceil(Math.random()*3) * 1000);
}
```
此代码，会在启动后数秒抛出错误，当发生错误、程序退出时，主进程会自动重启一个工作进程。

# 零停机重启
一般情况下，当更新代码后，必须重启Node，才能使更新的代码功能生效。重启当然是会令程序功能暂时中断的，虽然这个过程非常短，可能只有几秒。普通的程序可能不需要但心这个问题，但如果是非常重要的程序，不允许间断的程序，又该如何？

使用集群(Cluster），用“零停机重启”方案，可以很简单的实现这个需求。

实现代码：
```javascript
const cluster = require("cluster");
const os = require("os");

//主进程
if(cluster.isMaster){
    console.log("master process id :",process.pid);
    //cpu数量（几核）
    const cpus = os.cpus().length;
    console.log(`Clustering to ${cpus} CPUS`);

    for(let i=0; i<cpus; i++){

        //分派子进程
        cluster.fork();
    }

    //如果工作进程关闭了，重启一个
    /*
    cluster.on("exit",function(worker,code){
        if(code != 0 && !worker.suicide){
            console.log("worker crashed. Starting a new worker");
            cluster.fork();
        }
    });
    */

    //服务器收到这个消息
    process.on("SIGINT",function(){
        console.log("ctrl+c");
        process.exit();
    });


        
    var express = require("express")();
    express.listen(9000);
    express.get("/restart",function(req,res,next){
        const workers = Object.keys(cluster.workers);
            
        //重启函数
        function restart_worker(i){
            if(i >= workers.length) return;

            //第i个工作进程
            var worker = cluster.workers[workers[i]];
            console.log(`Stoping worker:${worker.process.pid}`);

            //中断工作进程
            worker.disconnect();

            //工作进程退出时
            worker.on("exit",function(){

                /*
                if(!worker.suicide){
                    console.log("suicide");
                    return;
                }
                */

                //启动工作进程
                const new_worker = cluster.fork();

                //当新的工作进程，准备好，并开始监听新的连接时，迭代重启下一个工作子进程
                new_worker.on("listening",function(){
                    restart_worker(i+1);
                })
            });
        }

        //重启第一个工作进程
        restart_worker(0);

        res.end("restart ok");

    });

}else{
    //子进程执行内容

    const http = require("http");
    const pid = process.pid;
    
    http.createServer(function(req,res){
        console.log(`Handing request from ${pid}`);
        res.end(`Hello from ${pid}\n`);
    }).listen(8000,function(){
        console.log(`Started ${pid}`);

    })
}
```

代码解析：
集群主进程用express提供web服务；集群工作进程用http提供web服务；当主进程收到指定消息时（代码中是访问restart路径），开始“零停机重启”操作。

实现重点是：停掉一个工作进程，并重启一个新的工作进程，当新的工作进程启动好，并进入监听状态时（即：可正常提供Web服务时），再重启下一个工作进程，直到全部重启完成。


# NodeJS如何实现真正的长连接？

# 用cookie实现websocket自动登录，session状态保留

注意，是websocket通信，一般常见的介绍是Http登录状态，websocket实现方案很少。

要实现的效果是：
输入帐号密码登录，登录后刷新、关闭页面均不影响登录状态、还会自动进入登录后的页面。

原理：
在服务端，当登录操作时，保存客户cookie，如果再次有连接，检查连接请求的cookie是否与登录客户保留的cookie一致，如果一致则认为已经登录，无需再次校验。

1、客户端index.html页面代码如下：
```htm
<html>
    <head>
    <meta charset="UTF-8">
    </head>
    <body>
    <div id="login" style="display: block;">
        <form >
            user: <input type="text" id="user" value="admin"/>
            
            pass: <input type="password" id="pass" value="adminpass"/>
            
            <input type="button" value="Submit" onclick="login();"/>
        </form>
    </div>
    <div id="logined" style="display: none;">
    </div>
    <script>
        function login(){
            var user = document.getElementById("user").value;
            var pass = document.getElementById("pass").value;
            
            socket_io.emit("login",{"user":user, "pass":pass});
            
        }
    </script>
    <script src="socket.io.js"></script>
    <script>
        var socket_io = io("http://127.0.0.1:8080");
        socket_io.on("login_ret",function(data){
            if(data.status == 0){
                document.getElementById("login").style.display = "none";
                document.getElementById("logined").style.display = "block";
                document.getElementById("logined").innerText = "You have logined:" + data.user;
            }
        });
    </script>
    </body>
</html>
```

2、服务端server.js代码如下：
```javascript
var app = require('express')().listen(8080);
var io = require('socket.io')(app);
//会话状态
var session = [];
//websocket连接
io.on('connection', function (socket) {

    //cookie是否存在
    if(socket.request.headers.cookie != undefined){

        //遍历会话，检查此是否有此cookie对应的登录状态
        for(i=0; i<session.length; i++){
            if(session[i].cookie == socket.request.headers.cookie){

                console.log(socket.request.headers.cookie);
                //登录成功
                socket.emit("login_ret",{"status":0, "message":"login success", "user":session[i].user});
                console.log(session[i].user,"login succcess by session");
                return;
            }
        }
    }
    //登录
    socket.on('login', function (data) {
        //登录校验
		if((data.user="admin") && (data.pass="adminpass")){
            if(socket.request.headers.cookie != undefined){
                //保存在会话中
                session.push({"cookie":socket.request.headers.cookie,"user":data.user});
            }
            //登录成功
            socket.emit("login_ret",{"status":0, "message":"login success", "user":data.user});
            console.log(data.user,"login success");
        }else{
            //登录失败
            socket.emit("login_ret",{"status":1, "message":"login failed", "user":data.user});
            console.log(data.user,"login failed");
        }
    });
});
```













https://www.fairysoftware.com/node.js_in_practice.html

https://blog.csdn.net/qq_39055970/article/details/119756714

https://www.fairysoftware.com/node.js_in_practice.html