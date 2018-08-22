# Solidity-
This is the note of solidity. By RuidongWang
#solidity笔记
[toc]
****
#**基本代码结构**

```
pragma solidity ^0.4.19;//声明solidity的版本号

contract ZombieFactory {
	//声明事件
    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16; //定义无符号整型
    uint dnaModulus = 10 ** dnaDigits; //10的16次方
	//结构体，类同C语言
    struct Zombie {
        string name;
        uint dna;
    }
	//定义公共类型数组，其他合约可以从这个数组读取数据（但不能写入数据）
    Zombie[] public zombies;
	//私有函数
    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna);
    }
	//私有函数
    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }
	//公共函数
    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}

```
***
#数据类型
**状态变量**是被永久地保存在合约中。也就是说它们被写入以太币区块链中. 想象成写入一个数据库。
四则运算同一般语言；
**乘方操作** `a = b ** c`；a的值为b的c次方
***
**`uint`指的是`uint256`指的是256位无符号整型。**
#数组
Solidity 支持两种数组: **静态**数组和**动态**数组；
```
Person[] people; // dynamic Array, we can keep adding to it
```
**记住：状态变量被永久保存在区块链中。所以在你的合约中创建动态数组来保存成结构的数据是非常有意义的。**

**公共数组`Person[] public people;`**其他合约可以从这个数组读取数据（但不能写入数据），这是一种有用的保存公共数据的模式。
数组尾部加入新的元素`array.push()`
***
#定义函数
**函数定义语法**
```
function eatHamburgers(string _name, uint _amount) {

}
```
这是一个名为`eatHamburgers`的函数，它接受两个参数：一个`string`类型的 和 一个 `uint`类型的。现在函数内部还是空的。
**注:**习惯上函数里的变量都是以( _ )开头 (但不是硬性规定) 以区别全局变量。
函数定义
```
eatHamburgers("vitalik", 100);
```
***
#私有/公共函数
Solidity 定义的函数的属性默认为**公共**。 这就意味着任何一方 (或其它合约) 都可以调用你合约里的函数。
```
uint[] numbers;
function _addToArray(uint _number) private {
  numbers.push(_number);
}
```
**私有函数命名用( _ )起始。**

**函数返回值**
```
string greeting = "What's up dog";
function sayHello() public returns (string) {
  return greeting;
}
```
把函数定义为 view, 意味着它只能读取数据不能更改数据:
```
function sayHello() public view returns (string) {
}
```
Solidity 还支持 pure 函数, 表明这个函数甚至都不访问应用里的数据
```
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```
这个函数甚至都不读取应用里的状态---它的返回值完全取决于它的输入参数，在这种情况下我们把函数定义为 pure.

**注:**可能很难记住何时把函数标记为 pure/view。 幸运的是， Solidity 编辑器会给出提示，提醒你使用这些修饰符。
***
#Keccak256 和 类型转换
**Keccak256**
Ethereum 内部有一个散列函数keccak256，它用了SHA3版本。一个散列函数基本上就是把一个字符串转换为一个256位的16进制数字。字符串的一个微小变化会引起散列数据极大变化。
`keccak256("aaaab");`
**类型转换方法**
例：`uint8(b)`
***
#事件
**事件**是合约和区块链通讯的一种机制。你的前端应用“监听”某些事件，并做出反应。
```
// 这里建立事件
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  //触发事件，通知app
  IntegersAdded(_x, _y, result);
  return result;
}
```
你的 app 前端可以监听这个事件。JavaScript 实现如下:
```
YourContract.IntegersAdded(function(error, result) { 
  // 干些事
}
```
***
#映射（Mapping）和地址（Address）
##Addresses （地址）
**地址属于特定用户（或智能合约）**

##Mapping（映射）
**映射**是另一种在 Solidity 中存储有组织数据的方法。
```
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
mapping (address => uint) public accountBalance;
//或者可以用来通过userId 存储/查找的用户名
mapping (uint => string) userIdToName;
```
映射本质上是存储和查找数据所用的键-值对。在第一个例子中，键是一个`address`，值是一个`uint`，在第二个例子中，键是一个`uint`，值是一个`string`。
***
#Msg.sender
在 Solidity 中，有一些全局变量可以被所有函数调用。 其中一个就是`msg.sender`，它指的是当前调用者（或智能合约）的`address`。
```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下
  favoriteNumber[msg.sender] = _myNumber;
  // 存储数据至映射的方法和将数据存储在数组相似
}

function whatIsMyNumber() public view returns (uint) {
  // 拿到存储在调用者地址名下的值
  // 若调用者还没调用 setMyNumber， 则值为 `0`
  return favoriteNumber[msg.sender];
}
```
在这个小小的例子中，任何人都可以调用`setMyNumber`在我们的合约中存下一个`uint`并且与他们的地址相绑定。 然后，他们调用`whatIsMyNumber`就会返回他们存储的`uint`。
***
#Require
`require`使得函数在执行过程中，当不满足某些条件时抛出错误，并停止执行：
```
function sayHiToVitalik(string _name) public returns (string) {
  // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
  // (敲黑板: Solidity 并不支持原生的字符串比较, 我们只能通过比较
  // 两字符串的 keccak256 哈希值来进行判断)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 如果返回 true, 运行如下语句
  return "Hi!";
}
```
如果你这样调用函数 `sayHiToVitalik（“Vitalik”）` ,它会返回“Hi！”。而如果调用的时候使用了其他参数，它则会抛出错误并停止执行。

因此，在调用一个函数之前，用 `require`验证前置条件是非常有必要的。
(**注by Wang**：这就类似于在C语言中检查函数参数的条件是否满足。)
***
#继承（is）
**合约`inheritance`(继承)**
```
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```
由于`BabyDoge`是从`Dog`那里`inherits`（继承)过来的。 这意味着当你编译和部署了`BabyDoge`，它将可以访问`catchphrase()`和`anotherCatchphrase()`和其他我们在`Doge`中定义的其他公共函数。

这可以用于逻辑继承（比如表达子类的时候，`Cat`是一种`Animal`）。 但也可以简单地将类似的逻辑组合到不同的合约中以组织代码。
***
#引入（Import）
（注by Wang：这就类似于C语言中的头文件导入，多文件模式）
在 Solidity 中，当你有多个文件并且想把一个文件导入另一个文件时，可以使用`import`语句：
```
import "./someothercontract.sol";//导入的文件名与路径
contract newContract is SomeOtherContract {
}
```
***
#Storage与Memory
在Solidity 中，有两个地方可以存储变量 ——`storage`或`memory`。
`Storage`变量是指永久存储在区块链中的变量。`Memory`变量则是临时的，当外部函数对某合约调用完成时，内存型变量即被移除。 你可以把它想象成存储在你电脑的硬盘或是RAM中数据的关系。

大多数时候你都用不到这些关键字，默认情况下 Solidity 会自动处理它们。 
**状态变量（在函数之外声明的变量）**默认为“存储”形式，并永久写入区块链；
而在**函数内部声明**的变量是`memory`型的，它们函数调用结束后消失。

需要手动声明存储类型，主要用于处理函数内的**结构体**和**数组**时：
```
contract SandwichFactory {
    struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 看上去很直接，不过 Solidity 将会给出警告
    // 告诉你应该明确在这里定义 `storage` 或者 `memory`。

    // 所以你应该明确定义 `storage`:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
    // 在存储里，另外...
    mySandwich.status = "Eaten!";
    // ...这将永久把 `sandwiches[_index]` 变为区块链上的存储

    // 如果你只想要一个副本，可以使用`memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...这样 `anotherSandwich` 就仅仅是一个内存里的副本了
    // 另外
    anotherSandwich.status = "Eaten!";
    // ...将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
    // 不过你可以这样做:
    sandwiches[_index + 1] = anotherSandwich;
    // ...如果你想把副本的改动保存回区块链存储
  }
}
```
***
#函数可见性
##internal 和 external
除`public`和`private`属性之外，Solidity 还使用了另外两个描述函数可见性的修饰词：`internal`（内部） 和`external`（外部）。

`internal`和`private`类似，不过， 如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数。(子合约可以)
`external`与`public`类似，只不过这些函数只能在合约之外调用 - 它们**不能被合约内的其他函数调用**。稍后我们将讨论什么时候使用`external`和`public`。

声明函数`internal`或`external`类型的语法，与声明`private`和`public`类 型相同：
```
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // 因为eat() 是internal 的，所以我们能在这里调用
    eat();
  }
}
```

***
#与其他合约的交互
如果我们的合约需要和区块链上的其他的合约会话，则需先定义一个`interface`(接口)。
先举一个简单的栗子。 假设在区块链上有这么一个合约：
```
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```
这是个很简单的合约，您可以用它存储自己的幸运号码，并将其与您的以太坊地址关联。 这样其他人就可以通过您的地址查找您的幸运号码了。

现在假设我们有一个外部合约，使用`getNum`函数可读取其中的数据。

首先，我们定义 LuckyNumber 合约的**interface**：
```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```
请注意，这个过程虽然看起来像在定义一个合约，但其实内里不同：

首先，我们只**声明了要与之交互的函数**—— 在本例中为`getNum`—— 在其中我们没有使用到任何其他的函数或状态变量。

其次，我们并**没有使用大括号（`{`和`}`）定义函数体**，我们**单单用分号（`;`）结束了函数声明**。这使它看起来像一个合约框架。

编译器就是靠这些特征认出它是一个接口的。

在我们的 app 代码中使用这个接口，合约就知道其他合约的函数是怎样的，应该如何调用，以及可期待什么类型的返回值。
***
#使用接口
继续前面`NumberInterface`的例子，我们既然将接口定义为：
```
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```
我们可以在合约中这样使用：
```
contract MyContract {
  address NumberInterfaceAddress = 0xab38...;
  // ^ 这是FavoriteNumber合约在以太坊上的地址
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 现在变量 `numberContract` 指向另一个合约对象

  function someFunction() public {
    // 现在我们可以调用在那个合约中声明的 `getNum`函数:
    uint num = numberContract.getNum(msg.sender);
    // ...在这儿使用 `num`变量做些什么
  }
}
```
通过这种方式，只要将您合约的可见性设置为`public`(公共)或`external`(外部)，它们就可以与以太坊区块链上的任何其他合约进行交互。
***
#多返回值处理
多返回值处理例子：
```
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 这样来做批量赋值:
  (a, b, c) = multipleReturns();
}

// 或者如果我们只想返回其中一个变量:
function getLastReturnValue() external {
  uint c;
  // 可以对其他字段留空:
  (,,c) = multipleReturns();
}
```
(**注by Wang:** 获取全部返回值时，使用()依次获取，获取部分返回值时，不需要的留空)
***
#if 语句
`if`语句的语法在Solidity中，与在 JavaScript 中差不多：
```
function eatBLT(string sandwich) public {
  // 看清楚了，当我们比较字符串的时候，需要比较他们的 keccak256 哈希码
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```
***
#第三课
***
#智能协议的永固性
到现在为止，我们讲的 Solidity 和其他语言没有质的区别，它长得也很像 JavaScript.

但是，在有几点以太坊上的 DApp 跟普通的应用程序有着天壤之别。

第一个例子，在你把智能协议传上以太坊之后，它就变得**不可更改**, 这种永固性意味着你的代码永远不能被调整或更新。

你编译的程序会一直，永久的，不可更改的，存在以太网上。这就是Solidity代码的安全性如此重要的一个原因。如果你的智能协议有任何漏洞，即使你发现了也无法补救。你只能让你的用户们放弃这个智能协议，然后转移到一个新的修复后的合约上。

但这恰好也是智能合约的一大优势。 代码说明一切。 如果你去读智能合约的代码，并验证它，你会发现， 一旦函数被定义下来，每一次的运行，程序都会严格遵照函数中原有的代码逻辑一丝不苟地执行，完全不用担心函数被人篡改而得到意外的结果。

##外部依赖关系
在第2课中，我们将加密小猫（CryptoKitties）合约的地址硬编码到DApp中去了。有没有想过，如果加密小猫出了点问题，比方说，集体消失了会怎么样？ 虽然这种事情几乎不可能发生，但是，如果小猫没了，我们的 DApp 也会随之失效 -- 因为我们在 DApp 的代码中用“硬编码”的方式指定了加密小猫的地址，如果这个根据地址找不到小猫，我们的僵尸也就吃不到小猫了，而按照前面的描述，我们却没法修改合约去应付这个变化！

因此，我们不能硬编码，而要采用“函数”，以便于 DApp 的关键部分可以以参数形式修改。

比方说，我们不再一开始就把猎物地址给写入代码，而是写个函数 setKittyContractAddress, 运行时再设定猎物的地址，这样我们就可以随时去锁定新的猎物，也不用担心加密小猫集体消失了。
***
#Ownable Contracts
我们确实是希望这个地址能够在合约中修改，但我可没说让每个人去改它呀。

要对付这样的情况，通常的做法是指定合约的“所有权” - 就是说，给它指定一个主人、，只有主人对它享有特权。

#OpenZeppelin库的Ownable 合约
下面是一个`Ownable`合约的例子： 来自 OpenZeppelin Solidity 库的`Ownable`合约。 OpenZeppelin 是主打安保和社区审查的智能合约库，您可以在自己的 DApps中引用。等把这一课学完，您不要催我们发布下一课，最好利用这个时间把 OpenZeppelin 的网站看看，保管您会学到很多东西！

把楼下这个合约读读通，是不是还有些没见过代码？别担心，我们随后会解释。
```
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```
下面有没有您没学过的东东？

构造函数：`function Ownable()`是一个**constructor**(构造函数)，构造函数不是必须的，它与合约同名，构造函数一生中唯一的一次执行，就是在合约最初被创建的时候。

函数修饰符：`modifier onlyOwner()`。 修饰符跟函数很类似，不过是用来修饰其他已有函数用的， 在其他语句执行前，为它检查下先验条件。 在这个例子中，我们就可以写个修饰符`onlyOwner`检查下调用者，确保只有合约的主人才能运行本函数。我们下一章中会详细讲述修饰符，以及那个奇怪的_;。

`indexed`关键字：别担心，我们还用不到它。

所以`Ownable`合约基本都会这么干：

合约创建，构造函数先行，将其`owner`设置为`msg.sender`（其部署者）

为它加上一个修饰符`onlyOwner`，它会限制陌生人的访问，将访问某些函数的权限锁定在`owner`上。

允许将合约所有权转让给他人。

`onlyOwner`简直人见人爱，大多数人开发自己的 Solidity DApps，都是从复制/粘贴`Ownable`开始的，从它再继承出的子类，并在之上进行功能开发。

既然我们想把`setKittyContractAddress`限制为`onlyOwner`，我们也要做同样的事情。
***
#onlyOwner 函数修饰符
函数修饰符看起来跟函数没什么不同，不过关键字`modifier`告诉编译器，这是个`modifier`(修饰符)，而不是个`function`(函数)。它不能像函数那样被直接调用，只能被添加到函数定义的末尾，用以改变函数的行为。

咱们仔细读读 onlyOwner:
```
/**
 * @dev 调用者不是‘主人’，就会抛出异常
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
onlyOwner 函数修饰符是这么用的：

contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  //注意！ `onlyOwner`上场 :
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```
注意`likeABoss`函数上的`onlyOwner`修饰符。 当你调用`likeABoss`时，首先执行 onlyOwner 中的代码， 执行到`onlyOwner`中的`_;`语句时，程序再返回并执行`likeABoss`中的代码。

可见，尽管函数修饰符也可以应用到各种场合，但最常见的还是放在函数执行之前添加快速的`require`检查。

因为给函数添加了修饰符`onlyOwner`，使得唯有合约的主人（也就是部署者）才能调用它。

>注意：主人对合约享有的特权当然是正当的，不过也可能被恶意使用。比如，万一，主人添加了个后门，允许他偷走别人的僵尸呢？

> 所以非常重要的是，部署在以太坊上的 DApp，并不能保证它真正做到去中心，你需要阅读并理解它的源代码，才能防止其中没有被部署者恶意植入后门；作为开发人员，如何做到既要给自己留下修复bug 的余地，又要尽量地放权给使用者，以便让他们放心你，从而愿意把数据放在你的 DApp 中，这确实需要个微妙的平衡。

***
#Gas
厉害！现在我们懂了如何在禁止第三方修改我们的合约的同时，留个后门给咱们自己去修改。

让我们来看另一种使得 Solidity 编程语言与众不同的特征：

##Gas - 驱动以太坊DApps的能源
在 Solidity 中，你的用户想要每次执行你的 DApp 都需要支付一定的**gas**，**gas**可以用**以太币**购买，因此，用户每次跑 DApp 都得**花费**以太币。

一个 DApp 收取多少 gas 取决于**功能逻辑的复杂程度**。每个操作背后，都在计算完成这个操作所需要的计算资源，（比如，存储数据就比做个加法运算贵得多）， 一次操作所需要花费的 gas 等于这个操作背后的所有运算花销的总和。

由于运行你的程序需要花费用户的真金白银，在以太坊中代码的编程语言，比其他任何编程语言都更强调优化。同样的功能，使用笨拙的代码开发的程序，比起经过**精巧优化**的代码来，运行花费更高，这显然会给成千上万的用户带来大量不必要的开销。

##为什么要用 gas 来驱动？
**以太坊就像一个巨大、缓慢、但非常安全的电脑。当你运行一个程序的时候，网络上的每一个节点都在进行相同的运算，以验证它的输出 —— 这就是所谓的“中心化”** 由于数以千计的节点同时在验证着每个功能的运行，这可以确保它的数据不会被被监控，或者被刻意修改。

可能会有用户用无限循环堵塞网络，抑或用密集运算来占用大量的网络资源，为了防止这种事情的发生，以太坊的创建者为以太坊上的资源制定了价格，想要在以太坊上运算或者存储，你需要先付费。

>注意：如果你使用**侧链**，倒是不一定需要付费，比如咱们在 Loom Network 上构建的 CryptoZombies 就免费。你不会想要在以太坊主网上玩儿“魔兽世界”吧？ - 所需要的 gas 可能会买到你破产。但是你可以找个算法理念不同的侧链来玩它。我们将在以后的课程中咱们会讨论到，什么样的 DApp 应该部署在太坊主链上，什么又最好放在侧链。

##省 gas 的招数：结构封装 （Struct packing）
在第1课中，我们提到除了基本版的`uint`外，还有其他变种`uint：uint8，uint16，uint32`等。

通常情况下我们不会考虑使用`uint`变种，因为无论如何定义`uint`的大小，Solidity 为它保留256位的存储空间。例如，使用`uint8`而不是`uint（uint256）`不会为你节省任何**gas**。

除非，把`uint`绑定到`struct`里面。

如果一个`struct`中有多个`uint`，则尽可能使用较小的`uint`, Solidity 会将这些`uint`打包在一起，从而占用较少的存储空间。例如：
```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// 因为使用了结构打包，`mini` 比 `normal` 占用的空间更少
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```
所以，当`uint`定义在一个 struct 中的时候，尽量使用最小的整数子类型以节约空间。 并且把同样类型的变量放一起（即在 struct 中将把变量按照类型依次放置），这样 Solidity 可以将存储空间最小化。例如，有两个 struct：

`uint c; uint32 a; uint32 b;` 和 `uint32 a; uint c; uint32 b;`

前者比后者需要的gas更少，因为前者把`uint32`放一起了。
***
#时间单位
Solidity 使用自己的本地时间单位。

变量`now`将返回当前的unix时间戳（自1970年1月1日以来经过的秒数）。我写这句话时 unix 时间是 `1515527488`。

注意：Unix时间传统用一个32位的整数进行存储。这会导致“2038年”问题，当这个32位的unix时间戳不够用，产生溢出，使用这个时间的遗留系统就麻烦了。所以，如果我们想让我们的 DApp 跑够20年，我们可以使用64位整数表示时间，但为此我们的用户又得支付更多的 gas。真是个两难的设计啊！

Solidity 还包含`秒(seconds)，分钟(minutes)，小时(hours)，天(days)，周(weeks) 和 年(years)`等时间单位。它们都会转换成对应的秒数放入`uint`中。所以`1分钟`就是`60`，`1小时`是`3600`（60秒×60分钟），`1天`是`86400`（24小时×60分钟×60秒），以此类推。

下面是一些使用时间单位的实用案例：
```
uint lastUpdated;

// 将‘上次更新时间’ 设置为 ‘现在’
function updateTimestamp() public {
  lastUpdated = now;
}

// 如果到上次`updateTimestamp` 超过5分钟，返回 'true'
// 不到5分钟返回 'false'
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```
***
#将结构体作为参数传入
由于结构体的存储指针可以以参数的方式传递给一个`private`或`internal`的函数，因此结构体可以在多个函数之间相互传递。

遵循这样的语法：
```
function _doStuff(Zombie storage _zombie) internal {
  // do stuff with _zombie
}
```
这样我们可以将某僵尸的引用直接传递给一个函数，而不用是通过参数传入僵尸ID后，函数再依据ID去查找。
***
#公有函数的安全性
现在来修改`feedAndMultiply`，实现冷却周期。

回顾一下这个函数，前一课上我们将其可见性设置为`public`。你必须仔细地检查所有声明为`public`和 `external`的函数，一个个排除用户滥用它们的可能，谨防安全漏洞。请记住，如果这些函数没有类似 `onlyOwner`这样的函数修饰符，用户能利用各种可能的参数去调用它们。

检查完这个函数，用户就可以直接调用这个它，并传入他们所希望的`_targetDna`或`species`。打个游戏还得遵循这么多的规则，还能不能愉快地玩耍啊！

仔细观察，这个函数只需被`feedOnKitty()`调用，因此，想要防止漏洞，最简单的方法就是设其可见性为 `internal`。
***
#带参数的函数修饰符
之前我们已经读过一个简单的函数修饰符了：onlyOwner。函数修饰符也可以带参数。例如：
```
// 存储用户年龄的映射
mapping (uint => uint) public age;

// 限定用户年龄的修饰符
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// 必须年满16周岁才允许开车 (至少在美国是这样的).
// 我们可以用如下参数调用`olderThan` 修饰符:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // 其余的程序逻辑
}
```
看到了吧，`olderThan`修饰符可以像函数一样接收参数，是“宿主”函数`driveCar`把参数传递给它的修饰符的。
***
#“view” 函数不花 “gas”
当玩家从外部调用一个view函数，是不需要支付一分 gas 的。

这是因为`view`函数不会真正改变区块链上的任何数据 - 它们只是读取。因此用`view`标记一个函数，意味着告诉 web3.js，运行这个函数只需要查询你的本地以太坊节点，而不需要在区块链上创建一个事务（事务需要运行在每个节点上，因此花费 gas）。

稍后我们将介绍如何在自己的节点上设置 web3.js。但现在，你关键是要记住，在所能只读的函数上标记上表示“只读”的“`external view `声明，就能为你的玩家减少在 DApp 中 gas 用量。

>注意：如果一个 view 函数在另一个函数的内部被调用，而调用函数与 view 函数的不属于同一个合约，也会产生调用成本。这是因为如果主调函数在以太坊创建了一个事务，它仍然需要逐个节点去验证。所以标记为 view 的函数只有在外部调用时才是免费的。

***
#存储非常昂贵
Solidity 使用`storage`(存储)是相当昂贵的，”写入“操作尤其贵。

这是因为，无论是写入还是更改一段数据， 这都将永久性地写入区块链。”永久性“啊！需要在全球数千个节点的硬盘上存入这些数据，随着区块链的增长，拷贝份数更多，存储量也就越大。这是需要成本的！

为了降低成本，不到万不得已，避免将数据写入存储。这也会导致效率低下的编程逻辑 - 比如每次调用一个函数，都需要在 memory(内存) 中重建一个数组，而不是简单地将上次计算的数组给存储下来以便快速查找。

在大多数编程语言中，遍历大数据集合都是昂贵的。但是在 Solidity 中，使用一个标记了`external view`的函数，遍历比 storage 要便宜太多，因为 view 函数不会产生任何花销。 （gas可是真金白银啊！）。

我们将在下一章讨论for循环，现在我们来看一下看如何如何在内存中声明数组。

在内存中声明数组
在数组后面加上 memory关键字， 表明这个数组是仅仅在内存中创建，不需要写入外部存储，并且在函数调用结束时它就解散了。与在程序结束时把数据保存进 storage 的做法相比，内存运算可以大大节省gas开销 -- 把这数组放在view里用，完全不用花钱。

以下是申明一个内存数组的例子：
```
function getArray() external pure returns(uint[]) {
  // 初始化一个长度为3的内存数组
  uint[] memory values = new uint[](3);
  // 赋值
  values.push(1);
  values.push(2);
  values.push(3);
  // 返回数组
  return values;
}
```
这个小例子展示了一些语法规则，下一章中，我们将通过一个实际用例，展示它和 for 循环结合的做法。

>注意：内存数组 必须 用长度参数（在本例中为3）创建。目前不支持 `array.push()`之类的方法调整数组大小，在未来的版本可能会支持长度修改。

***
#使用`for`循环
`for`循环的语法在 Solidity 和 JavaScript 中类似。

来看一个创建偶数数组的例子：
```
function getEvens() pure external returns(uint[]) {
  uint[] memory evens = new uint[](5);
  // 在新数组中记录序列号
  uint counter = 0;
  // 在循环从1迭代到10：
  for (uint i = 1; i <= 10; i++) {
    // 如果 `i` 是偶数...
    if (i % 2 == 0) {
      // 把它加入偶数数组
      evens[counter] = i;
      //索引加一， 指向下一个空的‘even’
      counter++;
    }
  }
  return evens;
}
```
这个函数将返回一个形为`[2,4,6,8,10]`的数组。
***
#第四课
***
#可支付
截至目前，我们只接触到很少的**函数修饰符**。 要记住所有的东西很难，所以我们来个概览：

我们有决定函数何时和被谁调用的可见性修饰符:`private`意味着它只能被合约内部调用；`internal`就像`private`但是也能被继承的合约调用；`external`只能从合约外部调用；最后`public`可以在任何地方调用，不管是内部还是外部。

我们也有状态修饰符， 告诉我们函数如何和区块链交互:`view`告诉我们运行这个函数不会更改和保存任何数据；`pure`告诉我们这个函数不但不会往区块链写数据，它甚至不从区块链读取数据。这两种在被从合约外部调用的时候都不花费任何gas（但是它们在被内部其他函数调用的时候将会耗费gas）。

然后我们有了自定义的`modifiers`，例如在第三课学习的:`onlyOwner`和`aboveLevel`。 对于这些修饰符我们可以自定义其对函数的约束逻辑。

这些修饰符可以同时作用于一个函数定义上：
```
function test() external view onlyOwner anotherModifier { /* ... */ }
```
在这一章，我们来学习一个新的修饰符 payable.

##`payable`修饰符
`payable`方法是让 Solidity 和以太坊变得如此酷的一部分 —— 它们是一种可以接收以太的特殊函数。

先放一下。当你在调用一个普通网站服务器上的API函数的时候，你无法用你的函数传送美元——你也不能传送比特币。

但是在以太坊中， 因为钱 (以太), 数据 (事务负载)， 以及合约代码本身都存在于以太坊。你可以在同时调用函数 并付钱给另外一个合约。

这就允许出现很多有趣的逻辑， 比如向一个合约要求支付一定的钱来运行一个函数。

###来看个例子
```
contract OnlineStore {
  function buySomething() external payable {
    // 检查以确定0.001以太发送出去来运行函数:
    require(msg.value == 0.001 ether);
    // 如果为真，一些用来向函数调用者发送数字内容的逻辑
    transferThing(msg.sender);
  }
}
```
在这里，`msg.value`是一种可以查看向合约发送了多少以太的方法，另外`ether`是一个內建单元。

这里发生的事是，一些人会从 web3.js 调用这个函数 (从DApp的前端)， 像这样 :
```
// 假设 `OnlineStore` 在以太坊上指向你的合约:
OnlineStore.buySomething().send(from: web3.eth.defaultAccount, 
                                value: web3.utils.toWei(0.001))
```
注意这个`value`字段， JavaScript 调用来指定发送多少(0.001)`以太`。如果把事务想象成一个信封，你发送到函数的参数就是信的内容。 添加一个`value`很像在信封里面放钱 —— 信件内容和钱同时发送给了接收者。

>注意： 如果一个函数没标记为`payable`， 而你尝试利用上面的方法发送以太，函数将拒绝你的事务。

***
#提现
在上一章，我们学习了如何向合约发送以太，那么在发送之后会发生什么呢？

在你发送以太之后，它将被存储进以合约的以太坊账户中， 并冻结在哪里 —— 除非你添加一个函数来从合约中把以太提现。

你可以写一个函数来从合约中提现以太，类似这样：
```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```
注意我们使用`Ownable`合约中的`owner`和`onlyOwner`，假定它已经被引入了。

你可以通过`transfer`函数向一个地址发送以太， 然后`this.balance`将返回当前合约存储了多少以太。 所以如果100个用户每人向我们支付1以太， this.balance 将是100以太。

你可以通过`transfer`向任何以太坊地址付钱。 比如，你可以有一个函数在`msg.sender`超额付款的时候给他们退钱：
```
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```
或者在一个有卖家和卖家的合约中， 你可以把卖家的地址存储起来， 当有人买了它的东西的时候，把买家支付的钱发送给它`seller.transfer(msg.value)`。

有很多例子来展示什么让以太坊编程如此之酷 —— 你可以拥有一个不被任何人控制的去中心化市场。
***
#随机数
##用 keccak256 来制造随机数。
Solidity 中最好的随机数生成器是 keccak256 哈希函数.

我们可以这样来生成一些随机数
```
// 生成一个0到100的随机数:
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```
这个方法首先拿到`now`的时间戳、`msg.sender`、 以及一个自增数`nonce`（一个仅会被使用一次的数，这样我们就不会对相同的输入值调用一次以上哈希函数了）。

然后利用`keccak`把输入的值转变为一个哈希值, 再将哈希值转换为`uint`, 然后利用 % 100 来取最后两位, 就生成了一个0到100之间随机数了。

##这个方法很容易被不诚实的节点攻击(弊端)
在以太坊上, 当你在和一个合约上调用函数的时候, 你会把它广播给一个节点或者在网络上的 transaction 节点们。 网络上的节点将收集很多事务, 试着成为第一个解决计算密集型数学问题的人，作为“工作证明”，然后将“工作证明”(Proof of Work, PoW)和事务一起作为一个 block 发布在网络上。

一旦一个节点解决了一个PoW, 其他节点就会停止尝试解决这个 PoW, 并验证其他节点的事务列表是有效的，然后接受这个节点转而尝试解决下一个节点。

这就让我们的随机数函数变得可利用了

我们假设我们有一个硬币翻转合约——正面你赢双倍钱，反面你输掉所有的钱。假如它使用上面的方法来决定是正面还是反面 (random >= 50 算正面, random < 50 算反面)。

如果我正运行一个节点，我可以 只对我自己的节点 发布一个事务，且不分享它。 我可以运行硬币翻转方法来偷窥我的输赢 — 如果我输了，我就不把这个事务包含进我要解决的下一个区块中去。我可以一直运行这个方法，直到我赢得了硬币翻转并解决了下一个区块，然后获利。

##所以我们该如何在以太坊上安全地生成随机数呢
因为区块链的全部内容对所有参与者来说是透明的， 这就让这个问题变得很难，它的解决方法不在本课程讨论范围，你可以阅读 这个 [StackOverflow 上的讨论](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract) 来获得一些主意。 一个方法是利用 oracle 来访问以太坊区块链之外的随机数函数。


当然， 因为网络上成千上万的以太坊节点都在竞争解决下一个区块，我能成功解决下一个区块的几率非常之低。 这将花费我们巨大的计算资源来开发这个获利方法 — 但是如果奖励异常地高(比如我可以在硬币翻转函数中赢得 1个亿)， 那就很值得去攻击了。

所以尽管这个方法在以太坊上不安全，在实际中，除非我们的随机函数有一大笔钱在上面，你游戏的用户一般是没有足够的资源去攻击的。

因为在这个教程中，我们只是在编写一个简单的游戏来做演示，也没有真正的钱在里面，所以我们决定接受这个不足之处，使用这个简单的随机数生成函数。但是要谨记它是不安全的。
***
#重构通用逻辑
在出现大量代码复用的时候，创建`modifier`（自定义修饰符）将复用代码简化
***
#第五课：ERC721 标准和加密收藏品
***
# 以太坊上的代币
让我们来聊聊**代币**。

如果你对以太坊的世界有一些了解，你很可能听过人们聊到代币——尤其是 ERC20 代币.

一个 代币 在以太坊基本上就是一个遵循一些共同规则的智能合约——即它实现了所有其他代币合约共享的一组标准函数，例如 transfer(address _to, uint256 _value) 和 balanceOf(address _owner).

在智能合约内部，通常有一个映射， mapping(address => uint256) balances，用于追踪每个地址还有多少余额。

所以基本上一个代币只是一个追踪谁拥有多少该代币的合约，和一些可以让那些用户将他们的代币转移到其他地址的函数。

##它为什么重要呢？
由于所有 ERC20 代币共享具有相同名称的同一组函数，它们都可以以相同的方式进行交互。

这意味着如果你构建的应用程序能够与一个 ERC20 代币进行交互，那么它就也能够与任何 ERC20 代币进行交互。 这样一来，将来你就可以轻松地将更多的代币添加到你的应用中，而无需进行自定义编码。 你可以简单地插入新的代币合约地址，然后哗啦，你的应用程序有另一个它可以使用的代币了。

其中一个例子就是交易所。 当交易所添加一个新的 ERC20 代币时，实际上它只需要添加与之对话的另一个智能合约。 用户可以让那个合约将代币发送到交易所的钱包地址，然后交易所可以让合约在用户要求取款时将代币发送回给他们。

交易所只需要实现这种转移逻辑一次，然后当它想要添加一个新的 ERC20 代币时，只需将新的合约地址添加到它的数据库即可。

##其他代币标准
对于像货币一样的代币来说，ERC20 代币非常酷。 但是要在我们僵尸游戏中代表僵尸就并不是特别有用。

首先，僵尸不像货币可以分割 —— 我可以发给你 0.237 以太，但是转移给你 0.237 的僵尸听起来就有些搞笑。

其次，并不是所有僵尸都是平等的。 你的2级僵尸"Steve"完全不能等同于我732级的僵尸"H4XF13LD MORRIS 💯💯😎💯💯"。（你差得远呢，Steve）。

有另一个代币标准更适合如 CryptoZombies 这样的加密收藏品——它们被称为**ERC721代币**.

**ERC721**代币是不能互换的，因为每个代币都被认为是唯一且不可分割的。 你只能以整个单位交易它们，并且每个单位都有唯一的 ID。 这些特性正好让我们的僵尸可以用来交易。

>请注意，使用像 ERC721 这样的标准的优势就是，我们不必在我们的合约中实现拍卖或托管逻辑，这决定了玩家能够如何交易／出售我们的僵尸。 如果我们符合规范，其他人可以为加密可交易的 ERC721 资产搭建一个交易所平台，我们的 ERC721 僵尸将可以在该平台上使用。 所以使用代币标准相较于使用你自己的交易逻辑有明显的好处。

***
#ERC721 标准, 多重继承

##实现一个代币合约（多重继承）
在实现一个代币合约的时候，我们首先要做的是将接口复制到它自己的 Solidity 文件并导入它，`import ./erc721.sol`。 接着，让我们的合约继承它，然后我们用一个函数定义来重写每个方法。

但等一下——`ZombieOwnership`已经继承自`ZombieAttack`了 —— 它如何能够也继承于`ERC721`呢？

幸运的是在Solidity，你的合约可以继承自多个合约，参考如下：
```
contract SatoshiNakamoto is NickSzabo, HalFinney {
  // 啧啧啧，宇宙的奥秘泄露了
}
```
正如你所见，当使用多重继承的时候，你只需要用逗号`,`来隔开几个你想要继承的合约。在上面的例子中，我们的合约继承自`NickSzabo`和`HalFinney`。

##ERC721 标准
让我们来看一看 ERC721 标准：
```
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```
这是我们需要实现的方法列表，我们将在接下来的章节中逐个学习。

虽然看起来很多，但不要被吓到了！我们在这里就是准备带着你一步一步了解它们的。

>注意： ERC721目前是一个 草稿，还没有正式商定的实现。在本教程中，我们使用的是 OpenZeppelin 库中的当前版本，但在未来正式发布之前它可能会有更改。 所以把这 一个 可能的实现当作考虑，但不要把它作为 ERC721 代币的官方标准。

### balanceOf 和 ownerOf
####balanceOf
```
  function balanceOf(address _owner) public view returns (uint256 _balance);
```
这个函数只需要一个传入`address`参数，然后返回这个`address`拥有多少代币。

在我们的例子中，我们的“代币”是僵尸。你还记得在我们 DApp 的哪里存储了一个主人拥有多少只僵尸吗？

####ownerOf
```
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
```
这个函数需要传入一个代币 ID 作为参数 (我们的情况就是一个僵尸 ID)，然后返回该代币拥有者的`address`。

同样的，因为在我们的 DApp 里已经有一个`mapping`(映射) 存储了这个信息，所以对我们来说这个实现非常直接清晰。我们可以只用一行`return`语句来实现这个函数。

>注意：要记得，`uint256`等同于`uint`。我们从课程的开始一直在代码中使用`uint`，但从现在开始我们将在这里用 uint256，因为我们直接从规范中复制粘贴。

####ERC721: 转移标准
注意 ERC721 规范有两种不同的方法来转移代币：
```
function transfer(address _to, uint256 _tokenId) public;
```
```
function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```
第一种方法是代币的拥有者调用`transfer`方法，传入他想转移到的`address`和他想转移的代币的 `_tokenId`。

第二种方法是代币拥有者首先调用`approve`，然后传入与以上相同的参数。接着，该合约会存储谁被允许提取代币，通常存储到一个`mapping (uint256 => address)`里。然后，当有人调用`takeOwnership`时，合约会检查`msg.sender`是否得到拥有者的批准来提取代币，如果是，则将代币转移给他。

你注意到了吗，`transfer`和`takeOwnership`都将包含相同的转移逻辑，只是以相反的顺序。 （一种情况是代币的发送者调用函数；另一种情况是代币的接收者调用它）。

所以我们把这个逻辑抽象成它自己的私有函数`_transfer`，然后由这两个函数来调用它。 这样我们就不用写重复的代码了。

####ERC721: 批准
现在，让我们来实现`approve`。

记住，使用`approve`或者`takeOwnership`的时候，转移有2个步骤：

你，作为所有者，用新主人的`address`和你希望他获取的`_tokenId`来调用`approve`

新主人用`_tokenId`来调用`takeOwnership`，合约会检查确保他获得了批准，然后把代币转移给他。

因为这发生在2个函数的调用中，所以在函数调用之间，我们需要一个数据结构来存储什么人被批准获取什么。

####ERC721: takeOwnership
最后一个函数`takeOwnership`， 应该只是简单地检查以确保`msg.sender`已经被批准来提取这个代币或者僵尸。若确认，就调用`_transfer`；
***
#预防溢出
恭喜你，我们完成了 ERC721 的实现。

并不是很复杂，对吧？很多类似的以太坊概念，当你只听人们谈论它们的时候，会觉得很复杂。所以最简单的理解方式就是你自己来实现它。

不过要记住那只是最简单的实现。还有很多的特性我们也许想加入到我们的实现中来，比如一些额外的检查，来确保用户不会不小心把他们的僵尸转移给`0`地址（这被称作 “烧币”, 基本上就是把代币转移到一个谁也没有私钥的地址，让这个代币永远也无法恢复）。 或者在 DApp 中加入一些基本的拍卖逻辑。（你能想出一些实现的方法么？）

但是为了让我们的课程不至于离题太远，所以我们只专注于一些基础实现。如果你想学习一些更深层次的实现，可以在这个教程结束后，去看看 OpenZeppelin 的 ERC721 合约。

##合约安全增强: 溢出和下溢
我们将来学习你在编写智能合约的时候需要注意的一个主要的安全特性：防止溢出和下溢。

什么是**溢出(overflow)**?

假设我们有一个`uint8`, 只能存储8 bit数据。这意味着我们能存储的最大数字就是二进制`11111111`(或者说十进制的2 ^ 8 - 1 = 255).

来看看下面的代码。最后 number 将会是什么值？
```
uint8 number = 255;
number++;
```
在这个例子中，我们导致了溢出------虽然我们加了1， 但是`number`出乎意料地等于 0了。 (如果你给二进制`11111111`加`1`, 它将被重置为`00000000`，就像钟表从`23:59`走向`00:00`)。

下溢(`underflow`)也类似，如果你从一个等于`0`的`uint8`减去`1`, 它将变成`255`(因为`uint`是无符号的，其不能等于负数)。

虽然我们在这里不使用 uint8，而且每次给一个 uint256 加 1 也不太可能溢出 (2^256 真的是一个很大的数了)，在我们的合约中添加一些保护机制依然是非常有必要的，以防我们的 DApp 以后出现什么异常情况。

##使用 SafeMath
为了防止这些情况，OpenZeppelin 建立了一个叫做 SafeMath 的**库(library)**，默认情况下可以防止这些问题。

不过在我们使用之前…… 什么叫做库?

一个**库**是 Solidity 中一种特殊的合约。其中一个有用的功能是给原始数据类型增加一些方法。

比如，使用 SafeMath 库的时候，我们将使用`using SafeMath for uint256`这样的语法。 SafeMath 库有四个方法 —`add`，`sub`，`mul`， 以及`div`。现在我们可以这样来让`uint256`调用这些方法：
```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```
我们将在下一章来学习这些方法，不过现在我们先将 SafeMath 库添加进我们的合约。

##SafeMath 第二部分
来看看 SafeMath 的部分代码:
```
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```
首先我们有了`library`关键字 — 库和`合约`很相似，但是又有一些不同。 就我们的目的而言，库允许我们使用`using`关键字，它可以自动把库的所有方法添加给一个数据类型：
```
using SafeMath for uint;
// 这下我们可以为任何 uint 调用这些方法了
uint test = 2;
test = test.mul(3); // test 等于 6 了
test = test.add(5); // test 等于 11 了
```
注意`mul`和`add`其实都需要两个参数。 在我们声明了`using SafeMath for uint`后，我们用来调用这些方法的`uint`就自动被作为第一个参数传递进去了(在此例中就是`test`)

我们来看看 add 的源代码看 SafeMath 做了什么:
```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}
```
基本上`add`只是像 + 一样对两个`uint`相加， 但是它用一个`assert`语句来确保结果大于 a。这样就防止了溢出。

**`assert`和`require`相似，若结果为否它就会抛出错误。`assert`和`require`区别在于，`require`若失败则会返还给用户剩下的gas，`assert`则不会。**所以大部分情况下，你写代码的时候会比较喜欢 `require`，`assert`只在代码可能出现严重错误的时候使用，比如`uint`溢出。

所以简而言之，`SafeMath`的`add`，`sub`，`mul`， 和`div`方法只做简单的四则运算，然后在发生溢出或下溢的时候抛出错误。

###在我们的代码里使用 SafeMath。
为了防止溢出和下溢，我们可以在我们的代码里找`+`，`-`，`*`，或`/`，然后替换为`add`,`sub`,`mul`,`div`.

比如，与其这样做:
```
myUint++;
```
我们这样做：
```
myUint = myUint.add(1);
```

##SafeMath 第三部分
太好了，这下我们的 ERC721 实现不会有溢出或者下溢了。

回头看看我们在之前课程写的代码，还有其他几个地方也有可能导致溢出或下溢。

比如， 在 ZombieAttack 里面我们有：
```
myZombie.winCount++;
myZombie.level++;
enemyZombie.lossCount++;
```
我们同样应该在这些地方防止溢出。（通常情况下，总是使用 SafeMath 而不是普通数学运算是个好主意，也许在以后 Solidity 的新版本里这点会被默认实现，但是现在我们得自己在代码里实现这些额外的安全措施）。

不过我们遇到个小问题 —`winCount`和`lossCount`是`uint16`， 而` level`是`uint32`。 所以如果我们用这些作为参数传入 SafeMath 的`add`方法。 它实际上并不会防止溢出，因为它会把这些变量都转换成 `uint256`:
```
function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}

// 如果我们在`uint8` 上调用 `.add`。它将会被转换成 `uint256`.
// 所以它不会在 2^8 时溢出，因为 256 是一个有效的 `uint256`.
```
这就意味着，我们需要再实现两个库来防止`uint16`和`uint32`溢出或下溢。我们可以将其命名为 `SafeMath16`和`SafeMath32`。

代码将和 SafeMath 完全相同，除了所有的`uint256`实例都将被替换成`uint32`或`uint16`。

我们已经将这些代码帮你写好了，打开`safemath.sol`合约看看代码吧。

现在我们需要在 ZombieFactory 里使用它们。
***
#注释
僵尸游戏的 Solidity 代码终于完成啦。

在以后的课程中，我们将学习如何将游戏部署到以太坊，以及如何和 Web3.js 交互。

不过在你离开第五课之前，我们来谈谈如何 给你的代码添加注释.

注释语法
Solidity 里的注释和 JavaScript 相同。在我们的课程中你已经看到了不少单行注释了：
```
// 这是一个单行注释，可以理解为给自己或者别人看的笔记
```
只要在任何地方添加一个 // 就意味着你在注释。如此简单所以你应该经常这么做。

不过我们也知道你的想法：有时候单行注释是不够的。毕竟你生来话痨。

所以我们有了多行注释：
```
contract CryptoZombies { 
  /* 这是一个多行注释。我想对所有花时间来尝试这个编程课程的人说声谢谢。
  它是免费的，并将永远免费。但是我们依然倾注了我们的心血来让它变得更好。

   要知道这依然只是区块链开发的开始而已，虽然我们已经走了很远，
   仍然有很多种方式来让我们的社区变得更好。
   如果我们在哪个地方出了错，欢迎在我们的 github 提交 PR 或者 issue 来帮助我们改进：
    https://github.com/loomnetwork/cryptozombie-lessons

    或者，如果你有任何的想法、建议甚至仅仅想和我们打声招呼，欢迎来我们的电报群：
     https://t.me/loomnetworkcn
  */
}
```
特别是，最好为你合约中每个方法添加注释来解释它的预期行为。这样其他开发者（或者你自己，在6个月以后再回到这个项目中）可以很快地理解你的代码而不需要逐行阅读所有代码。

Solidity 社区所使用的一个标准是使用一种被称作 natspec 的格式，看起来像这样：
```
/// @title 一个简单的基础运算合约
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice 现在，这个合约只添加一个乘法
contract Math {
  /// @notice 两个数相乘
  /// @param x 第一个 uint
  /// @param y  第二个 uint
  /// @return z  (x * y) 的结果
  /// @dev 现在这个方法不检查溢出
  function multiply(uint x, uint y) returns (uint z) {
    // 这只是个普通的注释，不会被 natspec 解释
    z = x * y;
  }
}
```
`@title`（标题）和`@author`（作者）很直接了.

`@notice`（须知）向**用户**解释这个方法或者合约是做什么的。`@dev`（开发者）是向开发者解释更多的细节。

`@param`（参数）和`@return`（返回）用来描述这个方法需要传入什么参数以及返回什么值。

注意你并不需要每次都用上所有的标签，它们都是可选的。不过最少，写下一个`@dev`注释来解释每个方法是做什么的。
***
***
***
***
***
***
