## UAVCAN Specification 3.Data structure description language

## Contents
### Data structure description language
- File hierarchy
	- Service data structure
 - Syntax
	 - Attribute definition
	 - Directives
	 - Comments
	 - Service response marker
 - Primitive data types 
 - Naming rules
	 - Mandatory
	 - Optional
	 - Advisory
 - Data type compatibility
	 - The concept of data type compatibility
	 - Signature
 - Data type ID
 - Data serialization
	 - Serialized data format
	 - Byte order and bit order
 - Standard data types
 - Vendor-specific data types

## Data structure description language（ 数据结构描述语言）

The Data Structure Description Language (DSDL) is used to define data structures for exchange via the CAN bus. The DSDL definitions are used to automatically generate the message serialization/deserialization code for a certain programming language. The tool that generates source code from DSDL definition files is called the DSDL compiler.  
数据结构描述语言（DSDL）用于定义通过 CAN 总线进行交换的数据结构。DSDL 定义用于为某种编程语言自动生成消息序列化/反序列化代码。把 DSDL 定义文件转换生成源代码的工具称为 DSDL 编译器。

### File hierarchy (文件层次结构)
Each DSDL definition file specifies exactly one data structure that can be used for message broadcasting or a pair of structures that can be used for service invocation data exchange.  
每个 DSDL 定义文件精确地指定一个数据结构，该结构可用于消息广播，或者可用于服务调用数据交换的一对结构。

The DSDL source file must be named using the data type name and default data type ID (if needed) as shown below:  
DSDL 源文件必须使用数据类型名称和默认数据类型 ID（如果需要）进行命名，如下所示：

```
[default data type ID.]<data type name>.uavcan
```

A defined data structure must be contained in a namespace, which may in turn be nested within another namespace. A namespace that is not nested in another namespace is called a root namespace. For example, all standard data types are contained in the root namespace uavcan, which contains nested namespaces: equipment, protocol, etc.  
已定义的数据结构必须包含在命名空间中，而命名空间又可以嵌套在另一个命名空间中。没有嵌套在其他命名空间中的命名空间称为根命名空间。例如，所有标准数据类型都包含在根命名空间 uavcan 中，其中包含嵌套的命名空间：equipment、protocol 等。

The namespace hierarchy is mapped directly to the file-system directory structure, as shown in the example below:  
命名空间层次结构直接映射到文件系统目录结构，如下例所示：
```
+ uavcan                        <-- Root namespace 
    + equipment                 <-- Nested namespace 
        + ...
    + protocol                  <-- Nested namespace 
        + 341.NodeStatus.uavcan <-- Definition of data type 
							        "uavcan.protocol.NodeStatus" 
							        with default data type ID 341
        + ...
    + Timestamp.uavcan          <-- Definition of data type 
								    "uavcan.Timestamp", 
								    default data type ID is not assigned
```

Notes:  
备注：
 - It is not necessary to explicitly define a default data type ID for non-standard data types (i.e., for vendor-specific or application-specific data types).  
 - 没必要为非标准数据类型明确定义一个默认数据类型 ID （比如，适用于特定于供应商或特定于应用程序的数据类型）。  
	- If the default data type ID is not defined by the DSDL definition, it will need to be assigned by the application at run time.  
	- 如果 DSDL 没有定义默认数据类型ID，则需要在运行时由应用程序分配它。
	- All standard data types have default data type ID values defined.  
	- 所有标准数据类型都定义了默认数据类型ID值。  
 - Data type names are case sensitive, i.e., names foo.Bar and foo.bar are considered different. Names that differ only in case should be avoided, because it may cause problems on file systems that are not case-sensitive.  
 - 数据类型名称区分大小写，比如 foo.Bar 和 foo.bar 被认为是不同的。应该避免仅在大小写情况下不同的名称，因为它可能会在不区分大小写的文件系统上造成问题。
 - Data types may contain nested data structures.  
 - 数据类型可能包含嵌套的数据结构。
 	- Some data structures may be designed for such nesting only, in which case they are not required to have a dedicated data type ID.  
	- 有些数据结构可能只设计用于这种嵌套，在这种情况下，它们不需要专用的数据类型ID。  
 - Full data type name is a unique identifier of a data type constructed from the root namespace, all nested namespaces (if any), and the data type name itself, joined via the dot symbol (.), e.g., uavcan.protocol.file.Read.  
 - 完整数据类型名称是数据类型的唯一标识符，该数据类型由根命名空间、所有嵌套命名空间（如果有的话）和数据类型名称本身构成，通过点符号（.）连接，例如，uavcan.protocol.file.Read。  
	 - The length of the Full data type name must not exceed 80 characters.  
	 - 完整数据类型名称的长度不能超过80个字符。  
	 - Refer to the naming rules below for the limitations imposed on the character set.  
	 - 有关字符集的限制，请参阅下面的命名规则。  
	 
#### Service data structure (服务数据结构)
Since a service invocation consists of two network exchange operations, the DSDL definition for a service must define two structures:  
由于服务调用包含两个网络交换操作，服务的 DSDL 定义必须定义两个结构:

 - Request part - for request transfer (client to server).  
 - 请求部分 —— 用于请求传输(客户端到服务器)。
 <br/>
 - Response part - for response transfer (server to client).  
 - 响应部分 —— 用于响应传输(服务器到客户端)。

Both request and response structures are contained within the same DSDL definition file, separated by a special statement (see the syntax chapter below).  
请求和响应结构都包含在同一个 DSDL 定义文件中，由一条特殊语句分隔（参见下面的语法章节）。

Service invocation data structures cannot be nested.  
服务调用数据结构不能嵌套。

### Syntax （语法）

A data structure definition consists of attributes and directives. Any line of the definition file may contain at most one attribute definition or at most one directive. The same line cannot contain an attribute definition and a directive at the same time.  
数据结构的定义由属性和指令组成。定义文件的任何行最多只能包含一个属性定义或指令。同一行不能同时包含属性定义或指令。

Lines should be separated with the ASCII line feed character (\n, code 10); vendor-specific data type definitions are allowed to use any other line ending convention.  
行之间用ASCII码（\n，code 10）分隔；厂家自己定义的特殊数据类型可以使用任何其他行结束约定。

An attribute can be either of the following:  
属性定义可以是一下中的任何一种：

 - Field - a variable that can be modified by the application and exchanged via the network.  
 - 字段 —— 可以由应用程序修改并通过网络交换的变量。
 <br/>
 - Constant - an immutable value that does not participate in network exchange.   
 - 常量 —— 不参与网络交换的固定值。

A directive is a statement that provides instructions to the DSDL compiler.  
指令是向 DSDL 编译器提供描述的语句。

Aside from attributes and directives, a DSDL definition may contain the following entities:  
除了属性和指令外，DSDL 定义还可以包含以下实体：

- Comments  
- 注释
<br/>
- Service response marker  
- 服务响应标志

The details are explained below.  
具体情况说明如下。

A DSDL definition for a message data type may contain only the following:  
DSDL定义的消息数据类型只能包含以下内容:

 - Attribute definitions (zero or more)  
 - 属性定义（0个或更多）
 <br/>
 - Directives (zero or more)  
 - 指令（0个或更多）
 <br/>
 - Comments (optional)  
 - 注释（可选）

A DSDL definition for a service data type may contain only the following:  
DSDL定义的服务数据类型只能包含以下内容:

 - Request part attribute definitions (zero or more)  
-请求阶段的属性定义（0个或更多）
<br/>
 - Response part attribute definitions (zero or more)  
 - 响应阶段的属性定义（0个或更多）
<br/>
 - Directives (zero or more)  
 - 指令（0个或更多）
 <br/>
 - Comments (optional)  
 - 注释（可选）
 <br/>
 - Service response marker (exactly one)  
 - 服务响应标记（只有一个）

#### Attribute definition （属性定义）
Field definition patterns:  
字段定义模式：

1. cast_mode field_type field_name
2. cast_mode field_type[X] field_name
3. cast_mode field_type[<X] field_name
4. cast_mode field_type[<=X] field_name
5. void_type

Constant definition pattern:  
常数定义模式：

1. cast_mode constant_type constant_name = constant_initializer

Each component is discussed below.  
下面将讨论每个组件。

#### Field type （字段类型）
The field type can be either a primitive data type (primitive data types are defined below) or a nested data structure.  
字段类型可以是原始数据类型（下面定义了原始数据类型），也可以是嵌套数据结构。

A primitive data type can be referred simply by name, e.g., float16, bool.  
原始数据类型可以简单的通过名称使用，比如：float16，bool。

A nested data structure can be referred by either of these two:  
一个嵌套的数据结构可以通过以下两种方式之一进行引用:

 - Short name, e.g., NodeStatus, if both the referred and the referring data types are located in the same namespace. For example, it is possible to access ns1.ns2.Type1 from ns1.ns2.Type2 using a short name, but not from ns1.ns2.ns3.Type3 or ns1.ns4.Type4.  
 - 短名称引用，所引用的数据类型位于同一命名空间时可以使用，例如 NodeStatus。可以在 ns1.ns2.Type2 时访问 ns1.ns2.Type1，但不能访问 ns1.ns2.ns3.Type3 或者 ns1.ns4.Type4。
 
 - Full name, e.g., uavcan.protocol.NodeStatus. A full name allows access to the data type from any namespace.  
 - 全称，例如，uavcan.protocol.NodeStatus。全名允许从任何命名空间访问数据类型。

A field type name can be appended with a statement in square brackets to define an array:  
字段类型名可以用方括号中的语句来定义数组：

 - Syntax [X] is used to define a static array of size exactly X items.  
 - 语法 [X] 用于定义大小正好为 X 的静态数组。
<br/>
 - Syntax [<X] is used to define a dynamic array of size from 0 to X-1 items, inclusively.  
 - 语法 [<X] 用于定义一个从 0 到 X-1 项的动态数组。
<br/>
 - Syntax [<=X] is used to define a dynamic array of size from 0 to X items, inclusively.  
 - 语法 [<=X] 用于定义一个从 0 到 X 项大小的动态数组。

In the array definition statements above, X must be a valid integer literal according to the rules defined in the section dedicated to constant definitions.  
在上面的数组定义语句中，X 必须是一个有效的整数，这是根据专门用于常量定义的部分中定义的规则确定的。

Arrays of maximum size with less than one item are not allowed. Multidimensional arrays are not allowed.  
不允许使用小于 1 项的数组。不允许多维数组。

#### Field name and constant name （字段名和常量名）
For a message data type, all attributes must have a unique name within the data type.  
对于消息数据类型，所有属性在数据类型中必须有唯一的名称。

For a service data type, all attributes must have a unique name within the same part (request/response) of the data type. In other words, service-type attributes can have the same name as long as they are separated by the service response marker.  
对于服务数据类型，所有属性必须在数据类型的相同部分（请求/响应）中具有唯一的名称。换句话说，只要服务响应标记将服务类型属性分隔开，它们就可以具有相同的名称。

#### Cast mode （强转模式）
Cast mode defines the rules of conversion from the native value of a certain programming language to the serialized field value. Cast mode may be left undefined, in which case the default will be used. Possible cast modes are defined below.  
强制转换模式定义了从某种编程语言的原生值到序列化字段值的转换规则。强制转换模式可以处于未定义状态，在这种情况下将使用默认模式。可能的转换模式定义如下。

- __saturated__ - This is the default cast mode, which will be used if the attribute definition does not specify the cast mode explicitly. For integers, it prevents an integer overflow - for example, attempting to write 0x44 to a 4-bit field will result in a bitfield value of 0x0F. For floating point values, it prevents overflow when casting to a lower precision floating point representation - for example, 65536.0 will be converted to a float16 as 65504.0; infinity will be preserved.  
- __饱和处理__ —— 这是默认的转换模式，如果属性定义没有显式地指定转换模式，就会使用这种模式。对于整数，它可以防止整数溢出——例如，尝试将0x44写入4bit的字段将得到0x0F的位字段值。对于浮点值，它可以防止在将其转换为精度较低的浮点表示形式时发生溢出 —— 例如，将65536.0转换为 float16 (如65504.0) ; 不会产生无穷大。  

<br/>
- __truncated__ - For integers, it discards the excess most significant bits - for example, attempting to write 0x44 to a 4-bit field will produce 0x04. For floating point values, overflow during downcasting will produce an infinity.  
- __截断__ —— 对于整数它有可能丢弃重要的符号位，如将 0x44 写入 4bit的字段会得到0x04。对于浮点数，转换时有可能得到正无穷。  

#### Constant definition (常量定义)
A constant must be a primitive scalar type (i.e., arrays and nested data structures are not allowed as constant types).  
常量必须是原始标量数值类型(即，数组和嵌套数据结构不允许作为常量类型)。

A constant must be assigned with a constant initializer, which must be one of the following:  
常量必须使用常量初始化器赋值，该初始化器必须是下列之一：

 - Integer zero (0).  
 - 整数零（0）。
 <br/>
 - Integer literal in base 10, starting with a non-zero character. E.g., 123, -12.  
 - 十进制整数，非零字符开头。如：123，-12。
 <br/>
 - Integer literal in base 16 prefixed with 0x. E.g., 0x123, -0x12, +0x123.  
 - 十六进制整数，以0x开头，如：0x123，-0x12，+0x123等。
 <br/>
 - Integer literal in base 2 prefixed with 0b. E.g., 0b1101, -0b101101, +0b101101.  
 - 二进制整数，以0b开头。如：0b1101，-0b101101，+0b101101。
 <br/>
 - Integer literal in base 8 prefixed with 0o. E.g., 0o123, -0o777, +0o777.  
 - 八进制整数，以0o开头，比如：0o123，-0o777，+0o777。
 <br/>
 - Floating point literal. Fractional part with an optional exponent part, e.g., 15.75, 1.575E1, 1575e-2, -2.5e-3, +25E-4. Not-a-number (NaN), positive infinity, and negative infinity are intentionally not supported in order to maximize cross-platform compatibility.  
 - 浮点数。指数部分可选的小数部分，为了跨平台不兼容支持NAN、正无穷和负无穷。
 <br/>
 - Boolean true or false.  
 - 布尔类型。true或者false。
 <br/>
 - Single ASCII character, ASCII escape sequence, or ASCII hex literal in single quotes. E.g., 'a', '\x61', '\n'.  
 - 单ASCII字符、ASCII转义序列或单引号中的ASCII十六进制文字。例如，'a'， '\x61'， '\n'。

The DSDL compiler must implicitly convert the type of an initializer expression to the constant type if the target type can allocate the value with no data loss. If a data loss occurs (e.g., integer overflow, floating point number decays to infinity), the DSDL compiler should refuse to compile such data type.  
如果目标类型可以在不丢失数据的情况下分配值，则 DSDL 编译器必须隐式地将初始化器表达式的类型转换为常量类型。如果发生数据丢失（例如，整数溢出、浮点数衰减到无穷大），DSDL 编译器应该拒绝编译这样的数据类型。

Note that constants do not affect the serialized data layout as they are never exchanged via the network.  
注意，常量不会影响序列化数据布局，因为它们从不通过网络交换。

#### Void type （空数据类型）
Void type is a special field type that is intended for data alignment purposes. The specification defines 64 distinct void types as follows:  
Void 类型是一种特殊的字段类型，用于数据对齐。该规范定义了 64 种不同的 void 类型，如下：

 - void1 - 1 padding bit;  
 - void2 - 2 padding bits;
 - …
 - void63 - 63 padding bits;
 - void64 - 64 padding bits.

A field of type void does not have a name and its cast mode cannot be specified. During message serialization, all void fields must be populated with zero bits; during deserialization, contents of the void fields should be ignored.  
类型为 void 的字段没有名称，无法指定其强制转换模式。在消息序列化过程中，所有的空字段必须用 0 bits 填充；在反序列化期间，应该忽略 void 字段的内容。

#### Directives （指令）
A directive is a single case-sensitive word starting with an “at sign” (@), possibly followed by space-separated arguments:  
指令是一个大小写敏感的单词，以“at sign”(@)开头，后面可能跟以空格分隔的参数：

 - @directive
 - @directive arg1 arg2

All valid directives are documented below.  
所有有效的指令都记录在下面。

__Union__
Keyword:  @union.

This directive instructs the DSDL compiler that the current message or the current part of a service data type (request or response) is a tagged union. A tagged union is a data structure that may encode either of its fields at a time. Such a data structure contains one implicit field, a union tag that indicates what particular field the data structure is holding at the moment. Unions are required to have at least two fields.  
此指令告诉 DSDL 编译器当前消息或服务数据类型（请求或响应）的当前部分是一种标记联合体。标记联合体是一种数据结构，一次可以对其任意一个字段进行编码。这样的数据结构包含一个隐式字段，一个标记联合体可以表示数据结构当前保持的特定字段。联合体被要求至少有两个字段。

This directive must be placed before the first attribute definition.  
这个指令必须放在第一个属性定义之前。

The encoding rules for tagged unions are defined in the following chapters.  
标记联合体的编码规则在下面的章节中定义。

#### Comments （注释）
A DSDL description may contain comments starting from a number sign # up to the end of the line. Comments must be ignored by the DSDL compiler.  
DSDL 描述可以包含从数字符号 # 开始到行的末尾的注释。注释的内容会被 DSDL 编译器忽略掉。

#### Service response marker （服务响应标志）
A service response marker separates the request and response parts of a service data type definition. The marker consists of three minus symbols (-) in a row on a dedicated line:  
服务响应标记将服务数据类型定义的请求和响应部分分开。该标记一行独立的的 3 个负号（-）组成：
```
	---
```

### Primitive data types （原始数据类型）
These types are assumed to be built-in. They can be directly referenced from any data type of any namespace. The DSDL compiler should implement these types using the native types of the target programming language. Example mapping to native types is given for C/C++.  
原始数据类型被认为是内置的。它们可以直接在任何命名空间的任何数据类型中被引用。DSDL 编译器应该使用目标编程语言的原生类型实现原始数据类型。在 C / C++ 中给出了映射到原生类型的示例。

There is also a set of special built-in types voidX, where X is the number of bits, 1 to 64 inclusive; e.g. void7. They are used for data alignment purposes.  
另外还有一组特殊的内置类型 voidX，其中 X 为bit的位数，包括1到64位；例如 void7。它们主要用于数据对齐。

| Name (名称)| Bit length （bit 长度）| Possible representation in C/C++ （C/C++的表示）| Value range （取值范围） | Binary representation （二进制表示） |
| :--- | :--- | :--- | :--- | :--- |
| bool | 1 | bool (can be optimized for bit arrays) | {0, 1} | One bit |
| intX | 2 ≤ X ≤ 64	| int8_t, int16_t, int32_t, int64_t | $[-(2^{X})/2, 2^{X}/2 - 1]$ | Two’s complement |
| uintX | 2 ≤ X ≤ 64 | uint8_t, uint16_t, uint32_t, uint64_t | $[0, 2^{X} - 1]$ ||
| float16 | 16 |float| ±65504 | IEEE754 binary16 |
| float32 | 32 |float| $Approx. ±10^{39}$ | IEEE754 binary32 |
| float64 | 64 |double| $Approx. ±10^{308}$ | IEEE754 binary64 |
| voidX | 1 ≤ X ≤ 64 | N/A | N/A | X zero bits (set to zero when encoding, ignore when decoding) |


### Naming rules (命名规则)
#### Mandatory (强制规则)
Field names, constant names, and type names must contain only ASCII alphanumeric characters and underscores ([A-Za-z0-9_]), and must begin with an ASCII alphabetic character ([A-Za-z]). Violation of this rule must be detected by the DSDL compiler and treated as a fatal error.  
字段名、常量名和类型名必须只包含ASCII字母数字字符和下划线（[A-Za-z0-9_]），并且必须以ASCII字母字符([A-Za-z])开头。违反此规则时会被 DSDL 编译器检测到，并将其视为严重错误。

#### Optional （可选的规则）
The following rules are recommended only (DSDL compilers are not required to enforce them):  
以下规则只建议（DSDL 编译器不会对其做强制要求）：

 - Field and namespace names should be all-lowercase words separated with underscores, and may include numbers, (e.g.: field_name, my_namespace_7).  
 - 字段和命名空间的名称应该是全小写的单词，用下划线分隔，可以包括数字（例如：field_name, my_namespace_7）。
<br/>
 - Constant names should be all-uppercase words separated with underscores, and may include numbers (e.g.: CONSTANT_NAME).  
 - 常量名称应该是全大写的单词，用下划线分隔，可以包括数字（例如：CONSTANT_NAME）。
<br/>

 - Data type names should be in camel case (first letter of all words in uppercase) and may include numbers (e.g.: TypeName, TypeName2).  
 - 数据类型名称应该是驼峰式的（所有单词的首字母是大写的），并且可以包含数字（例如：TypeName, TypeName2）。

#### Advisory （建议）
The following rules should be considered by the application designer, but should not be enforced:  
应用程序设计人员应该考虑以下规则，但不必强制执行:

 - Message names should be nouns or adjectives; service names should be verbs.  
 - 信息名称应是名词或形容词；服务名称应该是动词。
 - The name of a message that carries a command should end with the word “Command”; the name of a message that carries state information should end with the word “Status”.  
 - 载有命令的讯息的名称应以“Command”字结尾；带有状态信息的消息的名称应该以“Status”结尾。
 - The name of a service that is designed to obtain or to store data should begin with the word “Get” or “Set”, respectively.  
 - 用于获取或存储数据的服务的名称应分别以“Get”或“Set”开头。

### Data type compatibility （数据类型兼容性）
#### The concept of data type compatibility （数据类型兼容性的概念）
It is vital that all nodes exchanging some particular data structure use compatible DSDL definitions of it. Different DSDL definitions are considered compatible if they share the aspects listed below.  
所有用于节点交换的特定数据结构都必须使用相互兼容的 DSDL 定义。不同的 DSDL 定义如果出现下列情况，则会被误认为它们是兼容的。		

##### Binary layout （二进制的布局）
Encoded binary representation must be interpreted by different nodes in the same way. This implies that compatible data structures must have the same field types in the same order.  
编码的二进制表示必须由不同的节点以相同的方式进行解释。这意味着兼容的数据结构必须具有相同顺序的字段类型。

First definition:  
第一个定义：
```
uint8 a
uint8 b
```

Second definition:  
第二个定义
```
uint12 a
uint4 b
```

Even though the bit length of the data structures above is the same, the binary layout is clearly not compatible.  
尽管上面的数据结构的位长是相同的，但是二进制的布局显然是不兼容的。				 
				
##### Field name and order （字段名和顺序）
Encoded binary representation must be interpreted by different nodes in the same way. This implies that compatible data structures must have the same field types and names in the same order.  
编码的二进制表示必须由不同的节点以相同的方式进行解释。这意味着兼容的数据结构必须具有相同的字段类型和名称，并且顺序相同。

First definition:  
第一个定义：
```
uint8 a
uint8 b
```

Second definition:  
第二个定义：
```
uint8 b
uint8 a
```
Even though the first and the second definitions share the same binary layout (two fields of type uint8), they feature different field names and therefore are semantically incompatible.  				
尽管第一个和第二个定义共享相同的二进制布局（两个uint8类型的字段），但它们具有不同的字段名称，因此在语义上不兼容。

##### Constant definitions （常量的定义） 
Despite the fact that different constant values may render the different representations of the same data structure semantically incompatible, UAVCAN does not consider constant definitions in signature computation, because fixing constant definitions renders data types not extensible, i.e., any modification of a constant would break backward compatibility.  
尽管不同的常数值可能导致同一数据结构的不同表示在语义上不兼容，但 UAVCAN 在签名计算中不考虑常量定义，因为如果固定常量定义则会导致数据类型不可扩展，比如，对常数的任何修改都会破坏向后兼容性。

#### Signature (签名)
Incompatible data types may break the data exchange and cause unpredictable system behavior. In order to ensure type compatibility the concept of a data type signature is introduced. This section explains both the data type signature and a number of associated/auxiliary concepts.  
不兼容的数据类型可能会中断数据交换并导致不可预知的系统行为。为了保证类型的兼容性，引入了数据类型签名的概念。本节解释数据类型签名和一些相关/辅助概念。

Signature usage is discussed in the topic: CAN bus transport layer.  
签名的使用将在“CAN总线传输层”中讨论。

##### Signature hash function （签名哈希函数）

The data type signature is based on CRC-64-WE:  
这种数据类型的签名基于 CRC-64-WE：

 - Name: CRC-64-WE
 - Description: http://reveng.sourceforge.net/crc-catalogue/17plus.htm#crc.cat-bits.64
 - Initial value: 0xFFFFFFFFFFFFFFFF
 - Poly: 0x42F0E1EBA9EA3693
 - Reverse: no
 - Output XOR: 0xFFFFFFFFFFFFFFFF
 - Check: 0x62EC59E3F1A4F00A

Source code in Python:  
Python语言的源码：
```
# License: CC0, no copyright reserved
class Signature:
    MASK64 = 0xFFFFFFFFFFFFFFFF
    POLY = 0x42F0E1EBA9EA3693

    def __init__(self, extend_from=None):
        if extend_from is not None:
            self._crc = (int(extend_from) & Signature.MASK64) ^ Signature.MASK64
        else:
            self._crc = Signature.MASK64

    def add(self, data_bytes):
        if isinstance(data_bytes, str):
            data_bytes = map(ord, data_bytes)
        for b in data_bytes:
            self._crc ^= (b << 56) & Signature.MASK64
            for _ in range(8):
                if self._crc & (1 << 63):
                    self._crc = ((self._crc << 1) & Signature.MASK64) ^ Signature.POLY
                else:
                    self._crc <<= 1

    def get_value(self):
        return (self._crc & Signature.MASK64) ^ Signature.MASK64

```

##### Normalized data type definition (正则化数据类型定义)
Some traits of a data type are important to ensure compatibility and some are not. A normalized data type definition is a definition that does not have such unimportant traits.  
并不是所有的数据类型定义都兼容性有着决定性影响。正则化数据类型定义就是定义那些没有重要影响的特征。

To obtain a normalized definition of a given data type, the following actions must be performed on its definition:  
要获得一个给定的数据类型的正则化定义，必须对其定义执行以下操作：

 - Remove comments.  
 - 删除注释。 
  <br/>
  
 - Remove all constant definitions.  
 - 删除所有的常量定义。
  <br/>
 
 - Ensure that all cast specifiers are explicitly defined; if not, add default cast specifiers.  
 - 确保所有的强制类型转换被明确定义；如果没有，添加默认的的强制类型转换说明符。
   <br/>
   
 - For dynamic arrays, replace the max length specifier in the form [<X] to the form [<=Y].  
 - 对于动态数组，将表单中的最大长度说明符从[<X]替换为[<=Y]。
   <br/>
   
 - For nested data structures, replace all short names with full names.  
 - 对于嵌套数据结构，将所有短名称替换为全名称。
   <br/>
   
 - Remove unimportant whitespace (empty lines, leading whitespace, trailing whitespace, more than  one whitespace between tokens).  
 - 删除不重要的空白（空行、前导空白、后置空白、标记之间的多个空白）。
   <br/>
   
 - Prepend the DSDL definition with the full data type name on a separate line.  
 - 预先将 DSDL 定义用完整的数据类型名称放在独丽的一行中。
   <br/>
   
 - Replace newline characters with the ASCII line-feed character (code: 0x0A; escape sequence: \n).  
 - 换行符使用ASCII换行字符（代码：0x0A；转义序列：\n）。
   <br/>

Example for message type A in the namespace root:  
举个例子，根命名空间中的消息类型 A ：

```
#
# A header comment.
# Note that the formatting is broken deliberately.
#

union

float16 foo

float16 BAR    = 12.34  # This is BAR
truncated uint8    bar

int32  FOO =   - 42
```

Normalized definition:
正则化定义：

```
root.A
union
saturated float16 foo
truncated uint8 bar
```

Example for service type A in the namespace root:  
举个例子，根命名空间中的服务类型 A ：
```
#
# A header comment.
# Note that the formatting is broken deliberately.
#

B foobar
float16 foo
float16 BAR    = 12.34  # This is BAR
---

truncated uint8    foo
int32  BAR =   -42
root.ns1.B baz
```
Normalized definition:  
正则化定义：

```
root.A
root.B foobar
saturated float16 foo
---
truncated uint8 foo
root.ns1.B baz
```

##### DSDL signature (DSDL签名)
DSDL signature is the product of the application of the signature hash function to a normalized data type definition.  
DSDL 签名是签名哈希函数应用于正则化数据类型定义的产物。

It is important to understand that DSDL signature is not the same as data type signature (because the data type may reference definitions stored in several files).  
务必理解 DSDL 签名与数据类型签名不同（因为数据类型可能引用存储在多个文件中的定义）。

##### Data type signature（数据类型签名）
Data type signature is a 64-bit integer value that is guaranteed to be equal for compatible data types.  
数据类型签名是一个64位的整数值，对兼容的数据类型确保相等。

Hence, it is said that data types are compatible if their names and signatures are equal.  
因此，如果数据类型的名称和签名都是相同的，那么它们就是兼容的。

The following is the data type signature computation algorithm for a given data type, where hash is the signature hash function described earlier:  
下面是对一个已给定的数据类型，生成数据类型签名的计算算法，其中哈希是前面描述的签名哈希函数：

 - Initialize the hash value with the DSDL signature of the given type.  
 - 使用 DSDL 签名给定的数据类型初始化哈希值。
 - Starting from the top of the DSDL definition, do the following for each nested data structure:  
 - 从 DSDL 定义的顶层开始，对每个嵌套的数据结构执行以下操作：
	 - Extend the current hash value with the data type signature of the nested data structure.  
	 - 使用嵌套的数据结构的数据类型签名扩展当前的哈希值。
 - The resulting hash value will be the data type signature.  
 - 产生的哈希值将是数据类型签名。


The hash extension algorithm:  
哈希扩展算法：

 - Save the current hash value.  
 - 保存当前哈希值。
 - Feed the value the hash needs to be extended with to the hash function, byte by byte, LSB first.  
 - 将需要扩展的哈希值按字节提供给哈希函数，LSB在前。
 - Feed the saved hash value to the hash function, byte by byte, LSB first.  
 - 将保存的哈希值逐个字节地提供给哈希函数，LSB在前。

The data type signature computation algorithm has the following properties:  
数据类型签名计算算法具有如下性质：

Data type signature and DSDL signature of the same type are equal if the data type does not contain nested data structures.  
如果数据类型不包含嵌套的数据结构，则相同类型的数据类型签名和 DSDL 签名是相等的。

Data type signature is guaranteed to match only if all nested data structures are compatible.
The implementation of the algorithm explained above can be found in one of the existing implementations.  
只有在所有嵌套数据结构都兼容的情况下，才能保证数据类型签名匹配。
上述算法的实现可以在现有的实现中找到。

##### Aggregate signature （合集签名）
Aggregate data type signature is a signature computed for a set of data types.  
合集数据类型签名是为一组数据类型计算的签名。

Aggregate data type signature is useful for checking the compatibility between two sets of data types.  
合计数据类型签名对于检查两组数据类型之间的兼容性非常有用。

Two sets of data types X and Y are compatible if each data type in X has exactly one compatible data type in Y and both sets are of equal size.  
如果每个 X 中的数据类型恰好对应一个 Y 中的兼容数据类型，并且两个数据类型的大小相同，则两组数据类型 X 和 Y 是兼容的。

One possible way to ensure compatibility between X and Y is to directly check the compatibility of each element from X against each element from Y. UAVCAN, however, defines a computationally less expensive algorithm based on data type signature extension (as defined above).  
确保 X 和 Y 之间兼容性的一种可以用的的方法是直接检查 X 中的每个元素与 Y 中的每个元素的兼容性。然而，UAVCAN 定义了一种基于数据类型签名扩展的计算开销较低的算法（如上所述）。

The algorithm that computes the aggregate signature for the set of data types A is defined as follows, where hash is the signature hash function described earlier:  
为数据类型 A 集合计算聚合签名的算法定义如下，其中哈希是指前面描述的签名哈希函数：

- Sort A by full data type name lexicographically in descending order (encoding is ASCII, order is A to Z).  
- 把完整数据类型名称按字典降序排序（编码为ASCII，顺序为A到Z）。
<br/>
- Initialize the hash value with the data type signature of the first data type from A.  
- 使用来自 A 的第一个数据类型的数据类型签名初始化哈希值。
<br/>
- For each data type a from A, starting from the second, do the following:  
- 对于每个来自 A 的数据类型 a，从第二个数据类型开始，执行以下操作：
	<br/>
	- Extend the current hash value with the data type signature of a.  
	- 使用 a 的数据类型签名扩展当前哈希值。

Then, two sets of data types are considered to be compatible if their aggregate signatures are equal.  
然后，如果两组数据类型的合集签名相同，则认为它们是兼容的。

### Data type ID (数据类型 ID)
The set of possible data type ID values is limited, so devices from different vendors may occasionally reuse the same data type ID for different purposes. A part of the data type ID space is reserved for standard data types. Since all UAVCAN nodes should share the same configuration for standard data types, collisions are unlikely to occur in that range. Another part of the data type ID space is dedicated for vendor-specific (or application-specific) IDs, and this range of IDs is always collision prone.  
可用的数据类型ID值是有限的，因此来自不同供应商的设备可能偶尔为不同的目的重用相同的数据类型ID。数据类型ID空间的一部分是为标准数据类型保留的。由于所有UAVCAN节点都应该共享标准数据类型的相同配置，所以在这个范围内不太可能发生冲突。数据类型ID空间的另一部分专用于特定于供应商（或特定于应用程序）的ID，并且这个ID范围总是容易发生冲突。

In order to allow the nodes from different vendors to be used in the same application, the end user must be able to change the ID of any non-standard data type on each node. Vendors are advised to provide the end user with an option to change the ID of any standard message type as well.  
为了允许在同一应用程序中使用来自不同供应商的节点，最终用户必须能够更改每个节点上任何非标准数据类型的ID。建议供应商也向最终用户提供一个选项来更改任何标准消息类型的ID。

Notes:  
备注：

 - The reserved ID ranges listed in the Application level conventions.  
 - 在应用程序层列出保留未未使用的ID范围。
<br/>
 - Message types and service types do not share the same set of possible data ID values (i.e., a message type and a service type can share the same data type ID with no conflict). You can learn more about this in the CAN bus transport layer specification.  
 - 消息类型和服务类型独立计算自己的数据 ID （即，一个消息类型和服务类型可以共享相同的数据类型ID，且不存在冲突)。您可以在 CAN 总线传输层规范中了解更多相关信息。
<br/>
 - Changing ID does not affect data type compatibility.  
 - 更改ID不会影响数据类型兼容性。


### Data serialization (数据的序列化)
#### Serialized data format （序列化数据的格式）
Serialized data are an ordered sequence of bit fields; no header or footer is provided, and fields are not implicitly aligned. DSDL authors are advised (but not required) to manually align fields at byte boundaries using the void data type, in order to simplify data layouts and improve the performance of serialization and deserialization routines.  
序列化数据是指位域的有序序列；没有提供头或位，字段没有隐式对齐。建议 DSDL 的编辑者（但不是必需的）使用 void 数据类型在字节边界手动对齐字段，以简化数据布局并提高序列化和反序列化例程的性能。

The rules of encoding of different data types are defined below.  
下面定义了不同数据类型的编码规则。

##### Primitive types
| Type(类型) | Bit length（bit长度） | Binary representation（二进制表达） |
| :--- | :--- | :--- |
| intX | X | Two’s complement signed integer |
| uintX | X | Plain bits |
| bool | 1 | Single bit |
| float16 |	16|	IEEE754 binary16 |
| float32 |	32|	IEEE754 binary32 |
| float64 |	64|	IEEE754 binary64 |
| voidX | X | X zero bits; ignore when decoding |

##### Nested data structures （嵌套的数据结构）
Nested data structures are encoded in much the same way as if they were standalone top-level data structures, with one exception for tail array optimization, as defined below.  
嵌套数据结构的编码方式与独立的顶层数据结构的编码方式基本相同，只有一个例外，即尾部数组优化，定义如下。

##### Fixed size arrays（固定大小的数组）
Fixed-size arrays are encoded as a plain sequence of items, with each item encoded independently in place, with no alignment. No extra data is added.  
固定大小的数组被当作项目的普通序列编码，每个项目单独编码，没有对齐。没有添加额外的数据。

Essentially, a fixed-size array of size X will be encoded exactly in the same way as a sequence of X fields of the same type in a row. Hence, the following data type:  
本质上，大小为 X 的固定大小的数组将采用与相同类型的 X 字段分别单列一行，完全相同的方式进行编码。因此，数据类型如下：
```
AnyType[3] array
```

will have the same binary layout as:  
将和下面的编码有相同的二进制布局: 

```
AnyType array_0
AnyType array_1
AnyType array_2
```

The only difference is the representation in the target programming language.  
惟一的区别是在目标编程语言中的表示。

##### Dynamic arrays （动态数组）
Dynamic array encoding rules are sophisticated; hence it is recommended to review the existing implementations for a deeper understanding.  
动态数组编码规则复杂；因此，建议您回顾现有的实现，以便更深入地理解。

There is no difference between the following two array definitions other than text representation for the human benefit:  
除了文本表示之外，以下两个数组定义之间没有区别：
```
AnyType[<42]  a # Max size 41
AnyType[<=41] b # Max size 41
```

Both forms are equivalent in terms of the data serialization.  
这两种形式在数据序列化方面是等价的。

Array items can be of variable length (for example, if the item’s type is a data structure that itself contains dynamic arrays). As a result, array maximum and minimum size may not not always be the same.  
数组项可以是可变长度的（例如，如果项的类型是一个本身包含动态数组的数据结构）。因此，数组的最大和最小大小不可能总是相同的。

Normally, a dynamic array will be encoded as a sequence of encoded items, prepended with an unsigned integer field representing the number of contained items - the length field. The bit width of the length field is a function of the maximum number of items in the array: ⌈log2(X + 1)⌉, where X is the maximum number of items in the array. For example, if the maximum number of items is 251, the length field bit width must be 8 bits, or if the maximum number of items is 1, the length field bit width will be just a single bit.  
通常，一个动态数组将被编码为一个编码项序列，以一个无符号整数字段作为前缀，表示包含的项的数量——长度字段。长度字段的bit宽度是函数数组：⌈log2(X + 1)⌉ 的最大值，其中 X 是数组的最大条目数。例如，如果项的最大数目是 251，则长度字段位宽必须为 8 bit，或者如果项的最大数目是 1，则长度字段位宽将仅为 1 bit。

The transport layer provides a data length for every received data transfer (with an 8-bit resolution); thus, in some cases, the array length information would be redundant as it can be inferred from the overall transfer length reported by the transport layer. Elimination of the dynamic array length field is called tail array optimization, and it can be done if all the conditions below are satisfied:  
传输层为每个接收到的数据传输提供数据长度（具有 8 bit 分辨率）；因此，在某些情况下，数组长度信息是冗余的，因为它可以从传输层报告的整个传输长度推断出来。删除动态数组长度字段称为尾部数组优化，满足以下条件即可:

 - The minimum bit length of an item type is not less than 8 bits - because the transport layer reports the transfer length with an 8-bit resolution.  
 - 项目类型的最小位长不小于8位——因为传输层以 8 bit 为最小传输长度单位发送报告。
 <br/>
 
 - The array is the last field in the top-level data structure - because, otherwise, a much more complicated logic would be required to derive the length.  
 - 数组是顶层数据结构中的最后一个字段——因为，不然的话，派生长度将需要更复杂的逻辑。
 
The second condition means that tail array optimization would be possible only if there are no other fields in the output bit stream after the dynamic array. Only one array can be tail optimized. Below are some examples that illustrate when a tail array optimization is possible and when it is not.  
第二个条件意味着，只有在动态数组之后的输出位流中没有其他字段时，才有可能进行尾部数组优化。只能对一个数组进行尾部优化。下面是一些例子，说明什么时候可以进行尾部数组优化，什么时候不可以。

```
# Type root.A
uint8 foo
# Tail array optimization is possible here
# It will be encoded just like a fixed-size array
# The number of elements will be inferred from the transfer length
uint8[<9] array

# Type root.B
float16 foo
# Tail array optimization will not be possible because item length is less than 8 bits
uint7[<=8] array

# Type root.C
# Tail array optimization will not be possible because the array is not the last field
uint8[<=8] array
float16 bar

# Type root.D
# Tail array optimization will not be possible because item length is less than 8 bits
bool[<=42] array

# Type root.E
# Tail array optimization will not be possible because the minimum item length is zero
root.D[<=42] array     # Refer to root.D definition above
```

The same rules are applied to more sophisticated cases, for example when multiple nested types with dynamic arrays are used. Such structure may produce very complex binary layouts, but the same array encoding rules are still applicable.  
同样的规则也适用于更复杂的情况，例如使用带有动态数组的多个嵌套类型。这种结构可能会产生非常复杂的二进制布局，但是同样的数组编码规则仍然适用。

```
# Type root.Z
# Refer to root.A definition above
# This array will be tail optimized
# Arrays inside root.A will not be tail optimized
root.A[<=2] array

# Type root.Y
# This array will NOT be tail optimized
# Arrays inside root.A will NOT be tail optimized
root.A[<=2] array
# The last field here effectively disables any arrays from being tail optimized
float16 baz

# Type root.Q
int4 fooz
# Just an array that can be tail optimized, no nested types
float64[<=64] array

# Type root.X
# This array will NOT be tail optimized
# Because the minimum bit length of root.Q is 4 bits (less than 8 bits)
# However, the last array in the last item will be tail optimized
root.Q[<=12] array
```

Application designers are advised to keep in mind the tail array optimization rules when designing custom data types.  
在设计自定义数据类型时，建议应用程序设计人员牢记尾数组优化规则。

##### Unions (联合体)
Unions are encoded as two subsequent entities:  
联合体被编码为两个子序列实体：

- The union tag;  
- 联合体标签；
<br/>
- The selected field.  
- 选择的字段。

The union tag is an unsigned integer, whose bit length is a function of the number of fields in the union: ⌈log2(N)⌉, where N is the number of fields in the union. The value encoded in the union tag is the index of the selected field. Field indexes are assigned according to the order in which they are defined, starting from zero; i.e. the first defined field gets index 0, second defined field gets index 1, and so on.  
联合体标签是一个无符号整数，其 bit 长度是一个函数的值：[log2(N)]，其中 N 是联合体中的字段的数量。联合体标记中编码的值是所选字段的索引。字段索引是按照定义它们的顺序分配的，从 0 开始；即第一个定义的字段得到索引 0，第二个定义的字段得到索引 1，依此类推。

Consider the following example:  
看下面的例子：
```
#
# This is a union.
#

@union                  # In this case, the union tag requires 2 bits

uint16 FOO = 42         # A regular constant attribute

uint16 a                # Index 0
uint8 b                 # Index 1
float64 c               # Index 2

uint32 BAR = 42         # Another regular constant
```

In order to encode the value b, which, according to the definition, has the data type uint8, the union tag should be assigned the value 1. The following structure will have identical layout:  
为了对值 b 进行编码（根据定义，b 的数据类型为 uint8），应该将 union 标记赋值为 1。下面的结构将有相同的布局：  

```
uint2 tag
uint8 b
```

By way of an example, if the value of b was 7, the resulting encoded byte sequence would be (in binary):  
举个例子，如果 b 的值是 7，那么编码后的字节序列将是（二进制）:  
```
01000001 11000000
```

#### Byte order and bit order （字节顺序和位顺序）
Byte order: least significant byte (LSB) first, also known as little-endian.  
字节顺序：首先是最低有效字节（LSB），也称为小端。

Bit order: bits are filled from the most significant to the least significant, i.e., the most significant bit has index 0.  
位顺序：从最重要的位到最不重要的位。比如，最重要的位的序列为 0。

The resulting bit sequence must be aligned to 1 byte; pad bits must be set to zero.  
产生的 bit 序列必须对齐到 1 字节；垫位必须设置为 0。

For the purpose of example, the following data will be encoded according to the defined order:  
为了便于举例，以下数据将按照定义的顺序进行编码：

 1. 0xbeda of bit length 12
 2. -1 of bit length 3
 3. -5 of bit length 4
 4. -1 of bit length 2
 5. 0x88 of bit length 4

The resulting byte sequence is shown on the following diagram:  
得到的字节序列如下图所示：
<br/>
<div align=center>![Alt text](./picture/1581253916886.png)
<br/>

We recommend you review the existing implementations.  
建议回顾一下现有的实现。

### Standard data types （标准数据类型）
The DSDL definitions of the standard data structures are available in the DSDL repository, which is linked here, and in a later section of the specification.  
标准数据结构的 DSDL 定义在 DSDL仓库中可以获得。仓库地址如下。
```
https://github.com/UAVCAN
```

Information concerning development and maintenance of the standard DSDL definitions is available here.  
有关标准 DSDL 定义的开发和维护的信息可以在 01.Introduction 找到。


### Vendor-specific data types （生产商的特定数据类型）
Vendors must define their specific data types in a separate namespace, which should typically be named to match their company name. Separation of the vendor’s definitions into a dedicated namespace ensures that no name conflicts will occur in systems that utilize vendor-specific data types from different providers. Note that, according to the naming requirements, the name of a DSDL namespace must start with an alphabetic character; therefore, a company whose name starts with a digit will have to resort to a mangled name, e.g. by moving the digits towards the end of the name, or by spelling the digits in English (e.g. 42 - fortytwo).  
供应商必须在单独的命名空间中定义其特定的数据类型，通常应将其命名为与公司名称匹配。将供应商的定义分离到专用的命名空间中，可以确保在使用来自不同供应商的特定于供应商的数据类型的系统中不会发生命名冲突。注意，根据命名要求，DSDL 命名空间的名称必须以字母字符开头；因此，名字以数字开头的公司将不得不使用一个修饰过的名字，例如，将数字移到名字的末尾，或者用英语拼写数字（如42 - fortytwo）。


Defining vendor-specific data types within the standard namespace uavcan is explicitly prohibited. The standard namespace will always be used only for standard definitions.  
明确禁止在标准命名空间 uavcan 中定义特定于供应商的数据类型。标准命名空间将始终仅用于标准定义。

Generally speaking it is desirable for “generic” data types to be included into the standard set. Vendors should strive to design their data types as generic and as independent of their specific use cases as possible. The SI system of measurement units should be preferred, as data type definitions that make unnecessary deviations from SI will not be accepted into the standard set.  
一般来说，“通用”数据类型应该包含在标准集中。供应商应该努力将他们的数据类型设计为通用的，并且尽可能独立于他们的特定用例。测量单位的国际单位制（SI）应该是首选的，避免引起不必要的偏差。
