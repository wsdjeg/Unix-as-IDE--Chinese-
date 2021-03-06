﻿版本控制
========

现在，版本控制已经被视为专业软件开发中不可或缺的部分。图形化的集成开发环境例如 Eclipse 和 Visual Studio 已经包含版本控制，并且引入了对工业标准版本控制系统的支持。现代版本控制系统的血统可以追溯到来自于 ``diff`` 和 ``patch`` 等 Unix 程序的概念。而且依旧有很多人坚持认为最好的版本控制系统是命令行形式的。

在这个系列的最后一篇文章中，我会跟随常见开源版本控制系统发展的脚步，在最早的版本控制中从 ``diff`` 和 ``patch`` 的基本概念开始。

``diff``\、 ``patch`` 和 RCS
----------------------------

版本控制的一个核心概念就是 *unified diff*\，即统一差别格式，一种使用人机皆可理解的表达方式来表现文件中的变化。 ``diff`` 命令最初是由 Douglas McIlroy 在1974年随着第五版Unix发行的，所以它可以称得上是现代系统中仍在使用的最老的命令之一。

统一差别格式（ *unified diff* ）是最常见的可协作格式，可以由比较一个文件的两个不同版本来产生，它遵循以下语法： ::
    
    $ diff -u example.{1,2}.c
    --- example.c.1    2012-02-15 20:15:37.000000000 +1300
    +++ example.c.2    2012-02-15 20:15:57.000000000 +1300
    @@ -1,8 +1,9 @@
    #include <stdio.h>
    +#include <stdlib.h> 
    
    int main (int argc, char* argv[])
    {
        printf("Hello, world!\n");
    -   return 0;
    +   return EXIT_SUCCESS;
    }

在这个例子中，第二个文件比第一个文件中添加了一个头文件，并且其 ``main()`` 函数返回使用了标准的 ``EXIT_SUCCESS`` 而不是数字 ``0``\。并且注意开头， ``diff`` 还给出了了相应元数据，例如比较的文件名还有文件的最后修改时间。

在代码量较大的情况下，一种原始的版本控制方法即是交换`diff`输出。这些输出被称作 *patches*\，即补丁，补丁可以用 ``patch`` 来补到原基础代码上。我们可以这样将上面例子的 ``diff`` 输出保存成补丁： ::
    
    $ diff -u example.{1,2}.c > example.patch

然后我们就可以把补丁发送给一个尚在使用旧版源文件的开发者，然后他就可以像这样自动化打补丁： ::

    $ patch example.1.c < example.patch

补丁文件可以包含本目录及子目录内多组 ``diff`` 文件比较输出的结果，这样补丁就可以很容易地应用到源代码树上了。

使用 ``diff`` 输出来跟踪改动所涉及的操作足够规则用以保存改动历史，
人们开发出了 `Source Code Control System <http://en.wikipedia.org/wiki/Source_Code_Control_System>`_ 和 `Revision Control
System <http://en.wikipedia.org/wiki/Revision_Control_System>`_ （译注：源代码控制系统，暂无中文维基页面，出于可信度考虑不引用其他百科） 来实现这一目的，基本上已经取代了 ``diff`` 输出的方法。RCS提供了“锁定（lock）”文件的功能，防止一个文件在被“签出（check out）”时被其他人修改。这个概念给其他更成熟的版本控制系统铺平了道路。

RCS保留了简单易用的优势。将一个文件纳入版本控制，只需要键入 ``ci <filename>`` 并且提供一个合适的文件描述： ::

    $ ci example.c
    example.c,v  <--  example.c
    enter description, terminated with single '.' or end of file:
    NOTE: This is NOT the log message!
    >> example file
    >> .
    initial revision: 1.1
    done

这样就在该文件目录内创建了一个新文件 ``example.c,v``\，用来跟踪文件的修改。修改文件之前，你需要 *签出（check out）* ，然后修改，最后再把它 *签入（check in）* 回去： ::
    
    $ co -l example.c
    example.c,v  -->  example.c
    revision 1.1 (locked)
    done
    $ vim example.c
    $ ci -u example.c
    example.c,v  <--  example.c
    new revision: 1.2; previous revision: 1.1
    enter log message, terminated with single '.' or end of file:
    >> added a line
    >> .
    done

你可以使用 ``rlog`` 来查看一个项目的修改历史： ::
    
    $ rlog example.c
    
    RCS file: example.c,v
    Working file: example.c
    head: 1.2
    branch:
    locks: strict
    access list:
    symbolic names:
    keyword substitution: kv
    total revisions: 2;	selected revisions: 2
    description:
    example file
    ----------------------------
    revision 1.2
    date: 2012/02/15 07:39:16;  author: tom;  state: Exp;  lines: +1 -0
    added a line
    ----------------------------
    revision 1.1
    date: 2012/02/15 07:36:23;  author: tom;  state: Exp;
    Initial revision

使用 `rcsdiff -u` 命令获得两个修订版本之间统一差别（ ``diff`` ）格式的补丁文件： ::

    $ rcsdiff -u -r1.1 -r1.2 ./example.c
    ===================================================================
    RCS file: ./example.c,v
    retrieving revision 1.1
    retrieving revision 1.2
    diff -u -r1.1 -r1.2
    --- ./example.c	2012/02/15 07:36:23	1.1
    +++ ./example.c	2012/02/15 07:39:16	1.2
    @@ -4,6 +4,7 @@
    int main (int argc, char* argv[])
    {
        printf("Hello, world!\n");
    +   printf("Extra line!\n");
        return EXIT_SUCCESS;
    }

这样的用法可能让你觉得简单的补丁文件已经不再是一种版本控制的方法了。实际上它们依然很是很常用于像上面那样的场合，并且对于集中式和分散式的版本控制系统来说依然是很重要的。

CVS 和 Subversion
-----------------

为了解决多个开发者基于同一个代码做修改的问题，人们就开发了 *中心化版本控制系统* ，最早的是 `协作版本系统 (CVS) <http://zh.wikipedia.org/wiki/%E5%8D%94%E4%BD%9C%E7%89%88%E6%9C%AC%E7%B3%BB%E7%B5%B1>`_ ，之后出现了稍微高级一些的 `Subversion <http://zh.wikipedia.org/wiki/Subversion>`_\。这些系统的核心特性就是任何时刻或者任何修订版代码的官方版本都能从作为代码仓库的 *中心服务器* 上得到。如此获得的一个代码库被称为 *工作副本*\。

对于这些系统来说，基本的操作单位叫做 *变更集（changeset）*\。早期此类系统中最常见的向用户表现 *变更集* 的方式就是通过提供一个原型 ``diff`` 格式输出。这两种版本控制系统的工作方式都是记录变更集，而不是记录不同版本的原始文件本身。

这一代版本控制系统还引入了一些其他概念。例如 *分支（branch）* 一个项目，使得一个项目的多个不同版本可以同时存在同时修改，最终通过一些列测试和审查合并到主线或者叫 *主干（trunk）* 中。类似的一个概念是 *标签*\，可以将代码库的一个特定的版本标记为对应软件的一个发布版本。 ``merge``\（合并）的概念也引入了，允许手动解决因为对同一个文件的同部分修改所造成冲突。


Git 和 Mercurial
----------------

后一代版本控制系统则是 *分布式* ，或者叫 *无中心式* 的系统。在这些系统中工作副本包括了代码和项目的完整历史，所以不需要中心服务器就可以向这个项目提交修改。在开源、Unix 友好的环境中，突出的此类系统是 Git 和 Mercurial，它们的客户端程序是 ``git`` 和 ``hg``\。

这两种系统中，交换变更集的操作是 ``push``\（推送）， ``pull``\（拉取）和 ``merge``\（合并），一个仓库的修改可以被另一个接受。这样的无中心系统允许一种很复杂但是受到严格控制的开发生态。Git 就是 Linux Trovalds 为管理 Linux 内核开发工作而开发的分布式版本控制系统。

Git 和 Mercurial 不同于 CVS 和 Subversion，它们的基本操作单位不是修改集，而是压缩保存的完整的文件（blob）。这样搜索某单个文件的历史或者查阅某文件的两个版本间的修改成本会略高，但是对于每个修订版 ``git log --patch`` 命令仍然能输出统一 ``diff`` 格式，即便是在 ``diff`` 命令出现四十年之后的今天： ::
    
    commit c1e5559ddb09f8d02b989596b0f4100ad1aab422
    Author: Tom Ryder <tom@sanctum.geek.nz>
    Date:   Thu Feb 2 01:14:21 2012
    
        Changed my mind about this one.
    
    diff --git a/vim/vimrc b/vim/vimrc
    index cfbe8e0..65a3143 100644
    --- a/vim/vimrc
    +++ b/vim/vimrc
    @@ -47,10 +47,6 @@ set shiftwidth=4
     set softtabstop=4
     set tabstop=4
    
    -" Heresy
    -inoremap <C-a> <Home>
    -inoremap <C-e> <End>
    -
     " History
     set history=1000

这两种系统在功能甚至命令集上都有很多交叠，应该使用哪一种已经导致了 `大量争论 <http://stackoverflow.com/questions/35837/what-is-the-difference-between-mercurial-and-git>`_\。我见过的有关这两种系统的最好的介绍是 Scott Chacon 的 `Pro Git <http://progit.org/>`_ 和 Joel Spolsky 的 `Hg Init <http://hginit.com/>`_\。

结语
----

这是本系列文章的最后一篇。我试着给出了一个简捷的概览，介绍 Linux shell 里就已提供的一些基本工具和它们所提供的基本功能，而这些基本功能也恰好是一些专业 IDE 能够提供的。有时候我必须略过一些内容即便我想详细说明。不过我希望这些文章依然能让一个不熟悉在 Linux 系统上开发的人了解到这不起眼的 shell 也能成为非常全面的开发环境，并且完全使用的是免费、高度成熟且标准化的软件工具。
