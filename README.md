# CTF Writeups & Linux Notes

picoCTF writeups + notes on Linux internals and security fundamentals.

> 中文为主，每篇都带英文标题（方便搜索）。
> Most notes are in Chinese; English titles are listed for each.

## 关于

- 2026-09-20 开始学网络安全
- 平台：[picoCTF](https://picoctf.org/)
- 环境：Windows 11 + WSL2 (Ubuntu)

## 目录

### picoCTF

| Problem | Difficulty | Topic |
|---|---|---|
| [First Grep](picoCTF/First-Grep.md) | Easy | Searching large files with `grep` |

### Linux 笔记 / Linux Notes

| 标题 / Title | 讲什么 / What it covers |
|---|---|
| [为什么 `rm` 掉的文件还占着磁盘](linux/deleted-files.md)<br>**Why Deleted Files Still Take Up Disk Space** | `(deleted)` 标记、`lsof` 反查、为什么 `df` 和 `du` 对不上 |
| [Linux 入侵排查：三条线，一个 `/proc`](linux/incident-response.md)<br>**Linux Incident Response: Three Lines of Investigation, One `/proc`** | 挖矿 / 后门 / 自我删除的三种特征、可疑度判据、`kill` 与信号 |
| [为什么 MD5 不能用来存密码](linux/hashing.md)<br>**Why MD5 Shouldn't Be Used for Storing Passwords** | 雪崩效应、彩虹表、为什么"快"是致命的、盐（salt）能防什么不能防什么 |
| [为什么 18K 的文件能压到 101 字节](linux/compression.md)<br>**Why an 18K File Compresses to 101 Bytes** | `gzip` / `tar` 的三件事、压缩效果取决于重复度、`-t` 为什么安全 |
| [解压一个压缩包，文件为什么会跑到别的目录](linux/path-traversal.md)<br>**Why Extracting an Archive Can Write Files Outside the Target Directory** | 路径穿越（tar traversal / zip slip）、为什么 `tar` 不拦、解压前先看 |

## Topics covered / 涉及的主题

`linux` `bash` `command-line` `security` `reverse-engineering` `file-systems` `processes` `hashing` `cryptography` `compression` `path-traversal` `incident-response`

## ⚠️ 关于 flag

**本仓库所有 flag 均已打码。**

原因：picoCTF 的题目使用**静态 flag**（所有人一样）。
直接写出来等于给答案，会破坏题目。

想学的话，自己去做一遍 —— 过程才是重点。

## 为什么写这些

- 检验自己是不是真的懂了（能写清楚才算懂）
- 我踩过的坑，别人可能也在踩
- 攒一个能给别人看的东西

