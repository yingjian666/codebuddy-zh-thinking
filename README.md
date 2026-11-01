# codebuddy-zh-thinking

**Keywords / 搜索词**：CodeBuddy 中文思考 · CodeBuddy thinking Chinese · CodeBuddy skill · CodeBuddy plugin · 强制简体中文思维链 · force thinking in Chinese · chain-of-thought Chinese · system prompt skill injection · DeepSeek 中文思考 · AI Agent 中文推理 · Simplified Chinese CoT

让 **CodeBuddy** 里模型的**内部思考过程**（思维链、逐步规划、工具调用前后的推理、自我审查）强制使用**简体中文**。

> 这是 [dsh-zh-thinking](https://github.com/yingjian666/dsh-zh-thinking)（DeepSeek Harness 版）思路在 CodeBuddy 上的实现，与 [workbuddy-zh-thinking](https://github.com/yingjian666/workbuddy-zh-thinking)（WorkBuddy 版）是姊妹仓库。三者思路同源、代码各自独立，按所用产品二选一安装即可。

## 真机实测矩阵（2026-11-01）

| CodeBuddy 接入的模型 | 结果 |
|---|---|
| DeepSeek（第三方中转 API） | ✅ 思考英文 → 中文，全量生效 |
| 官方 GLM | ❌ 依旧英文 |

GLM 不生效是模型训练偏好：GLM 的推理语言压不过提示层引导。这不是安装问题——同一份技能、同一个内核，换 DeepSeek 就生效。**选一个推理语言听提示的模型，是本方案的前提。**

* * *

## 快速安装

```bash
git clone https://github.com/yingjian666/codebuddy-zh-thinking.git
cd codebuddy-zh-thinking
node install.mjs
```

**然后重启 CodeBuddy**。新会话里丢一段英文材料（英文报错、英文文档都行），观察思考块的语言。

卸载：

```bash
node install.mjs --uninstall
```

> **WorkBuddy 用户请移步姊妹仓库**：[workbuddy-zh-thinking](https://github.com/yingjian666/workbuddy-zh-thinking)（同一思路的 WorkBuddy 版，独立安装、独立维护）。

* * *

## 原理：为什么这样做

CodeBuddy（同 WorkBuddy，均为 CodeBuddy Code 内核家族——WorkBuddy 即该内核的定制发行版）**没有对外暴露**系统提示词装配接口，扩展点只有：MCP、Skill、插件 hooks、记忆注入。

其中唯一能进系统提示的落点是：

> **Skill 的 `description` 字段会随技能清单每轮拼进系统提示。**

这是本项目全部核心：把「必须用简体中文思考」的强制规则前置写进 `SKILL.md` 的 `description`，让它常驻在模型眼前。

### 一个被证伪的猜想

先试的是往用户级 `MEMORY.md` 写规则。日志证实该文件每轮重读、规则确实进了上下文——**但思考仍是英文**。一条「用户指令」压不住模型的默认习惯。换成 `description` 后立刻生效。这个对照实验是整件事的关键证据。

* * *

## 实现的两层

| 层 | 载体 | 位置强度 | 生效范围 |
|---|---|---|---|
| 1 | `SKILL.md` 的 `description` | 系统提示（技能清单，每轮常驻） | 全局 |
| 2 | `~/.codebuddy/MEMORY.md` 标记块 | 用户指令（推测路径，未真机验证） | 全局 |

层 1 是决定性的，层 2 是保险（CodeBuddy 侧该路径尚未验证，desc 层不受影响）。`manage.mjs off` 只摘层 2，**要彻底关闭得禁用或删除这个 skill**。

* * *

## 目录结构

```
.
├── SKILL.md          # 技能定义，description 里就是强制规则（核心）
├── manage.mjs        # MEMORY.md 规则块的 on/off/status/path
├── install.mjs       # 安装/卸载到 ~/.codebuddy/skills/
├── CHANGELOG.md
├── LICENSE
└── README.md
```

* * *

## 命令

安装后：

```bash
node ~/.codebuddy/skills/zh-thinking/manage.mjs status
node ~/.codebuddy/skills/zh-thinking/manage.mjs on
node ~/.codebuddy/skills/zh-thinking/manage.mjs off

# 直接跑仓库里的也可以，不依赖安装位置
node manage.mjs on
```

Windows 下若 `$HOME` 不展开，用绝对路径：
`node "C:/Users/<用户名>/.codebuddy/skills/zh-thinking/manage.mjs" on`

脚本是幂等的：`on` 按 `<!-- zh-thinking:begin -->` / `<!-- zh-thinking:end -->` 标记定位，已存在则原样替换、不会堆叠；`off` 只删标记块，`MEMORY.md` 里其他记忆一字不动。

* * *

## 生效时机

- **层 1（description）**：改完需要**重启 CodeBuddy**。技能清单在启动时索引，重启后每轮都在。
- **层 2（MEMORY.md）**：改完下一轮即进上下文。

注意：当前会话不会回溯改变——已发出的请求，系统提示早就组装完了。

* * *

## 边界与代价（请先读这段）

- **这是强引导，不是物理强制。** 本项目只能做到"在系统提示里加一条约束"，做不到"重写整个提示词"。
- **模型必须服从提示。** DeepSeek 系实测全量生效；官方 GLM 实测**不生效**。选对模型是前提，不是缺陷。
- **规则只管语言。** 代码、命令、路径、API 名、报错堆栈必须保留原文——翻译了会破坏可执行性。
- **依赖 skill 处于启用状态。** 在界面上禁用该 skill，约束立刻消失。
- **description 可能被压缩。** 技能数量激增时应用可能截断描述，核心句必须留在**最前面**。

* * *

## 兼容性

- CodeBuddy CN（`out/codebuddy/main.js` 版本 2026-10 前后）实测有效（DeepSeek 第三方 API）。
- 不依赖任何第三方 npm 包，纯 Node 内置模块。
- 修改的文件都在用户目录 `~/.codebuddy/` 下，**不触碰应用安装目录**，升级 CodeBuddy 不会失效。

* * *

## 许可证 / 致谢

MIT · Copyright (c) 2026 yingjian666

思路延续自两个项目，特此致谢：

- [dsh-zh-thinking](https://github.com/yingjian666/dsh-zh-thinking) —— 同作者的 DSH 版
- [wodongx123/dsh-language-control](https://github.com/wodongx123/dsh-language-control) —— 「让 Agent 用中文思考」思路的最初来源（MIT License, Copyright (c) 2026 wodongx123）

姊妹仓库：[workbuddy-zh-thinking](https://github.com/yingjian666/workbuddy-zh-thinking)。本项目针对 CodeBuddy 的扩展点独立实现，与上述项目代码无共用。
