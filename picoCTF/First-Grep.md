# picoCTF — First Grep

> 难度：Easy ｜ 平台：picoCTF ｜ 日期：2026-09-20

## 题目

给一个文本文件，里面藏着 flag。

考察点：**当数据量大到人眼看不过来时，用命令行工具代替眼睛。**

## 环境

- Windows 11 + WSL2 (Ubuntu)
- 用到：`file`、`grep`

## 解题过程

1. 下载题目文件（文件名就叫 `file`，14546 字节）
2. `file file` → 返回 `ASCII text, with very long lines`
3. 用 `grep` 搜 flag

**最终命令：**

```bash
grep -o "picoCTF{[^}]*}" file
```

**输出（flag 已打码）：**

```
picoCTF{grep_is_good_to_find_things_********}
```

## 卡住的地方

### 1. WSL 里 Windows 路径怎么写（卡了约 15 分钟）

一开始敲：

```bash
ls C:\Users\Admin\Downloads\file
```

报错 `No such file or directory`，而且报错信息里的路径变成了
`C:UsersAdminDownloadsfile` —— **所有反斜杠都不见了**。

**原因**：Linux 里 `\` 是**转义符**，会被 shell 吃掉。

**正确写法：**

```bash
ls /mnt/c/Users/Admin/Downloads/file
```

- 开头用 `/mnt/c/`，不是 `C:\`
- 分隔符用正斜杠 `/`

### 2. 家目录路径写死，失败

从提示符里看到了用户名，就照着猜 `/home/<用户名>/`，结果报错。

**原因**：当时是从「管理员 PowerShell」里敲 `wsl` 进去的，
WSL 直接给了 **root** 身份，家目录是 `/root`。

**教训**：不要写死路径，用 `~`；或者先跑 `whoami` / `echo $HOME` 确认。

### 3. 以为文件是"乱码"

打开看到一堆无意义字符（`r9f<|r(MYy|BM.7Vxt?...`），
以为文件损坏、需要解码。

**实际**：这些字符是**题目故意生成的**，目的就是把 flag 淹掉，
逼你用搜索工具。文件只有 1 行、14546 字节 —— 这就是
`file` 提示 `with very long lines` 的意思。

## 关键认识

**不要试图"看懂"文件，要学会"搜索"文件。**

**为什么用 `-o`：**

| 命令 | 结果 |
|---|---|
| `grep "picoCTF" file` | 输出**整行** —— 14546 个字符刷屏 |
| `grep -o "picoCTF" file` | 只输出**匹配到的那段** |

**为什么模式写成 `"picoCTF{[^}]*}"`：**

| 部分 | 含义 |
|---|---|
| `picoCTF{` | flag 固定开头 |
| `[^}]*` | 任意数量的、**不是 `}` 的字符** |
| `}` | flag 固定结尾 |

`[^}]*` 是精髓 —— **利用 flag「以 `}` 结尾」的格式特征**，
精确框出 `{` 和 `}` 之间的内容。

如果只用 `.*`（贪婪匹配），会一路吃到文件末尾。

## 总结

1. `/mnt/c/` 是 WSL 访问 Windows 文件的入口
2. 路径里必须用正斜杠，反斜杠会被 shell 吃掉
3. `~` 比写死路径安全
4. **`file` 是排查陌生文件的第一步** —— 先摸清是什么，再动手
5. **"看不懂"不是障碍，"不会搜"才是**
6. 管理员 PowerShell 里敲 `wsl` 会进 root，敲错 `rm` 没有确认提示

## 命令速查

| 命令 | 作用 |
|---|---|
| `pwd` | 当前目录 |
| `whoami` | 当前用户 |
| `echo $HOME` | 家目录 |
| `ls -la` | 列出所有文件（含隐藏） |
| `file 文件名` | 判断文件类型 |
| `grep "内容" 文件` | 搜索文本 |
| `grep -o "模式" 文件` | 只输出匹配部分 |
| `grep -c "内容" 文件` | 统计匹配行数 |
| `/mnt/c/` | WSL 里的 C 盘 |
| `~` | 家目录 |
