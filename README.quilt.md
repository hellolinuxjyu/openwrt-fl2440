QUILT 使用说明
===
---------------------------------
在 openwrt 开发过程中是通过 quilt 工具来进行 patch 管理的，因此可以通过先安装 quilt 工具，然后通过相应的命令来建立 patch , 具体说明如下。

使用 quilt 管理源码，它会在代码树中创建两个特殊的目录 patches 和 .pc。quilt 在 patches 目录中保存他管理的所有 patch。 而在 .pc 目录中保存自己内部工作状态，用户不需要了解这个目录。在 patches 目录中有个特殊文件 series 其中记录着 quilt 当前管理的 patch。patch 按照加入的顺序排列。即 quilt 是使用堆栈的概念来管理 patch 的应用的。为此 quilt push 为应用 patch , quilt pop 为撤销 patch。

*** 注 ***
openwrt 编译系统自配 quilt ，因此在其编译的时候如有 patch 将自动打入，apply patch 顺序貌似以文件名排序相关。
然后对于 patch 和项目之间仅仅需要在项目包里面创建 patches 目录，并将想要在编译时 apply 的文件放入即可，如下是例子：

    $ tree mklibs
    mklibs/
    ├── include
    │   └── elf.h
    ├── Makefile
    └── patches
        ├── 001-missing_stdio.patch
        ├── 002-disable_symbol_checks.patch
        ├── 003-no_copy.patch
        ├── 004-libpthread_link.patch
        ├── 005-readelf_fixes.patch
        ├── 006-duplicate_syms.patch
        ├── 007-uclibc_init.patch
        ├── 008-gc_sections.patch
        ├── 009-uclibc_libgcc_link.patch
        └── 010-missing_unistd.patch

    2 directories, 12 files

上述为 mklibs 目录，其中 Makefile 是用来配置编译相关，而 patches 里面存放的则是相应对标准包的修改。注意，所有的改动都是在以 mklibs 为根目录的修改，否则不能被 quilt push 进入源码中。

1. 安装
    
    $ sudo apt-get install quilt

2. 查看参数
    
    $ quilt help

    Usage: quilt [--trace[=verbose]] [--quiltrc=XX] command [-h] ...
           quilt --version
    Commands are:
            add       fold    new       remove    top
            annotate  fork    next      rename    unapplied
            applied   graph   patches   revert    upgrade
            delete    grep    pop       series
            diff      header  previous  setup
            edit      import  push      shell
            files     mail    refresh   snapshot

    Global options:

    --trace
            Runs the command in bash trace mode (-x). For internal debugging.

    --quiltrc file
            Use the specified configuration file instead of ~/.quiltrc (or
            /etc/quilt.quiltrc if ~/.quiltrc does not exist).  See the pdf
            documentation for details about its possible contents.  The
            special value "-" causes quilt not to read any configuration
            file.

    --version
            Print the version number and exit immediately.

3. 具体使用步骤

    $ quilt new 001-changes-commit.patch
    # 修改文件
    $ quilt edit file.cxx
    # 添加文件(-P 表示指定某个patch文件)
    $ quilt add [-P patch] file1.cxx
    # 删除文件(-P 表示指定某个patch文件)
    $ quilt remove [-P patch] file1.cxx
    # 将 patch 保存为文件
    $ quilt refresh

    这样就创建了一个 001-changes-commit.patch 的 patch 文件。

    # 查看当前 patch 修改的内容
    $ quilt diff

    # 查看当前已修改的文件列表(-v 用户友好； -a 显示所有，-l 显示patch)
    $ quilt files

4. patch 管理

    # 删除某个 patch 文件
    $ quilt delete 0001-changes-commit.patch
    # 撤销所有的 patch 变化
    $ quilt pop -a 
    # 应用所有的 patch 变化
    $ quilt push -a 

    同时可以查看当前状态
    # 查询当前已应用的 patch 列表
    $ quilt applied
    # 查询当前未应用的 patch 列表
    $ quilt unapplied

    # 查看当前或栈顶 patch
    $ quilt top

    # 查看所有 patches
    $ quilt series

    # 导入
    $ quilt import 
    # 撤销
    $ quilt revert