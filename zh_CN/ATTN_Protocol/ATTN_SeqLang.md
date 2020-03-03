# 信息序列化规范 - Sequence Language

Sequence Language（以下简称SL），旨在提供一种轻便高效的结构化数据存储格式。SL与平台无关，可应用于通信、数据存储等领域。


> 编码方式（TV-LC）

在双方达成一致的情况下，信息中的值名称是冗余的。编码首先用一个VarInt（[Variable-length quantity](https://en.wikipedia.org/wiki/Variable-length_quantity)，以下简称VarInt）代表值的个数（**T**otal count），随后是每个值（**V**alue）。每个值由两部分组成：长度（**L**ength） + 内容（**C**ontent）。长度L采用VarInt来表示；内容C均由值序列化为二进制编码。编码的最小单位为字节。

原始消息：

    Message {
      int     id = 1;       // Value_1
      string msg = "hello"; // Value_2
    }

一个包含两个值的SL示例：

    T  V1[L1-C1]          V2[L2-C2]       
    02    04 00000001     05 68656c6c6f
    
值的长度L可以为0，表示这个值为空（Null）；值应当严格按照描述文件约定的顺序进行排列。

> 描述文件

描述文件采用json格式，对特定SL编码的字段含义、值类型进行描述。

字段含义

| 名 | 值 | 是否必须 |
|  ---- | ---- | ---- |
| name  | 名称 | Y |
| type  | 类型 | Y |
| desc  | 描述 | N |

示例：

    [
        {
            "name":"id",
            "type":"number",
            "desc":"int"
        },
        {
            "name":"msg",
            "type":"string"
        }
    ]

嵌套类型
    
    [
        {
            "name":"id",
            "type":"number"
        },
        {
            "name":"msg",
            "type":[
                {
                    "name":"msg_id",
                    "type":"number"
                },
                {
                    "name":"msg_content",
                    "type":"string"
                }
            ]
        }
    ]

复杂类型的子类型编码也应遵循 TV-LC 规范

> 各种类型的详细编码规范

| 类型 | 描述 |
|  ---- | ---- |
| raw | 二进制数据 |
| number | 有符号整数类型（[Big-endian](https://en.wikipedia.org/wiki/Endianness#Big-endian)），根据值长度L来确定具体是何种类型(如L=4字节代表int类型) |
| decimal | 有符号有理数，用一个VarInt来表示指数位，其余部分表示尾数（Big-endian） |
| string | 默认采用 [UTF-8](https://en.wikipedia.org/wiki/UTF-8) 编码的字符串 |


> decimal 类型编码规范说明







