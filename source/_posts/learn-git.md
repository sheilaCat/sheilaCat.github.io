title: git学习笔记
date: 2015-11-17 22:19:57
tags: [git,github]
categories: 读书笔记
---
## 版本控制（VCS）的类型

* 本地版本控制系统
* 集中化的版本控制系统(Centralized Version Control Systems）
* 分布式版本控制系统(Distributed Version Control System）
分布式版本控制系统的客户端，每一次的提取操作，实际上都是一次对代码仓库的完整备份。这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。

<!-- more -->

## Git基础
* 直接记录快照，而非差异比较
* 近乎所有操作都是本地执行
* 时刻保持数据完整性
在保存到Git之前，所有数据都要进行内容的校验和（checksum）计算，并将此结果作为数据的唯一标识和索引。Git使用SHA-1算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个SHA-1哈希值，作为指纹字符串。该字串由40个十六进制字符（0-9及a-f）组成。
* 多数操作仅添加数据
* 文件的三种状态
对于任何一个文件，在 Git 内都只有三种状态：`已提交（committed）`，`已修改（modified）`和`已暂存（staged）`。已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交保存；已暂存表示把已修改的文件放在下次提交时要保存的清单中。
由此我们看到 Git 管理项目时，文件流转的三个工作区域：Git 的工作目录，暂存区域，以及本地仓库。

### 在工作目录中初始化新仓库
```javascript
$ git init
```

### 记录每次更新到仓库

![img](http://7xoefy.com1.z0.glb.clouddn.com/FileStatusLifecycle.png)

### 检查当前文件状态
```javascript
$ git status
```

### 暂存已修改文件
```javascript
$ git add
```
`git add`命令（这是个多功能命令，根据目标文件的状态不同，此命令的效果也不同：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等）


### 忽略某些文件

```javascript
$ cat .gitignore
*.[oa]
*~
```

文件 .gitignore 的格式规范如下：

* 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。
* 可以使用标准的 glob 模式匹配。
* 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。星号（*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。


### 查看已暂存和未暂存的更新

```javascript
$ git diff
```
`git diff`会使用文件补丁的格式显示具体添加和删除的行。
此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后***还没有暂存起来的变化内容***。
若要看已经暂存起来的文件和上次提交时的快照之间的差异，可以用`git diff --cached`命令。（Git 1.6.1及更高版本还允许使用`git diff --staged`，效果是相同的）

### 提交更新
`git commit`命令，之后会进入文本编辑环境要求输入本次提交的说明。
`git commit -m "xxx"`
`git commit -a -m "xxx"` 跳过使用暂存区域，`-a`选项Git自动把所有已经跟踪过的文件暂存起来一并提交。

### 移除文件
要从Git中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。`git rm`，并连带从工作目录中删除指定的文件。

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。比如一些大型日志文件或者一堆 .a 编译文件，不小心纳入仓库后，要移除跟踪但不删除文件，以便稍后在 .gitignore 文件中补上，用 --cached 选项即可：
```javascript
$ git rm --cached readme.txt
```

### 移动文件
```javascript
$ git mv file_from file_to
```
重命名文件，其实，运行`git mv`就相当于运行了下面三条命令
```javascript
$ mv README.txt README
$ git rm README.txt
$ git add README
```

### 查看提交历史
`git log`命令

`git log -p -2`命令     `-p`选项展开显示每次提交的内容差异，用`-2`则仅显示最近的两次更新

`git log --stat`仅显示简要的增改行数统计

```javascript

选项 说明

    -p 按补丁格式显示每个更新之间的差异。

    --stat 显示每次更新的文件修改统计信息。

    --shortstat 只显示 --stat 中最后的行数修改添加移除统计。

    --name-only 仅在提交信息后显示已修改的文件清单。

    --name-status 显示新增、修改、删除的文件清单。

    --abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。

    --relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）。

    --graph 显示 ASCII 图形表示的分支合并历史。

    --pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。

```

更有意思的是`format`，可以定制要显示的记录格式，这样的输出便于后期编程提取分析：
```javascript

$ git log --pretty=format:"%h - %an, %ar : %s"

    ca82a6d - Scott Chacon, 11 months ago : changed the version number

    085bb3b - Scott Chacon, 11 months ago : removed unnecessary test code

    a11bef0 - Scott Chacon, 11 months ago : first commit

```

列出了常用的格式占位符写法及其代表的意义。
```javascript

选项 说明

    %H 提交对象（commit）的完整哈希字串

    %h 提交对象的简短哈希字串

    %T 树对象（tree）的完整哈希字串

    %t 树对象的简短哈希字串

    %P 父对象（parent）的完整哈希字串

    %p 父对象的简短哈希字串

    %an 作者（author）的名字

    %ae 作者的电子邮件地址

    %ad 作者修订日期（可以用 -date= 选项定制格式）

    %ar 作者修订日期，按多久以前的方式显示

    %cn 提交者(committer)的名字

    %ce 提交者的电子邮件地址

    %cd 提交日期

    %cr 提交日期，按多久以前的方式显示

    %s 提交说明

```

还有时间限制，比如`--since`和`--until`，`git log --since=2.weeks`
```javascript

选项 说明

    -(n) 仅显示最近的 n 条提交

    --since, --after 仅显示指定时间之后的提交。

    --until, --before 仅显示指定时间之前的提交。

    --author 仅显示指定作者相关的提交。

    --committer 仅显示指定提交者相关的提交。

```

### 使用图形化工具查阅提交历史
随Git一同发布的gitk，基本上相当于`git log`命令的可视化版本。

### 撤销操作

#### 修改最后一次提交
```javascript
$ git commit --amend
```

#### 取消已经暂存的文件
```javascript
$ git reset HEAD filename
```

#### 取消对文件的修改
取消对工作目录文件的修改
```javascript
$ git checkout -- filename
```

### 远程仓库的使用

#### 查看当前的远程库
`git remote`
`git remote -v`显示对应的克隆地址

#### 添加远程仓库
`git remote add [shortname] [url]`

#### 查看远程仓库信息
`git remote show [remote-name]`

#### 远程仓库的删除和重命名
`git remote rename`命令修改某个远程仓库在本地的简称
`git remote rm branchname`移除对应的远端仓库


参考http://git.oschina.net/progit/

最后附上一张GIT API总结图，图片来自网络

![img](http://7xoefy.com1.z0.glb.clouddn.com/git-api.png)