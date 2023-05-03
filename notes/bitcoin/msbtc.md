# Mastering Bitcoin

## Chapter 1
比特币由这些构成：

-   一个去中心化的点对点网络（比特币协议）
-   一个公共的交易账簿（区块链）
-   一个去中心化的数学的和确定性的货币发行（分布式挖矿）
-   一个去中心化的交易验证系统（交易脚本）

**全节点客户端**

完整客户端或“全节点”是存储比特币交易的全部历史（每个用户每次交易）的客户端，管理用户的钱包，并且可以直接在比特币网络上启动交易。全节点处理协议的所有方面，并可以独立地验证整个区块链和任何交易。全节点客户端消耗大量计算机资源（例如，超过250 GB的磁盘，2 GB的RAM），但可以提供完全自主和独立的交易验证。


Alice <--> Joe 
Alice不愿意太冒险，只决定将10美元转换成比特币。她给Joe 10美元现金

Joe然后在他的智能手机钱包上选择发送，并显示一个包含两个输入的屏幕：

- 收款方的比特币地址
- 以比特币（BTC）或其当地法币（USD）计价的发送金额

***确认交易***

起初，Alice的钱包把与Joe的这笔交易显示为“**未确认**”。这意味着交易已传播到网络，但尚未记录在比特币交易账簿即区块链中。确认，就是一个交易必须包含在一个区块中，并被添加到区块链，这样的情况平均每10分钟发生一次。


## Chapter 2

比特币系统由用户、交易和矿工组成，其中用户主要使用密钥控制钱包，交易会被广播到整个比特币网络，矿工通过算力竞争生产出具备所有节点共识的区块链，区块链是一个分布式的公开权威账簿， 包含了比特币网络发生的所有的交易。

Alice -> Bob
交易信息：
```json
{
  "txid": "0627052b6f28912f2703066a912ea577f2ce4da4caa5a5fbd8a57286c345c2f2",
  "size": 258,
  "version": 1,
  "locktime": 0,
  "fee": 50000,
  "inputs": [
    {
      "coinbase": false,
      "txid": "7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18",
      "output": 0,
      "sigscript": "483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adf",
      "sequence": 4294967295,
      "pkscript": "76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac",
      "value": 10000000,
      "address": "1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK",
      "witness": []
    }
  ],
  "outputs": [
    {
      "address": "1GdK9UzpHBzqzX2A9JFP3Di4weBwqgmoQA",
      "pkscript": "76a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788ac",
      "value": 1500000,
      "spent": false,
      "spender": null
    },
    {
      "address": "1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK",
      "pkscript": "76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac",
      "value": 8450000,
      "spent": false,
      "spender": null
    }
  ],
  "block": {
    "height": 277316,
    "position": 64
  },
  "deleted": false,
  "time": 1388185914,
  "rbf": false,
  "weight": 1032
}
```

Alice支付Bob咖啡时使用一笔之前的交易作为输入。在以前的章节中，Alice从她朋友Joe那里用现金换了点比特币。那笔交易创建了被Alice的密钥锁定的比特币。在她支付Bob咖啡店的新交易中使用了之前的交易作为输入，输出是支付咖啡的金额和多余部分的找零。交易形成了一条链，最近交易的输入对应以前交易的输出。Alice用密钥签名解锁了之前交易的输出，从而向比特币网络证明她拥有这笔钱。她将买咖啡的这笔支付到Bob的地址上，明确指明要求是Bob签名才能消费这笔钱，否则就“阻止”那笔输出。这就实现了在Alice和Bob之间价值转移。*UTXO*

Alice的钱包应用首先会找到一些足够支付给Bob所需金额的输入。大多数钱包应用都会跟踪着的属于钱包的地址的所有可用输出。因此Alice的钱包会包含她用现金从Joe那里购买的比特币的交易输出副本。全节点客户端含有整个区块链中所有交易的所有未消费输出副本。这使得钱包既能拿这些输出构建交易，又能在收到新交易时很快地验证其输入是否正确。但是，全节点客户端占太大的硬盘空间，所以大多数钱包使用轻量级客户端，只保存用户自己的未消费输出。


交易的输出是以脚本(Script)的形式创建的，这个脚本设置了对兑换金额的“产权限制”，只能引入这个脚本的一个解后才能解除预置实现提款。简单点说就是，Alice的交易输出会包含一个脚本，这个脚本说 “谁能出示一个对应Bob地址对应密钥的签名，这个输出就支付给谁”。因为只有Bob的钱包的私钥可以匹配这个地址，所以只有Bob的钱包可以提供这个签名以兑换这笔输出。因此Alice会需要Bob的签名来限制输出的使用。


### Change

这个交易还会包含第二个输出。因为Alice的未花费交易输出金额是0.10比特币，对0.015 比特币一杯的咖啡来说太多了，需要找零给Alice 0.085比特币。Alice钱包创建给她的找零与付给Bob的支付在同一个交易里面。**可以说，Alice的钱包将她的金额分成了两个支付：一个给Bob，一个给自己。她可以在以后的交易里继续消费这笔找零输出。**

最后，为了让这笔交易尽快地被网络处理，**Alice的钱包会多付一小笔费用**。这个不是明显地包含在交易中的；而是通过输入和输出的差值所隐含的。如果Alice创建找零时只找 0.0845比特币，而不是 0.085比特币的话，就会有 0.0005比特币（50万聪） 。两笔输出加起来小于 0.10，所以这个 0.10 比特币的输入就没有被完整的消费了。这个差值会就被矿工收取当作交易费，作为矿工将交易放到区块里，最终打包到区块链帐薄中的费用。

任何比特币网络节点（其它客户端）收到一个之前没见过的有效交易时会立刻将它转发给它连接的其它节点。 因此，这个交易迅速地从P2P网络中传播开来，几秒内就能到达大多数节点。

### Mining

新交易不断地从用户钱包和其他应用流入比特币网络。当比特币网络上的节点看到这些交易时，会先将它们放到节点自行维护的一个**临时的未经验证的交易池**中。当矿工构建一个新区块时， 会将这些交易从这个交易池中拿出来放到一个新区块中，然后通过尝试解决一个难题（也叫工作量证明）以证明这个新区块的有效性

这些交易被加进新区块时，以交易费用和其它的一些规则进行排序。矿工一旦从网络上收到一个新区块， 就知道自己在这个区块上的解题竞赛已经输掉了，然后马上开始下一个新区块的挖矿。它会立刻将一些交易和最新那个区块的数字指纹放在一起开始构建下一个新区块，并开始工作量证明计算。**每个矿工会在他的区块中包含一个特殊的交易，将新生成的比特币（当前每区块为6.25比特币）作为矿工费支付到他自己的比特币地址，再加上块中所有交易的交易费用的总和作为自己的报酬。**如果他找到了使得新区块有效的解法，他就会得到这笔报酬，因为这个新区块被加入到了区块链总账中，他添加的这笔报酬交易也会变成可消费的。 参与矿池挖矿的Jing设置了他的软件，构建新区块时会将报酬地址设为整个矿池的地址。然后根据各自上一轮贡献的工作量将所得的报酬分给Jing和其他参与矿池挖矿的矿工。

Alice的交易被网络拿到后放进未验证交易池中。一旦被挖矿软件验证，它就被包含在由Jing的矿池生成的新区块（称为候选块）中。参与该矿池的所有矿工立即开始计算候选块的工作证明。大约在Alice的钱包将这个交易发送出来五分钟后，Jing的ASIC矿机发现了新区块的正解并将这个新区块发布到网络上，一旦结果被其它矿机验证成功，它们就会立即开始下一个新区块的竞赛。

Jing的ASIC矿机发现了新区块的正解并将之发布为第277,316号区块，包含420个交易，包括Alice的交易。将Alice交易包含在区块中就算做对该交易的一次"确认"。

既然Alice的这笔交易已经成为区块的一部分被嵌入到了区块链中，它就成为了整个分布式比特币账簿的一部分，并对所有比特币客户端应用可见。每个比特币客户端都能独立地验证这笔交易是有效且可消费的。全节点客户端可以追溯钱款的来源，从第一次有比特币在区块里生成的那一刻开始，按交易与交易间的关系顺藤摸瓜。
# Hack and Debug with gdb or LLDB, using Bitcoin's code as an example

```cpp
/*
 * Copyright CC0 Angel Leon <@gubatron>
 */
```
**Update**: I believe now it's better to use `lldb`, at least on MacOS, here's a [LLDB to GDB command map](https://lldb.llvm.org/lldb-gdb.html)

Here's how to use `gdb` to debug issues you might be having hacking bitcoinclassic (or any other C++ program)

If you're in Ubuntu you might want to have the following packages installed

```bash
sudo apt install git build-essential pkg-config dh-autoreconf libtool autotools-dev automake libssl-dev libevent-dev bsdmainutils libzmq3-dev libminiupnpc-dev libboost-all-dev
```

We'll also need to add an apt repo to install berkley db dependencies

```bash
sudo add-apt-repository ppa:bitcoinclassic/bitcoinclassic
sudo apt-get update
sudo apt-get install libdb4.8-dev libdb4.8++-dev
```

Check out the main repo
`git clone https://github.com/bitcoinclassic/bitcoinclassic`

Make sure to have it forked on github to your account, now let's add your remote clone to our local instance
`cd bitcoinclassic`
`git remote add myusername https://github.com/myusername/bitcoinclassic`

Now when you create a branch or have commits to push you push them to your `myusername` remote.

## First build ever
`./autogen.sh`

`./configure --enable-debug`

`make -j 8`

## Further builds
### From Scratch
`make clean; ./configure --enable debug; make -j 8`

### Normal builds
To compile ongoing code changes
`make -j 8`

If everything went fine, you should see output similar to...
```
AR       libbitcoin_wallet.a
  CXXLD    bitcoin-cli
  CXXLD    bitcoin-tx
  CXXLD    bitcoind
  CXXLD    test/test_bitcoin
make[1]: Nothing to be done for `all-am'.
```

# Running gdb

First make sure you're in the folder where our executable `bitcoind` is
`cd src`

## Running gdb

On my mac I have to run it with sudo
```
$ sudo gdb
Password:
GNU gdb (GDB) 8.0.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-apple-darwin16.7.0".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word".
(gdb) 
```

## Loading the symbol table of our executable

Now let's load the executable we're about to debug, with the `file` command.
```
(gdb) file bitcoind
Reading symbols from bitcoind...warning: dsym file UUID doesn't match the one in /Users/gubatron/workspace.frostwire/bitcoinclassic/src/bitcoind
warning: can't find symbol '_Z10ParseHexUVRK8UniValueRKNSt3__112basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE' in minsymtab
warning: can't find symbol '_Z21getAllNetMessageTypesv' in minsymtab
warning: can't find symbol '_ZN9Streaming10BufferPool10writeInt32Ej' in minsymtab
done.
```


You can alternatively start `gdb` and pass the executable as an argument
```
$ gdb bitcoind
GNU gdb (GDB) 8.0.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-apple-darwin16.7.0".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from bitcoind...
warning: dsym file UUID doesn't match the one in /Users/gubatron/workspace.frostwire/bitcoinclassic/src/bitcoind

warning: can't find symbol '_Z10ParseHexUVRK8UniValueRKNSt3__112basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE' in minsymtab

warning: can't find symbol '_Z21getAllNetMessageTypesv' in minsymtab

warning: can't find symbol '_ZN9Streaming10BufferPool10writeInt32Ej' in minsymtab
done.
```

## Setting breakpoints

If you know where you want your program to start before hand you can set breakpoints before you run it.
Now that we have a symbol table loaded, let's set up a few breakpoints. We do that with the `b <filename>:<linenumber>` command
```
(gdb) b main.cpp:3560
Breakpoint 1 at 0x1001c8ac8: file main.cpp, line 3560.
(gdb) b bitcoind.cpp:174
Breakpoint 2 at 0x1000091a2: file bitcoind.cpp, line 174.
(gdb) b init.cpp:164
Breakpoint 3 at 0x1000b22c8: file /usr/local/include/boost/signals2/detail/slot_groups.hpp, line 164.
(gdb) b init.cpp:174
Breakpoint 4 at 0x1000b25c3: file init.cpp, line 174.
```

## Running our executable from gdb
Since the file is loaded, we can tell `gdb` to "`run`", the `run` command can receive the arguments we pass to our executable.
If we want to setup a break point at the start and run it, we can do a `b main` and then `run <my program args`, or we can use the shortcut `start <myprogram args>`

* * *
**macOS caveat**: If you're doing this tutorial from a Mac and you're getting an error like this:
```
(gdb) run -server -keypool=1 -rest -discover=0 -testnet -datadir=/Users/gubatron/tmp/bitcoind
Starting program: /Users/gubatron/workspace.frostwire/bitcoinclassic/src/bitcoind -server -keypool=1 -rest -discover=0 -testnet -datadir=/Users/gubatron/tmp/bitcoind
During startup program terminated with signal ?, Unknown signal.
```

create or edit your `~/.gdbinit` file and add the following line and restart gdb:
`set startup-with-shell off`
* * *


So let's start our program and set a breakpoint at the start (make sure the datadir folder exists beforehand):
```
(gdb) start -server -keypool=1 -rest -discover=0 -testnet -datadir=/Users/gubatron/tmp/bitcoind
Temporary breakpoint 1 at 0x1000099e4: file bitcoind.cpp, line 187.
Starting program: /Users/gubatron/workspace.frostwire/bitcoinclassic/src/bitcoind -server -keypool=1 -rest -discover=0 -testnet -datadir=/Users/gubatron/tmp/bitcoind
[New Thread 0x1903 of process 30164]
warning: unhandled dyld version (15)

Thread 2 hit Temporary breakpoint 1, main (argc=7, argv=0x7fff5fbff628) at bitcoind.cpp:187
187	    SetupEnvironment();
```

Since this view isn't the friendliest of all, here's a cool trick, press `Ctrl-x-s`
<img src="https://i.imgur.com/2RjRiJ9.png"/>

With `Ctrl-x-2` you get multiple windows, so you can see the stack.

<img src="https://i.imgur.com/EfTlj2I.png"/>

# Debugging bitcoind with LLDB

```bash
MacBook-Pro:src gubatron$ sudo lldb bitcoind
Password:
(lldb) target create "bitcoind"
Current executable set to 'bitcoind' (x86_64).

(lldb) b main
Breakpoint 1: where = bitcoind`main + 36 at bitcoind.cpp:202, address = 0x00000001000097e4
(lldb) run -server -keypool=1 -rest -discover=0 -testnet -datadir=/Users/gubatron/tmp/bitcoind                                                                  There is a running process, kill it and restart?: [Y/n] Y
Process 85476 exited with status = 9 (0x00000009) 
Process 85483 launched: '/Users/gubatron/workspace.frostwire/bitcoin-abc/src/bitcoind' (x86_64)
Process 85483 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x00000001000097e4 bitcoind`main(argc=7, argv=0x00007fff5fbff630) at bitcoind.cpp:202
   199 	}
   200 	
   201 	int main(int argc, char *argv[]) {
-> 202 	    SetupEnvironment();
   203 	
   204 	    // Connect bitcoind signal handlers
   205 	    noui_connect();
Target 0: (bitcoind) stopped.
```

Then type `gui` and enjoy the view.

<img src="https://i.imgur.com/HgYVMDN.png">

[LLDB to GDB command map](https://lldb.llvm.org/lldb-gdb.html)




[](https://bitcoincore.reviews/feed.xml "Atom Feed") [](https://twitter.com/bitcoincoreprs "Twitter")

[Bitcoin Core PR Review Club](https://bitcoincore.reviews/)

| [Home](https://bitcoincore.reviews/) | [Meetings](https://bitcoincore.reviews/meetings/) | [Code of Conduct](https://bitcoincore.reviews/code-of-conduct/) | [Hosting](https://bitcoincore.reviews/hosting/) |

_These notes were written by [LarryRuane](https://github.com/LarryRuane/) to accompany the [Review Club meeting for PR 22350](https://bitcoincore.reviews/22350) and [GDB video tutorial](https://vimeo.com/576956296/df0b66fbfc). The original can be found on [Larry’s gist](https://gist.github.com/LarryRuane/8c6e8de82f6e2b360ca54dd751388af6)._

# Using debuggers with Bitcoin Core

Please also refer to Fabian Jahr’s [excellent documentation](https://github.com/fjahr/debugging_bitcoin) and [video](https://youtu.be/6aPSCDAiqVI). In this document, I’ll cover only some of what his document does in slightly greater detail, while trying not to duplicate too much, and focused on `gdb` and Linux.

Video version of (most of) this document: https://vimeo.com/576956296/df0b66fbfc _NOTE if you watch the video_: Near the end, I had problems attaching to a running `bitcoind` and entering TUI mode (TUI mode is explained below). Please see the TUI section below for the fix to this problem. (Short version: start `gdb` with the `-tui` option.)

There are two debuggers commonly used with `bitcoind` (and the other c++ executables), `lldb` and `gdb`. The `lldb` debugger is used on MacOS and on Linux with `clang` builds. The `gdb` debugger is used on Linux, and can be used with either `gcc` or `clang` builds. This document won’t discuss `lldb`, but it’s similar to `gdb`.

## gdb documentation

`gdb` has been around since the 1980s and has [excellent documentation](https://sourceware.org/gdb/current/onlinedocs/gdb/). There’s so much there that I don’t know; I’ll present the small subset of things I use most often.

## build with optimization disabled

Debugging with optimizations enabled (the default setting) is very difficult and confusing because many variables can’t be seen (they’re “optimized out”), functions are in-lined, loops unrolled, etc. Single-stepping gives the experience of “I wonder where we’ll go next?” So always build without optimizations:

```
$ ./configure CXXFLAGS='-O0 -g'
```

To tell if optimization is enabled, run `grep '^CXXFLAGS src/Makefile`; if you see `-O0` then optimization is disabled.

Be careful not to make any performance measurements with optimization disabled.

### debug build shortcut

Sometimes you need to debug only a small part of the code. Instead of rebuilding everything, you can manually change the definition of `CXXFLAGS` in `src/Makefile` so that it specifies `-O0` instead of `-O2`, `touch` the files you want to debug, and run `make`.

## running `bitcoind` with `gdb`

There are two ways of debugging (or sometimes called controlling) a `bitcoind` instance with `gdb`.

### starting `bitcoind` from the debugger

Make sure you’re in the `src` directory (so the debugger can find source files). Run

```
gdb bitcoind
Copyright (C) 2020 Free Software Foundation, Inc.
(...)
Reading symbols from bitcoind...
(gdb) run
Starting program: /g/bitcoin/src/bitcoind 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
2021-07-09T21:53:44Z Bitcoin Core version v21.99.0-ea728a30665a (release build)
(...)
```

The `run` command can be abbreviated as `r` – all `gdb` commands can be abbreviated as long as unique. Type `help` to see a list of topics, and `help <topic>` or `help <command>` for detailed help on that topic or command.

If you want to pass arguments to the program, just type them on the `run` command line

```
(gdb) run -regtest -debug
```

Sometimes the arguments string is rather long, so instead you can specify it when starting `gdb` (so that way your shell history has it).

```
$ gdb --args bitcoind -regtest -debug
```

Then you only need to type `run`. If you want to change the arguments, you can specify them on the `run` command, and they’ll replace the initial ones. If you type `run` again, the program will be run with the most recent arguments.

When you `quit` out of `gdb`, the `bitcoind` process dies too. Note that `bitcoind` is not given a chance to shut down gracefully; it’s like sending the process a hard-kill (SIGKILL or -9). But this hasn’t ever caused me problems (and it’s very good to know if this does cause problems!).

### attach to a running `bitcoind`

Rather than starting `bitcoind` from within `gdb`, you can attach to an existing `bitcoind`. Example:

```
gdb --pid 1234
```

This will interrupt the running `bitcoind` and attach to it. It will be suspended, so it may lose P2P connections after a minute or so, but this has never been a problem for me. It’s not possible to affect the command line arguments. When you `quit`, the `bitcoind` continues running.

## command recall within `gdb`

Like any modern shell, `gdb` has command history, which isn’t preserved across `gdb` sessions. Initially, it’s in “emacs” mode (control-p to bring up the most recent command, control-r to search, etc.), but typing the two keys ESCAPE control-j silently puts it into “vi” mode. This setting doesn’t persist across (gdb) executions.

If you type just an Enter (return, empty line), `gdb` will re-execute the last command. This is very nice for single-stepping – type `next` or just `n`, then return-return- etc. to step over each line, or `step` or `s` to continue to step downward.

## useful gdb commands

A few other important commands (the ones I use most often):

-   `fini` – finishes running the current function, printing its return value, then stops in the debugger again
-   `b` – set a breakpoint
-   `i b` – (information about breakpoints) show breakpoints
-   `d <n>` – delete breakpoint number n
-   `d` – delete all breakpoints (it will ask you to confirm)
-   `dis <n>` – disable breakpoint n (instead of deleting it)
-   `ena <n>` – enable breakpoint n
-   `dis` – disable all breakpoints
-   `ena` – enable all breakpoints
-   `until <lineno>` – continue until the given line number is reached, then stop (this is a temporary breakpoint)
-   `c` – continue execution (run freely)
-   `bt` – show stack trace (backtrace)
-   `up`, `down`, `f<n>` – move up and down the call stack, or go to a particular stack frame
-   `l` – show (list) the source code around the current debugger location (then pressing Enter will show the next few lines; `l-` shows previous lines)
-   `i thr` – (information threads) show the list of running threads with their IDs, the `*` indicates the thread the debugger is currently attached to
-   `thr <n>` – switch to thread n (small integer)
-   `thr apply all bt` – show the stack traces of all threads
-   `quit` – quit, leave the debugger

You can set breakpoints by function name:

```
(gdb) b ConnectBlock
```

If not unique, prepend with the object name:

```
(gdb) b CChainState::ConnectBlock
```

Or set breakpoints by filename and line number:

```
(gdb) b validation.cpp:1728
```

If you set the breakpoint by function name, or if you specify the line number of the open brace, then when `gdb` stops there, the function arguments may not be set up correctly (will print as garbage), so single-step (`next`) to the next line.

## command and symbol completion

Type TAB to [extend or complete the current filename, command, or symbol](https://sourceware.org/gdb/current/onlinedocs/gdb/Completion.html#Completion).

## recommended ~/.gdbinit settings

When `gdb` starts up, it opens and reads the file `.gdbinit` in your home directory if it exists. Here you can set things you’d like all the time. Here are the settings I use:

```
set print pretty
set logging on
set history save on
```

The `set history save on` appends your typed commands into `.gdb_history` on exit; `set logging on` appends gdb output into `gdb.txt` as you go.

## gdb.txt

The `gdb.txt` file is very useful; a large data structure such as `m_chainparams.GetConsensus()` is almost 200 lines of output when printed; it’s almost impossible to find things in it by visual inspection. But if you open `gdb.txt` in an editor after printing something large, you can explore it in the editor (most editors let you find matching braces and brackets, for example, or you can just search for things). Also, you have the entire history of the state of the variables or stack traces you’ve printed out, which can be great for later analysis.

Unfortunately, when you print variables, the print request that you’ve typed does not go into `gdb.txt`. So if you’re looking at `gdb.txt` in the editor, you’ll see the contents of variables, but may not know which they are! A workaround I often use is to print a string just before (or after) printing the variable, reminding myself which variable this is. For example,

```
(gdb) p "m_chainparams.GetConsensus()"
$6 = "m_chainparams.GetConsensus()"
(gdb) p m_chainparams.GetConsensus()
(... large variable ...)
```

(You can use the command history so you don’t have to retype all of that.)

## GUI (TUI) mode

One of `gdb`’s lesser-known features is its built-in GUI mode. Well, it’s not a real GUI, it’s a [TUI](https://sourceware.org/gdb/current/onlinedocs/gdb/TUI.html) (text user interface). Here’s [an example](https://photos.app.goo.gl/ZZj9EngPd6mZzak79). It’s best to start `gdb` in TUI mode by specifying `-tui` on the command line. Once in `gdb`, switch between regular and TUI mode by typing the two keys control-x a.

There’s one problem with TUI mode: If you don’t start `gdb` with the `-tui` option, but instead switch to TUI mode by typing control-x a, then nothing the debugger prints gets appended to `gdb.txt`. A workaround for this is to exit TUI mode (control-x a), print the variable or stack trace, then re-enter TUI mode (control-x a).

There also seems to be a problem with attaching to a running process without specifying `-tui` if you then enter TUI mode by typing control-x a. The terminal control becomes completely confused. If attaching to a running process (and if you want to use TUI), always specify `-tui` on the command line, for example:

```
gdb --pid 1234 -tui
```

It also seems to be (at least for me) that if you start `gdb` with `-tui`, then you can’t switch out of it (control-x a). Trying to do that causes `gdb` to crash. So you must stay in TUI mode for the entire session, but that’s probably best anyway.

If the program you’re debugging prints to the terminal, the TUI display gets completely confused (because it thinks it alone is updating the window). Luckily, it’s easy to fix, just type control-L. But, for this reason, it’s advisable to start `bitcoind` with the `-noprinttoconsole` argument, and watch what it’s doing using `tail -f ~/.bitcoin/debug.log` in another window. Or start `bitcoind` normally and attach to it with `gdb` from another window.

Also very helpful is [single key mode](https://sourceware.org/gdb/current/onlinedocs/gdb/TUI-Single-Key-Mode.html#TUI-Single-Key-Mode) (enable and disable by typing control-x s) which allows you to type just a single character to do the common things like next and step.

## Debugging unit tests

It’s easy to use the debugger on unit tests, for example (still in the `src` directory):

```
$ gdb --args test/test_bitcoin --run_test=getarg_tests/logargs
(gdb) b getarg_tests::logargs::test_method
Breakpoint 1 at 0x261aa0: file test/getarg_tests.cpp, line 196.
(gdb)
```

Notice the nonobvious way the names of the test functions are constructed.

## functional (python) tests

### python breakpoints

It’s often helpful to set a breakpoint in the Python test; adding this line will set a breakpoint; the test will suspend and you will be at the python debugger prompt (`(Pdb)`):

```
import pdb; pdb.set_trace()
```

You must run the functional test directly, rather than using `test_runner.py`. It helps to specify `--timeout-factor 0` on the python script command line. The python debugger is pretty basic; type `help` to see the list of commands.

When the python test is suspended in the debugger, there generally will be one or more `bitcoind` processes that the test has launched. To see these, run a command like

```
$ ps alx | grep bitcoind
```

Their command line arguments show where their data directories are; it’s often use to look at their `debug.log` files. You can attach to them with `gdb`, set breakpoints there, continue, then continue in the python debugger.

### pretty printing python variables

Another thing useful in debugging python programs is to add these lines near the top of the file:

```
import pprint
pp = pprint.PrettyPrinter(indent=4)
```

To print out a complicated variable such as a dictionary during the execution of the test, add a line like:

```
pp.pprint(complicated_var)
```

This will display the variable in a much more readable format.

## a “breakpoint” hack

Sometimes you’d like to attach `gdb` to a running `bitcoind` when a certain condition occurs at a particular code location. A weird trick is to add code like to the code path you’re interested in:

```
{ static int spin = 1; while(spin); }
```

(This can be conditioned on some state.) This acts as a hardcoded breakpoint. When the `bitcoind` reaches this code, it will infinite loop; when you notice (or guess) that this has happened, you attach to the process with `gdb`, type `i thr` to see which thread is the one you’re likely interested in, then type `thr <id>`, and you should be at this infinite loop. Then type `set var spin=0` and then you can print variables, single-step, set breakpoints and continue, or whatever. You don’t have to have started `bitcoind` in the debugger ahead of time.

When I don’t know how to cause `bitcoind` to execute a particular code path that I’m interested in debugging or understanding, I’ve set one of these “spin” landmines and then run the entire functional test suite. When it seems to be hung, if I run `top` and see a `bitcoind` steady at 100% CPU, I attach to it, find the right thread, and then begin debugging. It’s a hack, but this has been helpful many times.




# 必读–如何阅读org文档

如何打开org文档

可以直接在github上打开org文档，但是为了方便的跳转到源代码，请使用Emacs编辑器打开org文档！Windows环境的[下载链接](http://mirrors.ustc.edu.cn/gnu/emacs/windows/),Linux下直接使用apt-get或者yum直接安装emacs即可;

如何使用TAB按键

在*和**、***以及更多星号开头的标题上敲击键盘上的Tab按键，可以展开和隐藏这个标题里的内容;

如何理解行首的冒号

行首的冒号(:)是方便org文档输出到HTML文档重点标注代码和命令，除了在#+BEGIN_EXAMPLE和#+END_EXAMPLE里原样输出;

如何生成HTML文档

菜单栏里选择Org->Export/Publish,会调用导出HTML的选项;

如何链接到源代码

比如file:~/.bitcoin ，将光标移动到file开头的链接上，鼠标点击，就会自动跳转到源代码了，如果是目录，就会打开目录。

下载bitcoin源代码

从[github网站](https://github.com/bitcoin/bitcoin)上直接下载或者使用命令行工具:

git clone --branch v0.8.2 https://github.com/bitcoin/bitcoin.git
    

注意将bitcoin源代码目录放在~目录下,目录名为bitcoin，以便迅速在Emacs编辑器中打开bitcoin源代码,Windows下的目录一般为:

C:\Documents and Settings\Administrator\Applicatin Data

如果没有这个目录，可以用如下命令查看目录路径

set appdata

# [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)准备工作

假设你已经对STL及gdb有了一些基本认识，熟悉C++编程。

## [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#1%E4%BA%A7%E7%94%9F%E8%B0%83%E8%AF%95%E4%BF%A1%E6%81%AF)1.产生调试信息

在configure.ac文件里增加2行代码(注意：行首有冒号)：

AC_INIT([Bitcoin]...
: ${CFLAGS="-g -ggdb"}
: ${CXXFLAGS="-g -ggdb"}

按照doc/build-unix.md文件里的的要求重新配置并编译：

./autogen.sh
./configure
make -B //如果是第一次编译，不需要-B

这样在输出的.o文件及elf文件里就会包含有调试信息,否则默认会使用-O2优化选项。

## [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#2gdb%E9%87%8C%E5%A2%9E%E5%8A%A0%E5%AF%B9stl%E7%9A%84%E6%94%AF%E6%8C%81)2.gdb里增加对stl的支持

bitcoin里大量使用了stl，方便在Linux、Windows、Mac间移植。 7.0以后的gdb已经增加了对Python的支持,通过Python，增加gdb对STL的支持： [http://sourceware.org/gdb/wiki/STLSupport](http://sourceware.org/gdb/wiki/STLSupport)

# [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E5%BC%80%E5%A7%8B%E5%88%86%E6%9E%90)开始分析

## [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E9%9D%99%E6%80%81%E5%88%86%E6%9E%90)静态分析

### [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%BF%AB%E9%93%BE%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%E7%B4%A2%E5%BC%95%E6%96%87%E4%BB%B6%E5%8F%8A%E7%9B%B8%E5%85%B3%E7%9B%AE%E5%BD%95)配置文件、快链数据文件、索引文件及相关目录:

在root下有一个目录.bitcoin,首先介绍下几个文件: [~/.bitcoin](https://github.com/happyg1t/bitcoin-analysis/blob/master/~/.bitcoin)

bitcoin.conf

配置文件，bitcoind启动的时候会读取这个文件

debug.log

调试信息输出到这个文件里

peers.dat

存储的其他peer的信息

wallet.dat

钱包文件

blocks

快链(block chain)存储的地方

chainstate

快链的状态

testnet3

用于测试的快链,有一个不同的起始块(genesis block),testnet1中的起始块,就是目前大家在交易的块链。

### [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E6%BA%90%E4%BB%A3%E7%A0%81%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)源代码文件说明:

.h文件及.cpp文件中类的定义及说明: [Doxygen自动产生的说明](https://dev.visucore.com/bitcoin/doxygen)

## [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E5%8A%A8%E6%80%81%E5%88%86%E6%9E%90)动态分析

### [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E9%85%8D%E7%BD%AEbitcoinconf)配置bitcoin.conf

testnet=1

使用测试网络,详见[bitcoin.conf的详细配置](https://en.bitcoin.it/wiki/Running_Bitcoin)

### [](https://github.com/happyg1t/bitcoin-analysis/blob/master/bitcoin%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.org#%E5%BC%80%E5%A7%8B%E8%B0%83%E8%AF%95)开始调试

gdb bitcoind

好了,从bitcoind里读取symbol的时间可能会稍微长点:

Reading symbols from bitcoin/src/bitcoind...done.

开始调试:

b main
run

int main(int argc, char* argv[])
{
    bool fRet = false;

    // Connect bitcoind signal handlers
    noui_connect();

    fRet = AppInit(argc, argv);

    if (fRet && fDaemon)
        return 0;

    return (fRet ? 0 : 1);
 }

main()函数的重点是AppInit(),noui_connect()是用来链接前端Qt的初始化程序，图形界面我们就不分析了，直接进入到AppInit()

1. bool AppInit(int argc, char* argv[])
2. {
3.     boost::thread_group threadGroup;//使用boost的多线程,使得前端Qt程序运行流畅
4.     boost::thread* detectShutdownThread = NULL;//在程序启动期间，如果按了Ctrl-C键退出，则让多线程处理完后退出

5.     bool fRet = false;
6.     try
7.     {
8.         //
9.         // Parameters
10.         //
11.         // If Qt is used, parameters/bitcoin.conf are parsed in qt/bitcoin.cpp's main()
12.         ParseParameters(argc, argv);
13.         if (!boost::filesystem::is_directory(GetDataDir(false)))
14.         {
15.             fprintf(stderr, "Error: Specified data directory \"%s\" does not exist.\n", mapArgs["-datadir"].c_str());
16.             return false;
17.         }
18.         ReadConfigFile(mapArgs, mapMultiArgs);
19.         // Check for -testnet or -regtest parameter (TestNet() calls are only valid after this clause)
20.         if (!SelectParamsFromCommandLine()) {
21.             fprintf(stderr, "Error: Invalid combination of -regtest and -testnet.\n");
22.             return false;
23.         }

24.         if (mapArgs.count("-?") || mapArgs.count("--help"))
25.         {
26.             // First part of help message is specific to bitcoind / RPC client
27.             std::string strUsage = _("Bitcoin Core Daemon") + " " + _("version") + " " + FormatFullVersion() + "\n\n" +
28.                 _("Usage:") + "\n" +
29.                   "  bitcoind [options]                     " + _("Start Bitcoin server") + "\n" +
30.                 _("Usage (deprecated, use bitcoin-cli):") + "\n" +
31.                   "  bitcoind [options] <command> [params]  " + _("Send command to Bitcoin server") + "\n" +
32.                   "  bitcoind [options] help                " + _("List commands") + "\n" +
33.                   "  bitcoind [options] help <command>      " + _("Get help for a command") + "\n";

34.             strUsage += "\n" + HelpMessage(HMM_BITCOIND);
35.             strUsage += "\n" + HelpMessageCli(false);

36.             fprintf(stdout, "%s", strUsage.c_str());
37.             return false;
38.         }

39.         // Command-line RPC
40.         bool fCommandLine = false;
41.         for (int i = 1; i < argc; i++)
42.             if (!IsSwitchChar(argv[i][0]) && !boost::algorithm::istarts_with(argv[i], "bitcoin:"))
43.                 fCommandLine = true;

44.         if (fCommandLine)
45.         {
46.             int ret = CommandLineRPC(argc, argv);
47.             exit(ret);
48.         }
49. #ifndef WIN32
50.         fDaemon = GetBoolArg("-daemon", false);
51.         if (fDaemon)
52.         {
53.             fprintf(stdout, "Bitcoin server starting\n");

54.             // Daemonize
55.             pid_t pid = fork();
56.             if (pid < 0)
57.             {
58.                 fprintf(stderr, "Error: fork() returned %d errno %d\n", pid, errno);
59.                 return false;
60.             }
61.             if (pid > 0) // Parent process, pid is child process id
62.             {
63.                 CreatePidFile(GetPidFile(), pid);
64.                 return true;
65.             }
66.             // Child process falls through to rest of initialization

67.             pid_t sid = setsid();
68.             if (sid < 0)
69.                 fprintf(stderr, "Error: setsid() returned %d errno %d\n", sid, errno);
70.         }
71. #endif
72.         SoftSetBoolArg("-server", true);

73.         detectShutdownThread = new boost::thread(boost::bind(&DetectShutdownThread, &threadGroup));
74.         fRet = AppInit2(threadGroup);
75.     }
76.     catch (std::exception& e) {
77.         PrintExceptionContinue(&e, "AppInit()");
78.     } catch (...) {
79.         PrintExceptionContinue(NULL, "AppInit()");
80.     }

代码里增加了一些中文注释，作为对英文注释的补充。

-   try..catch{}方式处理这段代码，因为这里涉及大量磁盘读取，有可能碰到无法打开文件，磁盘空间满等其他问题。
-   ParseParameters()及ReadConfigFile()都是在读取一些参数，需要注意的是，命令行参数的优先级高于配置文件，比如执行了如下语句

bitcoind -testnet=0

则在bitcoin.conf中配置的testnet则无效了。

参数最终储存在mapArgs及mapMultiArgs中,定义如下:

map<string, string> mapArgs;
map<string, vector<string> > mapMultiArgs;

使用了STL的map定义的,后面会经常使用这几个函数GetBoolArg()、GetArg()来查看参数设置,如:

bool GetBoolArg(const std::string& strArg, bool fDefault)
{
    if (mapArgs.count(strArg))
    {
        if (mapArgs[strArg].empty())
            return true;
        return (atoi(mapArgs[strArg]) != 0);
    }
    return fDefault;
    }

经常会看到这样调用GetBoolArg():

GetBoolArg("-daemon", false)

如果没有找到相关设置，则返回假。

-   24行到38行是打印帮助提示。
-   介绍下面的内容前,先介绍下RPC(远程过程调用),命令行方式下RPC的使用方法:

bitcoind -daemon //后台运行
bitcoind getinfo //获取状态信息
bitcoind -stop//停止daemon进程

-   39行到48行判断是否是上面第二行(getinfo)的语句:首先判断参数是否有’-‘或’/’,并且不包含’bitcoin:’,则执行RPC(使用的是JSON-RPC调用协议)

bitcoin:URI是用于请款的,所以应该排除这种情况: bitcoin://1F2EUzKR1XsLRCtEnsnpDQZ13XJgS6P3ZK?amount=0.001&message=donation

接着执行CommandLineRPC(),具体执行RPC(远程过程调用).

-   49行到71行是处理-daemon参数,假设这样运行bitcond

bitcoind -daemon

通过fork()创建一个子进程,创建成功,父进程则在64行的时候返回,子进程接着执行下面的初始化,包括AppInit2()。 在.bitcoin目录下创建了一个bitcoind.pid的文件，记录了子进程的PID。

[file:~/bitcoin/src/init.cpp::bool AppInit2 boost thread_group threadGroup](https://github.com/happyg1t/bitcoin-analysis/blob/master/~/bitcoin/src/init.cpp)

/** Initialize bitcoin.
 *  @pre Parameters should be parsed and config file should be read.
 */
bool AppInit2(boost::thread_group& threadGroup)
{
    // ****************** Step 1: setup   设置
    ...
    // ****************** Step 2: parameter interactions   参数互动(主要是一些参数设置)
    ...
    // ****************** Step 3: parameter-to-internal-flags   参数传入内部标记(bool型变量)
    ...
    // ****************** Step 4: application initialization: dir lock, daemonize, pidfile, debug log   应用初始化:锁定目录,后台运行,调试信息
    ...
    // ****************** Step 5: verify wallet database integrity   确认钱包数据库的完整性
    ...
    // ****************** Step 6: network initialization   网络初始化
    ...
    // ****************** Step 7: load block chain   加载块链
    ...
    // ****************** Step 8: load wallet   加载钱包
    ...
    // ****************** Step 9: import blocks   导入块数据
    ...
    // ****************** Step 10: load peers   导入peers
    ...
    // ****************** Step 11: start node   开始节点(挖矿程序在这里)
    ...
    // ****************** Step 12: finished  完成
    ...
}

-   AppInit2里包含了bitcoin的大部分初始程序，包括读取’块索引’、加载块链、加载100个预产生的keys,导入peers.dat中的信息,以及初始化其他线程。

未完待续,Contact me:

-   BitMessage:BM-2cVh8hpF9jcvDtFJCDddx518EEs76SvTUE
-   BTC:1F2EUzKR1XsLRCtEnsnpDQZ13XJgS6P3ZK

<2014-01-19 星期日>
