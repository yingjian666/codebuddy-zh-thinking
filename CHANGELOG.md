# Changelog

## 0.1.0 — 2026-11-01

首个版本（从 [workbuddy-zh-thinking](https://github.com/yingjian666/workbuddy-zh-thinking) 拆分而来，两仓库独立维护）。

**做了什么**

- 用 Skill 的 `description` 字段把「必须用简体中文思考」的规则写进 CodeBuddy 的系统提示（技能清单每轮拼入系统提示，desc 是唯一能进系统提示的自定义常驻文本）。
- `manage.mjs` 管理用户级 `~/.codebuddy/MEMORY.md` 的规则块，作为第二层保险，可幂等开关。
- `install.mjs` 一键安装 / 卸载到 `~/.codebuddy/skills/zh-thinking/`。

**真机实测（2026-11-01）**

| CodeBuddy 接入的模型 | 结果 |
|---|---|
| DeepSeek（第三方中转 API） | ✅ 思考英文 → 中文，全量生效 |
| 官方 GLM | ❌ 依旧英文 |

GLM 不生效属模型训练偏好，提示层压不住；选对模型是本方案的前提。

**被证伪的方案**

先试往用户级 `MEMORY.md` 写规则：文件确实每轮重读、规则确实进了上下文，但思考仍为英文。换成 `description` 后立刻生效（DeepSeek）。对照实验是关键证据。
