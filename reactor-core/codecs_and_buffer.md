# Codecs and Buffer

字节码操作对大量数据管道配置类的应用来说是一个核心关注点。它广泛应用于`reactor-net`模块来序列化和反序列化从IO中接收或发送的字节码。

`reactor.io.buffer.Buffer`是Java中`ByteBuffer`的装饰器，增加了一系列操作。目的是通过`ByteBuffer`的限制和读取或者覆盖预分配的字节来减少对字节的复制。追踪`ByteBuffer`的位置能把开发者折腾的头痛不已。`Buffer`帮我们处理了这些问题，我们决定把这个简单的工具带给用户。

A simple Buffer manipulation as seen in one of our Groovy Spock test:

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

A useful application for Buffer is Buffer.View, which is returned by multiple operations such as split(). It simply provides for a copy-free way to scan and introspect bytes from ByteBuffer. Buffer.View is also a kind of Buffer, so the same operations are exposed.

Reusing the same bytes for chunked read using a delimiter and Buffer.View:

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

Playing with Buffer can feel a bit low-level for the common marshalling/unmarshalling use cases and Reactor comes with a series of pre-defined converters called Codec. Some Codec will be requiring the appropriate extra dependency in the classpath to work, like Jackson for JSON manipulation.

Codec works in two ways, first it implements Function to encode anything directly and return the encoded data, usually under the form of Buffer. This is great but it does only work with stateless Codecs, the alternative is to use the returned encoding function from Codec.encoder().

**Codec.encoder() vs Codec.apply(Source)**

* Codec.encoder() returns a unique encoding function that should not be shared between different Threads.
* Codec.apply() directly encodes (and save an encoder allocation), but the Codec itself should be shared between Threads in that case

> Reactor Net handles that difference for you in fact by calling Codec.encoder for each new connection.

Codec can also decode data into from source type, usually Buffer for most of the Codec implementations. To decode a source data, we must retrieve a decoding function from Codec.decoder(). Unlike in the encoding side, there isn’t any apply shortcut as the method is already overloaded for the encoding purpose. Like the encoding side, the decoding function should not be shared between Threads.

There are two forms of Codec.decoder() functions, one will simply return the decoded data where Codec.decoder(Consumer) will call the passed consumer on each decoding event.

**Codec.decoder() vs Codec.decoder(Consumer)**

* Codec.decoder() is a blocking decoding function, it will return directly the decoded data from the passed source.
* Codec.decoder(Consumer) can be used for non-blocking decoding, it will return nothing (null) and only invoke the passed Consumer once decoded. It can be combined with any asynchronous facility.

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
