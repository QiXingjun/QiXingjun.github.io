---
layout: post
title: 深入理解Java NIO--Part Two
categories: Java NIO
description: 深入理解Java NIO
keywords: Java，NIO，IO
---

## 1. 分散（Scatter）/聚集（Gather）

Java NIO开始支持scatter/gather，scatter/gather用于描述从Channel中读取或者写入到Channel的操作。 

分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。 

聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel。因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。 

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。 

### 1.1 Scattering Reads 

Scattering Reads是指数据从一个channel读取到多个buffer中。如下图描述： 

![scattering-reads](http://i.imgur.com/5z4GV6H.png)

代码示例如下： 

```java
ByteBuffer header = ByteBuffer.allocate(128);  
ByteBuffer body   = ByteBuffer.allocate(1024);  
  
ByteBuffer[] bufferArray = { header, body };  
  
channel.read(bufferArray);  
```

注意buffer首先被插入到数组，然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。 

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(即：消息大小不固定的消息)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。 

### 1.2 Gathering Writes 

Gathering Writes是指数据从多个buffer写入到同一个channel。如下图描述： 

![gathering-writes](http://i.imgur.com/QVTS2xq.png)


代码示例如下： 

```java
ByteBuffer header = ByteBuffer.allocate(128);  
ByteBuffer body   = ByteBuffer.allocate(1024);  
  
//write data into buffers  
  
ByteBuffer[] bufferArray = { header, body };  
  
channel.write(bufferArray);  
```

buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。 

## 2. 通道之间的数据传输

在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel传输到另外一个channel。 

### 2.1 transferFrom() 

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中（注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个简单的例子： 

```java 
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");  
FileChannel      fromChannel = fromFile.getChannel();  
  
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");  
FileChannel      toChannel = toFile.getChannel();  
  
long position = 0;  
long count = fromChannel.size();  
  
toChannel.transferFrom(position, count, fromChannel);  
```

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。 

此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。 

### 2.2 transferTo() 

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子： 

```java 
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");  
FileChannel      fromChannel = fromFile.getChannel();  
  
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");  
FileChannel      toChannel = toFile.getChannel();  
  
long position = 0;  
long count = fromChannel.size();  
  
fromChannel.transferTo(position, count, toChannel);  
```

是不是发现这个例子和前面那个例子特别相似？除了调用方法的FileChannel对象不一样外，其他的都一样。 

上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。 

## 3. 选择器（Selector）

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。 

### 3.1 为什么使用Selector? 

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。 

但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。在这里，只要知道使用Selector能够处理多个通道就足够了。 

下面是单线程使用一个Selector处理3个channel的示例图： 

![selector](http://i.imgur.com/MmcZ8Eo.png)

### 3.2 Selector的创建 

通过调用Selector.open()方法创建一个Selector，如下： 

```java 
Selector selector = Selector.open();  
```

### 3.3 向Selector注册通道 

为了将Channel和Selector配合使用，必须将channel注册到selector上。通过SelectableChannel.register()方法来实现，如下： 

```java 
channel.configureBlocking(false);  
SelectionKey key = channel.register(selector,  
    Selectionkey.OP_READ);  
```

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。 

注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件： 

* Connect
* Accept
* Read
* Write

通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。 

这四种事件用SelectionKey的四个常量来表示： 

* SelectionKey.OP_CONNECT
* SelectionKey.OP_ACCEPT
* SelectionKey.OP_READ
* SelectionKey.OP_WRITE

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下： 

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;  
```

在下面还会继续提到interest集合。 

### 3.4 SelectionKey 

在上一小节中，当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。这个对象包含了一些你感兴趣的属性： 

* interest集合
* ready集合
* Channel
* Selector
* 附加的对象（可选）

下面我会描述这些属性。 

#### 3.4.1 interest集合 

就像向Selector注册通道一节中所描述的，interest集合是你所选择的感兴趣的事件集合。可以通过SelectionKey读写interest集合，像这样： 

```java
int interestSet = selectionKey.interestOps();  
  
boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT；  
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;  
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;  
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;  
```

可以看到，用“位与”操作interest 集合和给定的SelectionKey常量，可以确定某个确定的事件是否在interest 集合中。 

#### 3.4.2 ready集合 

ready 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个ready set。Selection将在下一小节进行解释。可以这样访问ready集合：
 
```java
int readySet = selectionKey.readyOps(); 
```

可以用像检测interest集合那样的方法，来检测channel中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型： 

```java
selectionKey.isAcceptable();  
selectionKey.isConnectable();  
selectionKey.isReadable();  
selectionKey.isWritable();  
```

#### 3.4.3 Channel + Selector 

从SelectionKey访问Channel和Selector很简单。如下： 

```java
Channel  channel  = selectionKey.channel();  
Selector selector = selectionKey.selector();  
```s

#### 3.4.4 附加的对象 

可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加更多信息与通道一起使用的Buffer，或是包含聚集数据的某个对象。使用方法如下： 

```java
selectionKey.attach(theObject);  
Object attachedObj = selectionKey.attachment();  
```

还可以在用register()方法向Selector注册Channel的时候附加对象。如： 

```java 
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);  
```

### 3.5 通过Selector选择通道 

一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。 

下面是select()方法： 

* int select()
* int select(long timeout)
* int selectNow()

select()阻塞到至少有一个通道在你注册的事件上就绪了。 

select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。 

selectNow()不会阻塞，不管什么通道就绪都立刻返回（此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。 

select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。 

**selectedKeys()**

一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示： 

```java
Set selectedKeys = selector.selectedKeys();  
```

当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。 

可以遍历这个已选择的键集合来访问就绪的通道。如下： 

```java
Set selectedKeys = selector.selectedKeys();  
Iterator keyIterator = selectedKeys.iterator();  
while(keyIterator.hasNext()) {  
    SelectionKey key = keyIterator.next();  
    if(key.isAcceptable()) {  
        // a connection was accepted by a ServerSocketChannel.  
    } else if (key.isConnectable()) {  
        // a connection was established with a remote server.  
    } else if (key.isReadable()) {  
        // a channel is ready for reading  
    } else if (key.isWritable()) {  
        // a channel is ready for writing  
    }  
    keyIterator.<tuihighlight class="tuihighlight"><a href="javascript:;" style="display:inline;float:none;position:inherit;cursor:pointer;color:#7962D5;text-decoration:underline;" onclick="return false;">remove</a></tuihighlight>();  
}  
```

这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。 

注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。 

SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。 

### 3.6 wakeUp() 

某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。 

如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法上，下个调用select()方法的线程会立即“醒来（wake up）”。 

### 3.7 close() 

用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。 

### 3.8 完整的示例 

这里有一个完整的示例，打开一个Selector，注册一个通道注册到这个Selector上(通道的初始化过程略去),然后持续监控这个Selector的四种事件（接受，连接，读，写）是否就绪。
 
```java 
Selector selector = Selector.open();  
channel.configureBlocking(false);  
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);  
while(true) {  
  int readyChannels = selector.select();  
  if(readyChannels == 0) continue;  
  Set selectedKeys = selector.selectedKeys();  
  Iterator keyIterator = selectedKeys.iterator();  
  while(keyIterator.hasNext()) {  
    SelectionKey key = keyIterator.next();  
    if(key.isAcceptable()) {  
        // a connection was accepted by a ServerSocketChannel.  
    } else if (key.isConnectable()) {  
        // a connection was established with a remote server.  
    } else if (key.isReadable()) {  
        // a channel is ready for reading  
    } else if (key.isWritable()) {  
        // a channel is ready for writing  
    }  
    keyIterator.<tuihighlight class="tuihighlight"><a href="javascript:;" style="display:inline;float:none;position:inherit;cursor:pointer;color:#7962D5;text-decoration:underline;" onclick="return false;">remove</a></tuihighlight>();  
  }  
}  
```

## 4. 文件通道

Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。 

FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。 

打开FileChannel 

在使用FileChannel之前，必须先打开它。但是，我们无法直接打开一个FileChannel，需要通过使用一个InputStream、OutputStream或RandomAccessFile来获取一个FileChannel实例。下面是通过RandomAccessFile打开FileChannel的示例： 

Java代码 
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");  
FileChannel inChannel = aFile.getChannel();  


从FileChannel读取数据 

调用多个read()方法之一从FileChannel中读取数据。如： 

Java代码 
ByteBuffer buf = ByteBuffer.allocate(48);  
int bytesRead = inChannel.read(buf);  


首先，分配一个Buffer。从FileChannel中读取的数据将被读到Buffer中。 

然后，调用FileChannel.read()方法。该方法将数据从FileChannel读取到Buffer中。read()方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1，表示到了文件末尾。 

向FileChannel写数据 

使用FileChannel.write()方法向FileChannel写数据，该方法的参数是一个Buffer。如： 

Java代码 
String newData = "New String to write to file..." + System.currentTimeMillis();  
  
ByteBuffer buf = ByteBuffer.allocate(48);  
buf.clear();  
buf.put(newData.getBytes());  
  
buf.flip();  
  
while(buf.hasRemaining()) {  
    channel.write(buf);  
}  


注意FileChannel.write()是在while循环中调用的。因为无法保证write()方法一次能向FileChannel写入多少字节，因此需要重复调用write()方法，直到Buffer中已经没有尚未写入通道的字节。 

关闭FileChannel 

用完FileChannel后必须将其关闭。如： 

Java代码 
channel.close();  


FileChannel的position方法 

有时可能需要在FileChannel的某个特定位置进行数据的读/写操作。可以通过调用position()方法获取FileChannel的当前位置。 

也可以通过调用position(long pos)方法设置FileChannel的当前位置。 

这里有两个例子： 

Java代码 
long pos = channel.position();  
channel.position(pos +123);  


如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 —— 文件结束标志。 

如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“文件空洞”，磁盘上物理文件中写入的数据间有空隙。 

FileChannel的size方法 

FileChannel实例的size()方法将返回该实例所关联文件的大小。如： 

Java代码 
long fileSize = channel.size();  


FileChannel的truncate方法 

可以使用FileChannel.truncate()方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。如：

Java代码 
channel.truncate(1024);  


这个例子截取文件的前1024个字节。 

FileChannel的force方法 

FileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到FileChannel里的数据一定会即时写到磁盘上。要保证这一点，需要调用force()方法。 

force()方法有一个boolean类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。 

下面的例子同时将文件数据和元数据强制写到磁盘上： 

Java代码 
channel.force(true);  


Socket 通道 Top



（本部分原文链接，作者：Jakob Jenkov，译者：郑玉婷，校对：丁一） 
Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel： 

打开一个SocketChannel并连接到互联网上的某台服务器。
一个新连接到达ServerSocketChannel时，会创建一个SocketChannel。
打开 SocketChannel 

下面是SocketChannel的打开方式： 

Java代码 
SocketChannel socketChannel = SocketChannel.open();  
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));  


关闭 SocketChannel 

当用完SocketChannel之后调用SocketChannel.close()关闭SocketChannel： 

Java代码 
socketChannel.close();  


从 SocketChannel 读取数据 

要从SocketChannel中读取数据，调用一个read()的方法之一。以下是例子： 

Java代码 
ByteBuffer buf = ByteBuffer.allocate(48);  
int bytesRead = socketChannel.read(buf);  


首先，分配一个Buffer。从SocketChannel读取到的数据将会放到这个Buffer中。 

然后，调用SocketChannel.read()。该方法将数据从SocketChannel 读到Buffer中。read()方法返回的int值表示读了多少字节进Buffer里。如果返回的是-1，表示已经读到了流的末尾（连接关闭了）。 

写入 SocketChannel 

写数据到SocketChannel用的是SocketChannel.write()方法，该方法以一个Buffer作为参数。示例如下： 

Java代码 
String newData = "New String to write to file..." + System.currentTimeMillis();  
  
ByteBuffer buf = ByteBuffer.allocate(48);  
buf.clear();  
buf.put(newData.getBytes());  
  
buf.flip();  
  
while(buf.hasRemaining()) {  
    channel.write(buf);  
}  


注意SocketChannel.write()方法的调用是在一个while循环中的。Write()方法无法保证能写多少字节到SocketChannel。所以，我们重复调用write()直到Buffer没有要写的字节为止。 

非阻塞模式 

可以设置 SocketChannel 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用connect(), read() 和write()了。 

connect() 

如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用finishConnect()的方法。像这样： 

Java代码 
socketChannel.configureBlocking(false);  
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));  
  
while(! socketChannel.finishConnect() ){  
    //wait, or do something else...  
}  


write() 

非阻塞模式下，write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用write()。前面已经有例子了，这里就不赘述了。 

read() 

非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。 

非阻塞模式与选择器 

非阻塞模式与选择器搭配会工作的更好，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。Selector与SocketChannel的搭配使用会在后面详讲。 



ServerSocket 通道 Top


（本部分原文链接，作者：Jakob Jenkov，译者：郑玉婷，校对：丁一） 
Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道，就像标准IO中的ServerSocket一样。ServerSocketChannel类在 java.nio.channels包中。 

这里有个例子： 

Java代码 
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();  
  
serverSocketChannel.socket().bind(new InetSocketAddress(9999));  
  
while(true){  
    SocketChannel socketChannel =  
            serverSocketChannel.accept();  
  
    //do something with socketChannel...  
}  


打开 ServerSocketChannel 

通过调用 ServerSocketChannel.open() 方法来打开ServerSocketChannel.如： 

Java代码 
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();  


关闭 ServerSocketChannel 

通过调用ServerSocketChannel.close() 方法来关闭ServerSocketChannel. 如： 

Java代码 
serverSocketChannel.close();  


监听新进来的连接 

通过 ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候，它返回一个包含新进来的连接的 SocketChannel。因此，accept()方法会一直阻塞到有新连接到达。 

通常不会仅仅只监听一个连接，在while循环中调用 accept()方法. 如下面的例子： 

Java代码 
while(true){  
    SocketChannel socketChannel =  
            serverSocketChannel.accept();  
  
    //do something with socketChannel...  
}  


当然，也可以在while循环中使用除了true以外的其它退出准则。 

非阻塞模式 

ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接，返回的将是null。 因此，需要检查返回的SocketChannel是否是null。如： 

Java代码 
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();  
  
serverSocketChannel.socket().bind(new InetSocketAddress(9999));  
serverSocketChannel.configureBlocking(false);  
  
while(true){  
    SocketChannel socketChannel =  
            serverSocketChannel.accept();  
  
    if(socketChannel != null){  
        //do something with socketChannel...  
    }  
}  


Datagram 通道 Top



（本部分原文链接，作者：Jakob Jenkov，译者：郑玉婷，校对：丁一） 
Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。 

打开 DatagramChannel 

下面是 DatagramChannel 的打开方式： 

Java代码 
DatagramChannel channel = DatagramChannel.open();  
channel.socket().bind(new InetSocketAddress(9999));  


这个例子打开的 DatagramChannel可以在UDP端口9999上接收数据包。 

接收数据 

通过receive()方法从DatagramChannel接收数据，如： 

Java代码 
ByteBuffer buf = ByteBuffer.allocate(48);  
buf.clear();  
channel.receive(buf);  


receive()方法会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据，多出的数据将被丢弃。 

发送数据 

通过send()方法从DatagramChannel发送数据，如: 

Java代码 
String newData = "New String to write to file..." + System.currentTimeMillis();  
  
ByteBuffer buf = ByteBuffer.allocate(48);  
buf.clear();  
buf.put(newData.getBytes());  
buf.flip();  
  
int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));  


这个例子发送一串字符到”jenkov.com”服务器的UDP端口80。 因为服务端并没有监控这个端口，所以什么也不会发生。也不会通知你发出的数据包是否已收到，因为UDP在数据传送方面没有任何保证。 

连接到特定的地址 

可以将DatagramChannel“连接”到网络中的特定地址的。由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。 

这里有个例子: 

Java代码 
channel.connect(new InetSocketAddress("jenkov.com", 80));  


当连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子： 

Java代码 
int bytesRead = channel.read(buf);  
int bytesWritten = channel.write(but);  


管道（Pipe） Top


（本部分原文链接，作者：Jakob Jenkov，译者：黄忠，校对：丁一） 
Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。 

这里是Pipe原理的图示： 




创建管道 

通过Pipe.open()方法打开管道。例如： 

Java代码 
Pipe pipe = Pipe.open();  


向管道写数据 

要向管道写数据，需要访问sink通道。像这样： 

Java代码 
Pipe.SinkChannel sinkChannel = pipe.sink();  


通过调用SinkChannel的write()方法，将数据写入SinkChannel,像这样： 

Java代码 
String newData = "New String to write to file..." + System.currentTimeMillis();  
ByteBuffer buf = ByteBuffer.allocate(48);  
buf.clear();  
buf.put(newData.getBytes());  
  
buf.flip();  
  
while(buf.hasRemaining()) {  
    <b>sinkChannel.write(buf);</b>  
}  


从管道读取数据 

从读取管道的数据，需要访问source通道，像这样： 

Java代码 
Pipe.SourceChannel sourceChannel = pipe.source();  


调用source通道的read()方法来读取数据，像这样： 

Java代码 
ByteBuffer buf = ByteBuffer.allocate(48);  
  
int bytesRead = inChannel.read(buf);  


read()方法返回的int值会告诉我们多少字节被读进了缓冲区。 