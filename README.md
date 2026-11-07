# DSHIO Standards

**The third-party standard for DSH (DeepSeek Harness) plugins.**
**DSH（DeepSeek Harness）插件的第三方标准体系。**

[中文说明见下方](#中文说明) · [Website](https://dshio.org) · [Checker](https://github.com/moldkey/dshio-checker)

---

## What this is

DSHIO is an independent third-party standard system for DSH plugins, written and
maintained by DSHIO.

**It is not an official DeepSeek specification.** The standard is derived from the
observed behaviour of the official development kit, but:

- It does **not** require the official development kit
- It does **not** require any specific language, build tool, or dependency
- Conformance is judged by **observable runtime behaviour**, not by source structure

Anyone who follows the standard produces a conforming plugin — however they build it.

---

## Standards

| No. | Title | Layer | Status |
|---|---|---|---|
| DSHIO/T 00—2026 | Terms and Definitions | Basic | Trial |
| DSHIO/T 01—2026 | Rules for Writing Standards | Basic | Trial |
| DSHIO/T 10—2026 | Plugin Standard | Product | Trial |
| DSHIO/T 20—2026 | General Rules for Testing | Method | Trial |
| DSHIO/T 21—2026 | Test Conditions | Method | Trial |
| DSHIO/T 32—2026 | Certification Rules | Management | Trial · Not in effect |
| 01 Framework | Standards System Framework | — | Trial |

**Trial period: 2026-11-07 → 2027-02-07.** Standards automatically become
"current" when the period ends; no re-publication is needed.

---

## Current scope

| Stage | Name | Status |
|---|---|---|
| T0 | Static check | ✅ Available |
| T1 | Install | ✅ Available |
| T2 | List | ✅ Available |
| T3 | Mount | ✅ Available |
| T4 | Boot | ✅ Available |
| **T5** | **Tool invocation** | 🕐 **Planned** |

**Why T5 is only planned:** it involves model cost and nondeterminism
(see DSHIO/T 32 clause 4.3).

**Manual-judgement clauses (5.5.1, 5.5.2) are the author's own responsibility.**
DSHIO does not issue verdicts on them. Producing an opinion on them is a paid service.

---

## Certification status

**DSHIO issues *inspection reports*, not *certificates*.**

| Output | Available | Notes |
|---|---|---|
| Inspection report | ✅ Yes | Covers T0–T4, with environment fingerprint, repeat runs, overall verdict |
| Certificate | ❌ No | DSHIO/T 32 is still "Trial · Not in effect" |
| L3 level | 🕐 Planned | Requires T5 |

**An inspection report is not a certificate. Any claim that a report constitutes
certification is false.**

---

## How to cite

```
DSHIO/T 10—2026, clause 5.2.2
```

Do **not** write `DSHIO-DEV` (retired code name) or omit the standard number.

---

## Repository layout

```
standards/                   Standard text (Markdown)
  DSHIO-T-00-2026-...md
  DSHIO-T-01-2026-...md
  DSHIO-T-10-2026-...md
  DSHIO-T-20-2026-...md
  DSHIO-T-21-2026-...md
  DSHIO-T-32-2026-...md
  01-...md
machine/                     Machine-readable clauses
  DSHIO-T-10.json            17 clauses that can be checked automatically
```

The Markdown source is in Simplified Chinese; an English edition is in progress.

---

## Related

| Project | What it is |
|---|---|
| **https://dshio.org** | Web edition of the standards |
| [dshio-checker](https://github.com/moldkey/dshio-checker) | Open-source conformance checker |
| [dsh-deepseekbase](https://github.com/moldkey/dsh-deepseekbase) | Data tools + standards skills for DSH |

---

## License

- **Standard text** (`standards/`, `machine/`) — [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- **Everything else** — MIT

Quoting, redistribution and translation are permitted with attribution.

---

## Independence

DSHIO is an independent third-party standards body with no affiliation to
DeepSeek and no endorsement from it.

The standards are independently derived and are not an official specification.
For anything official, use these:

- [DeepSeek](https://www.deepseek.com)
- [DeepSeek Harness](https://deepseek.com/harness)
- [Official repository](https://github.com/deepseek-ai/deepseek-harness)
- [API docs](https://api-docs.deepseek.com)

---

# 中文说明

**DSH（DeepSeek Harness）插件的第三方标准体系。**

## 这是什么

DSHIO 是一套针对 DSH 插件的独立第三方标准，由 DSHIO 编写与维护。

**不是 DeepSeek 官方规范。** 标准从官方开发套件的实际行为归纳而来，但：

- 不要求使用官方开发套件
- 不要求特定编程语言、构建工具或依赖
- 判定以**可观测的运行时行为**为准，不以源码结构为准

任何人按标准写出的插件都是合格的 —— 不论他怎么实现。

## 标准清单

| 标准号 | 名称 | 层级 | 状态 |
|---|---|---|---|
| DSHIO/T 00—2026 | 术语和定义 | 基础 | 试行 |
| DSHIO/T 01—2026 | 标准编写规则 | 基础 | 试行 |
| DSHIO/T 10—2026 | 插件标准 | 产品 | 试行 |
| DSHIO/T 20—2026 | 测试方法总则 | 方法 | 试行 |
| DSHIO/T 21—2026 | 测试条件 | 方法 | 试行 |
| DSHIO/T 32—2026 | 认证规则 | 管理 | 试行 · 未启用 |
| 01 框架 | 标准体系框架 | — | 试行 |

**试行期：2026-11-07 至 2027-02-07**，期满自动转为现行，不需重新发布。

## 当前实施范围

| 阶段 | 名称 | 状态 |
|---|---|---|
| T0 | 静态检查 | ✅ 已提供 |
| T1 | 安装 | ✅ 已提供 |
| T2 | 列装 | ✅ 已提供 |
| T3 | 挂载 | ✅ 已提供 |
| T4 | 启动 | ✅ 已提供 |
| **T5** | **工具调用** | 🕐 **待规划** |

**T5 为何待规划：** 涉及模型成本与随机性（见 DSHIO/T 32 第 4.3 条）。

**人工判断条款（5.5.1、5.5.2）由开发者自行判定。**
DSHIO 不出具这两条判定结论；如需出具意见，属付费服务。

## 认证状态

**DSHIO 出具「检验报告」，不出具「认证证书」。**

| 产出 | 是否提供 | 说明 |
|---|---|---|
| 检验报告 | ✅ 提供 | 覆盖 T0–T4，含环境指纹、重复性、总体判定 |
| 认证证书 | ❌ 不提供 | DSHIO/T 32 整体仍为「试行 · 未启用」 |
| L3 级别 | 🕐 待规划 | 需要 T5 |

**获得检验报告不等于获得认证。任何把检验报告说成认证的表述均为虚假。**

## 怎么引用

```
DSHIO/T 10—2026 第 5.2.2 条
```

不要写 `DSHIO-DEV`（已废止的旧代号），也不要省略标准号。

## 许可

- **标准文本**（`standards/`、`machine/`）— [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- **其他内容** — MIT

引用、转载、翻译均允许，注明出处即可。

## 独立性声明

DSHIO 是独立第三方标准组织，与 DeepSeek 无隶属关系，未获其授权或背书。

标准为独立归纳，非官方规范。官方信息请以 DeepSeek 官方渠道为准。
