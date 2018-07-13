* 1. In the standard IO API you work with byte streams and character streams. In NIO you work with channels and buffers. Data is always read from a channel into a buffer, or written from a buffer to a channel.
* 2. Java NIO contains the concept of "selectors". A selector is an object that can monitor multiple channels for events (like: connection opened, data arrived etc.). Thus, a single thread can monitor multiple channels for data.
  To use a Selector you register the Channel's with it. Then you call it's select() method. This method will block until there is an event ready for one of the registered channels. Once the method returns, the thread can then process these events. Examples of events are incoming connection, data received etc.
* 3. Channel is like a stream in IO; all data is read from and write to channels. 4 channels listed:
  * FileChannel
  * DatagramChannel
  * SocketChannel
  * ServerSocketChannel
* 4. A FileChannel example
```
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
* 5. A buffer is essentially a block of memory into which you can write data, which you can then later read again. This memory block is wrapped in a NIO Buffer object, which provides a set of methods that makes it easier to work with the memory block.
  * Using a Buffer to read and write data typically follows this little 4-step process:
    * Write data into the Buffer
    * Call buffer.flip()
    * Read data out of the Buffer
    * Call buffer.clear() or buffer.compact()
  * Capacity, Position and Limit
    * The meaning of position and limit depends on whether the Buffer is in read or write mode. Capacity always means the same, no matter the buffer mode.
    * Position
      * When you write data into the Buffer, you do so at a certain position. Initially the position is 0. When a byte has been written into the Buffer the position is advanced to point to the next cell in the buffer to insert data into. Position can maximally become capacity - 1.

      * When you read data from a Buffer you also do so from a given position. When you flip a Buffer from writing mode to reading mode, the position is reset back to 0. As you read data from the Buffer you do so from position, and position is advanced to next position to read.
    * Limit
      * In write mode the limit of a Buffer is the limit of how much data you can write into the buffer. In write mode the limit is equal to the capacity of the Buffer.
      * When flipping the Buffer into read mode, limit means the limit of how much data you can read from the data. Therefore, when flipping a Buffer into read mode, limit is set to write position of the write mode. In other words, you can read as many bytes as were written (limit is set to the number of bytes written, which is marked by position).
    * Mark
      * A buffer's mark is the index to which its position will be reset when the rest method is invoked.  The mark is not always defined, but when it is defined it is never negative and is never greater than the position.  If the mark is defined then it is discarded when the position or the limit is adjusted to a value smaller than the mark.  If the mark is not defined then invoking the reset method causes an InvalidMarkException to be thrown.
  * Methods
    * clear makes a buffer ready for a new sequence of channel-read or relative put operations: It sets the limit to the capacity and the position to zero.
    * flip makes a buffer ready for a new sequence of channel-write or relative get operations: It sets the limit to the current position and then sets the position to zero.
    * rewind makes a buffer ready for re-reading the data that it already contains: It leaves the limit unchanged and sets the position to zero.
* 6. A Selector is a Java NIO component which can examine one or more NIO Channel's, and determine which channels are ready for e.g. reading or writing. This way a single thread can manage multiple channels, and thus multiple network connections.
  * The Channel must be in non-blocking mode to be used with a Selector. This means that you cannot use FileChannel's with a Selector since FileChannel's cannot be switched into non-blocking mode. Socket channels will work fine though.
  ```
  channel.configureBlocking(false);

  SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
  ```
  * Notice the second parameter of the register() method. This is an "interest set", meaning what events you are interested in listening for in the Channel, via the Selector. There are four different events you can listen for:
    * Connect
    * Accept
    * Read
    * Write
  * code snippet
```
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

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

    keyIterator.remove();
  }
}

```
* 7 Pipe
```
Pipe pipe = Pipe.open();

Pipe.SinkChannel sinkChannel = pipe.sink();

String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}

Pipe.SourceChannel sourceChannel = pipe.source();

ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```

