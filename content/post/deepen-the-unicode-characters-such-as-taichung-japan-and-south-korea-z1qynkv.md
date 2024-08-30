---
title: 深入C++控制台中日韩等Unicode字符乱码问题
slug: deepen-the-unicode-characters-such-as-taichung-japan-and-south-korea-z1qynkv
url: >-
  /post/deepen-the-unicode-characters-such-as-taichung-japan-and-south-korea-z1qynkv.html
date: '2024-08-27 20:24:41+08:00'
lastmod: '2024-08-30 19:24:41+08:00'
toc: true
tags:
  - C++
  - 乱码
  - Unicode
keywords: C++,乱码,Unicode
isCJKLanguage: true
---



在 C++ 编程中，控制台输出中文乱码一直是一个令人头疼的问题。但是目前中文互联网上的文章，大多都是告诉你哪些语句可以解决这个问题，并没有深层次的分析，因此这篇文章从 Windows 运行库的角度来分析问题的根本原因。

## 解决办法

首先介绍目前能想到的最完美的解决办法：

```cpp
// 文件编码改成 UTF-8，同时在 CMakeLists.txt 中添加 add_compile_options("/utf-8")
SetConsoleOutputCP(CP_UTF8); // 相应的，当你将 locale 设置为 utf8 之后，你应该将 ConsoleOutputCP 设置为 utf8，因为默认的 CP_ACP 根本不支持中文以外的字符
setlocale(LC_ALL, ".utf-8"); // 如果你想向控制台输出emoji或者除了中文之外的字符，那么强烈推荐你将locale设置为.utf8，因为中文系统默认的`936`GBK代码页压根就不支持那些字符。
wcout.imbue(locale(wcout.getloc(), new codecvt_utf8_utf16<wchar_t>)); // 将 wcout 接受的 utf16 编码的 wchar_t 流转换为 utf8 buffer
```

该方法可以让 `std::wcout`​、`std::cout`​、`printf` ​支持所有 Unicode 字符，让 `wprintf` ​支持 Unicode 双字节字符

以下是结合微软 C++ 运行时 ucrtbased.dll 和 msvcp140.dll 的源码和 IDA 调试得到的不同的控制台输出函数输出 Unicode 字符的兼容性和要求

|放在同一个单元格内的函数都或多或少的调用了最底下的函数|中文字符|中文字符 + 日韩等 Unicode **BMP 基本平面的字符**<br />**U+0000 到 U+FFFF**<br />|Unicode **BMP** + Emoji 等 **SMP 辅助平面的字符**<br />**U+010000 到 U+10FFFF**<br />|
| ------------------------------------------------------| --------| --------------------------| --------------------|
|​`wprintf（C标准库）`​，<br />`_putws（C标准库）`​，<br />`fputwchar（C标准库）`​，<br />`putwchar（C标准库）`​，<br />`fputwc（C标准库）`​<br />|​`setlocale(LC_ALL, "");`​<br />|​`setlocale(LC_ALL, ".utf8")`​|❌|
|​`std::wcout（C++标准库）`​|​`setlocale(LC_ALL, "");`​<br />|​`SetConsoleOutputCP(CP_UTF8);`​<br /><br />`setlocale(LC_ALL, ".utf8")` ​或 `wcout.imbue(locale(".utf8"))`​<br />|​`项目编码(UTF-8)`​ <br />`SetConsoleOutputCP(CP_UTF8);`​<br />`wcout.imbue(locale(wcout.getloc(), new codecvt_utf8_utf16<wchar_t>));`​<br />|
|​`_cwprintf（Microsoft特定）`​，<br />`_putwch（Microsoft特定）`​，<br />`WriteConsoleW（Windows API）`​<br />|✔|✔|✔`项目编码(UTF-8)`​|
|​`std::cout`​<br />`printf`​<br />|​`项目编码(GBK or ANSI)`​<br />`SetConsoleOutputCP(CP_ACP);`​<br />|​`项目编码(UTF-8)`​<br />`SetConsoleOutputCP(UTF-8);`​|​`项目编码(UTF-8)`​<br />`SetConsoleOutputCP(UTF-8);`​<br />|

PS:

1. ​`setlocale(LC_ALL, "")`​ ​为设置 locale 为实现定义的本机环境（ANSI），不设置的话默认是 `setlocale( LC_ALL, "C" )`​
2. ​`locale: ".utf-8"`​：即可通过 `setlocale(LC_ALL, ".utf8")`​ ​设置也可通过 `wcout.imbue`​ ​设置
3. ​`项目编码(UTF-8)`​ ​指：文件编码改成 UTF-8，同时在 CMakeLists.txt 中添加 add_compile_options("/utf-8")
4. ​`setlocale`​ 会影响 `wcout`​ ​的 `locale`​
5. ​`std::wcout.imbue`​ ​不会影响其他 `locale`​
6. GB18030 虽然支持所有 Unicode 字符，但是 Windows 压根不支持 GB18030 的代码页，`setlocale(LC_ALL, ".54936")`​ ​返回 null

---

## 不同函数的分析

下面按照向控制台输出内容的不同 API 来进行分析

> 术语说明：
>
> * `partial code point`​：因为 Windows MSVC 上的 L 字符串 `wchar_t` ​类型为 16 位，所以只能表示 Unicode Layer0 中的字符，对于 emoji 这种字符，需要两个 `wchar_t` ​的字符才能表示清楚，所以将这两个 `wchar_t` ​类型的值分别称为 `partial code point`​
> * ​`fh`​：file hanle，POSIX 格式的，stdout 为 2
> * ​`double_translation`​：First, convert from the source multibyte to Unicode, then from Unicode back to multibyte, but in the console codepage.

### 1. `fputwc`​(`wprintf`​,`_putws`​)

​`fputwc` ​最终会调用 `_fputwc_nolock_internal`​，其内部细节如果

1. ​`_wctomb_internal`​ 将 wide char 转换成 multibytes，但是不支持 `partial code point`​

   * 如果没有设置 `locale`​，merely casts it to char (if in range)，并 `ptd.get_errno().set(EILSEQ);`​
   * 如果设置了 `locale` ​为 `.utf-8`​，则调用 `__crt_mbstring::__c32rtomb_utf8(char* s, char32_t c32, ...)`​，支持 `partial code point`​，但需要传入 `char32_t`​
   * 其他情况下调用 `WideCharToMultiByte` ​和对应的 `locale->locinfo->_public._locale_lc_codepage`​，支持 `partial code point`​，但需要传入 `wchar_t[2]`​
2. call `_fputc_nolock_internal`​ -> `_write_nolock(fh, buffer, buffer_size, ...)`​，将成功转换后的 `multibytes` ​输出到 `stdout` ​中，该函数在 `locale`​ 为 `".utf8"` ​以及 `ConsoleOutputCP` ​为 `CP_UTF8` ​的情况下支持写入 emoji，然而上一步不支持 emoji，所以这一步支持也没用

   对于需要 `double_translation` ​的 fh（stdout 正是如此），如果 fh 的 textmode 为 ansi（stdout 正是如此），则调用 `write_double_translated_ansi_nolock`​

   这个函数从 locale codepage 获取 buffer 的编码，将 multibyte 中每个**字符**转换为对应的 `wchar_t[2]`​（这里之所以是 `[2]` ​是因为 Windows 知道 Unicode BMP 之后的字符都需要 32 位来表示），然后调用 `WideCharToMultiByte` ​将 `wchar_t[2]` ​转换为 `ConsoleOutputCP` ​对应的编码下的 multibytes，这样终端才能正确展示这些字符

> ​`wprintf` ​最终也会在 `class stream_output_adapter` ​中对每一个 `wchar_t` ​调用 `_fputwc_nolock_internal`​，所以必然不支持 emoji
>
> ​`_putws` ​最终也是调用 `_fputwc_nolock_internal`​，所以也不支持 emoji

### 2. `WriteConsoleW`​(`_cwprintf`​)

> ​`_cwprintf` ​与 `wprintf` ​不同的是，`_cwprintf` ​调用的 `output_adapter` ​是 `console_output_adapter`​，其内部会调用 `_putwch`​，`_putwch` ​又会调用 `WriteConsoleW`​

​`WriteConsoleW` ​是 kernelbase.dll 实现的，天生支持 emoji，我看不懂它的 IDA 伪代码，也找不到它的原代码，所以不知道它的细节是怎么样的

### 3. `std::wcout`​

~~​`wcout`​~~ ~~没有~~ ~~​`double_translation`​~~ ~~步骤，没有~~ ~~​`double_translation`​~~ ~~步骤意味着：用户使用~~ ~~​`wcout`​~~ ~~时，~~​**~~如果指定的~~** **~~​`locale`​~~** **~~中的~~** **~~​`std::codecvt`​~~** **~~将~~** **~~​`wchar_t *`​~~**  **~~转换为了编码 A，则也应该调用~~** **~~​`SetConsoleOutputCP(编码A);`​~~**

> ~~当 locale 为~~  ~~​`"C"`​~~  ~~时会调用 fputwc，虽然 fputwc 有~~ ~~​`double_translation`​~~ ~~步骤，但是因为 locale 为~~  ~~​`"C"`​~~​ ~~，在~~  ~~​`_wctomb_internal`​~~ ~~这一步就 return 了，并不会~~ ~~​`double_translation`​~~ ~~步骤。~~

1. call std::wstreambuf::xsputn
2. call std::wfilebuf::overflow

   * 如果没有设置 locale，则 locale 默认为 `"C"`​ ，则 `this->_Pcvt` ​为 null，程序只会简单的调用 fputwc
   * 如果设置了正确的 locale（`setlocale` ​不返回 null，`std::locale` ​不报错），则会调用 `std::codecvt::out` ​将 `__int16 _Meta` ​转换为 bytes
   * 然后调用 `fwrite`​，fwrite 最终调用 `_write_nolock`​，还是会经过 `double_translation` ​步骤

要注意的是 `".utf8"`​ 这个 locale 实现的 `std::codecvt`​ 并不能处理 `patial code point`​ 的情况，因为其 `out`​ 方法会调用 `WideCharToMultiByte`​，但是仅传入了 `wchar_t`​ 而不是 `wchar_t[2]`​。相反，`codecvt_utf8_utf16`​ 这个类因为是自己实现了 utf16 到 utf8 的转换，所以可以处理这种情况。

---

## 参考文献

* [Windows 中控制台应用的 Unicode 支持调查 - 胡玮文的博客](https://www.huww98.cn/Unicode-in-Windows-console)
* [setlocale, _wsetlocale | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/setlocale-wsetlocale?view=msvc-170)
* [WideCharToMultiByte function (stringapiset.h) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-widechartomultibyte)
* [_cprintf, _cprintf_l, _cwprintf, _cwprintf_l | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/cprintf-cprintf-l-cwprintf-cwprintf-l?view=msvc-170)
* [Microsoft Windows library files - Wikipedia](https://en.wikipedia.org/wiki/Microsoft_Windows_library_files#MSVCRT.DLL,_MSVCP*.DLL_and_CRTDLL.DLL)
* [彻底弄懂 Unicode 编码 - 李宇仓 | Li Yucang](https://liyucang-git.github.io/2019/06/17/%E5%BD%BB%E5%BA%95%E5%BC%84%E6%87%82Unicode%E7%BC%96%E7%A0%81/)
* [std::locale - cppreference.com](https://en.cppreference.com/w/cpp/locale/locale)
* [std::locale::locale - cppreference.com](https://en.cppreference.com/w/cpp/locale/locale/locale)
* [String literal - cppreference.com](https://en.cppreference.com/w/cpp/language/string_literal)
* [std::codecvt_utf8_utf16 - cppreference.com](https://en.cppreference.com/w/cpp/locale/codecvt_utf8_utf16)
* [关于字符编码，你所需要知道的（ASCII,Unicode,Utf-8,GB2312…） | 简单生活 — Kevin Yang 的博客](https://www.imkevinyang.com/2010/06/%E5%85%B3%E4%BA%8E%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81%EF%BC%8C%E4%BD%A0%E6%89%80%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84)
* [关于字符编码那些你应该知道的事情](https://blog.shoyuf.top/stories/About-character-encoding)

## 附录

过程中使用的测试代码

```cpp

int main() {
    SetConsoleOutputCP(CP_UTF8);
    setlocale(LC_ALL, ".utf-8");
    wcout.imbue(locale(wcout.getloc(), new std::codecvt_utf8_utf16<wchar_t>));
    // wcout.imbue(locale(""));
    // auto localname = setlocale(LC_ALL, "");
    // auto localname = setlocale(LC_ALL, ".utf8");
    // cout << "localname: " << localname << endl;
    // printf(localname);
    // wcout.imbue(locale(""));
    // std::wcout << L"🍌" << std::endl;
    // wprintf(L"🍌\n");
    // _cwprintf(L"🍌\n");
    // wprintf（C标准库）
    wprintf(L"%S", u8"汉字한국의🍌\n");
    printf("1,printf,汉字한국의🍌\n");
    std::cout << "4,wcout,汉字한국의🍌" << std::endl;
    wprintf(L"1,wprintf,汉字한국의🍌\n");

    // _putws（C标准库）
    _putws(L"2,_putws,汉字한국의🍌");

    // putwchar（C标准库）
    std::wcout << L"3,putwchar,";
    putwchar(L'你');
    _putch('\n');

    // std::wcout（用于宽字符）
    std::wcout << L"4,wcout,汉字한국의🍌" << std::endl;

    // WriteConsoleW（Windows API）
    HANDLE hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
    DWORD charsWritten;
    WriteConsoleW(hConsole, L"5,WriteConsoleW,汉字한국의🍌\n", sizeof(L"5,WriteConsoleW,汉字한국의🍌\n") / 2, &charsWritten, NULL);

    // _cwprintf（Microsoft特定）
    _cwprintf(L"6,_cwprintf,汉字한국의🍌\n");

    // _putwch（Microsoft特）
    std::cout << "7,_putwch,";
    _putwch(L'你');
    _putch('\n');

    // fputwc（C标准库）
    std::cout << "8,fputwc,";
    fputwc(L'你', stdout);
    _putch('\n');

    return 0;
}
```

​

‍

‍
