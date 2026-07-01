---
created: 2026-07-01T16:23:55+08:00
---

# Protocol Buffers

- **官方文档：** https://protobuf.dev/。

## 一、Protocol Buffers 是什么

- **核心定位：** Protocol Buffers 是 Google 开发的一整套数据交换方案，不只是一个数据交换格式，而是从“定义数据”到“生成代码”再到“传输数据”的完整体系。直观理解，`.proto` 是数据协议说明书，生成代码是各语言使用这份说明书的工具，二进制格式是数据真正落盘或传输时的样子。

- **组成部分：** Protocol Buffers 主要包括四类东西。

    - **`.proto` 定义语言：** 用来描述数据长什么样，包括字段名、字段类型和字段编号。

    - **`protoc` 编译器：** 根据 `.proto` 文件生成不同语言的代码。

    - **语言运行时库：** 让生成出来的代码能在 Java、Go、C++ 等语言中正常序列化和解析数据。

    - **二进制序列化格式：** 写入文件或通过网络传输时使用的紧凑格式，主要给程序高效读取，不是给人直接阅读。

## 二、Protocol Buffers 解决什么问题

- **API 结构约定：** `.proto` 文件负责定义结构化数据，相当于双方提前约定好的数据格式，便于对接。

- **高效序列化：** Protocol Buffers 可以把对象序列化成更紧凑的二进制数据，适合文件或网络里的高频读写。

- **跨语言传递：** 只要不同语言使用同一份 `.proto` 定义，并分别生成自己的代码，就可以互相读写同一种数据。

    - Protocol Buffers 支持为 C++、C#、Dart、Go、Java、Kotlin、Objective-C、Python、Rust、Ruby、PHP 等语言生成代码。

    ```text
    Java 程序
      ↓ 写入 Protobuf 二进制数据
    文件 / 网络
      ↓ 读取 Protobuf 二进制数据
    C++ 程序
    ```

## 三、Protocol Buffers 的使用流程

1. **定义数据结构：** 先在 `.proto` 文件中定义 `message`，可以把 `message` 理解成一种跨语言的数据结构（结构体）。

    ```proto
    message Person {
      string name = 1;
      int32 id = 2;
      string email = 3;
    }
    ```

2. **生成语言代码：** 使用 `protoc` 根据 `.proto` 文件生成 Java、Go、C++ 等语言对应的类型和读写代码。

3. **业务代码使用：** 在业务代码里直接使用生成出来的类型，不需要自己手写二进制解析逻辑。

    ```go
    p := &pb.Person{
        Name:  "John Doe",
        Id:    1234,
        Email: "jdoe@example.com",
    }
    ```

4. **序列化和解析：** 写入文件或网络前，把对象序列化成 Protobuf 二进制数据；读取时，再按照同一份 `.proto` 定义把二进制数据解析回对象。

## 四、Protocol Buffers 的字段编号为什么重要

- **字段编号：** `string name = 1` 里的 `1` 不是默认值，而是字段编号。Protobuf 序列化时主要靠字段编号识别数据，不是靠字段名，所以字段编号也是协议兼容演进的关键。

    - 如果把 `string name = 1` 改成 `string username = 1`，只要编号仍然是 `1`，底层仍会把它当成同一个字段。

    - 后续在 `.proto` 中新增字段时，只要遵守字段编号兼容规则，旧程序和新程序就更容易在同一套协议下演进。

- **类比理解：** 可以把 `.proto` 理解成一张表格模板，字段编号就是这张模板里的固定列号。

    ```text
    Person:
    - 1 号字段：name
    - 2 号字段：id
    - 3 号字段：email
    ```

    ```text
    name = John Doe
    id = 1234
    email = jdoe@example.com
    ```

    - Java 往模板里填数据并保存成二进制文件后，C++ 读取这个文件时，也按照同一张模板理解 `1`、`2`、`3` 号字段，所以能正确读出来。

## 五、Protobuf 和 Kratos 的关系

- **Kratos 中的角色：** Kratos 用 `.proto` 来描述接口协议，包括服务方法、请求参数和响应参数。写完 `.proto` 后，再用工具生成 Go 代码，业务代码就能直接使用生成出来的类型，不需要手写请求结构体、响应结构体和底层解析逻辑。

## 六、Protobuf vs JSON

- **数据形态：** JSON 是人类容易阅读的文本格式（可以直接看出字段名和字段值），Protobuf 写入文件或网络时是二进制格式（需要结合 `.proto` 定义才能正确解析）。

- **Protobuf 优势：** Protobuf 的体积更小，读写速度通常更快。
