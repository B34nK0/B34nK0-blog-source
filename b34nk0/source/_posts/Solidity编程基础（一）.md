---
title: Solidity编程基础(一)
date: 2022-01-18 23:57:39
categories: 区块链
tags: Solidity
---

# 概要

由ETH为代表的第二代区块链技术，相比于第一代区块链技术而言，最大的特点就是智能合约的出现，让去中心化应用成为了可能。ETH节点为智能合约提供运行环境：EVM（Ethereum Virtual Machine）以太坊虚拟机。EVM是一个动态运行沙盒，可以将以太坊上所有的智能合约和周围环境全部隔离。因此，EVM上运行的智能合约无法访问网络、文件系统或者在EVM上运行的其他进程。

Solidity是一个基于合约高级编程语言，它是静态类型语言，支持继承、库和复杂的用户定义两类型等功能。它可以被编译成EVM的汇编语言，从而被链上的节点所执行。其他语言还有Serpent、Vyper和LLL，同样可被编程成EVM的汇编语言从而在其节点上运行。

solidity的IDE环境可使用：[Remix](https://remix.ethereum.org/)

<!--more-->  

# sol文件结构
## 编译开发
pragma关键字沿用c、c++编译指令概念。
```cpp
pragma solidity ^0.4.0;
```
以上指令表明编译器版本需要高于0.4.0才可编译。

```javascript
pragma solidity >= 0.4.22 < 0.6.0;
```

可以使用更复杂的规则来指定编译器的版本，表达式遵循 npm 版本语义。

## 引入其他文件
1、全局引入：
```javascript
import "filename";
```

2、自定义命名空间
此语句将从 “filename” 中导入所有的全局符号到当前全局作用域中（不同于 ES6，Solidity 是向后兼容的）。
```javascript
import * as symbolName from "filename";
```
...创建一个新的全局符号 symbolName，其成员均来自 "filename" 中全局符号。

另一种语法不属于 ES6，但或许更简便：
```javascript
import "filename" as symbolName;
```

3、多包引入
```javascript
import {symbol1 as alias, symbol2} from "filename";
```
创建新的全局符号 alias 和 symbol2，分别从 "filename" 引用 symbol1 和 symbol2 。

4、路径
上文中的 filename 总是会按路径来处理，以 `/`作为目录分割符、以 `.` 标示当前目录、以  `..`表示父目录。 当 `.` 或 `..` 后面跟随的字符是 `/` 时，它们才能被当做当前目录或父目录。 只有路径以当前目录 `.` 或父目录 `..` 开头时，才能被视为相对路径。

用 `import "./x" as x;` 语句导入当前源文件同目录下的文件 x 。 如果用 `import "x" as x;` 代替，可能会引入不同的文件（在全局 include directory 中）。

Remix 提供一个为 github 源代码平台的自动重映射，它将通过网络自动获取文件： 比如，你可以使用 `import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;` 导入一个 map 迭代器。

关于编译器的路径配置可以根据自己的编译器进行查找

# 注释
## 代码注释

```javascript
// 这是一个单行注释。

/*
这是一个
多行注释。
*/
```

## 文档注释

```javascript
pragma solidity ^0.4.0;

/** @title 形状计算器。 */
contract shapeCalculator {
    /** @dev 求矩形表明面积与周长。
    * @param w 矩形宽度。
    * @param h 矩形高度。
    * @return s 求得表面积。
    * @return p 求得周长。
    */
    function rectangle(uint w, uint h) returns (uint s, uint p) {
        s = w * h;
        p = 2 * (w + h);
    }
}
```
# 合约
在 Solidity 中，合约类似于面向对象编程语言中的类。 每个合约中可以包含 状态变量、 函数、 函数修饰器、事件、 结构类型、 和 枚举类型 的声明，且合约可以从其他合约继承。

采用关键字 contract声明
```javascript
pragma solidity ^0.4.0;

contract ContractExample{
   
}
```

## 状态变量
状态变量是永久地存储在合约存储中的值，在合约里声明而不属于任何函数的都是状态变量。

```javascript
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData; // 状态变量
    // ...
}
```
## 类型
Solidity 是一种静态类型语言，这意味着每个变量（状态变量和局部变量）都需要在编译时指定变量的类型（或至少可以推导出变量类型 -- 类型推导）。 Solidity 提供了几种基本类型，可以用来组合出复杂类型。

### 值类型
以下类型也称为值类型，因为这些类型的变量将始终按值来传递。 也就是说，当这些变量被用作函数参数或者用在赋值语句中时，总会进行**值拷贝**。

#### 1、 布尔类型
`bool` ：可能的取值为字面常数值 `true` 和 `false` 。

**运算符：**

`!` （逻辑非）
`&&` （逻辑与， "and" ）
`||` （逻辑或， "or" ）
`==` （等于）
`!=` （不等于）
运算符 `||` 和 `&&` 都遵循同样的**短路（ short-circuiting ）规则**。就是说在表达式 f(x) || g(y) 中， 如果 f(x) 的值为 true ，那么 g(y) 就不会被执行，即使会出现一些副作用。

#### 2、整型
`int / uint` ：分别表示有符号和无符号的不同位数的整型变量。 支持关键字 `uint8` 到 `uint256` （无符号，从 8 位到 256 位）以及 `int8` 到 `int256`，以 8 位为步长递增。 `uint` 和 `int` 分别是 `uint256` 和 `int256` 的别名。

**运算符：**

比较运算符： `<= ， < ， == ， != ， >= ， > （返回布尔值）`
位运算符： `& ， | ， ^ （异或）， ~ （位取反）`
算数运算符： `+ ， - ， 一元运算 - ， 一元运算 + ， * ， / ， % （取余） ， ** （幂）， << （左移位） ， >> （右移位）`
除法总是会截断的（仅被编译为 EVM 中的 DIV 操作码）， 但如果操作数都是 **字面常数（literals） （或者字面常数表达式），则不会截断。**

**除以零或者模零运算都会引发运行时异常。**

移位运算的结果取决于运算符左边的类型。 
表达式 `x << y 与 x * 2^y` 是等价的， `x >> y 与 x / 2^y` 是等价的。这意味对一个负数进行移位会导致其符号消失。 **按负数位移动会引发运行时异常。**


#### 3、地址

`address`：地址类型存储一个 **20 字节**的值（以太坊地址的大小）。 地址类型也有成员变量，并作为所有合约的基础。

**运算符：**`<=， <， ==， !=， >= 和 >`

**ps**：
从 0.5.0 版本开始，合约不会从地址类型派生，但仍然可以显式地转换成**地址类型**。

关于地址类型相关内容，在后文介绍

#### 4、定长字节数组
关键字有`：bytes1， bytes2， bytes3， ...， bytes32`。`byte 是 bytes1` 的别名。

可以通过16进制字面量或者数字字面量来设定bytesi：
bytes1 aa = 0x30;	//16进制字面量
bytes2 bb = 10;		//数字字面量

也可通过字符来设定bytesi：
bytes1 dd = 'a';

**运算符：**

比较运算符：`<=， <， ==， !=， >=， > （返回布尔型）`
位运算符： `&， |， ^ （按位异或）， ~ （按位取反）， << （左移位）， >> （右移位）`
索引访问：如果 x 是 bytesI 类型，那么 `x[k] （其中 0 <= k < I）返回第 k 个字节（只读）`。
该类型可以和作为右操作数的任何整数类型进行移位运算（但返回结果的类型和左操作数类型相同），右操作数表示需要移动的位数。 进行负数位移运算会引发运行时异常。

**成员变量：**
`.length` 表示这个字节数组的长度（只读）.

#### 5、有理数和整型字面量

 - 整数字面量：1,10，-1，-100
 - 字符字面量："test"、'test'，双引号或者单引号都可以
 - 地址字面量：0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF
 - 16进制字面量，以0x为前缀：0x9aaa
 - 数字字面量：7.5,0.2

整数字面常数由范围在 **0-9** 的一串数字组成，表现成**十进制**。 例如，69 表示数字 69。 Solidity 中是没有八进制的，因此前置 0 是无效的。

十进制**小数字面常数**带有一个 `.`，至少在其一边会有一个数字。 比如：`1.`，`.1`，和 `1.3`。

科学符号也是支持的，尽管指数必须是整数，但**底数可以是小数**。 比如`：2e10， -2e10， 2e-10， 2.5e1`。

数值字面常数表达式本身支持任意精度，除非它们被转换成了非字面常数类型（也就是说，当它们出现在非字面常数表达式中时就会发生转换）。 这意味着**在数值常量表达式中, 计算不会溢出而除法也不会截断**。

例如， (2^800 + 1) - 2^800 的结果是字面常数 1 （属于 uint8 类型），尽管计算的中间结果已经超过了 以太坊虚拟机Ethereum Virtual Machine(EVM) 的机器字长度。 此外， `.5 * 8` 的结果是整型 4 （尽管有非整型参与了计算）。

只要操作数是整型，任意整型支持的运算符都可以被运用在数值字面常数表达式中。 **如果两个中的任一个数是小数，则不允许进行位运算。如果指数是小数的话，也不支持幂运算（因为这样可能会得到一个无理数）。**

#### 6、枚举类型

```javascript
pragma solidity ^0.4.0;

contract test {
    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
    ActionChoices choice;
    ActionChoices constant defaultChoice = ActionChoices.GoStraight;

    function setGoStraight() public {
        choice = ActionChoices.GoStraight;
    }

    // 由于枚举类型不属于 |ABI| 的一部分，因此对于所有来自 Solidity 外部的调用，
    // "getChoice" 的签名会自动被改成 "getChoice() returns (uint8)"。
    // 整数类型的大小已经足够存储所有枚举类型的值，随着值的个数增加，
    // 可以逐渐使用 `uint16` 或更大的整数类型。
    function getChoice() public view returns (ActionChoices) {
        return choice;
    }

    function getDefaultChoice() public pure returns (uint) {
        return uint(defaultChoice);
    }
}
```

#### 7、函数类型

函数类型是一种表示函数的类型。

 - 可以将一个函数赋值给另一个函数类型的变量
 - 也可以将一个函数作为参数进行传递
 - 还能在函数调用中返回函数类型变量。

 函数类型有两类：- 内部（internal） 函数和 外部（external） 函数：

 - 内部函数

只能在当前合约内被调用（更具体来说，在当前代码块内，包括内部库函数和继承的函数中），因为它们不能在当前合约上下文的外部被执行。 调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

 - 外部函数

由一个地址和一个函数签名组成，可以通过外部函数调用传递或者返回。

函数类型表示成如下的形式

```javascript
function (<parameter types>) {internal(默认)|external} [pure|constant|view|payable] [returns (<return types>)]
```

例如：ER20协议的sendTokens函数

```javascript
function sendTokens(address receiver, uint256 amount) public returns (bool) {
}
```

 1. 该函数有两个参数，分别是流量类型addreess的receiver（接收地址），uint256的amount（数量）。
 2. 函数默认internal内部函数，如果为外部函数external时需要显示声明。
 3. 如果没有返回结果，则必须省略`return`关键字


### 引用类型
比起之前讨论过的值类型，在处理复杂的类型（即占用的空间超过 256 位的类型）时，我们需要更加谨慎。 由于拷贝这些类型变量的开销相当大，我们不得不考虑它的存储位置，是将它们保存在 ** 内存memory ** （并不是永久存储）中， 还是 ** 存储storage ** （保存状态变量的地方）中。

#### 1、数据的存储

关于数据的存储位置：**memory、storage、calldata**

 - 根据上下文不同，大多数时候数据有默认的位置，但也可以通过**在类型名后增加关键字 storage 或 memory** 进行修改。
 -  函数参数（包括返回的参数）的数据位置默认是 memory， 局部变量的数据位置默认是 storage，状态变量的数据位置强制是 storage （持久化）。
 - 也存在第三种数据位置， **calldata ，这是一块只读的，且不会永久存储的位置，用来存储函数参数**。 **外部函数的参数（非返回参数）的数据位置被强制指定为 calldata** ，效果跟 memory 差不多。

#### 2、数据传递方式

相比于值类型的值拷贝方式，因为复杂类型占用空间较大，复制时占用空间较大，所以采用**引用传递**方式。

 - **状态变量向局部变量赋值**时仅仅传递一个引用，而且这个引用总是指向状态变量，因此后者改变的同时前者也会发生改变。
 -  从一个 内存memory 存储的引用类型向另一个 内存memory 存储的引用类型赋值并不会创建拷贝。

```javascript
pragma solidity ^0.4.0;

contract C {
    uint[] x; // 状态变量 x 的数据存储位置是 storage

    // 函数参数 memoryArray 的数据存储位置是 memory
    function f(uint[] memoryArray) public {
        x = memoryArray; // memory 将整个数组拷贝到 storage 中，可行
        var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
        y[7]; // 返回第 8 个元素，可行
        y.length = 2; // 通过 y 修改 x，可行
        delete x; // 清除数组，同时修改 y，可行
        // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
        // 但 storage 是“静态”分配的：
        // y = memoryArray;
        // 下面这一行也不可行，因为这会“重置”指针，
        // 但并没有可以让它指向的合适的存储位置。
        // delete y;

        g(x); // 调用 g 函数，同时移交对 x 的引用
        h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}
```

#### 3、引用类型

常见的引用类型有：

 1. 数组（Array）
 2. 不定长字节数组（Bytes）
 3. 字符串（String）
 5. 结构体（Struct）

##### 1、数组
数组可以在声明时指定长度，也可以动态调整大小。 对于 存储storage 的数组来说，元素类型可以是任意的（即元素也可以是数组类型，映射类型或者结构体）。 对于 内存memory 的数组来说，元素类型不能是映射类型，如果作为 public 函数的参数，它只能是 **ABI 类型**（后文介绍）。

一个元素类型为 T，固定长度为 k 的数组可以声明为 **T[k]**，而动态数组声明为 **T[]**。

举个例子，一个长度为 5，元素类型为 uint 的动态数组的数组，应声明为 uint[][5] （注意这里跟其它语言比，数组长度的**声明位置**是反的）。 
要访问第三个动态数组的第二个元素，你应该使用 x[2][1]（数组下标是从 0 开始的，且**访问数组时的下标顺序与声明时相反**，也就是说，x[2] 是从右边减少了一级）。

**bytes 和 string 类型**的变量是特殊的数组。 bytes 类似于 byte[]，但它**在 calldata 中会被“紧打包”**（译者注：将元素连续地存在一起，不会按每 32 字节一单元的方式来存放）。 string 与 bytes 相同，但（暂时）不允许用长度或索引来访问。

如果想要访问以字节表示的字符串 s，请使用 `bytes(s).length / bytes(s)[7] = 'x';`。
注意这时你访问的是 **UTF-8** 形式的低级 bytes 类型，而不是单个的字符。

**可以将数组标识为 public，从而让 Solidity 创建一个 getter。 之后必须使用数字下标作为参数来访问 getter。**

**创建内存数组**
可使用 **new 关键字**在内存中创建变长数组。 与 存储storage 数组相反的是，你 不能 通过修改成员变量 .length 改变 内存memory 数组的大小。

```javascript
pragma solidity ^0.4.16;

contract C {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
        bytes memory b = new bytes(len);
        // 这里我们有 a.length == 7 以及 b.length == len
        a[6] = 8;
        //不能修改memory数组的长度，错误信息：表达式必须是个左值表达式
		//a.length = 100;
    }

	//storage 数组
	uint[]b;
	function g() {
		b = new uint[](7);
		//可以修改storage的数组
		b.length = 10;
		b[9] = 100;
	}
}
```

**成员**

 1. length:

数组有 length 成员变量表示当前数组的长度。 动态数组可以在 存储storage （而不是 内存memory ）中通过改变成员变量 .length 改变数组大小。 并不能通过访问超出当前数组长度的方式实现自动扩展数组的长度。 一经创建，内存memory 数组的大小就是固定的（但却是动态的，也就是说，它依赖于运行时的参数）。

 2. push:

变长的 存储storage 数组以及 bytes 类型（而不是 string 类型）都有一个叫做 push 的成员函数，它用来附加新的元素到数组末尾。 这个函数将返回新的数组长度。

##### 2、不定长字节数组（Bytes）
Bytes是一个动态数组，它与byte[]不同。byte[]数组为每个元素分配32字节，而bytes对所有字节进行**紧打包**,Bytes可以声明为一个设定长度的状态变量，如下面的代码：

```javascript
bytes localBytes = new bytes(0);
//bytes可以直接赋值
localBytes = "This is a test";
//数组成员，附加一个元素到数组末尾
localBytes.push(byte(10));
//数组成员，表示当前数组长度
return localBytes.length;
```

##### 3、字符串（String）
不像c语言需要'\0'结尾，其存储为UTF-8编码

##### 4、结构体(Struct)
结构体采用struct关键字声明， 由变量构成，是一个组合数据类型。

```javascript
pragma solidity ^0.4.11;

contract CrowdFunding {
    // 定义的新类型包含两个属性。
    struct Funder {
    	//地址类型
        address addr;
 		//uint
        uint amount;
    }

	//4个变量
    struct Campaign {
        address beneficiary;
        uint fundingGoal;
        uint numFunders;
        uint amount;
        mapping (uint => Funder) funders;
    }

    uint numCampaigns;
    //字典映射
    mapping (uint => Campaign) campaigns;

	//创建一个Campaign并存储在字典里
    function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
        campaignID = numCampaigns++; // campaignID 作为一个变量返回
        // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
        campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
    }

	//通过id映射获取Campaign并创建Funder存储到Campaign的成员funders数组中
    function contribute(uint campaignID) public payable {
    	//引用关系，所以可以直接修改局部变量c从而修改到campaigns状态变量的数据
        Campaign storage c = campaigns[campaignID];
        // 以给定的值初始化，创建一个新的临时 memory 结构体，
        // 并将其拷贝到 storage 中。
        // 注意你也可以使用 Funder(msg.sender, msg.value) 来初始化。
        c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
        c.amount += msg.value;
    }

    function checkGoalReached(uint campaignID) public returns (bool reached) {
        Campaign storage c = campaigns[campaignID];
        if (c.amount < c.fundingGoal)
            return false;
        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}
```
上面的合约只是一个简化版的众筹合约，但它已经足以让我们理解结构体的基础概念。 结构体类型可以作为元素用在映射和数组中，其自身也可以包含映射和数组作为成员变量。

注意在函数中使用结构体时，一个结构体是如何赋值给一个局部变量（默认存储位置是 存储storage ）的。 在这个过程中并没有拷贝这个结构体，而是保存一个引用，所以对局部变量成员的赋值实际上会被写入状态。

当然，你也可以直接访问结构体的成员而不用将其赋值给一个局部变量，就像这样， campaigns[campaignID].amount = 0。


#### 4、字典/映射

字典是一种Key/Value对。在声明时的形式为 mapping(_KeyType => _ValueType)。 其中 _KeyType 可以是除了映射、变长数组、合约、枚举以及结构体以外的几乎所有类型。 _ValueType 可以是包括映射类型在内的任何类型。

上文我们在实现众筹合约时，即使用到了` mapping (uint => Campaign) campaigns;`字典，字典不支持迭代，但可以在此基础上实现一个可迭代的[映射](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol)
#### 5、特殊情况
变量定义是放在栈上的，因为栈的容量限制，只能容纳16层，如果定义太多本地变量会导致栈溢出。
解决方案可以将变量封装在结构体里、或者使用memory在内存上创建。