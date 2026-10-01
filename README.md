# CTF Writeups & Linux 笔记

picoCTF 解题记录 + 一些 Linux 底层概念的整理。

## 关于

- 2026-09-20 开始学网络安全
- 平台：[picoCTF](https://picoctf.org/)
- 环境：Windows 11 + WSL2 (Ubuntu)

## 目录

### picoCTF

| 题目 | 难度 | 考点 |
|---|---|---|
| [First Grep](picoCTF/First-Grep.md) | Easy | 用 `grep` 搜索大文件 |

### Linux 笔记

| 标题 | 讲什么 |
|---|---|
| [为什么 `rm` 掉的文件还占着磁盘](linux/deleted-files.md) | `(deleted)` 标记、`lsof` 反查、为什么 `df` 和 `du` 对不上 |
| [Linux 入侵排查：三条线，一个 `/proc`](linux/incident-response.md) | 挖矿 / 后门 / 自我删除的三种特征、可疑度判据、`kill` 与信号 |
| [为什么 MD5 不能用来存密码](linux/hashing.md) | 雪崩效应、彩虹表、为什么"快"是致命的、盐（salt）能防什么不能防什么 |

## ⚠️ 关于 flag

**本仓库所有 flag 均已打码。**

原因：picoCTF 的题目使用**静态 flag**（所有人一样）。
直接写出来等于给答案，会破坏题目。

想学的话，自己去做一遍 —— 过程才是重点。

## 为什么写这些

- 检验自己是不是真的懂了（能写清楚才算懂）
- 我踩过的坑，别人可能也在踩
- 攒一个能给别人看的东西

