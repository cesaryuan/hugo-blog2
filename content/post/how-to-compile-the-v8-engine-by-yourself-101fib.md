---
title: 如何自己编译V8引擎
slug: how-to-compile-the-v8-engine-by-yourself-101fib
url: /post/how-to-compile-the-v8-engine-by-yourself-101fib.html
date: '2023-11-22 12:10:49'
lastmod: '2023-11-29 19:37:19'
toc: true
tags:
  - V8
  - C++
  - Chrome
keywords: V8,C++,Chrome
description: >-
  本文介绍了在安装 Visual Studio 2022 后，设置 Windows 工作环境和安装 depot_tools
  的步骤。其中包括安装必要的组件、配置环境变量以及使用 gclient 初始化 depot_tools。
isCJKLanguage: true
---

## 先决条件

> 参考：
>
> * [https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#setting-up-windows](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#setting-up-windows "[Chromium Docs - Checking out and Building Chromium for Windows](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#Setting-up-Windows)")
> * [https://gist.github.com/jhalon/5cbaab99dccadbf8e783921358020159#setting-up-windows](https://gist.github.com/jhalon/5cbaab99dccadbf8e783921358020159#setting-up-windows)

Once you have Visual Studio 2022 installed, you will need to install the following additional components:

* Desktop Development with C++
* C++ ATL for Latest v143 Build Tools (x86 & x64)
* C++ MFC for Latest v143 Build Tools (x86 & x64)
* C++ CMake tools for Windows
* Windows 11 SDK (10.0.22621.0)

如果没有在上一步安装 Windows 11 SDK，那么需要download the [Windows 11 SDK (10.0.22621.755)](https://go.microsoft.com/fwlink/p/?linkid=2196241)然后安装，并且确保勾选了 "<span style="font-weight: bold;" data-type="strong">Debugging Tools for Windows&quot;。</span>

如果在上一步安装了 Windows 11 SDK，那么需要点击修改，如下图

​![image](https://raw.githubusercontent.com/cesaryuan/hugo-blog2/main/static/imgs/202311291938531.png)

> 如果SDK没有安装在默认位置：`C:\Program Files (x86)\Windows Kits\10`​，那么需要设置环境变量`WINDOWSSDKDIR`​为SDK安装位置
>
> ```pwsh
> $env:WINDOWSSDKDIR = "D:\Windows Kits\10";
> [Environment]::SetEnvironmentVariable('WINDOWSSDKDIR', $env:WINDOWSSDKDIR, 'User')
> ```

## 安装`depot_tools`​

> 参考：
>
> * [https://gist.github.com/jhalon/5cbaab99dccadbf8e783921358020159#download-chrome-tools-and-v8](https://gist.github.com/jhalon/5cbaab99dccadbf8e783921358020159#download-chrome-tools-and-v8)
> * [https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#install](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#install)

通过`scoop install depot_tools`​安装depot_tools，接着干以下几件事

*  <span style="font-weight: bold;" data-type="strong">[Scoop已完成]</span>  Modify the <span style="font-weight: bold;" data-type="strong">PATH</span> System Variable and put `%SCOOP%\depot_tools\current`​ at the front.
* <span style="font-weight: bold;" data-type="strong">[Scoop已完成] </span> Add the <span style="font-weight: bold;" data-type="strong">DEPOT_TOOLS_WIN_TOOLCHAIN</span> User Variable and set it to <span style="font-weight: bold;" data-type="strong">0</span>.

  * This tells depot_tools to use your locally installed version of Visual Studio
* Add the <span style="font-weight: bold;" data-type="strong">vs2022_install</span> User Variable and set it to your installation path of Visual Studio.

  ```pwsh
  $env:vs2022_install = "D:\VisualStudio\2022\Community";
  [Environment]::SetEnvironmentVariable('vs2022_install', $env:vs2022_install, 'User')
  ```

  * For 2022 Community，默认位置是 `C:\Program Files (x86)\Microsoft Visual Studio\2022\Community`​

Once done, open up `cmd.exe`​, `cd`​ to your `depot_tools`​ directory and then run the following command:

```cmd
set http_proxy=http://127.0.0.1:7890 & set https_proxy=http://127.0.0.1:7890
gclient
```

On the first run, `gclient`​ will install all the Windows-specific bits needed to work with the code, including msysgit and python.

* If you run `gclient`​ from a non-cmd shell (e.g., cygwin, PowerShell), it may appear to run properly, but msysgit, python, and other tools may not get installed correctly.
* If you see strange errors with the file system on the first run of gclient, you may want to [disable Windows Indexing](https://tortoisesvn.net/faq.html#cantmove2).

After running gclient open a command prompt and type `where python`​ and confirm that the depot_tools `python.bat`​ comes ahead of any copies of `python.exe`​, like so:

```
C:\dev\depot_tools>where python
C:\dev\depot_tools\python.bat
C:\Users\User\AppData\Local\Microsoft\WindowsApps\python.exe
```

## 拉取V8代码

```pwsh
mkdir v8_code && cd v8_code
fetch v8 # When fetch completes, it will have created a hidden .gclient file and a directory called v8 in the working directory.
cd v8
git fetch
gclient sync # 每次切换分支都要重新运行
```

>  <span style="font-weight: bold;" data-type="strong">NOTE</span>: Usually, you can update your current v8 branch with `git pull`​. Note that if you’re not on a branch, `git pull`​ won’t work, and you’ll need to use `git fetch`​ instead.

官方推荐用cmd，但是测试用pwsh也么啥问题

## 构建 V8

> 参考：[Building V8 with GN · V8](https://v8.dev/docs/build-gn)

首先确保工作目录为`v8`​

### 自动构建

如果不需要修改构建参数`args.gn`​，可以直接运行

```pwsh
python3 tools\dev\gm.py x64.release
```

### 手动构建

> [_ITERATOR_DEBUG_LEVEL | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/standard-library/iterator-debug-level?view=msvc-170)
>
> [Windows VS2017编译WebRTC选项_use_custom_libcxx=false报错_thehunters的博客-CSDN博客](https://blog.csdn.net/thehunters/article/details/122239625)
>
> [v8环境搭建采坑记录-蒲公英云](https://demo.dandelioncloud.cn/article/details/1463314937429946369)
>
> [V8引擎编译 | Calvin&apos;s Marbles](http://www.calvinneo.com/2020/11/21/v8-compile/)

v8_monolithic和v8_static_library的区别是v8_monolithic会把`v8`​、`v8_libbase`​、`v8_libplatform`​三个lib文件整合成一个文件

```pwsh
python3 tools/dev/v8gen.py gen -b x64.release.sample -- is_clang=false v8_enable_object_print=true v8_enable_disassembler=true
autoninja -C out.gn\x64.release.sample v8_monolit h # only compile static

# it will generate args below
dcheck_always_on = false
is_component_build = false
is_debug = false
target_cpu = "x64"
use_custom_libcxx = false
v8_enable_sandbox = true
v8_monolithic = true
v8_use_external_startup_data = false

# Additional command-line args:
is_clang=false
v8_enable_object_print=true
v8_enable_disassembler=true


```

Or

```pwsh
# 最好先删除一下原来的目录
rm out/x64.debug -Recurse -ErrorAction Continue

# DEBUG
($build_args = (@"
	is_debug=true # 是否debug编译
	enable_iterator_debugging=false # debug模式需为false，否则 检测到“_ITERATOR_DEBUG_LEVEL”的不匹配项: 值“0”不匹配值“2”
	v8_enable_slow_dchecks=false # debug模式需为false，否则 error C3615: constexpr 函数不能生成常量表达式
	target_cpu="x64" 
	v8_enable_backtrace=true 
	v8_enable_sandbox=true
	v8_optimized_debug=false 
	is_component_build=false 
	v8_static_library=true # 编译静态库
	v8_enable_disassembler=true # 打印
	v8_enable_object_print=true # 打印
	use_custom_libcxx=false # 用于静态库的话需要设为false
	v8_use_external_startup_data=false # 避免外部文件依赖
	is_clang=false # MSVC 编译
"@ -replace " |#.*", "" -replace "[\r\n]+", " " -replace '"', '\"')) && 
gn gen out/x64.debug --args=$build_args
# RELEASE
($build_args = (@"
	is_debug=false # 是否debug编译
	target_cpu="x64" 
	v8_enable_sandbox=true
	is_component_build=false 
	v8_monolithic=true # 编译静态库
	v8_enable_disassembler=true # 打印
	v8_enable_object_print=true # 打印
	use_custom_libcxx=false # 用于静态库的话需要设为false
	v8_use_external_startup_data=false # 避免外部文件依赖
	is_clang=false # MSVC 编译
"@ -replace " |#.*", "" -replace "[\r\n]+", " " -replace '"', '\"')) && 
gn gen out/x64.debug --args=$build_args

# 构建
autoninja -C out\x64.debug
```

> 这里有一个坑，就是ninja会生成一堆下面这样的输出
>
> ```pwsh
> 注意: 包含文件:       D:\Windows Kits\10\include\10.0.20348.0\ucrt\wchar.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wconio.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wctype.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wdirect.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wio.h
> 注意: 包含文件:         D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_share.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wprocess.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wstdlib.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_wtime.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\sys/stat.h
> 注意: 包含文件:         D:\Windows Kits\10\include\10.0.20348.0\ucrt\sys/types.h
> 注意: 包含文件:     D:\VisualStudio\2022\Community\VC\Tools\MSVC\14.38.33130\include\xmemory
> 注意: 包含文件:      D:\VisualStudio\2022\Community\VC\Tools\MSVC\14.38.33130\include\cstdint
> 注意: 包含文件:       D:\VisualStudio\2022\Community\VC\Tools\MSVC\14.38.33130\include\stdint.h
> 注意: 包含文件:      D:\VisualStudio\2022\Community\VC\Tools\MSVC\14.38.33130\include\cstdlib
> 注意: 包含文件:       D:\Windows Kits\10\include\10.0.20348.0\ucrt\math.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_math.h
> 注意: 包含文件:       D:\Windows Kits\10\include\10.0.20348.0\ucrt\stdlib.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_malloc.h
> 注意: 包含文件:        D:\Windows Kits\10\include\10.0.20348.0\ucrt\corecrt_search.h
> ```
>
> 因此需要改动`build\toolchain\win\toolchain.gni`​文件，把两处`/showIncludes`​全部删掉

‍

## IDE设置

> [https://docs.google.com/document/d/1BpdCFecUGuJU5wN6xFkHQJEykyVSlGN8B9o3Kz2Oes8/edit?pli=1](https://docs.google.com/document/d/1BpdCFecUGuJU5wN6xFkHQJEykyVSlGN8B9o3Kz2Oes8/edit?pli=1)
>
> [https://v8.dev/docs/ide-setup](https://v8.dev/docs/ide-setup)

Author: [jkummerow@chromium.org](mailto:jkummerow@chromium.org)

Last update: 2022-04-14

1. Download VSCode.

    1. On Linux, you can use V8’s own tools/dev/update-vscode.sh --auto (both for initial download, and when you get the in-product notification that an update is available). It’ll extract the package to ~/vscode, and it’ll do its best to create convenient $PATH entries and a .desktop file; so on most distros with standard setup you can simply type code . in any directory afterwards to get started, or find it in whatever menu system your desktop environment uses.
    2. On other platforms, download from [https://code.visualstudio.com/](https://code.visualstudio.com/), and rely on VSCode’s own auto-updater.
2. In V8, run `python3 tools/dev/update-compile-commands.py`​.  
    This creates the compile_commands.json file that clangd requires for indexing. Re-run this tool every time you pull in (or make) significant changes (in particular: adding, moving, or removing files).
3. Install extensions:

    1. C/C++ (by Microsoft)
    2. Clang-Format (by xaver) (Ctrl+K, F to format current line/selection)
    3. clangd (by LLVM)
    4. GN (by npclaudiu)
    5. V8 Torque Language Support (by v8-torque)
    6. optional: ESLint (by Microsoft) (for tests and other .js files)
    7. optional: GitLens (by GitKraken)
    8. optional: Python (by Microsoft) (for V8’s Python tools; needs pylint and yapf installed separately)
4. Configure settings. See the example settings.json below for inspiration. Note that there are `~/.config/Code/User/settings.json`​ for system-wide config and `./.vscode/settings.json`​ for project-specific settings. Choose wisely which setting goes where :-)  
    Note: on Windows, the paths are `%AppData%\Code\User\settings.json`​ and `.\code\settings.json`​, respectively.
5. Optional: if you want to use VSCode’s “build tasks” feature to build from the IDE (default shortcut: Ctrl+Shift+B), set up a task to your liking. See the example ./.vscode/tasks.json below for inspiration.

Notes and gotchas:

* clangd sometimes crashes (e.g. when you keep making changes quicker than it can index them, it sometimes gets confused). VSCode will give up on restarting it if this happens too frequently. In that case, hit F1 (or Ctrl+Shift+P), type “reload”, and select “Developer: Reload Window” to reload VSCode.
* See also: [https://chromium.googlesource.com/chromium/src/+/master/docs/vscode.md](https://chromium.googlesource.com/chromium/src/+/master/docs/vscode.md)
* Feel free to leave comments on this doc or email me (address at the top) with additional suggestions or corrections!

Example settings.json:

```pwsh
{
   "clangd.arguments": [
       "-header-insertion=never"  // More annoying than helpful
   ],
   "clangd.onConfigChanged": "restart",
   "[cpp]": {
       "editor.defaultFormatter": "xaver.clang-format"
   },
   // Let clangd take care of these features.
   "C_Cpp.autocomplete": "Disabled",
   "C_Cpp.errorSquiggles": "Disabled",
   "C_Cpp.intelliSenseEngine": "Disabled",
   "editor.tabSize": 2,
   "editor.fontSize": 12,
   "editor.formatOnType": true,
   "editor.inlayHints.enabled": "off",  // Exceeds 80-col limit
   "editor.rulers": [80],
   "editor.fontFamily": "'DejaVu Sans Mono', 'Droid Sans Mono', 'monospace', 'Droid Sans Fallback'",
   "files.insertFinalNewline": true,
   "files.trimFinalNewlines": true,
   "files.trimTrailingWhitespace": true,
   "files.watcherExclude": {
       "<span style="font-weight: bold;" data-type="strong">/.git/objects/</span>": true,
       "<span style="font-weight: bold;" data-type="strong">/.git/subtree-cache/</span>": true,
       "<span style="font-weight: bold;" data-type="strong">/node_modules/*/</span>": true,
       "infra/**": true,
       // Index generated files for x64.debug, ignore others to declutter
       // search results.
       "out/{arm,ia32,mips}*/**": true,
       "out/x64.optdebug/**": true,
       "out/x64.release/**": true,
   },
   "[javascript]": {
       "editor.defaultFormatter": "xaver.clang-format"
   },
   "javascript.preferences.quoteStyle": "double",
   "python.linting.pylintPath": "/usr/bin/pylint",
   "python.formatting.provider": "yapf",
   "python.formatting.yapfPath": "/usr/bin/yapf",
   "python.formatting.yapfArgs": [
       "--style", "{based_on_style: google, indent_width: 2}"
   ],
   "[typescript]": {
       "editor.defaultFormatter": "xaver.clang-format"
   },
   "workbench.editor.splitSizing": "split",
}

```

Example tasks.json:

```pwsh
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
   {
     "label": "gm x64.debug",
     "type": "shell",
     "command": "tools/dev/gm.py x64.debug tests",
     "group": {
       "kind": "build",
       "isDefault": true
     },
     "presentation": {
       "reveal": "always",
       "panel": "dedicated",
       "clear": true
     },
     "problemMatcher": {
       "fileLocation": [
         "relative",
         "${workspaceFolder}out/x64.debug/"
       ],
       "pattern": {
         "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
         "file": 1,
         "line": 2,
         "column": 3,
         "severity": 4,
         "message": 5
       }
     }
   }
 ]
}

```
