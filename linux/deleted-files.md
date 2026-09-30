# 为什么 `rm` 掉的文件还占着磁盘？

> 一次 `df` 和 `du` 对不上的排查记录

## 起因

删了一个大文件，`df -h` 一看 —— 空间没回来。

```bash
$ ls -lh /tmp/huge.log
-rw-r--r-- 1 user user 2.0G /tmp/huge.log

$ rm /tmp/huge.log
$ df -h /tmp
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb        100G   78G   17G  83% /
```

**78G 还是 78G。文件删了，空间没还。**

## 一个反直觉的事实

`rm` **不删除文件内容**。

它只做一件事：

> **删掉「文件名 → inode」这条链接。**

用图书馆打比方：

| | 类比 |
|---|---|
| 文件名 | 书架上的**标签** |
| inode | 书本身（记着内容存在哪几个书架上） |
| `rm` | **撕掉标签**，书还在架子上 |

**一旦撕掉标签，你就"看不见"这本书了 —— 但它没被销毁。**

## 什么时候才算真的删掉

inode 上有一个**引用计数**（referenced count），记录"现在有几个东西指着我的内容"。

一删回到零，内核才回收内容。**有两个来源会占着这个计数：**

1. **硬链接** —— 每个指向它的文件名
2. **打开它的进程** —— 每个还在读/写它的进程

**只要有一项不为零 → 内容就在 → 空间就不还。**

这次是第 2 种：**有个进程还开着它。**

## 怎么找出是谁占着

```bash
$ lsof | grep deleted
```

`lsof` = list open files。它能列出**所有进程打开着的所有文件**，包括那些已经"没有名字"的。

输出里会出现这么一行：

```
COMMAND    PID  USER   FD   TYPE  DEVICE  SIZE/OFF   NODE   NAME
syslogd    812  root   7w   REG   8,16   2147483648  5231   /var/log/huge.log (deleted)
```

**关键在最后一列**：`/var/log/huge.log (deleted)`

- `(deleted)` = 这个文件的名字已经没了
- 但 `syslogd` 这个进程还开着它（`FD` 列是 `7w`，`w` = 只写）

**`lsof` 的数据其实来自 `/proc/<PID>/fd`** —— 直接看也一样：

```bash
$ sudo ls -l /proc/812/fd/7
lrwx------ 1 root root 64 Sep 29 22:10 /proc/812/fd/7 -> /var/log/huge.log (deleted)
```

**`/proc` 不是真的目录**，是内核现算给你看的窗口。所以它能显示"已删但未释放"这种硬盘上根本查不到的状态。

## 空间怎么拿回来

**别去 `rm` —— 已经删过了。要么让那个进程关掉文件，要么重启它。**

```bash
$ sudo kill 812          # 进程一死，它打开的所有 fd 自动关闭
$ df -h /tmp             # 空间回来了
```

或者更安全一点，重启那个服务：

```bash
$ sudo systemctl restart syslogd
```

## 为什么这事在安全排查里重要

**1. 反外挂 / 反取证都爱用这招**

程序跑起来之后，**第一件事是把自己删掉**：

```bash
$ cp /tmp/malware /tmp/runme
$ /tmp/runme &
$ rm /tmp/runme
```

这样一来：

- `ls`、`find` 找不到它
- 磁盘上"没有"这个文件

**但 `/proc/<PID>/exe` 还指着它：**

```bash
$ ls -l /proc/1234/exe
lrwxrwxrwx 1 root root 0 Sep 29 22:15 /proc/1234/exe -> /tmp/runme (deleted)
$ cp /proc/1234/exe /tmp/recovered    # 捞出来了
```

**→ 看到 `(deleted)` 就是一条强信号**：这个进程的文件"凭空消失"了。

**2. 磁盘满了但 `du` 说没满**

`df`（看文件系统层）和 `du`（看文件层）对不上，最常见的原因就是这个：

```bash
$ df -h / | tail -1
/dev/sda1   100G   95G   3G   96%   /     ← 快满了

$ sudo du -sh /var/log
1.2G    /var/log                          ← 才 1.2G？
```

差的那几十 G，就是被"删了但没释放"的文件占着。**`lsof | grep deleted` 一跑就现形。**

## 一句话总结

> **`rm` 删的是名字，不是内容。**
> **只要还有进程开着它（或还有硬链接指着它），内容就还在，空间就不还。**
> **`lsof | grep deleted` 找出是谁，`kill` 掉它，空间就回来了。**

## 相关命令速查

| 命令 | 作用 |
|---|---|
| `lsof` | 列出所有打开的文件（数据来自 `/proc/*/fd`） |
| `lsof \| grep deleted` | 找"已删除但仍被占用"的文件 |
| `lsof -p <PID>` | 看某个进程打开了哪些文件 |
| `ls -l /proc/<PID>/fd` | 同上，看原始数据 |
| `ls -l /proc/<PID>/exe` | 看进程在跑哪个可执行文件 |
| `cp /proc/<PID>/exe 目标` | 从运行中的进程里把文件捞出来 |
| `df -h` vs `du -sh` | 两者对不上 = 有这个问题的信号 |
