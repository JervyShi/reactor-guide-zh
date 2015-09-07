# Codecs and Buffer

字节码操作对大量数据管道配置类的应用来说是一个核心关注点。它广泛应用于`reactor-net`模块来序列化和反序列化从IO中接收或发送的字节码。

`reactor.io.buffer.Buffer`是Java中`ByteBuffer`的装饰器，增加了一系列操作。目的是通过`ByteBuffer`的限制和读取或者覆盖预分配的字节来减少对字节的复制。追踪`ByteBuffer`的位置能把开发者折腾的头痛不已。`Buffer`帮我们处理了这些问题，我们决定把这个简单的工具带给用户。

下面是在Groovy Spock测试中写的一个简单操作Buffer的例子：

```
import reactor.io.buffer.Buffer

//...

given: "an empty Buffer and a full Buffer"
                def buff = new Buffer()
                def fullBuff = Buffer.wrap("Hello World!")

when: "a Buffer is appended"
                buff.append(fullBuff)

then: "the Buffer was added"
                buff.position() == 12
                buff.flip().asString() == "Hello World!"
```

`Buffer`的一种十分有用的应用是`Buffer.View`，很多操作如`split()`就会返回它。它提供一种无需拷贝的方式来扫描和检索`ByteBuffer`中的字节码。`Buffer.View`也是一种`Buffer`，所以有些操作也是对外暴露的。

使用一个分隔符和`Buffer.View`来复用一些相同的字节码：

```
byte delimiter = (byte) ';';
byte innerDelimiter = (byte) ',';

Buffer buffer = Buffer.wrap("a;b-1,b-2;c;d;");

List<Buffer.View> views = buffer.split(delimiter);

int viewCount = views.size();
Assert.isTrue(viewCount == 4);

for (Buffer.View view : views) {
    System.out.println(view.get().asString()); //prints "a" then "b-1,b-2", then "c" and finally "d"

    if(view.indexOf(innerDelimiter) != -1){
      for(Buffer.View innerView : view.get().split(innerDelimiter)){
        System.out.println(innerView.get().asString()); //prints "b-1" and "b-2"
      }
    }
}
```

在正常的序列化/反序列化的场景中使用`Buffer`会显得有些层级过低，`Reactor`提供一些称为`Codec`的预定义转换器。一些`Codec`在使用时需要在classpath下增加一些额外的依赖，比如处理Json时需要用到Jackson。

`Codec`有两种工作方式，第一种是实现Funcation来直接编码并且返回编码后的数据，通常以Buffer的表单形式返回。这非常棒，但仅限于与无状态的`Codec`工作才能起效，另一个可选的方法是使用`Codec.encoder()`返回的编码函数。

**Codec.encoder() vs Codec.apply(Source)**

* Codec.encoder() 返回一个唯一编码函数，且不能在多个线程间共享。
* Codec.apply() 直接编码（并保存分配的编码器），但是`Codec`本身需要在多个线程间共享。

> Reactor Net模块实际上通过每个新线程调用一次`Codec.encoder`来避免线程安全问题

对于大部分实现了`Buffer`的`Codec`来说，`Codec`同样也能通过源类型解码数据。我们需要通过`Codec.decoder()`来获取一个解码函数用于解码数据。和编码不同的是，没有可重写的方法用于解码数据。和编码相同的是，解码方法也不是线程安全的。

有两种形式的`Codec.decoder()`函数，其中一个简单的返回解码数据，另一个`Codec.decoder(Consumer)`将会调用consumer处理每次解码事件。

**Codec.decoder() vs Codec.decoder(Consumer)**

* Codec.decoder() 是一种阻塞的解码函数，它将会直接返回传入源数据对应的解码数据。
* Codec.decoder(Consumer) 可以用于非阻塞式解码，调用时不会返回任何东西，仅仅在解码完毕时对传入Consumer进行回调。它可以与任何异步场景进行整合。

Using one of the predefined Codecs as verified in this Groovy Spock test:

```
import reactor.io.json.JsonCodec

//...

given: 'A JSON codec'
                def codec = new JsonCodec<Map<String, Object>, Object>(Map);
    def latch = new CountDownLatch(1)

when: 'The decoder is passed some JSON'
                Map<String, Object> decoded;
                def callbackDecoder = codec.decoder{
                  decoded = it
                  latch.countDown()
                }
                def blockingDecoder = codec.decoder()

                //yes this is real simple async strategy, but that's not the point here :)
                Thread.start{
                  callbackDecoder.apply(Buffer.wrap("{\"a\": \"alpha\"}"))
    }

    def decodedMap = blockingDecoder.apply(Buffer.wrap("{\"a\": \"beta\"}")

then: 'The decoded maps have the expected entries'
    latch.await()
                decoded.size() == 1
                decoded['a'] == 'alpha'
                decodedMap['a'] == 'beta'
```

Table 4. Available Core Codecs:

Name|Description|Required Dependency
----|-----------|-------------------
ByteArrayCodec|Wrap/unwrap byte arrays from/to Buffer.|N/A
DelimitedCodec|Split/Aggregate Buffer and delegate to the passed Codec for unit marshalling.|N/A
FrameCodec|Split/Aggregate Buffer into Frame Buffers according to successive prefix lengths.|N/A
JavaSerializationCodec|Deserialize/Serialize Buffers using Java Serialization.|N/A
PassThroughCodec|Leave the Buffers untouched.|N/A
StringCodec|Convert String to/from Buffer|N/A
LengthFieldCodec|Find the length and decode/encode the appropriate number of bytes into/from Buffer|N/A
KryoCodec|Convert Buffer into Java objects using Kryo with Buffers|com.esotericsoftware.kryo:kryo
JsonCodec,JacksonJsonCodec|Convert Buffer into Java objects using Jackson with Buffers|com.fasterxml.jackson.core:jackson-databind
SnappyCodec|A Compression Codec which applies a delegate Codec after unpacking/before packing Buffer|org.xerial.snappy:snappy-java
GZipCodec|A Compression Codec which applies a delegate Codec after unpacking/before packing Buffer|N/A
