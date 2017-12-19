### Hessian Grammar

```
           # starting production
top        ::= value

           # 8-bit binary data split into 64k chunks
binary     ::= 'x41' b1 b0 <binary-data>      # non-final chunk
           ::= 'B' b1 b0 <binary-data>        # final chunk
           ::= [x20-x2f] <binary-data>        # binary data of
                                                 #  length 0-15
           ::= [x34-x37] <binary-data>        # binary data of
                                                 #  length 0-1023

           # boolean true/false
boolean    ::= 'T'
           ::= 'F'

           # definition for an object (compact map)
class-def  ::= 'C' string int string*

           # time in UTC encoded as 64-bit long milliseconds since
           #  epoch
date       ::= x4a b7 b6 b5 b4 b3 b2 b1 b0
           ::= x4b b3 b2 b1 b0       # minutes since epoch

           # 64-bit IEEE double
double     ::= 'D' b7 b6 b5 b4 b3 b2 b1 b0
           ::= x5b                   # 0.0
           ::= x5c                   # 1.0
           ::= x5d b0                # byte cast to double
                                     #  (-128.0 to 127.0)
           ::= x5e b1 b0             # short cast to double
           ::= x5f b3 b2 b1 b0       # 32-bit float cast to double

           # 32-bit signed integer
int        ::= 'I' b3 b2 b1 b0
           ::= [x80-xbf]             # -x10 to x3f
           ::= [xc0-xcf] b0          # -x800 to x7ff
           ::= [xd0-xd7] b1 b0       # -x40000 to x3ffff

           # list/vector
list       ::= x55 type value* 'Z'   # variable-length list
           ::= 'V' type int value*   # fixed-length list
           ::= x57 value* 'Z'        # variable-length untyped list
           ::= x58 int value*        # fixed-length untyped list
           ::= [x70-77] type value*  # fixed-length typed list
           ::= [x78-7f] value*       # fixed-length untyped list

           # 64-bit signed long integer
long       ::= 'L' b7 b6 b5 b4 b3 b2 b1 b0
           ::= [xd8-xef]             # -x08 to x0f
           ::= [xf0-xff] b0          # -x800 to x7ff
           ::= [x38-x3f] b1 b0       # -x40000 to x3ffff
           ::= x59 b3 b2 b1 b0       # 32-bit integer cast to long

           # map/object
map        ::= 'M' type (value value)* 'Z'  # key, value map pairs
           ::= 'H' (value value)* 'Z'       # untyped key, value

           # null value
null       ::= 'N'

           # Object instance
object     ::= 'O' int value*
           ::= [x60-x6f] value*

           # value reference (e.g. circular trees and graphs)
ref        ::= x51 int            # reference to nth map/list/object

           # UTF-8 encoded character string split into 64k chunks
string     ::= x52 b1 b0 <utf8-data>         # non-final chunk
           ::= 'S' b1 b0 <utf8-data>         # string of length
                                             #  0-65535
           ::= [x00-x1f] <utf8-data>         # string of length
                                             #  0-31
           ::= [x30-x34] b0 <utf8-data>         # string of length
                                             #  0-1023

           # map/list types for OO languages
type       ::= string                        # type name
           ::= int                           # type reference

           # main production
value      ::= null
           ::= binary
           ::= boolean
           ::= class-def value
           ::= date
           ::= double
           ::= int
           ::= list
           ::= long
           ::= map
           ::= object
           ::= ref
           ::= string
```

#### 1. binary data
```
binary     ::= 'x41' b1 b0 <binary-data>      # non-final chunk
           ::= 'B' b1 b0 <binary-data>        # final chunk
           ::= [x20-x2f] <binary-data>        # binary data of
                                                 #  length 0-15
           ::= [x34-x37] b0 <binary-data>        # binary data of
                                                 #  length 0-1023
```

##### 1.1 长度为0-15的编码为
`[x20-x2f] <binary-data>`
    长度 len = code - x20
##### 1.2 长度16-1023的编码为
`[x34-x37] b0 <binary-data>`  
    长度 len = (code - x34) << 8 + b0
##### 1.3 长度 >= 1024 的非结束binary块编码为
`'x41' b1 b0 <binary-data>`
    长度 len = b1 << 8 + b0
##### 1.4 长度 >= 1024 的结束binary块编码为
`'B' b1 b0 <binary-data>`
    长度 len = b1 << 8 + b0

#### 2. boolean
```
boolean ::= T       T(0x54) represents true
        ::= F       F(0x46) represents false
```

#### 3. int
```
int        ::= 'I' b3 b2 b1 b0
           ::= [x80-xbf]             # -x10 to x3f
           ::= [xc0-xcf] b0          # -x800 to x7ff
           ::= [xd0-xd7] b1 b0       # -x40000 to x3ffff
```

##### 3.1 -16~47 范围内的整数编码为
    `[x80-xbf]`
    value = code -0x90
##### 3.2 -2048~2047 范围内的整数编码为
    `[xc0-xcf] b0`
    value = ((code - 0xc8) << 8) + b0
##### 3.3 -262144~262143 范围内的整数编码为
    `[xd0-xd7] b1 b0`
    value = ((code - xd4) << 16) + (b1 << 8) + b0
##### 3.4 32-bit有符号整数编码为
    `'I' b3 b2 b1 b0`
    value = (b3 << 24) + (b2 << 16) + (b1 << 8) + b0;

#### 4. long
```
long       ::= 'L' b7 b6 b5 b4 b3 b2 b1 b0
           ::= [xd8-xef]             # -x08 to x0f
           ::= [xf0-xff] b0          # -x800 to x7ff
           ::= [x38-x3f] b1 b0       # -x40000 to x3ffff
           ::= x59 b3 b2 b1 b0       # 32-bit integer cast to long
```

##### 4.1 -8~15 之间的整数被编码为
`[xd8-xef]`
    value = code - 0xe0
##### 4.2 -2048~2047 之间的整数被编码为
`[xf0-xff] b0`
    value = ((code - 0xf8) << 8) + b0
##### 4.3 -262144~262143 之间的整数被编码为
`[x38-x3f] b1 b0`
    value = ((code - 0x3c) << 16) + (b1 << 8) + b0
##### -2147483648~2147483647 之间的32-bit整数被编码为
`x59 b3 b2 b1 b0 `
    value = (b3 << 24) + (b2 << 16) + (b1 << 8) + b0
##### 超出上述范围的长整数被编码为
`'L' b7 b6 b5 b4 b3 b2 b1 b0`
    value = ((b64 << 56) + (b56 << 48) + (b48 << 40) + (b40 << 32) + (b32 << 24) + (b24 << 16) + (b16 << 8) + b8)

#### 5. double
```
double     ::= 'D' b7 b6 b5 b4 b3 b2 b1 b0
           ::= x5b                   # 0.0
           ::= x5c                   # 1.0
           ::= x5d b0                # byte cast to double
                                     #  (-128.0 to 127.0)
           ::= x5e b1 b0             # short cast to double
           ::= x5f b3 b2 b1 b0       # 32-bit float cast to double
```

##### 5.1 0.0和1.1分别被编码成`0x5b`和`0x5c`
##### 5.2 -128.0~127.0 范围内没有小数部分的 被编码成
`x5d b0`
    value = (double) b0
##### 5.3 -32768.0~32767 范围内没有小数部分被编码为
`x5e b1 b0`
    value = (double) ((b1 << 8) + b0)
##### 5.4 -2147483.625~2147483.625且小数部分为m * 2 ^ (-3) 的被编码为
`x5f b3 b2 b1 b0`
    value = (doule) (((b3 << 24) + (b2 << 16) + (b1 << 8) + b1) / 1000)
##### 5.5 其余编码为
`'D' b7 b6 b5 b4 b3 b2 b1 b0`
64-bit IEEE 浮点数
value = Double.longBitsToDouble((b64 << 56) + (b56 << 48) + (b48 << 40) + (b40 << 32) + (b32 << 24) + (b24 << 16) + (b16 << 8) + b8)

#### 6. null
    `null       ::= 'N'`

#### 7. string
```
string     ::= 'R' b1 b0 <utf8-data>         # non-final chunk
           ::= 'S' b1 b0 <utf8-data>         # string of length
                                             #  0-65535
           ::= [x00-x1f] <utf8-data>         # string of length
                                             #  0-31
           ::= [x30-x34] b0 <utf8-data>         # string of length
                                             #  0-1023
```
examples:
```
x00                         # "", empty string
x05             hello       # "hello"
x01 xc3 x83                 # "\u00c3"

S x00 x05       hello       # "hello" in long form

x52 x00 x07     hello,      # "hello, world" split into two chunks
x05             world
```

##### 7.1 长度0~31
`[x00-x1f] <utf8-data>`
长度 len = code
##### 7.2 长度0~1023
`[x30-x34] b0 <utf8-data>`
长度 len = ((code - x30) << 8) + b0
##### 7.3 非结束块
`'R' b1 b0 <utf8-data>`
字符串长度，非字节长度 len = (b1 << 8) + b0
##### 7.4 结束块
`'S' b1 b0 <utf8-data>`
字符串长度，非字节长度 len = (b1 << 8) + b0

#### 8. date
```
date       ::= x4a b7 b6 b5 b4 b3 b2 b1 b0  #Date represented by a 64-bit long of milliseconds since Jan 1 1970 00:00H, UTC.
           ::= x4b b3 b2 b1 b0       # minutes since epoch
```
时间戳

#### 9. object
```
class-def  ::= 'C' string int string*

object     ::= 'O' int value*
           ::= [x60-x6f] value*
```

##### 9.1 压缩格式: class定义
Hessian 2.0制定了一个简洁的对象格式，其中字段名只需要序列化一次。对于对象而言，仅需要按次序地序列化这些字段的取值.
 
类定义包括类型字符串,字段个数,所有的字段名. 类定义被存放在一个对象定义表中,每个类定义由一个唯一整数作为key标识，之后被对象实例通过key来引用。

##### 9.2 压缩格式: object定义
对象实例基于先前定义来创建一个新的对象实例. 类定义通过一个整型key在类定义表中进行查询。

examples:
```
class Car {
  String color;
  String model;
}

out.writeObject(new Car("red", "corvette"));
out.writeObject(new Car("green", "civic"));

---

C                        # object definition (#0)
  x0b example.Car        # type is example.Car
  x92                    # two fields
  x05 color              # color field name
  x05 model              # model field name

O                        # object def (long form)
  x90                    # object definition #0
                         #PS: 实际上这里 O 和 x90 会被编码为x60, 因为这里的类定义数量为1 < 16，当流中的类定义数量 > 15时，将会采用 'O' b0 形式，类定义索引key = b0 - x90
  x03 red                # color field value
  x08 corvette           # model field value

x60                      # object def #0 (short form)
  x05 green              # color field value
  x05 civic              # model field value
```
```
enum Color {
  RED,
  GREEN,
  BLUE,
}

out.writeObject(Color.RED);
out.writeObject(Color.GREEN);
out.writeObject(Color.BLUE);
out.writeObject(Color.GREEN);

---

C                         # class definition #0
  x0b example.Color       # type is example.Color
  x91                     # one field
  x04 name                # enumeration field is "name"

x60                       # object #0 (class def #0)
  x03 RED                 # RED value

x60                       # object #1 (class def #0)
  x90                     # object definition ref #0
  x05 GREEN               # GREEN value

x60                       # object #2 (class def #0)
  x04 BLUE                # BLUE value

x51 x91                   # object ref #1, i.e. Color.GREEN
```

#### 10. list
```
list       ::= x55 type value* 'Z'   # variable-length list
           ::= 'V' type int value*   # fixed-length list
           ::= x57 value* 'Z'        # variable-length untyped list
           ::= x58 int value*        # fixed-length untyped list
           ::= [x70-77] type value*  # fixed-length typed list
           ::= [x78-7f] value*       # fixed-length untyped list
```

##### 10.1 untyped list
是ArrayList对象（不包含子类）或者没有实现Serializable接口的list会被归为untyped list，编码时不会携带list的类信息过去，统一当做ArrayList处理
还有一种情况，在使用了`CollectionSerializer.setSendJavaType(false)`后，所有经过该Serializer处理的list，写入流时都不会携带list的类信息

##### 10.2 长度小于8个的List被编码为
`[x70-77] type value*` 或 `[x78-7f] value*`
type为list的类名(String)或类名索引(int)

##### 10.3 长度大于等于8的集合被编码为
`'V' type int value*` 或 `x58 int value*`

##### 10.4 Iterator和Enumration被编码为
`x55 type value* 'Z'` 或 `x57 value* 'Z'`

##### 10.5 list中的域不会被序列化，只会序列化list中的元素

##### examples
    public class SubList<E> extends ArrayList<E> {
        public String name = "sublist";
    }
    List<Car> list = new SubList<>();
    list.add(new Car("red", "corvette"));
    
    ---
    
    x71
        x1c com.krino.test.model.SubList    #type of list
    
        x43                                 #object definition
            x18 com.krino.test.model.car    #type of element
        x92                                 #two fields
            x05 color                       #color field name
            x05 model                       #model field name   
    
        x60                                 #object instance
            x03 red                         #color field value
            x08 corvette                    #model field value

#### 11. map
```
map        ::= 'M' type (value value)* 'Z'  # key, value map pairs
           ::= 'H' (value value)* 'Z'       # untyped key, value
```

##### 11.1 untyped map
类似list，是HashMap对象（不包含子类）或者没有实现Serializable接口的map会被归为untyped map，编码时不会携带map的类信息过去，统一当做HashMap处理
还有一种情况，在使用了`MapSerializer.setSendJavaType(false)`后，所有经过该Serializer处理的map，写入流时都不会携带map的类信息

##### 11.2 map的域不会被序列化，只会序列化map中的key和value

##### examples
    Map map = new HashMap();
    map.put(new Integer(1), "fee");
    map.put(new Integer(16), "fie");
    map.put(new Integer(256), "foe");
    
    ---
    
    H           # untyped map (HashMap for Java)
      x91       # 1
      x03 fee   # "fee"
    
      xa0       # 16
      x03 fie   # "fie"
    
      xc9 x00   # 256
      x03 foe   # "foe"  3个对象的顺序不固定
    
      Z

#### 12. ref
```
ref ::= x51 int
```
以一个整数作为key,引用之前所定义的list, map或者对象实例. 对于从输入流中读取出来的每个list, map或对象, 都在流中按位置次序作为标识, i.e. 第一个list或者map为'0', 下一个为'1'等等. 之后的引用能够使用先前的对象. Writers MAY生成引用. Parsers MUST 识别引用.

ref 可以引用未读完的元素。例如，循环链表将引用整个列表被读取之前的第一个链接。

#### 13. Reference Maps
Hessian 2.0包含3个内建的引用表：
    1.一个map/object/list引用表.
    2.一个类型定义表(class definition map).
    3.一个type(class name)表
值引用表使得hessian支持arbitrary graphs，还有递归和循环数据结构。
class和type表通过消除重复字符串而提高了hessian的效率。

##### 13.1 值引用
Hessian通过在字节码流中遇到它们时记录列表，对象和映射来支持arbitrary graphs。
解析器MUST在引用表中存储每个list, 对象和map。
被存储的对象能够被引用。

##### 13.2 class引用
每个类型定义被自动添加到类型表(class-map)中。解析器必须在初次使用一个类型时把它添加到类型表中，之后对象实例将通过引用来追踪自身的类型。

##### 13.3 type引用
map和list的值类型字符串被存储在type map中。解析器必须在初次碰到一个type字符串时把它加入到type map中。

#### 14. Bytecode map
Hessian被制定成一个字节码协议。**Hessian解析器本质上是一段针对头字节的选择分支语句**.

    x00 - x1f    # utf-8 字符串， 长度 0-32
    x20 - x2f    # 二进制数据， 长度 0-16
    x30 - x33    # utf-8 字符串， 长度 0-1023
    x34 - x37    # 二进制数据， 长度 0-1023
    x38 - x3f    # 3字节压缩长整型 (-x40000 to x3ffff)
    x40          # 保留 (expansion/escape)
    x41          # 8-bit 二进制数据的非终止块  ('A')
    x42          # 8-bit 二进制数据的终止块 ('B')
    x43          # 对象类型定义 ('C')
    x44          # 64-bit IEEE 编码的 double ('D')
    x45          # 保留
    x46          # boolean false ('F')
    x47          # 保留
    x48          # untyped map ('H')
    x49          # 32-bit 有符号整数 ('I')
    x4a          # 64-bit UTC millisecond date
    x4b          # 32-bit UTC minute date
    x4c          # 64-bit 有符号长整型 ('L')
    x4d          # 带类型的map ('M')
    x4e          # null ('N')
    x4f          # 对象实例 ('O')
    x50          # 保留
    x51          # 指向 map/list/object的引用 - 整数 ('Q')
    x52          # utf-8 字符串的非终止块 ('R')
    x53          # utf-8 字符串的终止块 ('S')
    x54          # boolean true ('T')
    x55          # 变长list/vector ('U')
    x56          # 定长list/vector ('V')
    x57          # 变长 untyped list/vector ('W')
    x58          # 定长 untyped list/vector ('X')
    x59          # 编码为32位int的长整型 ('Y')
    x5a          # list/map 结束符 ('Z')
    x5b          # double 0.0
    x5c          # double 1.0
    x5d          # double represented as byte (-128.0 to 127.0)
    x5e          # double represented as short (-32768.0 to 327676.0)
    x5f          # double represented as float
    x60 - x6f    # object with direct type
    x70 - x77    # 定长list with direct length(长度小于8)
    x78 - x7f    # 定长untyped list with direct length(长度小于8)
    x80 - xbf    # 一字节压缩 int (-x10 to x3f, x90 is 0)
    xc0 - xcf    # 两字节压缩 int (-x800 to x7ff)
    xd0 - xd7    # 三字节压缩 int (-x40000 to x3ffff)
    xd8 - xef    # 一字节压缩 long (-x8 to xf, xe0 is 0)
    xf0 - xff    # 两字节压缩 long (-x800 to x7ff, xf8 is 0)