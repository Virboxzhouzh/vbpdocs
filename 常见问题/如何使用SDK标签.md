---
sidebar_position: 10.4
---

# 如何使用SDK标签

SDK 工具包，包括头文件、静态库以及动态库，用户在编程的过程中将 SDK 标签静态载入到需要保护的函数当中，这样生成的可执行程序，在 Virbox Protector 加壳工具中就能够分析出 SDK 表示的函数，这样就能够找到用户的核心代码所在的位置。目前支持，VBProtectBegin（常规保护），VBVirtualizeBegin（虚拟化保护），VBMutateBegin（混淆化保护），VBSnippetBegin（碎片化代码保护）,VBProtectDecrypt(许可加解密)。[注意：SDK 标签能方便用户找到关键代码]

### 函数模块保护

#### 注意事项

- 只能静态加载，不支持动态加载dll（即 LoadLibrary 的方式）。

- VBProtectBegin、VBVirtualizeBegin、VBSnippetBegin 以及 VBMutateBegin 等接口，传入的字符串参数，不能与其他函数共用。

- 传入的字符串参数保证为 ANSII 码的形式，这样显示在界面上的函数名称才正确，否则就会显示为乱码。

- 每个 Begin 对应一个 End，总是成对出现，并且一个函数里面不要出现多对 Begin+End。

- 如果 SDK 表示的保护方式和工程文件中保存的保护方式冲突了，以工程文件中的保护方式为准。

- Begin+End 锁定代码最好大于 3 行代码。（因为锁定的代码正汇编生成的指令小于 15 个字节，那么不会显示在加壳工具界面上）

- SDK 动态库，分为 32 以及 64 位，在使用的时候要开发者根据要编译的程序位数进行加载对应的库。

- 目前明确不支持的语言：易语言、Java 程序、Unity3d。

- Begin/End 不支持嵌套使用。

- VBProtectDecrypt 被加密的字符串或者是缓冲区的长度必须是 16 的倍数。

  > eg：char g_test_string[16] = {"test_decrypt"};

- VBProtectDecrypt 传入缓冲区和传出缓冲区不能是同一个缓冲区。

- VBProtectDecrypt 传入的缓冲区只能放在函数外，即全局变量。具体的使用参照 demo。

- .Net 程序暂时不支持。

### 字符串加解密

- 加密的字符串必须是常量。

- 也可以使用 VBDecryptData 直接到数据加密，但数据和长度也必须是常量。

- 字符串解密支持的写法有以下几种：

  - 直接字符串解密：

    > VBDecryptStringA("test_string");

  - 局部静态变量：

    > static const char g_string[] = "test_string";

  - 全局变量：

    > char g_test_string[] = "test_string";
    >
    > const char g_test_string[] = "test_string";
    >
    > static const char g_test_string[] = "test_string";

- 如果程序过于复杂可能会导致解析不出加密的数据，而在加壳时报错，使用 -fpic 或 -fpie 加上 -O2 编译的 32 位 Linux 程序此类情况比较明显。建议降低代码的复杂度。

- 编译器可能会将相同的常量字符串合并为同一个，如果只加密了其中一个，会导致出错，如以下代码：

  > const char* a = "test_string";
  >
  > const char* b = VBDecryptStringA("test_string");
  >
  > printf("a = %s, b = %s\n", a, b);

  这种情况，打印字符串a，可能会是乱码。