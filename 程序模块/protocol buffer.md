# Protocol Buffers 

对于移动端，为了节约电量和流量，使用高度压缩的通信协议，Protocol Buffers 是一种轻便高效的结构化数据存储格式，适合做数据存储或 RPC 数据交换格式

可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式

## 安装

[52im/protobuf: Protocol Buffers - Google's data interchange format (github.com)](https://github.com/52im/protobuf)

```shell
tar -xzf protobuf-2.1.0.tar.gz
cd protobuf-2.1.0
./configure --prefix=$INSTALL_DIR
make
make check
make install
```



## 功能demo

结构化数据称为message

**编写文件**

```protobuf
# addressbook.proto
pakcage lm;
message helloworld{      //结构化的数据
	required int32 id = 1;  //基本数据
	required string str = 2;  //基本数据
	optional int32 opt = 3;  //可选成员
}
```

**编译**

```shell
protoc -I=$SRC_DIR --cpp_out=$DST_DIR/addressbook.proto
# 将生成两个文件
# lm.helloworld.pb.h ， 定义了 C++ 类的头文件
# lm.helloworld.pb.cc ， C++ 类的实现文件

```

**用读写应用来使用它**

写

```cpp
#include "lm.helloworld.pb.h"
…
 
 int main(void)
 {
   
  lm::helloworld msg1;
  msg1.set_id(101);
  msg1.set_str(“hello”);
     
  // Write the new address book back to disk.
  fstream output("./log", ios::out | ios::trunc | ios::binary);
         
  if (!msg1.SerializeToOstream(&output)) {
      cerr << "Failed to write msg." << endl;
      return -1;
  }        
  return 0;
 }
```

读

```cpp
#include "lm.helloworld.pb.h"
…
 void ListMsg(const lm::helloworld & msg) {
  cout << msg.id() << endl;
  cout << msg.str() << endl;
 }
  
 int main(int argc, char* argv[]) {
 
  lm::helloworld msg1;
  
  {
    fstream input("./log", ios::in | ios::binary);
    if (!msg1.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }
  
  ListMsg(msg1);
  …
 }
```

## 比起其他技术

protobuf更为快、简单，

添加新的消息中的field不会引发已发布程序的改变

语义清晰，无序解析器

比起XML，它功能简单，无法表示复杂的概念

XML可以直接读取编辑，protebuf是二进制存储，需要有.proto定义

## 实现原理

编码：Varint表示数字，每个byte的最高位为1表示后续的byte也是该数字的一部分，为0则表示不是，这个特性决定了对于小数字会很高效，但对大数字每个byte都有一位浪费了

解包：将二进制序列按照指定格式读到C++对应的结构类型

> xml的解包过程：从文件中读取字符串，转换为XML文档对象结构模型，再从XML文档对象结构模型中读取指定节点的字符串，再将字符串转换为指定类型的变量，耗用较多CPU