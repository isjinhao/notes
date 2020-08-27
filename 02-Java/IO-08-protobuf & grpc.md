## Protobuf 2

### 官方入门文档

This tutorial provides a basic Java programmer's introduction to working with protocol buffers. By walking through creating a simple example application, it shows you how to

- Define message formats in a `.proto` file.
- Use the protocol buffer compiler.
- Use the Java protocol buffer API to write and read messages.

This isn't a comprehensive guide to using protocol buffers in Java. For more detailed reference information, see the [Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto), the [Java API Reference](https://developers.google.com/protocol-buffers/docs/reference/java), the [Java Generated Code Guide](https://developers.google.com/protocol-buffers/docs/reference/java-generated), and the [Encoding Reference](https://developers.google.com/protocol-buffers/docs/encoding).

#### Why Use Protocol Buffers?

The example we're going to use is a very simple "address book" application that can read and write people's contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:

- Use Java Serialization. This is the default approach since it's built into the language, but it has a host of（许多） well-known problems (see Effective Java, by Josh Bloch pp. 213), and also doesn't work very well if you need to share data with applications written in C++ or Python.
- You can invent an ad-hoc（临时） way to encode the data items into a single string – such as encoding 4 ints as "12:3:-23:67". This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously（臭名昭著） space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.

Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.

#### Where to Find the Example Code

The example code is included in the source code package, under the "examples" directory. [Download it here.](https://developers.google.com/protocol-buffers/docs/downloads)

#### Defining Your Protocol Format

To create your address book application, you'll need to start with a `.proto` file. The definitions in a `.proto` file are simple: you add a *message* for each data structure you want to serialize, then specify a name and a type for each field in the message. Here is the `.proto` file that defines your messages, `addressbook.proto`.

```protobuf
syntax = "proto2";

package tutorial;

option java_package = "com.example.tutorial";
option java_outer_classname = "AddressBookProtos";

message Person {
    required string name = 1;
    required int32 id = 2;
    optional string email = 3;

    enum PhoneType {
        MOBILE = 0;
        HOME = 1;
        WORK = 2;
    }

    message PhoneNumber {
        required string number = 1;
        optional PhoneType type = 2 [default = HOME];
    }

    repeated PhoneNumber phones = 4;
}

message AddressBook {
	repeated Person people = 1;
}
```

As you can see, the syntax is similar to C++ or Java. Let's go through each part of the file and see what it does.

The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects. In Java, the package name is used as the Java package unless you have explicitly specified a `java_package`, as we have here. Even if you do provide a `java_package`, you should still define a normal `package` as well to avoid name collisions in the Protocol Buffers name space as well as in non-Java languages.

After the package declaration, you can see two options that are Java-specific: `java_package` and `java_outer_classname`. `java_package` specifies in what Java package name your generated classes should live. If you don't specify this explicitly, it simply matches the package name given by the `package` declaration, but these names usually aren't appropriate Java package names (since they usually don't start with a domain name). The `java_outer_classname` option defines the class name which should contain all of the classes in this file. If you don't give a `java_outer_classname` explicitly, it will be generated by converting the file name to camel case. For example, "my_proto.proto" would, by default, use "MyProto" as the outer class name.

Next, you have your message definitions. A message is just an aggregate containing a set of typed fields. Many standard simple data types are available as field types, including `bool`, `int32`, `float`, `double`, and `string`. You can also add further structure to your messages by using other message types as field types – in the above example the `Person` message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages. You can even define message types nested inside other messages – as you can see, the `PhoneNumber` type is defined inside `Person`. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of `MOBILE`, `HOME`, or `WORK`.

The " = 1", " = 2" markers on each element identify the unique "tag" that field uses in the binary encoding. Tag numbers 1-15 require one less byte to encode than higher numbers, so as an optimization you can decide to use those tags for the commonly used or repeated elements, leaving tags 16 and higher for less-commonly used optional elements. Each element in a repeated field requires re-encoding the tag number, so repeated fields are particularly good candidates for this optimization.

Each field must be annotated with one of the following modifiers:

- `required`: a value for the field must be provided, otherwise the message will be considered "uninitialized". Trying to build an uninitialized message will throw a `RuntimeException`. Parsing an uninitialized message will throw an `IOException`. Other than this, a required field behaves exactly like an optional field.
- `optional`: the field may or may not be set. If an optional field value isn't set, a default value is used. For simple types, you can specify your own default value, as we've done for the phone number `type` in the example. Otherwise, a system default is used: zero for numeric types, the empty string for strings, false for bools. For embedded messages, the default value is always the "default instance" or "prototype" of the message, which has none of its fields set. Calling the accessor to get the value of an optional (or required) field which has not been explicitly set always returns that field's default value.
- `repeated`: the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.

> **Required Is Forever** You should be very careful about marking fields as `required`. If at some point you wish to stop writing or sending a required field, it will be problematic to change the field to an optional field – old readers will consider messages without this field to be incomplete and may reject or drop them unintentionally. You should consider writing application-specific custom validation routines for your buffers instead. Some engineers at Google have come to the conclusion that using `required` does more harm than good; they prefer to use only `optional` and `repeated`. However, this view is not universal.
>
> 简单来说，不建议使用 `required`

You'll find a complete guide to writing `.proto` files – including all the possible field types – in the [Protocol Buffer Language Guide](https://developers.google.com/protocol-buffers/docs/proto). Don't go looking for facilities similar to class inheritance, though – protocol buffers don't do that.

#### Compiling Your Protocol Buffers

Now that you have a `.proto`, the next thing you need to do is generate the classes you'll need to read and write `AddressBook` (and hence `Person` and `PhoneNumber`) messages. To do this, you need to run the protocol buffer compiler `protoc` on your `.proto`:

1. If you haven't installed the compiler, [download the package](https://developers.google.com/protocol-buffers/docs/downloads) and follow the instructions in the README.

2. Now run the compiler, specifying the source directory (where your application's source code lives – the current directory is used if you don't provide a value), the destination directory (where you want the generated code to go; often the same as `$SRC_DIR` ), and the path to your `.proto` . In this case, you...:

   ```
   protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
   ```

   Because you want Java classes, you use the `--java_out`  option – similar options are provided for other supported languages.

This generates `com/example/tutorial/AddressBookProtos.java` in your specified destination directory.

#### The Protocol Buffer API

Let's look at some of the generated code and see what classes and methods the compiler has created for you. If you look in `AddressBookProtos.java`, you can see that it defines a class called `AddressBookProtos`, nested within which is a class for each message you specified in `addressbook.proto`. Each class has its own `Builder` class that you use to create instances of that class. You can find out more about builders in the [Builders vs. Messages](https://developers.google.com/protocol-buffers/docs/javatutorial#builders) section below.

Both messages and builders have auto-generated accessor methods for each field of the message; messages have only getters while builders have both getters and setters. Here are some of the accessors for the `Person` class (implementations omitted for brevity（简洁）):

```java
// required string name = 1;
public boolean hasName();
public String getName();

// required int32 id = 2;
public boolean hasId();
public int getId();

// optional string email = 3;
public boolean hasEmail();
public String getEmail();

// repeated .tutorial.Person.PhoneNumber phones = 4;
public List<PhoneNumber> getPhonesList();
public int getPhonesCount();
public PhoneNumber getPhones(int index);
```

Meanwhile, `Person.Builder` has the same getters plus setters:

```java
// required string name = 1;
public boolean hasName();
public java.lang.String getName();
public Builder setName(String value);
public Builder clearName();

// required int32 id = 2;
public boolean hasId();
public int getId();
public Builder setId(int value);
public Builder clearId();

// optional string email = 3;
public boolean hasEmail();
public String getEmail();
public Builder setEmail(String value);
public Builder clearEmail();

// repeated .tutorial.Person.PhoneNumber phones = 4;
public List<PhoneNumber> getPhonesList();
public int getPhonesCount();
public PhoneNumber getPhones(int index);
public Builder setPhones(int index, PhoneNumber value);
public Builder addPhones(PhoneNumber value);
public Builder addAllPhones(Iterable<PhoneNumber> value);
public Builder clearPhones();
```

As you can see, there are simple JavaBeans-style getters and setters for each field. There are also `has` getters for each singular（单数的） field which return true if that field has been set. Finally, each field has a `clear` method that un-sets the field back to its empty state.

Repeated fields have some extra methods – a `Count` method (which is just shorthand for the list's size), getters and setters which get or set a specific element of the list by index, an `add` method which appends a new element to the list, and an `addAll` method which adds an entire container full of elements to the list.

Notice how these accessor methods use camel-case naming, even though the `.proto` file uses lowercase-with-underscores. This transformation is done automatically by the protocol buffer compiler so that the generated classes match standard Java style conventions. You should always use lowercase-with-underscores for field names in your `.proto` files; this ensures good naming practice in all the generated languages. See the [style guide](https://developers.google.com/protocol-buffers/docs/style) for more on good `.proto` style.

For more information on exactly what members the protocol compiler generates for any particular field definition, see the [Java generated code reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated).

#### Enums and Nested Classes

The generated code includes a `PhoneType` Java 5 enum, nested within `Person`:

```protobuf
public static enum PhoneType {
  MOBILE(0, 0),
  HOME(1, 1),
  WORK(2, 2),
  ;
  ...
}
```

The nested type `Person.PhoneNumber` is generated, as you'd expect, as a nested class within `Person`.

#### Builders vs. Messages

The message classes generated by the protocol buffer compiler are all *immutable*. Once a message object is constructed, it cannot be modified, just like a Java `String`. To construct a message, you must first construct a builder, set any fields you want to set to your chosen values, then call the builder's `build()` method.

You may have noticed that each method of the builder which modifies the message returns another builder. The returned object is actually the same builder on which you called the method. It is returned for convenience so that you can string several setters together on a single line of code.

Here's an example of how you would create an instance of `Person`:

```protobuf
Person john =
  Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .addPhones(
      Person.PhoneNumber.newBuilder()
        .setNumber("555-4321")
        .setType(Person.PhoneType.HOME))
    .build();
```

#### Standard Message Methods

Each message and builder class also contains a number of other methods that let you check or manipulate the entire message, including:

- `isInitialized()`: checks if all the required fields have been set.
- `toString()`: returns a human-readable representation of the message, particularly useful for debugging.
- `mergeFrom(Message other)`: (builder only) merges the contents of `other` into this message, overwriting singular scalar fields, merging composite fields, and concatenating repeated fields.
- `clear()`: (builder only) clears all the fields back to the empty state.

These methods implement the `Message` and `Message.Builder` interfaces shared by all Java messages and builders. For more information, see the [complete API documentation for `Message`](https://developers.google.com/protocol-buffers/docs/reference/java/com/google/protobuf/Message).

#### Parsing and Serialization

Finally, each protocol buffer class has methods for writing and reading messages of your chosen type using the protocol buffer [binary format](https://developers.google.com/protocol-buffers/docs/encoding). These include:

- `byte[] toByteArray();`: serializes the message and returns a byte array containing its raw bytes.
- `static Person parseFrom(byte[] data);`: parses a message from the given byte array.
- `void writeTo(OutputStream output);`: serializes the message and writes it to an `OutputStream`.
- `static Person parseFrom(InputStream input);`: reads and parses a message from an `InputStream`.

These are just a couple of the options provided for parsing and serialization. Again, see the [`Message` API reference](https://developers.google.com/protocol-buffers/docs/reference/java/com/google/protobuf/Message) for a complete list.

> **Protocol Buffers and O-O Design** Protocol buffer classes are basically dumb（哑的） data holders (like structs in C); they don't make good first class citizens in an object model. If you want to add richer behaviour to a generated class, the best way to do this is to wrap the generated protocol buffer class in an application-specific class. Wrapping protocol buffers is also a good idea if you don't have control over the design of the `.proto` file (if, say, you're reusing one from another project). In that case, you can use the wrapper class to craft an interface better suited to the unique environment of your application: hiding some data and methods, exposing convenience functions, etc. **You should never add behaviour to the generated classes by inheriting from them**. This will break internal mechanisms and is not good object-oriented practice anyway.

#### Writing A Message

Now let's try using your protocol buffer classes. The first thing you want your address book application to be able to do is write personal details to your address book file. To do this, you need to create and populate instances of your protocol buffer classes and then write them to an output stream.

Here is a program which reads an `AddressBook` from a file, adds one new `Person` to it based on user input, and writes the new `AddressBook` back out to the file again. The parts which directly call or reference code generated by the protocol compiler are highlighted.

```java
import com.example.tutorial.AddressBookProtos.AddressBook;
import com.example.tutorial.AddressBookProtos.Person;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import java.io.IOException;
import java.io.PrintStream;

class AddPerson {
    // This function fills in a Person message based on user input.
    static Person PromptForAddress(BufferedReader stdin,
                                   PrintStream stdout) throws IOException {
        Person.Builder person = Person.newBuilder();

        stdout.print("Enter person ID: ");
        person.setId(Integer.valueOf(stdin.readLine()));

        stdout.print("Enter name: ");
        person.setName(stdin.readLine());

        stdout.print("Enter email address (blank for none): ");
        String email = stdin.readLine();
        if (email.length() > 0) {
            person.setEmail(email);
        }

        while (true) {
            stdout.print("Enter a phone number (or leave blank to finish): ");
            String number = stdin.readLine();
            if (number.length() == 0) {
                break;
            }

            Person.PhoneNumber.Builder phoneNumber =
                Person.PhoneNumber.newBuilder().setNumber(number);

            stdout.print("Is this a mobile, home, or work phone? ");
            String type = stdin.readLine();
            if (type.equals("mobile")) {
                phoneNumber.setType(Person.PhoneType.MOBILE);
            } else if (type.equals("home")) {
                phoneNumber.setType(Person.PhoneType.HOME);
            } else if (type.equals("work")) {
                phoneNumber.setType(Person.PhoneType.WORK);
            } else {
                stdout.println("Unknown phone type.  Using default.");
            }

            person.addPhones(phoneNumber);
        }

        return person.build();
    }

    // Main function:  Reads the entire address book from a file,
    //   adds one person based on user input, then writes it back out to the same
    //   file.
    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println("Usage:  AddPerson ADDRESS_BOOK_FILE");
            System.exit(-1);
        }

        AddressBook.Builder addressBook = AddressBook.newBuilder();

        // Read the existing address book.
        try {
            addressBook.mergeFrom(new FileInputStream(args[0]));
        } catch (FileNotFoundException e) {
            System.out.println(args[0] + ": File not found.  Creating a new file.");
        }

        // Add an address.
        addressBook.addPerson(
            PromptForAddress(new BufferedReader(new InputStreamReader(System.in)),
                             System.out));

        // Write the new address book back to disk.
        FileOutputStream output = new FileOutputStream(args[0]);
        addressBook.build().writeTo(output);
        output.close();
    }
}
```

#### Reading A Message

```java
import com.example.tutorial.AddressBookProtos.AddressBook;
import com.example.tutorial.AddressBookProtos.Person;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.PrintStream;

class ListPeople {
    // Iterates though all people in the AddressBook and prints info about them.
    static void Print(AddressBook addressBook) {
        for (Person person: addressBook.getPeopleList()) {
            System.out.println("Person ID: " + person.getId());
            System.out.println("  Name: " + person.getName());
            if (person.hasEmail()) {
                System.out.println("  E-mail address: " + person.getEmail());
            }

            for (Person.PhoneNumber phoneNumber : person.getPhonesList()) {
                switch (phoneNumber.getType()) {
                    case MOBILE:
                        System.out.print("  Mobile phone #: ");
                        break;
                    case HOME:
                        System.out.print("  Home phone #: ");
                        break;
                    case WORK:
                        System.out.print("  Work phone #: ");
                        break;
                }
                System.out.println(phoneNumber.getNumber());
            }
        }
    }

    // Main function:  Reads the entire address book from a file and prints all
    //   the information inside.
    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println("Usage:  ListPeople ADDRESS_BOOK_FILE");
            System.exit(-1);
        }

        // Read the existing address book.
        AddressBook addressBook =
            AddressBook.parseFrom(new FileInputStream(args[0]));

        Print(addressBook);
    }
}
```



### 简单应用

**proto文件**

```protobuf
syntax= "proto2";
// 永远不要修改生成的代码，只能修改这里的代码
// 使用方式 protoc --java_out=src/main/java  src/protobuf/Studuent.proto
// 产生class文件,具体其他生产方法，可以看官方文档.
package com.protobuf.test;

option optimize_for = SPEED;    // 加快解析速度，详细可以去官网查加快解析速度，不写默认是这个
option java_package="cn.isjinhao.proto2";
option java_outer_classname="MyDataInfo";

message Student{
    required string name=1;
    optional int32 id=2;
    optional string address=3;
}
```

**java 程序**

```java
public class Protobuf2Test {

    public static void main(String[] args) throws InvalidProtocolBufferException {

        MyDataInfo.Student student = MyDataInfo.Student.newBuilder().setName("张三").setAddress("北京").build();
        byte[] bytes = student.toByteArray();
        MyDataInfo.Student student1 = MyDataInfo.Student.parseFrom(bytes);
        System.out.println(student1.getName() + " " + student1.getAddress());
    }
}
// 张三 北京
```



### Netty集成Protobuf2

**proto文件**

```protobuf
syntax= "proto2";
// 永远不要修改生成的代码，只能修改这里的代码
// 使用方式 protoc --java_out=src/main/java  src/protobuf/Student2.proto
// 产生class文件,具体其他生产方法，可以看官方文档.
package com.protobuf.test;

option optimize_for = SPEED;    // 加快解析速度，详细可以去官网查加快解析速度，不写默认是这个
option java_package="cn.isjinhao.nettydemo.sixexample";
option java_outer_classname="MyDataInfo2";

message Student{
    required string name=1;
    optional int32 id=2;
    optional string address=3;
}
```

**Client**

```java
public class MyClient {
    public static void main(String[] args) throws Exception {
        //事件循环组,只有一个循环组
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventLoopGroup).
                    channel(NioSocketChannel.class).
                    handler(new MyClientInitializer());
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8899).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}
```

```java
public class MyClientInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline=ch.pipeline();
        /**
         * 主要是这四个ProtobufHander，用于protobuf编解码。
         */
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
 
        pipeline.addLast(new ProtobufDecoder(MyDataInfo2.Student.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());

        // 因为这里我们是针对MyDataInfo.Student数据，所以下面自定义的handler中，咱们的泛型也是MyDataInfo.Student类型
        pipeline.addLast(new MyClientHandler());
    }
}
```

```java
public class MyClientHandler extends SimpleChannelInboundHandler<MyDataInfo2.Student> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo2.Student msg) 
        throws Exception { }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        MyDataInfo2.Student student = 
            MyDataInfo2.Student.newBuilder().setName("北京").setAddress("上海").build();
        ctx.channel().writeAndFlush(student);
    }
}
```

**Server**

```java
public class MyServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new MyServerInitializer());
            ChannelFuture channelFuture = serverBootstrap.bind(8899).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

```java
public class MyServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
        pipeline.addLast(new ProtobufDecoder(MyDataInfo2.Student.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());
        pipeline.addLast(new MyServerHandler());
    }
}
```

```java
public class MyServerHandler extends SimpleChannelInboundHandler<MyDataInfo2.Student> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo2.Student msg) 
        throws Exception {
        System.out.println(msg.getName());
        System.out.println(msg.getAddress());
    }
}
// 启动后，这里会输出信息
```



### oneof 解决多数据载体问题

**proto文件**

```protobuf
syntax= "proto2";
// 永远不要修改生成的代码，只能修改这里的代码
// 使用方式 protoc --java_out=src/main/java  src/protobuf/Studuent3.proto
// 产生class文件,具体其他生产方法，可以看官方文档.
package com.protobuf.test;

option optimize_for = SPEED;    // 加快解析速度，详细可以去官网查加快解析速度，不写默认是这个
option java_package="cn.isjinhao.nettydemo.seventhexample";
option java_outer_classname="MyDataInfo3";

message MyMessage {
    enum DataType{
        StudentType = 1;
        DogType = 2;
        CatType = 3;
    }
    required DataType data_type = 1;
    oneof dataBody{
        Student student = 2;
        Dog dog = 3;
        Cat cat = 4;
    }
}

message Student{
    optional string name = 1;
    optional int32 id = 2;
    optional string address = 3;
}

message Dog {
    optional string name = 1;
    optional int32 id = 2;
}

message Cat {
    optional string name=1;
    optional string city=2;
}
```

**Server**

```java
public class MyServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new MyServerInitializer());
            ChannelFuture channelFuture = serverBootstrap.bind(8899).sync();
            channelFuture.channel().closeFuture().sync();

        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

```java
public class MyServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
        pipeline.addLast(new ProtobufDecoder(MyDataInfo3.MyMessage.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());
        pipeline.addLast(new MyServerHandler());
    }
}
```

```java
public class MyServerHandler extends SimpleChannelInboundHandler<MyDataInfo3.MyMessage> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo3.MyMessage msg) 
        throws Exception {
        if (msg.getDataType() == MyDataInfo3.MyMessage.DataType.StudentType) {
            MyDataInfo3.Student student = msg.getStudent();
            System.out.println(student.getName());
            System.out.println(student.getAddress());
        } else if (msg.getDataType() == MyDataInfo3.MyMessage.DataType.DogType) {
            MyDataInfo3.Dog dog = msg.getDog();
            System.out.println(dog.getName());
            System.out.println(dog.getId());
        } else if (msg.getDataType() == MyDataInfo3.MyMessage.DataType.CatType) {
            MyDataInfo3.Cat cat = msg.getCat();
            System.out.println(cat.getName());
            System.out.println(cat.getCity());
        }
    }
}
```

**Client**

```java
public class MyClient {
    public static void main(String[] args) throws Exception {
        //事件循环组,只有一个循环组
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventLoopGroup).
                    channel(NioSocketChannel.class).
                    handler(new MyClientInitializer());
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8899).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}
```

```java
public class MyClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline=ch.pipeline();
        /**
         * 主要是这四个ProtobufHander，用于protobuf编解码。
         */
        pipeline.addLast(new ProtobufVarint32FrameDecoder());

        pipeline.addLast(new ProtobufDecoder(MyDataInfo3.MyMessage.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());

        // 因为这里我们是针对MyDataInfo.Student数据，所以下面自定义的handler中，咱们的泛型也是MyDataInfo.Student类型
        pipeline.addLast(new MyClientHandler());
    }
}
```

```java
public class MyClientHandler extends SimpleChannelInboundHandler<MyDataInfo3.MyMessage> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo3.MyMessage msg) 
        throws Exception {}

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        int i = new Random().nextInt(3);
        MyDataInfo3.MyMessage myMessage = null;
        if (0 == i) {
            myMessage = MyDataInfo3.MyMessage.newBuilder().
                    setDataType(MyDataInfo3.MyMessage.DataType.StudentType).
                    setStudent(MyDataInfo3.Student.newBuilder().setName("王五").setAddress("深圳").build()).build();
        } else if (1 == i) {
            myMessage = MyDataInfo3.MyMessage.newBuilder().
                    setDataType(MyDataInfo3.MyMessage.DataType.DogType).
                    setDog(MyDataInfo3.Dog.newBuilder().setName("王五").setId(100).build()).build();
        } else if (2 == i) {
            myMessage = MyDataInfo3.MyMessage.newBuilder().
                    setDataType(MyDataInfo3.MyMessage.DataType.CatType).
                    setCat(MyDataInfo3.Cat.newBuilder().setName("王五").setCity("广东").build()).build();
        }
        ctx.channel().writeAndFlush(myMessage);
    }
}
```



## grpc

### 官方入门文档

#### Overview

In gRPC, a client application can directly call a method on a server application on a different machine as if it were a local object, making it easier for you to create distributed applications and services. As in many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a gRPC server to handle client calls. On the client side, the client has a stub (referred to as just a client in some languages) that provides the same methods as the server.

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/493a60ee-2c21-4f55-ba5a-3a4a4ea61a11" /></div>

gRPC clients and servers can run and talk to each other in a variety of environments - from servers inside Google to your own desktop - and can be written in any of gRPC’s supported languages. So, for example, you can easily create a gRPC server in Java with clients in Go, Python, or Ruby. In addition, the latest Google APIs will have gRPC versions of their interfaces, letting you easily build Google functionality into your applications.

#### Service definition

Like many RPC systems, gRPC is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. By default, gRPC uses [protocol buffers](https://developers.google.com/protocol-buffers) as the Interface Definition Language (IDL) for describing both the service interface and the structure of the payload messages. It is possible to use other alternatives if desired.

```protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}
message HelloRequest {
  string greeting = 1;
}
message HelloResponse {
  string reply = 1;
}
```

gRPC lets you define four kinds of service method:

- Unary（一元） RPCs where the client sends a single request to the server and gets a single response back, just like a normal function call.

```protobuf
rpc SayHello(HelloRequest) returns (HelloResponse);
```

- Server streaming RPCs where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. gRPC guarantees message ordering within an individual RPC call.

```protobuf
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
```

- Client streaming RPCs where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the messages, it waits for the server to read them and return its response. Again gRPC guarantees message ordering within an individual RPC call.

```protobuf
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
```

- Bidirectional streaming RPCs where both sides send a sequence of messages using a read-write stream. **The two streams operate independently**, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved.

```protobuf
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
```

You’ll learn more about the different types of RPC in the [RPC life cycle](https://grpc.io/docs/guides/concepts/#rpc-life-cycle) section below.

#### Using the API

Starting from a service definition in a `.proto` file, gRPC provides protocol buffer compiler plugins that generate client-and server-side code. gRPC users typically call these APIs on the client side and implement the corresponding API on the server side.

- On the server side, the server implements the methods declared by the service and runs a gRPC server to handle client calls. The gRPC infrastructure decodes incoming requests, executes service methods, and encodes service responses.
- On the client side, the client has a local object known as *stub* (for some languages, the preferred term is *client*) that implements the same methods as the service. The client can then just call those methods on the local object, wrapping the parameters for the call in the appropriate protocol buffer message type - gRPC looks after sending the request(s) to the server and returning the server’s protocol buffer response(s).

#### Synchronous vs. asynchronous

Synchronous RPC calls that block until a response arrives from the server are the closest approximation（近似） to the abstraction of a procedure call that RPC aspires to. On the other hand, networks are inherently（固有的） asynchronous and in many scenarios it’s useful to be able to start RPCs without blocking the current thread.

The gRPC programming API in most languages comes in both synchronous and asynchronous flavors（风味）. You can find out more in each language’s tutorial and reference documentation (complete reference docs are coming soon).

#### Unary RPC

First consider the simplest type of RPC where the client sends a single request and gets back a single response.

1. Once the client calls a stub method, the server is notified that the RPC has been invoked with the client’s [metadata](https://grpc.io/docs/guides/concepts/#metadata) for this call, the method name, and the specified [deadline](https://grpc.io/docs/guides/concepts/#deadlines) if applicable.
2. The server can then either send back its own initial metadata (which must be sent before any response) straight away, or wait for the client’s request message. Which happens first, is application-specific.
3. Once the server has the client’s request message, it does whatever work is necessary to create and populate a response. The response is then returned (if successful) to the client together with status details (status code and optional status message) and optional trailing metadata.
4. If the response status is OK, then the client gets the response, which completes the call on the client side.

#### Server streaming RPC

A server-streaming RPC is similar to a unary RPC, except that the server returns a stream of messages in response to a client’s request. After sending all its messages, the server’s status details (status code and optional status message) and optional trailing metadata are sent to the client. This completes processing on the server side. The client completes once it has all the server’s messages.

#### Client streaming RPC

A client-streaming RPC is similar to a unary RPC, except that the client sends a stream of messages to the server instead of a single message. The server responds with a single message (along with its status details and optional trailing metadata), typically but not necessarily after it has received all the client’s messages.

#### Bidirectional streaming RPC

In a bidirectional streaming RPC, the call is initiated by the client invoking the method and the server receiving the client metadata, method name, and deadline. The server can choose to send back its initial metadata or wait for the client to start streaming messages.

Client- and server-side stream processing is application specific. Since the two streams are independent, the client and server can read and write messages in any order. For example, a server can wait until it has received all of a client’s messages before writing its messages, or the server and client can play “ping-pong” – the server gets a request, then sends back a response, then the client sends another request based on the response, and so on.

#### Deadlines/Timeouts

gRPC allows clients to specify how long they are willing to wait for an RPC to complete before the RPC is terminated with a `DEADLINE_EXCEEDED` error. On the server side, the server can query to see if a particular RPC has timed out, or how much time is left to complete the RPC.

Specifying a deadline or timeout is language specific: some language APIs work in terms of（就...而言） timeouts (durations of time), and some language APIs work in terms of a deadline (a fixed point in time) and may or maynot have a default deadline.

#### RPC termination

In gRPC, both the client and server make independent and local determinations of the success of the call, and their conclusions may not match. This means that, for example, you could have an RPC that finishes successfully on the server side (“I have sent all my responses!") but fails on the client side (“The responses arrived after my deadline!"). It’s also possible for a server to decide to complete before a client has sent all its requests.

#### Cancelling an RPC

Either the client or the server can cancel an RPC at any time. A cancellation terminates the RPC immediately so that no further work is done.

> Warning: Changes made before a cancellation are not rolled back.

#### Metadata

Metadata is information about a particular RPC call (such as [authentication details](https://grpc.io/docs/guides/auth/)) in the form of a list of key-value pairs, where the keys are strings and the values are typically strings, but can be binary data. Metadata is opaque（不透明） to gRPC itself - it lets the client provide information associated with the call to the server and vice versa.

Access to metadata is language dependent.

#### Channels

A gRPC channel provides a connection to a gRPC server on a specified host and port. It is used when creating a client stub. Clients can specify channel arguments to modify gRPC’s default behaviour, such as switching message compression on or off. A channel has state, including `connected` and `idle`.

How gRPC deals with closing a channel is language dependent. Some languages also permit querying channel state.



### 一元RPC

**proto文件**

```protobuf
syntax= "proto3";

package com.protobuf.proto;
option java_package="cn.isjinhao.grpctest.demo1";
option java_outer_classname="StudentInfo0";
// 将里面的类分成多个文件（就以前都是所有类在一个文件中，这里就分开了）
option java_multiple_files=true;
service StudentService0{
    rpc GetRealNameByUserName(MyRequest0) returns (MyResponse0){}
}
// 请求的对象
message MyRequest0 {
    string username = 1;
}
message MyResponse0 {
    string realName = 2;
}
```

**impl**

```java
public class StudentService0Impl extends StudentService0Grpc.StudentService0ImplBase {
    @Override
    public void getRealNameByUserName
        (MyRequest0 request, StreamObserver<MyResponse0> responseObserver) {
        System.out.println(request.getUsername());
        responseObserver.onNext(MyResponse0.newBuilder().setRealName("张三").build());
        responseObserver.onCompleted();
    }
}
```

**Server**

```java
public class GrpcServer1 {
    private Server server;
    public void start() throws Exception {
        server = ServerBuilder.forPort(8899).
            addService(new StudentService0Impl()).build().start();
        System.out.println("server started ... ");
        // JVM 回调钩子，在虚拟机关闭的时候执行这个方法
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("关闭 jvm");
            GrpcServer1.this.stop();
        }));
    }
    public void stop() {
        if (null != this.server) {
            this.server.shutdown();
        }
    }
    private void awaitTermination() throws InterruptedException {
        if (null != this.server) {
            this.server.awaitTermination();
        }
    }
    public static void main(String[] args) throws Exception {
        GrpcServer1 grpcServer1 = new GrpcServer1();
        grpcServer1.start();
        grpcServer1.awaitTermination();
    }
}
```

**Client**

```java
public class GrpcClient1 {
    public static void main(String[] args) {
        ManagedChannel managedChannel = 
            ManagedChannelBuilder.
            forAddress("localhost", 8899).usePlaintext().usePlaintext().build();
        StudentService0Grpc.StudentService0BlockingStub studentService0BlockingStub = 
            StudentService0Grpc.newBlockingStub(managedChannel);
        MyResponse0 myResponse0 = 
            studentService0BlockingStub.
            getRealNameByUserName(MyRequest0.newBuilder().setUsername("李四").build());
        System.out.println(myResponse0.getRealName());
    }
}
```



### 服务器流式RPC

**proto文件**

```protobuf
syntax= "proto3";

package com.protobuf.proto;
option java_package="cn.isjinhao.grpctest.demo2";
option java_outer_classname="StudentInfo";
option java_multiple_files=true;
service StudentService {
    rpc GetStudentsByAge(MyAge) returns (stream StudentResponse){}
}
message StudentResponse {
    string name = 1;
    int32 age = 2;
    string city = 3;
}
message MyAge{
    int32 age=1;
}
```

**impl**

```java
public class StudentServiceImpl extends StudentServiceGrpc.StudentServiceImplBase {
    @Override
    public void getStudentsByAge
        	(MyAge request, StreamObserver<StudentResponse> responseObserver) {
        System.out.println("接收到客户端消息： " + request.getAge());
        responseObserver.
            onNext(StudentResponse.newBuilder().setName("张三").setAge(20).build());
        responseObserver.
            onNext(StudentResponse.newBuilder().setName("李四").setAge(30).build());
        responseObserver.
            onNext(StudentResponse.newBuilder().setName("王五").setAge(40).build());
        responseObserver.
            onNext(StudentResponse.newBuilder().setName("赵六").setAge(50).build());
        responseObserver.onCompleted();
    }
}
```

**Client**

```java
public class GrpcClient2 {
    public static void main(String[] args) {
        ManagedChannel managedChannel = 
            ManagedChannelBuilder.
            forAddress("localhost", 8899).usePlaintext().usePlaintext().build();
        StudentServiceGrpc.StudentServiceBlockingStub studentServiceBlockingStub = 
            StudentServiceGrpc.newBlockingStub(managedChannel);
        Iterator<StudentResponse> studentsByAge = 
            studentServiceBlockingStub.
            getStudentsByAge(MyAge.newBuilder().setAge(20).build());
        while (studentsByAge.hasNext()) {
            StudentResponse next = studentsByAge.next();
            System.out.println(next.getName() + "   " + next.getAge());
        }
    }
}
```



### 客户端流式RPC

**proto文件**

```protobuf
syntax= "proto3";

package com.protobuf.proto;
option java_package="cn.isjinhao.grpctest.demo3";
option java_outer_classname="StudentInfo3";
option java_multiple_files=true;
service StudentService3 {
    rpc get_students_wrapper_by_ages(stream MyAge3) returns (StudentResponseList3){}
}
message MyAge3 {
    int32 age=1;
}
message StudentResponse3 {
    string name = 1;
    int32 age = 2;
    string city = 3;
}
message StudentResponseList3 {
    repeated StudentResponse3 studentResponse3 = 1;
}
```

**impl**

```protobuf
public class StudentService3Impl extends StudentService3Grpc.StudentService3ImplBase {
    @Override
    public StreamObserver<MyAge3> getStudentsWrapperByAges
    		(StreamObserver<StudentResponseList3> responseObserver) {
        return new StreamObserver<MyAge3>() {
            // stream里每来一个数据，这里就会被调用一次
            @Override
            public void onNext(MyAge3 myAge3) {
                System.out.println("one Next  " + myAge3.getAge());
            }
            @Override
            public void onError(Throwable throwable) {
                System.out.println(throwable.getMessage());
            }
            // stream全部完成后，这个方法会被调用
            @Override
            public void onCompleted() {
                StudentResponse3 studentResponse31 = 
                    StudentResponse3.newBuilder().
                    setName("张三").setAge(20).build();
                StudentResponse3 studentResponse32 = 
                    StudentResponse3.newBuilder().
                    setName("李四").setAge(30).build();
                StudentResponseList3 build = 
                    StudentResponseList3.newBuilder().
                    addStudentResponse3(studentResponse31).
                    addStudentResponse3(studentResponse32).build();
                responseObserver.onNext(build);
                responseObserver.onCompleted();
            }
        };
    }
}
```

**Client**

```
public class GrpcClient3 {
    public static void main(String[] args) throws InterruptedException {
        ManagedChannel managedChannel = 
            ManagedChannelBuilder.
            forAddress("localhost", 8899).usePlaintext().usePlaintext().build();
        // 客户端发送流式请求的时候，需要使用异步的方式调用
        StudentService3Grpc.StudentService3Stub stub = 
        	StudentService3Grpc.newStub(managedChannel);

        StreamObserver<StudentResponseList3> streamObserver = 
        	new StreamObserver<StudentResponseList3>() {
            @Override
            public void onNext(StudentResponseList3 studentResponseList3) {
                studentResponseList3.getStudentResponse3List().
                	forEach(studentResponse3 -> {
                        System.out.println(studentResponse3.getName() + " " + 
                            studentResponse3.getAge() + " " + studentResponse3.getCity());
                });
            }
            @Override
            public void onError(Throwable throwable) { }

            @Override
            public void onCompleted() {
                System.out.println("completed");
            }
        };

        StreamObserver<MyAge3> studentsWrapperByAges = 
        	stub.getStudentsWrapperByAges(streamObserver);
        studentsWrapperByAges.onNext(MyAge3.newBuilder().setAge(20).build());
        studentsWrapperByAges.onNext(MyAge3.newBuilder().setAge(30).build());
        studentsWrapperByAges.onNext(MyAge3.newBuilder().setAge(40).build());
        studentsWrapperByAges.onNext(MyAge3.newBuilder().setAge(50).build());
        studentsWrapperByAges.onCompleted();
        TimeUnit.SECONDS.sleep(5);
    }
}
```



### 双向流式RPC

**proto文件**

```protobuf
syntax= "proto3";

package com.protobuf.proto;

option java_package="cn.isjinhao.grpctest.demo4";
option java_outer_classname="StudentInfo4";
option java_multiple_files=true;
service StudentService4 {
    rpc BiTalk(stream StreamRequest4) returns (stream StreamResponse4) { }
}
message StreamRequest4 {
    string request_info = 1;
}
message StreamResponse4 {
    string response_info = 2;
}
```

**impl**

```java
public class StudentService4Impl extends StudentService4Grpc.StudentService4ImplBase {
    @Override
    public StreamObserver<StreamRequest4> biTalk(StreamObserver<StreamResponse4> responseObserver) {
        return new StreamObserver<StreamRequest4>() {
            @Override
            public void onNext(StreamRequest4 streamRequest4) {
                System.out.println(streamRequest4.getRequestInfo());                responseObserver.onNext(StreamResponse4.newBuilder().setResponseInfo(UUID.randomUUID().toString()).build());
            }
            @Override
            public void onError(Throwable throwable) {
                System.out.println(throwable.getMessage());
            }
            @Override
            public void onCompleted() {
                responseObserver.onCompleted();
            }
        };
    }
}
```

**Client**

```java
public class GrpcClient4 {
    public static void main(String[] args) throws InterruptedException {
        ManagedChannel managedChannel = ManagedChannelBuilder.
            forAddress("localhost", 8899).usePlaintext().usePlaintext().build();
        // 客户端发送流式请求的时候，需要使用异步的方式调用
        StudentService4Grpc.StudentService4Stub stub = 
            	StudentService4Grpc.newStub(managedChannel);
        StreamObserver<StreamResponse4> streamObserver = 
            	new StreamObserver<StreamResponse4>() {
            @Override
            public void onNext(StreamResponse4 streamResponse4) {
                System.out.println(streamResponse4.getResponseInfo());
            }
            @Override
            public void onError(Throwable throwable) {
                System.out.println(throwable.getMessage());
            }
            @Override
            public void onCompleted() {
                System.out.println("completed");
            }
        };
        StreamObserver<StreamRequest4> streamRequest4StreamObserver = 
            	stub.biTalk(streamObserver);

        for (int i = 0; i < 10; i++) {
            TimeUnit.SECONDS.sleep(1);
            StreamRequest4 streamRequest4 = StreamRequest4.newBuilder().setRequestInfo(LocalDateTime.now().toString()).build();
            streamRequest4StreamObserver.onNext(streamRequest4);
        }
        TimeUnit.SECONDS.sleep(5);
    }
}
```



## Dubbo集成grpc









