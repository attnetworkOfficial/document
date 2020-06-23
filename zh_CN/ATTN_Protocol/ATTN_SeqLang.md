# 信息序列化规范 - Sequence Language

Sequence Language（以下简称SL），旨在提供一种轻便高效的结构化数据存储格式。SL与平台无关，可应用于通信、数据存储等领域。

## 编码方式

编码最小单位为字节，所有数据均按照以下方式编码 

| 名称 | 编码方式 | 描述 |
| ---- | ---- | ---- |
| **L**ength  | VarInt | 二进制数据长度，采用可变长编码：[Variable-length quantity](https://en.wikipedia.org/wiki/Variable-length_quantity) |
| **V**alue  | Byte Array | 二进制数据 |

> 数据类型

| 类型 | 描述 | json | 示例值 | 示例编码
|  ---- | ---- | ---- | ---- | ---- |
| raw | 二进制数据 | { "type":"raw" } | | |
| number | 有符号整数类型（[Big-endian](https://en.wikipedia.org/wiki/Endianness#Big-endian)）<br>第一bit为符号位。如果正数的第一bit是1，则在最左补一个空白字节 | { "type":"number" } | 128<br>-128 | 0x0080<br>0x80 |
| decimal | 有符号有理数：a * 10^b，len + a + b<br>len 采用VarInt编码 a、b按照 number 类型进行编码<br> len 表示 a 的数据长度，b 的数据长度可以由 decimal 总长度减去 len 算出 | { "type":"decimal" } | | |
| string | 将二进制数据作为字符串解析，默认采用 [UTF-8](https://en.wikipedia.org/wiki/UTF-8) 编码 |  { "type":"string" } | | |
| array | 数组，由一种特定类型组成，元素个数可变 | {<br>　"type":"array",<br>　"element" : { "type" : ??? }<br>} | | |
| dict | key-value 数据结构<br>key只支持string、number类型<br>value 可以为任意特定类型 | {<br>　"type":"dict",<br>　"key" : { "type" : "string" } <br>　"value" : { "type" : ??? }<br>} | | |
| object | 对象，可以由多种类型组成。每个对象的结构由其描述文件表示 | {<br>　"type":"object",<br>　"fields" : {<br>　　"field0" : { "type" : ??? },<br>　　"field1" : { "type" : ??? }<br>　}<br>} | | |


## 描述文件

值严格按照描述文件约定的顺序进行排列。
描述文件采用json格式，对特定SL编码的数据类型及结构进行描述。

> 一个用户信息数据结构示例：

    {
      "type"  : "object",
      // object 用 fields 字段记录各个值的类型
      "fields" : {
        "id"          : { "type":"number"  },
        "first_name"  : { "type":"string"  },
        "last_name"   : { "type":"string"  },
        "score"       : { "type":"decimal" },
        "phone"       : { "type":"string"  },
        "contacts" : { 
          "type" : "array",
          // array 类型用 element 字段记录元素类型
          "element" : {
            "type"  : "object",
            "fields" : {
              "id"     : { "type":"number" },
              "remark" : { "type":"string" }
            }
          }
        }
      }
    }

> 演示如何将具体的消息内容转换为字节码

原始消息内容：

    {
      "id"         : 11099822739479112,
      "first_name" : "John",
      "last_name"  : "Smith",
      "score"      : 128.32,
      "phone"      : "400-222-5555",
      "contacts"   : [
        { "id" : 39817873987985719, "remark" : "boss" },
        { "id" : 45405687374639045 }
      ]
    }

根据数据结构规范转换为字节码：

    00        05        10        15        20        25        30        35        40        45        50        55        60      64 < index
    4007276f3adf74da48044a6f686e05536d69746804023220020c3430302d3232322d353535351a0e08008d76253ac3e13704626f73730a0800a1503b6abf6fc500 < bytes
    40|message                                                                                                                       | < details
      07|id          |04|firs..|05|last_n..|04|score |0c|phone                 |1a|contacts                    |  |                  |
        276f3adf74da48  4a6f686e  536d697468  02|a | |  3430302d3232322d35353535  0e|contact-0                 |0a|contact-1         |
                                                3220 b                              08|id             04|remark|  08|id            |00
                                                    02                                008d76253ac3e137  626f7373    00a1503b6abf6fc5

字节码解析：

| 字节码 | 描述 | 值 |
| ---- | ---- | ---- |
| 40 | 数据总长度 | 64 |
| 07 | id 数据长度 | 7 |
| 276f3adf74da48 | id | 11099822739479112 |
| 04 | first_name 数据长度 | 4 |
| 4a6f686e | first_name | John |
| 05 | last_name 数据长度 | 5 |
| 536d697468 | last_name | Smith |
| 04 | score 数据长度 | 5 |
| 02 | score a长度 | 2 |
| 3220 | score a值 | 12832 |
| 02 | score b值（128.32 = 12832 * 10^-2） | 2 |
| 0c | phone 数据长度 | 12 |
| 3430302d3232322d35353535 | phone | 400-222-5555 |
| 1a | contacts 数据长度 | 26 |
| 0e | contact-0 数据长度 | 14 |
| 08 | contact-0 id 数据长度 | 8 |
| 008d76253ac3e137 | contact-0 id | 39817873987985719 |
| 04 | contact-0 remark 数据长度 | 4 |
| 626f7373 | contact-0 remark | boss |
| 0a | contact-1 数据长度 | 10 |
| 08 | contact-1 id 数据长度 | 8 |
| 00a1503b6abf6fc5 | contact-1 id | 45405687374639045 |
| 00 | contact-0 remark 数据长度（长度为零代表此值为null）| 0 |
