---
layout: post
title: 深入理解Java NIO--Part One
categories: Java NIO
description: 深入理解Java NIO
keywords: Java，NIO，IO
---

Java NIO（New IO）是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API.Java NIO提供了与标准IO不同的IO工作方式： 

**Channels and Buffers（通道和缓冲区）**：标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

**Asynchronous IO（异步IO）**：Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。

**Selectors（选择器）**：Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

## 1. Java NIO 概述

Java NIO 由以下几个核心部分组成： 

* Channels
* Buffers
* Selectors

虽然Java NIO 中除此之外还有很多类和组件，但在我看来，Channel，Buffer 和 Selector 构成了核心的API。

其它组件，如Pipe和FileLock，只不过是与三个核心组件共同使用的工具类。

### 1.1 Channel 和 Buffer 

基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流，数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。这里有个图示： 

![channel and buffer](http://i.imgur.com/QgqbGJS.png)

Channel和Buffer有好几种类型。下面是JAVA NIO中的一些主要Channel的实现： 

* FileChannel
* DatagramChannel
* SocketChannel
* ServerSocketChannel

正如你所看到的，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。 

以下是Java NIO里关键的Buffer实现： 

* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。 

Java NIO 还有个 MappedByteBuffer，用于表示内存映射文件。 

### 1.2 Selector 

Selector允许单线程处理多个 Channel。

如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。 

这是在一个单线程中使用一个Selector处理3个Channel的图示： 

![selector](http://i.imgur.com/XDdE2vm.png)


要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如:新连接进来，数据接收等。 

## 2. Java NIO vs IO

当学习了Java NIO和IO的API后，一个问题马上涌入脑海： 
应该何时使用IO，何时使用NIO呢？在本文中，会尽量清晰地解析Java NIO和IO的差异、它们的使用场景，以及它们如何影响您的代码设计。

### 2.1 Java NIO和IO的主要区别 

下表总结了Java NIO和IO之间的主要差别，我会更详细地描述表中每部分的差异。 

|IO	             |NIO   			 |
| -------------- | -----------------:|
|Stream oriented |Buffer oriented    |
|Blocking IO	 |Non blocking IO	 |
| 				 |Selectors			 |


#### 2.1.1 面向流与面向缓冲 

Java NIO和IO之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。 Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。 

#### 2.1.2 阻塞与非阻塞IO 

Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。 

#### 2.1.3 选择器（Selectors） 

Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。 

### 2.2 NIO和IO如何影响应用程序的设计 

无论您选择IO或NIO工具箱，可能会影响您应用程序设计的以下几个方面： 

* 对NIO或IO类的API调用。
* 数据处理。
* 用来处理数据的线程数。

#### 2.2.1 API调用 

当然，使用NIO的API调用时看起来与使用IO时有所不同，但这并不意外，因为并不是仅从一个InputStream逐字节读取，而是数据必须先读入缓冲区再处理。 

#### 2.2.2 数据处理 

使用纯粹的NIO设计相较IO设计，数据处理也受到影响。 

在IO设计中，我们从InputStream或 Reader逐字节读取数据。假设你正在处理一基于行的文本数据流，例如： 

```
Name: Anna  
Age: 25  
Email: anna@mailserver.com  
Phone: 1234567890  
```

该文本行的流可以这样处理： 

```java
InputStream input = … ; // get the InputStream from the client socket  
BufferedReader reader = new BufferedReader(new InputStreamReader(input));  
  
String nameLine   = reader.readLine();  
String ageLine    = reader.readLine();  
String emailLine  = reader.readLine();  
String phoneLine  = reader.readLine();  
```

请注意处理状态由程序执行多久决定。换句话说，一旦reader.readLine()方法返回，你就知道肯定文本行就已读完， readline()阻塞直到整行读完，这就是原因。你也知道此行包含名称；同样，第二个readline()调用返回的时候，你知道这行包含年龄等。 正如你可以看到，该处理程序仅在有新数据读入时运行，并知道每步的数据是什么。一旦正在运行的线程已处理过读入的某些数据，该线程不会再回退数据（大多如此）。下图也说明了这条原则： 

![read-from-block-stream](http://i.imgur.com/UgYXGyP.png)

而一个NIO的实现会有所不同，下面是一个简单的例子： 

```java
ByteBuffer buffer = ByteBuffer.allocate(48);  
  
int bytesRead = inChannel.read(buffer);  
```

注意第二行，从通道读取字节到ByteBuffer。当这个方法调用返回时，你不知道你所需的所有数据是否在缓冲区内。你所知道的是，该缓冲区包含一些字节，这使得处理有点困难。 
假设第一次 read(buffer)调用后，读入缓冲区的数据只有半行，例如，“Name:An”，你能处理数据吗？显然不能，需要等待，直到整行数据读入缓存，在此之前，对数据的任何处理毫无意义。 

所以，你怎么知道是否该缓冲区包含足够的数据可以处理呢？好了，你不知道。发现的方法只能查看缓冲区中的数据。其结果是，在你知道所有数据都在缓冲区里之前，你必须检查几次缓冲区的数据。这不仅效率低下，而且可以使程序设计方案杂乱不堪。例如： 

```java
ByteBuffer buffer = ByteBuffer.allocate(48);  
int bytesRead = inChannel.read(buffer);  
while(! bufferFull(bytesRead) ) {  
bytesRead = inChannel.read(buffer);  
}  
```

bufferFull()方法必须跟踪有多少数据读入缓冲区，并返回真或假，这取决于缓冲区是否已满。换句话说，如果缓冲区准备好被处理，那么表示缓冲区满了。 

bufferFull()方法扫描缓冲区，但必须保持在bufferFull()方法被调用之前状态相同。如果没有，下一个读入缓冲区的数据可能无法读到正确的位置。这是不可能的，但却是需要注意的又一问题。 

如果缓冲区已满，它可以被处理。如果它不满，并且在你的实际案例中有意义，你或许能处理其中的部分数据。但是许多情况下并非如此。下图展示了“缓冲区数据循环就绪”,即从一个通道里读数据，直到所有的数据都读到缓冲区里：

![read-from-channel-until-all-datas-read-into-buffer](http://i.imgur.com/pWQp2om.png)


### 2.3 总结 

NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。 

如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。同样，如果你需要维持许多打开的连接到其他计算机上，如P2P网络中，使用一个单独的线程来管理你所有出站连接，可能是一个优势。一个线程多个连接的设计方案如下图所示，即单线程管理多个连接：

![one-thread-manage-multi-connection](http://i.imgur.com/kLYnW4D.png)


如果你有少量的连接使用非常高的带宽，一次发送大量的数据，也许典型的IO服务器实现可能非常契合。下图说明了一个典型的IO服务器设计，即一个连接处理一个线程：

![one-thread-manage-oneconnection](http://i.imgur.com/fjadVr9.png)

## 3. 通道（Channel）

Java NIO的通道类似流，但又有些不同： 

* 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
* 通道可以异步地读写。
* 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

Java NIO中最重要的通道的实现： 

* FileChannel：从文件中读写数据。
* DatagramChannel：能通过UDP读写网络中的数据。
* SocketChannel：能通过TCP读写网络中的数据。
* ServerSocketChannel：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

### 3.1 基本的 Channel 示例 

下面是一个使用FileChannel读取数据到Buffer中的示例： 

```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");  
FileChannel inChannel = aFile.getChannel();  
  
ByteBuffer buf = ByteBuffer.allocate(48);  
  
int bytesRead = inChannel.read(buf);  
while (bytesRead != -1) {  
  
System.out.println("Read " + bytesRead);  
buf.flip();  
  
while(buf.hasRemaining()){  
System.out.print((char) buf.get());  
}  
  
buf.clear();  
bytesRead = inChannel.read(buf);  
}  
aFile.close();  
```

注意 buf.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据。


## 4. 缓冲区（Buffer）

Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。 

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。 

### 4.1 Buffer的基本用法 

使用Buffer读写数据一般遵循以下四个步骤： 

* 写入数据到Buffer
* 调用flip()方法
* 从Buffer中读取数据
* 调用clear()方法或者compact()方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。 

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。 

下面是一个使用Buffer的例子： 

```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");  
FileChannel inChannel = aFile.getChannel();  
  
//create buffer with capacity of 48 bytes  
ByteBuffer buf = ByteBuffer.allocate(48);  
  
int bytesRead = inChannel.read(buf); //read into buffer.  
while (bytesRead != -1) {  
  
  buf.flip();  //make buffer ready for read  
  
  while(buf.hasRemaining()){  
      System.out.print((char) buf.get()); // read 1 byte at a time  
  }  
  
  buf.clear(); //make buffer ready for writing  
  bytesRead = inChannel.read(buf);  
}  
aFile.close();  
```

### 4.2 Buffer的capacity,position和limit 

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。 

为了理解Buffer的工作原理，需要熟悉它的三个属性： 

* capacity
* position
* limit

position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。 

这里有一个关于capacity，position和limit在读写模式中的说明，详细的解释在插图后面。 

![capacity-position-limit](http://i.imgur.com/it7MHr3.png)

#### 4.2.1 capacity 

作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。 

#### 4.2.2 position 

当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1。 

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0。当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。 

#### 4.2.3 limit 

在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。 

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position） 

### 4.3 Buffer的类型 

Java NIO 有以下Buffer类型： 

* ByteBuffer
* MappedByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

这些Buffer类型代表了不同的数据类型。换句话说，就是可以通过char，short，int，long，float 或 double类型来操作缓冲区中的字节。 

### 4.4 Buffer的分配 

要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。

下面是一个分配48字节capacity的ByteBuffer的例子。 

```java
ByteBuffer buf = ByteBuffer.allocate(48);  
```

这是分配一个可存储1024个字符的CharBuffer： 

```java
CharBuffer buf = CharBuffer.allocate(1024);  
```

### 4.5 向Buffer中写数据 

写数据到Buffer有两种方式： 

* 从Channel写到Buffer。
* 通过Buffer的put()方法写到Buffer里。

从Channel写到Buffer的例子 

```java
int bytesRead = inChannel.read(buf); //read into buffer.  
```

通过put方法写Buffer的例子： 

```java
buf.put(127);  
```

put方法有很多版本，允许你以不同的方式把数据写入到Buffer中。例如，写到一个指定的位置，或者把一个字节数组写入到Buffer。 

### 4.6 flip()方法 

flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。 

换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。 

### 4.7 从Buffer中读取数据 

从Buffer中读取数据有两种方式： 

* 从Buffer读取数据到Channel。
* 使用get()方法从Buffer中读取数据。

从Buffer读取数据到Channel的例子： 

```java 
//read from buffer into channel.  
int bytesWritten = inChannel.write(buf);  
```

使用get()方法从Buffer中读取数据的例子 

```java 
byte aByte = buf.get();  
```

get方法有很多版本，允许你以不同的方式从Buffer中读取数据。例如，从指定position读取，或者从Buffer中读取数据到字节数组。

### 4.8 rewind()方法 

Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。 

### 4.9 clear()与compact()方法 

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。 

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。 

如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。 

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。 

### 4.10 mark()与reset()方法 

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如： 

```java 
buffer.mark();  
  
//call buffer.get() a couple of times, e.g. during parsing.  
  
buffer.reset();  //set position back to mark.  
```

### 4.11 equals()与compareTo()方法 

可以使用equals()和compareTo()方法比较两个Buffer。 

#### 4.11.1 equals() 

当满足下列条件时，表示两个Buffer相等： 

* 有相同的类型（byte、char、int等）。
* Buffer中剩余的byte、char等的个数相等。
* Buffer中所有剩余的byte、char等都相同。

如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只比较Buffer中的剩余元素。 

#### 4.11.2 compareTo()方法 

compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer： 

* 第一个不相等的元素小于另一个Buffer中对应的元素。
* 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。

（注：剩余元素是从 position到limit之间的元素） 
