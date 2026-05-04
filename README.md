# Code Auditor — AI 驱动的代码审计 Skill

> **教 AI 真正理解和运用静态语法纠错工具进行代码审计，而不是只会调用预设脚本。**

[![Trae IDE](https://img.shields.io/badge/Trae%20IDE-Skill-6C47FF)](https://www.trae.com.cn/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 项目简介

**Code Auditor** 是一个为 Trae IDE 设计的 AI 技能（Skill），它教会 AI 模型如何**真正理解**静态语法纠错工具的原理和用法，从而能够全自动化地对代码仓库进行语法检查、代码质量审计和安全扫描。

### 与普通 "脚本调用式" 审计的区别

| 普通做法 | Code Auditor 做法 |
|---------|------------------|
| AI 找到预设脚本 → 执行 → 粘贴结果 | AI **理解工具原理** → 根据场景灵活构建命令 → 智能解读输出 |
| 固定的工具组合，不懂为什么 | 理解编译器/Linter/格式化器的本质区别和各自定位 |
| 所有文件逐行读码审查（浪费 token） | **按需读码**：工具报错才读，warning/style 仅统计 |
| 默认全量扫描（或不扫描） | **先问用户**：根据策略选择不同深度的审计 |
| 不区分错误级别 | 区分确定性错误 vs 概率性警告 vs 约定性风格 |

---

## 核心思想

```
真正懂审计 = 理解工具原理 + 灵活构建命令 + 智能解读输出
```

本 Skill 将指导 AI 完成以下 10 步审计流程：

1. **发现目标** — 定位要审计的代码文件/目录
2. **克隆仓库** — 如果是 GitHub 链接，先 `git clone` 到本地
3. **语言检测** — 通过扩展名/配置文件识别语言
4. **询问策略** — 问用户要什么深度（语法检查/标准审计/深度安全）
5. **工具认知** — 根据策略选择工具类型
6. **构建命令** — 动态构造审计命令
7. **执行审计** — 运行工具，捕获输出
8. **解读结果** — 区分 error/warning/style
9. **按需读码** — 仅 error 才读码复核
10. **报告呈现** — 生成结构化审计报告

---

## 项目结构

```
.trae/skills/code-auditor/
├── SKILL.md              ← 技能主文件（教 AI 理解审计）
└── scripts/              ← 已删，改为纯教学（备份在 scripts-backup/）
```

> **理念**：我们选择教 AI "渔" 而不是给 AI "鱼"。不再提供预设脚本，而是让 AI 真正理解每个工具的工作原理，从而能灵活应对各种项目场景。

---

## 支持的 30+ 种编程语言

包括但不限于：

| 语言 | 编译器语法检查 | Linter / 其他工具 |
|------|--------------|------------------|
| C/C++ | `gcc -fsyntax-only`, `clang -fsyntax-only` | cppcheck, clang-tidy, sparse |
| Java | `javac -Xlint:all` | PMD |
| Python | `python -m compileall` | pyflakes, ruff, flake8 |
| JavaScript | `node --check` | ESLint, JSHint, quick-lint-js |
| TypeScript | `tsc --noEmit` | ESLint (@typescript-eslint) |
| Go | `go vet` | gofmt, golangci-lint, staticcheck |
| Rust | `rustc --emit metadata`, `cargo check` | rustfmt |
| Ruby | `ruby -c` | RuboCop |
| PHP | `php -l` | parallel-lint, PHPCS |
| Shell | `bash -n` | ShellCheck |
| C# | `dotnet build --no-restore` | dotnet format |
| Kotlin | `kotlinc -Werror` | ktlint, detekt |
| Swift | `swiftc -parse` | SwiftLint |
| Dart/Flutter | `dart analyze` | dart format |
| Lua | `luac -p` | luacheck |
| Perl | `perl -c` | PerlCritic |
| Haskell | `ghc -fno-code` | HLint |
| Elixir | `mix compile --warnings-as-errors` | Credo |
| 以及其他 15+ 种语言... | | |

---

## 三层审计策略

AI 在执行审计前会主动向用户确认审计深度：

### ❶ 快速语法检查
```
目的：最快速度验证代码是否能通过编译/解析
工具：仅编译器语法检查命令，不用 linter
耗时：数秒
适用：提交前快速验证、CI 预检查、修改后自查
```

### ❷ 标准质量审计
```
目的：平衡速度与覆盖面，常规代码审查
工具：编译器语法检查 + 主要 Linter
耗时：数十秒到数分钟
适用：MR/PR 审查、定期代码巡检、日常质量把控
```

### ❸ 深度安全审计
```
目的：最全面的检查，不遗漏任何潜在风险
工具：编译器 + Linter + 安全专项工具 + 最严格参数
耗时：数分钟到十分钟
适用：上线前安全检查、第三方代码审计、安全合规
```

---

## 审计示例

### 单文件 Python 审计

```bash
# 快速语法检查
python -m compileall -q main.py

# 标准质量审计
python -m compileall -q .
ruff check .

# 深度安全审计
python -m compileall -q .
ruff check .
flake8 .
```

### 完整 Rust 项目审计

```bash
# cargo check 在某些情况下可以替代编译检查
cargo check --workspace 2>&1
rustfmt --check src/**/*.rs
```

---

## 输出解读

AI 能智能识别工具输出的不同模式：

| 输出匹配 | 严重程度 | 处理方式 |
|---------|---------|---------|
| `error`, `ERROR`, `SyntaxError` | 真·错误 | 读码复核 |
| `warning`, `WARN`, `caution` | 潜在问题 | 仅统计 |
| `note`, `info`, `style` | 风格建议 | 仅统计 |
| `command not found` | 环境缺失 | 跳过并报告 |

---

## 安装使用

### 前提条件

- [Trae IDE](https://www.trae.com.cn/) 已安装
- 目标语言的审计工具已安装（如 `gcc`、`ruff`、`eslint` 等）

### 安装步骤

1. 克隆本仓库：
```bash
git clone https://github.com/your-username/code-auditor-skill.git
```

2. 将 `code-auditor` 文件夹复制到 Trae 的 skills 目录：
```bash
# Windows
copy -Recurse .trae/skills/code-auditor %USERPROFILE%/.trae/skills/code-auditor
```

3. 在 Trae IDE 中向 AI 发起审计请求即可自动触发。

---

## Skill 设计原则

1. **先问再动手** — 每次审计前先确认策略深度
2. **理解而非记忆** — 教 AI 理解工具原理，而非背诵命令
3. **按需读码** — 只有 error 才读码，节省 token
4. **容错执行** — 工具缺失时跳过，不中断审计
5. **非破坏性** — 所有命令均为只读
6. **灵活适配** — 根据项目结构/语言/环境动态调整

---

## 许可证

[MIT License](LICENSE)
