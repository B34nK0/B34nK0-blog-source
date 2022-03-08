---
title: Solidity编程基础（二）
date: 2022-03-08 14:32:09
categories: 区块链
tags: Solidity
---

@[TOC](Solidity编程基础二)
# 概要
本文延续专栏的编程[基础一](https://blog.csdn.net/BBinChina/article/details/122395501)进行学习,本文主要内容讲解Solidity的语句以及修饰符等内容

<!--more-->  
# 语句
## 条件语句
`if/else` 条件语句和其他语言一样，但需要注意的是`if(1) {...}`在Solidity是无效的，但也可以通过强制类型转换将1转成true，尽管与其他语言一样`{}`语句块只有一个语句时可以省略`{}`，但便于开发者查看，建议采用{不省略}的编程规范。

```javascript
 if (totalPoints > bet.line) {
 
 } else if (totalPoints < bet.line) {
 
 } else {
 
 }
```

## 循环语句
* while语句
```javascript
while(true)  {
	if (true) {
		break;
	}
}
```
* for语句
```javascript
for(uint i = 0; i < 10; i++)  {
	if (i == 2) {
		return;
	} else {
		///
	} 
	if (i == 1 ) {
		continue;
	} 
	///
}
```

1) break: 用来跳出现有循环
2) return: 用来从函数方法中返回
3) continue: 用来退出当前的循环，跳到下一次的循环开始

除了以上的` whille、for、if、else、break、continue、return`，还有 `? :`三元操作符
`a > b ? a : b 表示如果 a > b 则返回a，否则返回b`

# 修饰符
在上一章节 我们讲到函数类型时，给出其声明模板

```javascript
function (<parameter types>) {internal(默认)|external} [pure|constant|view|payable] [returns (<return types>)]
```

模板中的`external、internal、pure、constant、view、payable`即是我们常说的函数修饰符。而其中的 `external、internal、public、private`是针对函数的可见性而言，`pure、constant、view、payable`是针对对合约状态变量的修改能力做出规定。

由于 Solidity 有两种函数调用（**内部调用不会产生实际的 EVM 调用或称为“消息调用”，而外部调用则会产生一个 EVM 调用**）， **函数和状态变量**有四种可见性类型。 函数可以指定为 `external` ，`public` ，`internal` 或者 `private`，默认情况下函数类型为 `public`,`internal`。 对于**状态变量**，不能设置为 `external `，默认是` internal` 。

## 修饰符说明
* internal修饰符
1、声明的函数和状态变量只能通过内部访问。如在当前合约中调用，或继承的合约里调用。
2、需要注意的是不能使用`this.`前缀，`this.`表示通过外部方式访问（通过EVM调用栈）。
3、不指定任何修饰符时，函数默认为Internal修饰符，其类似于Java或C++里的protected。
* external修饰符
1、声明函数为外部函数，即可提供其他合约或通过交易来发起调用，通过`this.`如this.f()发起调用。
* public修饰符
1、声明函数为公开函数，可通过外部或者外部消息来调用。
2、对于public类型状态变量，编译器自动为所有 public 状态变量创建 getter 函数,getter函数在下文讲解。
* private修饰符
1、声明函数为私有函数，仅在当前合约中可访问，在继承的合约内不可访问。
2、同理声明状态变量为私有状态，仅在当前合约内访问。
* constant修饰符
1、声明函数**不能更改**区块链的状态变量，只可读取状态变量。包括**不能**改变变量、调用事件、创建另一个合约以及调用其他可能改变状态的函数。
* view修饰符
1、等同于constant
* pure修饰符
1、声明函数**不能读写**状态变量
* payable修饰符
1、声明函数可以从调用者接收ether，如果发送方没有提供ether，调用则可能会失败。
2、而声明为payable也表示该函数只能收取ether。

## 修饰符区别
* external 和 public 修饰符
1、eternal和public都是提供其他合约调用或者交易的方式调用，其主要区别在于合约调用函数的方式和输入参数的传输方式。
2、在合约中，从一个函数直接调用一个`public`函数时，同`private`和`internal`函数一样，代码的执行会通过`jump`汇编指令。但`external`必须通过`call`汇编指令。
3、internal调用会从只读的calldata里**复制**输入参数到内存或者栈上。
4、external只读取calldata数据，并不复制。
5、通过jump指令执行时，数组参数是通过指向内存的指针来传递的。
6、public函数不知道调用者是external还是internal，所以public函数会跟internal一样复制参数到内存上。
7、为了减少开销，尽量确定函数可使用external修饰符

示例总结：

```javascript
pragma solidity ^0.4.12;
contract Test {
	/*
	可以被internal、external调用，但interal调用时不会复制参数a到内存里
	因为复制操作，所以会产生gas额外的燃料费
	代码的执行通过jump指令
	*/
	function test(uint[20]a) public returns(uint) {
		return a[10]*2;
	}

	/*
	不复制参数到内存，直接从calldata读取
	代码的执行通过call指令
	*/
	function test1(uint[20]a) external returns(uint) {
		return a[10]*2;
	}
	
	/*
	同理复制参数到内存
	*/
	function test2(uint[20]a) internal returns(uint) {
		return a[10]*2;
	}
}
```

* internal 和 external修饰符
上述已经讲述了internal和external调用及参数传输方式
以下通过示例讲解internal之间的调用关系

```javascript
pragma solidity ^0.4.5;

contract FunctionTest {
	function internalFunc() internal{}
	function externalFunc() external{}

	function callFunc() {
		//内部调用,不能通过this访问
		internalFunc();
		//只能通过this访问外部函数
		this.externalFunc();//执行call指令
	}
}

contract FunctionTest1{
	
	function externalCall(FunctionTest ft) {
		//external函数供其他合约调用、不能调用ft的内部函数internalFunc();
		ft.externalFunc();
	}
}
```

## 自定义修饰符

修改器是一种合约属性，可被继承，同时还可被派生的**合约重写(override)**。下面我们来看一段示例代码：

```javascript
pragma solidity ^0.4.0;

contract owned {
    function owned() { owner = msg.sender; }
    address owner;

    //自定义修饰符onlyOwner，在当前合约没有使用到，可被继承
    //修饰符修饰的函数在执行函数时会先执行修饰符内容
    //_;表示占位符，函数体会在这个占位符之后执行
    //该修饰符进行调用者判断，如果不是合约持久着即抛出异常，否则正常执行被修饰函数内容
    modifier onlyOwner {
        if (msg.sender != owner)
            throw;
        _;
    }
}

//继承owned合约，同时继承其`onlyOwner`修饰符
contract mortal is owned {
	//onlyOwner修饰符用于判断调用者为合约持有者时执行selfdestruct销毁
    function close() onlyOwner {
        selfdestruct(owner);
    }
}


contract priced {
    //修饰符可以接收参数
    modifier costs(uint price) {
        if (msg.value >= price) {
            _;
        }
    }
}

//继承priced合约、owned合约
contract Register is priced, owned {
    mapping (address => bool) registeredAddresses;
    uint price;

	//构造函数
    function Register(uint initialPrice) { price = initialPrice; }

    // costs修饰符接收参数price
    // payable修饰符表示接收ether，该修饰符时必须的，否则函数自动拒绝调用的ether
    function register() payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }

	//合约持有者才能修改价格
    function changePrice(uint _price) onlyOwner {
        price = _price;
    }
}
```
### 自定义修饰符扩展

```javascript
contract Mutex {
	bool locked;
	modifier noReentrancy() {
		require (!locked, "Reentrant call.");
		//锁的粒度在于执行函数体
		locked=true;
		_;
		locked=false;
	}

	//采用互斥锁概念防止这个f函数体被重入，比如msg.sender.call里不能在调用f()。
	function f() public noReentrancy return(uint) {
		(bool success,) = msg.sender.call("");
		require(success);
		return 1;
	}
}
```

## public状态变量的getter函数
对于下面给出的合约，编译器会生成一个名为 data 的函数， 该函数不会接收任何参数并返回一个 uint ，即状态变量 data 的值。可以在声明时完成状态变量的初始化。

```javascript
pragma solidity ^0.4.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public {
        uint local = c.data();
    }
}
```

getter 函数具有外部可见性。如果在内部访问 getter（即没有 this. ），它被认为一个状态变量。 如果它是外部访问的（即用 this. ），它被认为为一个函数。

```javascript
pragma solidity ^0.4.0;

contract C {
    uint public data;
    function x() public {
        data = 3; // 内部访问
        uint val = this.data(); // 外部访问
    }
}
```

下一个例子稍微复杂一些：

```javascript
pragma solidity ^0.4.0;

contract Complex {
    struct Data {
        uint a;
        bytes3 b;
        mapping (uint => uint) map;
    }
    mapping (uint => mapping(bool => Data[])) public data;
}
```

这将会生成以下形式的函数

```javascript
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
```

请注意，因为没有好的方法来提供映射的键，所以**结构中的映射map被省略**。