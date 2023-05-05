# Bitcoin Core Analysis
Bitcoin Core 是比特币的一个参考实现。
## 环境配置
1. git clone https://github.com/bitcoin/bitcoin
2. ./autogen.sh
3. ./configure --enable-debug (debug optional)
4. make
5. (optional) sudo make install

编辑器配置 
bear -- make (for compile_commands.json)

## use gdb or lldb

编译之后
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
$ ./configure --enable-debug
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

## 对RPC接口中的创建交易函数进行分析

static RPCHelpMan createrawtransaction()
{
    return RPCHelpMan{
        "createrawtransaction", // 命令名称

        "\nCreate a transaction spending the given inputs and creating new outputs.\n"
        "Outputs can be addresses or data.\n"
        "Returns hex-encoded raw transaction.\n"
        "Note that the transaction's inputs are not signed, and\n"
        "it is not stored in the wallet or transmitted to the network.\n", // 命令帮助信息

        CreateTxDoc(), // 交易创建文档，用于生成命令帮助信息

        RPCResult{ // 返回结果
            RPCResult::Type::STR_HEX, // 返回HEX编码的字符串
            "transaction", // 返回的交易数据
        },

        RPCExamples{ // 命令使用示例
            HelpExampleCli("createrawtransaction", "\"[{\\\"txid\\\":\\\"myid\\\",\\\"vout\\\":0}]\" \"[{\\\"address\\\":0.01}]\"")
            + HelpExampleCli("createrawtransaction", "\"[{\\\"txid\\\":\\\"myid\\\",\\\"vout\\\":0}]\" \"[{\\\"data\\\":\\\"00010203\\\"}]\"")
            + HelpExampleRpc("createrawtransaction", "\"[{\\\"txid\\\":\\\"myid\\\",\\\"vout\\\":0}]\", \"[{\\\"address\\\":0.01}]\"")
            + HelpExampleRpc("createrawtransaction", "\"[{\\\"txid\\\":\\\"myid\\\",\\\"vout\\\":0}]\", \"[{\\\"data\\\":\\\"00010203\\\"}]\"")
        },
/*
下面是一个lambda函数，这个lambda函数表达式包含两个参数，分别是self和request。self是RPCHelpMan对象的引用，它表示当前的命令帮助信息；request是JSONRPCRequest对象的引用，它表示当前的JSON-RPC请求。

lambda函数表达式的返回值类型是UniValue，它是一个类似于JSON值的通用值类型，可以表示各种不同类型的数据，包括字符串、数字、数组和对象等。在这里，lambda函数表达式返回一个HEX编码的字符串，表示新的未签名交易。-> UniValue用于指定返回值类型为UniValue。

[&]表示将当前函数作用域内的所有变量都按引用捕获，以便在lambda函数中使用。这样做可以让lambda函数访问当前函数作用域内的变量，从而方便地实现一些逻辑。

总之，这个lambda函数表达式的作用是解析请求中的参数，创建新的未签名交易，并将其编码为HEX字符串返回。
*/

        [&](const RPCHelpMan& self, const JSONRPCRequest& request) -> UniValue
        {
            // 解析replaceable参数
            std::optional<bool> rbf;
            if (!request.params[3].isNull()) {
                rbf = request.params[3].get_bool();
            }

            // 构建新的交易，参数包括输入vin，输出vout及锁定脚本
            CMutableTransaction rawTx = ConstructTransaction(request.params[0], request.params[1], request.params[2], rbf);

            // 将新交易编码为HEX字符串并返回
            return EncodeHexTx(CTransaction(rawTx));
        },
    };
}
这段代码实现了比特币核心钱包中的createrawtransaction命令的处理逻辑。该命令用于创建一个新的未签名的交易，以便后续进行签名和广播。该函数包括以下几个参数，分别为：

inputs：一个数组，包含了输入交易的列表，每个输入交易由一个包含 txid 和 vout 字段的 JSON 对象组成，用于指定该输入交易的交易 ID 和输出索引。
outputs：一个数组，包含了输出交易的列表，每个输出交易由一个包含 address 或 data 字段的 JSON 对象组成，用于指定该输出交易的地址或数据。
locktime：交易锁定时间，一个整数值，可选参数。
replaceable：交易是否可替换，一个布尔值，可选参数。

该代码中的RPCHelpMan是比特币代码中的一个类，用于生成命令帮助信息。JSONRPCRequest是一个包含JSON-RPC请求的对象，通过该对象可以获取到createrawtransaction命令的参数信息。

该代码首先解析传入的参数，包括输入、输出和replaceable参数。然后，它调用ConstructTransaction函数创建一个新的交易，该函数将解析输入、输出和replaceable参数，并使用这些参数构建一个新的CMutableTransaction对象。最后，该代码将新的交易编码为HEX字符串并返回。

需要注意的是，由于createrawtransaction命令所创建的交易是未签名的，因此它不能直接广播到比特币网络中。在签名前，需要通过其他方式将该交易传递给需要签名的节点或者使用钱包软件进行签名。





