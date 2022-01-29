区块链技术一大特点就是去中心化，由此衍生出一种基于区块链技术的云平台，在这些平台上你可以发布并执行自己的代码。与传统云计算平台例如亚马逊，阿里云不同的是，你在以太坊发布的代码不会存储在某一台主机上，不能像运行在阿里云，腾讯云那样的平台上的程序那样，你能把代码托管到一个具体对象，然后还能针对性的调试和修改，在以太坊发布代码后，二进制代码会存储在无数个独立的主机上，因此代码一旦发布就很难修改，如果你要对代码中的某些数据进行改动，那么以太坊需要广播给网络里面所有主机，由于数据修改非常麻烦，因此你发布的代码想要做变更时，你需要支付一定的代价。

在以太坊上进行开发时，每个账号开始都有固定的虚拟资产叫”gas”，如果你发布后的代码想要进行改动时，会导致网络里所有存储了你二进制数据的主机进行改动，因此你需要支付一定的gas，这些gas是可以变现的，就类似于比特币，因此你要开发智能合约，衡量你代码水平的标准就是发布后你要尽可能少的去改动它，接下来我们看看如何编写并发布一个智能合约。

首先我们需要安装开发环境，首先在机器上安装nodejs，我发现在windows上安装比较方便，安装好nodejs后，我们需要通过npm安装truffle，它相当于智能合约的编译器，使用如下命令进行安装:

npm i --global truffle
安装好truffle后，我们还需安装一个叫Ganache的程序，它可以成为在以太网发布代码的客户端，下载链接为：
https://www.truffleframework.com/ganache
安装后所得界面如下：请添加图片描述
注意到启动后点击quick start，出现的画面如上，最左边一列叫adress，其实就是账号，第二列balance就是账号对应的以太，也叫gas，如果你要修改发布后的代码你需要从账号中支付gas给对方，通常你运行时顶部中间RPC SERVER对应ip为127.0.0.1，点击右上角齿轮按钮将其修改为你的本地ip，完成这些安装后就可以开发智能合约了。

在本地创建一个目录叫smart_contract，然后通过控制台cd到该目录，使用如下命令初始化开发环境:

truffle init
完成后目录下面会生成一系列文件夹，同时还有一个truffle-config.js文件，其中的文件夹contracts就是存放我们合约代码的地方。进入到contracts目录，由于我们开发智能合约的语言叫solidility，因此我们创建一个名为donation.sol的文件，然后写入如下代码：

pragma solidity ^0.5.16;

contract Donation {
    address public donatur;
    address payable donatee;
    uint public money;
    string public useless_variable;

    constructor() public {
        donatee = msg.sender;
        useless_variable = "hello world!";
    }

    function change_useless_variable(string memory param) public{
        useless_variable = param;
    }

    function donate() public payable {
        donatur = msg.sender;
        money = msg.value;
    }

    function receive_donation() public {
        donatee.transfer(address(this).balance);
    }
}
代码的语法我们不需要关心，我们只需要了解几点，首先一个合约可以用来接收以太，类似于银行账号能接收他人转账，我们看到里面有个变量叫useless_variable，这个变量的内容可以修改，后面我们发布该代码后，会对它进行修改，同时可以看到修改它是需要付出代价的。接下来用命令对代码进行编译：

truffle  compile
运行命令后界面如下：
请添加图片描述
这个命令会将我们的代码编译成在以太坊上可以运行的二进制代码，这些二进制会发布到以太坊中各个主机进行存储，我们可以将其理解为前面我们说过的区块链中的transation，发布后是极难修改的。接下来要把编译好的二进制发布到以太坊，回到根目录，然后进入migration目录，创建一个文件叫2_deploy_donation.js文件，添加代码如下：

var Donation = artifacts.require("./Donation.sol")

module.exports = function(deployer) {
    deployer.deploy(Donation)
}
我们可以看到，除了智能合约的代码外，其他事物代码是使用js编写，然后用Nodejs来运行。回到根目录，打开truffle-config.js文件，删掉里面所有内容，然后添加代码如下：

module.exports = {
  networks: {
    "development": {
      network_id: 5777,
      host: "10.211.55.7",
      port: 7545
    },
  }
};
注意这里host那里一定要修改成你本地的ip，network_id就是你打开Ganacache后看到的network_id，通常是5777，然后回到根目录，执行如下命令将编译好的二进制代码发布大以太坊:

truffle migration
命令执行后你会看到不少输出内容，其中最后一部分类似如下情况：
请添加图片描述
注意看它的contract address字段，它对应一串十六进制字符串，这个字段类似于银行账户的账号，在接收gas，或转账gas时需要用到它。此时你再打开Ganacache你会看到第一个账号的以太数值发生了改变：
请添加图片描述
这是因为我们发布的二进制数据其实对应transation，这些数据要存储到网络里面的各个独立主机上，这要销毁对方的算力和存储空间，因此我们需要给人家支付一定的”费用“。同时户主页到第一行对应的TX COUNT那里变成了4，可以理解成我们发布的合约二进制数据存储在4个区块上，要把这4个区块添加到公链，矿工就得像前面我们说的那样找到合适的字符串，这个过程也就是挖矿，我们支付的以太就是对矿工挖矿的报酬。点击最顶层的transation按钮会看到如下情况：
请添加图片描述
它显示我们创建了两个合约，一个对应donation.sol的代码，一个对应2_deploy_donation.js的代码。一旦合约发布后，除非你通过特定方法关闭它，要不然它会一直存储在区块链里，根据我们前面的描述，任何人都无法修改其内容，如果合约有Bug,你也不能修改，只能再次发布一个新合约。接下来我们看看如何与发布后的代码进行交互，执行如下命令：

truffle console
这样就进入到命令交互模式，我们执行一段指令，让发布的合约代码显示出变量useless_variable的内容，代码如下：

Donation.deployed().then(function(instance) {
       return instance.useless_variable.call()
    }
);
其中instance对应我们发布到以太坊的二进制代码，上面代码执行后结果如下：
请添加图片描述
可以看到执行代码后他打印除了变量对应的内容。我们再看看如何改变变量的内容，这里的”改变“本质上是再次增加transation的区块链，也就是说改变后变量原来的内容”hello world”还是会被记录在区块链中，这些信息不会被抹除，我们输入如下代码：

Donation.deployed().then(function(instance){return instance.change_useless_variable("how a nice day", {from:"104FC5D50F4461EF5Ce428Ad76Fc2aB573A8C7f8"});});
这里需要注意from字段的内容，它是我们再Ganacache中第一个账号对应的address，执行后输出结果如下：
请添加图片描述
这时候你再次打开Ganacache界面就会看到第一行对应的TX COUNT增加到5，这意味着修改导致一个新区块的添加。只能合约就像一个账号，它是可以接收转账的，对应的资金就是gas, 但是这里有一个很奇怪的情况就是，我们不能直接以gas问单位进行转换，gas还有一个下级单位叫wei ，类似于人民币中元下面还有角和分，在”转账“给智能合约时，我们需要以wei为单位进行转账，这就类似于我们要想转账1元钱，我们需要先把数目转换成分，于是1元等于100分，我们需要转账100分才行。

在以太坊的转换关系中 1gas = 1000000000000000000 wei（18个0），我们可以看到这点有点变态，也就是我们要给合约转账5gas，需要写入的数值为5后面跟18个0，我们看看对应代码：

Donation.deployed().then(function(instance){return instance.donate({from:"0x104FC5D50F4461EF5Ce428Ad76Fc2aB573A8C7f8", value:"5000000000000000000"});});
代码执行后结果如下：请添加图片描述
这是后再次打开Ganache可以看到情况如下：请添加图片描述
可以看到对应的balance数值建设了，正好减少5个以太也就是5个gas，同时区块数量增加到了6，接下来我们可以将转账再次从智能合约转回到我们的账号，代码如下：

Donation.deployed().then(function(instance){return instance.receive_donation({from:"104FC5D50F4461EF5Ce428Ad76Fc2aB573A8C7f8"![请添加图片描述](https://img-blog.csdnimg.cn/981b0721300f43d19e051cccbc062d1e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAdHlsZXJfZG93bmxvYWQ=,size_20,color_FFFFFF,t_70,g_se,x_16)
});});
这段代码跟前面几乎一样，只是把instance.donate改成instance.receive_donation，同时没有了value字段，执行后结果如下：
请添加图片描述
此时再次打开Ganacache看到情况如下：
请添加图片描述
可以看到5个以太转了回来，但是总数额少了一些，同时区块的数量达到8个，少掉的0.02个以太其实是作为费用支付给了”矿工“，因为他们要通过挖矿的方式把我们的转账信息记录下来。

智能合约的重要作用是任何信息只能添加，但是不能删除和修改。我们经常看到一些无良电商一开始为了吸引客流，给出了相当低廉的价格或优惠，一旦顾客购买后他会发来与当初承诺不符合的商品，要不缺斤少两，要不偷梁换柱，例如前阵子欧拉好猫当初在宣传它的车子时说使用的是高通芯片，结果客户买了后发现使用的是因特尔芯片，客户找他理论时，他里面将原来宣传改掉，你再次打开其网页是已经看不到早期它宣传的配置，使用智能合约就是为了防止这种耍无赖的情形出现，从而能保证交易双方的合法权益。

后面我们看看如何使用golang来开发一个类似于以太坊这样的去中心化交易系统。

