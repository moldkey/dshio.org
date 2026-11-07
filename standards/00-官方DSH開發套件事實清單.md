# 官方 DSH 開發套件事實清單（標準依據）

> 來源：sandbox 內 `@deepseek-ai/dsh@0.2.2-rc.1` 實測 + 官方 README
> 用途：DSHIO 標準的所有技術條款必須以此為依據
> 採集時間：2026-11-07

---

## 一、版本與包

| 項目 | 值 |
|---|---|
| DSH 版本 | `0.2.2-rc.1` |
| 全局包位置 | `/opt/node24/lib/node_modules/@deepseek-ai/dsh` |
| `dsh-tools` 位置 | `.../dsh/node_modules/@deepseek-ai/dsh-tools` |
| 官方插件數 | **150+**（見 `default-config.yml`）|

---

## 二、CLI 命令文法（官方 README 第 5 行）

```
dsh [--profile] <name> [options] [app-args...]
dsh plugin --profile <name> <pnpm-args...>
```

### 主要選項

| 選項 | 作用 |
|---|---|
| `--profile <name>` | 啟動哪個 profile |
| `--from-default-profile <name>` | 用出廠模板重建 profile |
| `--patch <path>` | 額外 patch 層（可重複）|
| `--dump-config` | 印出合成後的 profile 樹 |
| `--dump-config-schema` | 印出 entry/patch 的 JSON Schema |
| `--dump-default-config` | 印出不含用戶層的樹 |

### 出廠 profile（首次使用自動初始化）

`web` · `headless` · `sdk` · `sdk-minimal` · `acp`

**`desktop` 是保留名** —— Electron 擁有，CLI 拒絕 boot 和 config-dump。

---

## 三、Profile 目錄結構（關鍵）

```
$DSH_HOME/profiles/<name>/
├── package.json          ← 含 dsh.profile.bundles（有序列表）+ 插件依賴
└── cordis.patch.yml      ← 用戶自己的 patch 層
```

**`package.json` 裡：**
```json
{
  "dsh": {
    "profile": {
      "bundles": ["@deepseek-ai/dsh-base", "..."]
    }
  },
  "dependencies": { "...": "..." }
}
```

---

## 四、層級優先序（官方 README 第 42-44 行）

**合成順序（後者覆蓋前者）：**

```
1. 每个 bundle 的 patch，按 dsh.profile.bundles 顺序
2. profile 的 cordis.patch.yml
3. home 级 $DSH_HOME/cordis.patch.yml
4. --patch 覆盖层
```

**這條是標準必須寫死的 —— 插件作者要知道自己的配置在哪一層生效。**

---

## 五、出廠 bundle 名（從安裝目錄解析，再從 profile node_modules 解析）

`@deepseek-ai/dsh-base`
`@deepseek-ai/dsh-web-app`
`@deepseek-ai/dsh-headless`
`@deepseek-ai/dsh-sdk-app`
`@deepseek-ai/dsh-sdk-minimal`
`@deepseek-ai/dsh-acp-app`

---

## 六、版本兼容性（官方 README 第 39 行）

> Installation and profile startup enforce the declared `engines.dsh` range,
> or DSH peer ranges when it is absent, against the same runtime version shown
> by `dsh --version`.
>
> Incompatible plugins require an explicitly acknowledged exact-version exemption.

**三條硬規則：**
1. 有 `engines.dsh` 就按它判
2. 沒有就按 DSH peer ranges 判
3. 不兼容的要「explicitly acknowledged exact-version exemption」

**這解釋了為什麼我們測的插件有 654 個裝不上 —— 大部分是 `engines.dsh` 寫死。**

---

## 七、配置 Schema（`--dump-config-schema` 實測）

### entry 欄位

| 欄位 | 類型 | 說明 |
|---|---|---|
| `id` | string | entry id。省略時 loader 自己生成；patch 用配置的 id |
| `name` | string | 插件模組 specifier。相對路徑以 patch 文件為錨點 |
| `config` | any | 插件配置 |
| `group` | bool/null | 允許 patch 索引與插入，不選插件實現 |
| `disabled` | bool/null 或 `!!js` | Loader 會把其他真值強制轉成 disabled，但 schema 拒絕 |
| `inject` | array/object/null | 依賴注入 |
| `intercept` | object/null | 攔截 |
| `isolate` | object/null | 隔離 |

### 列表類型

- `entryList` — entry 數組
- `patchList` — patch 數組
- `includeConfig` — `{ path: "..." }`，相對 owner Loader 樹解析

---

## 八、官方插件清單（150+，節選）

### 工具類（`dsh-tool-*`）
```
dsh-tool-ask-user      dsh-tool-bash          dsh-tool-bash-persistent
dsh-tool-cordis        dsh-tool-executor-native dsh-tool-executor-ptc
dsh-tool-fs            dsh-tool-fs-search     dsh-tool-goal
dsh-tool-jobs          dsh-tool-present       dsh-tool-pwsh
dsh-tool-pwsh-persistent dsh-tool-schedule    dsh-tool-skill
dsh-tool-subagent      dsh-tool-subagent-control dsh-tool-todo
dsh-tool-web           dsh-tool-workflow      dsh-tool-working-directory
dsh-tool-call-timeout-policy
```

### 服務類
```
dsh-fs / dsh-fs-local / dsh-fs-patch / dsh-fs-sandbox / dsh-fs-observation-policy
dsh-llm / dsh-llm-deepseek / dsh-llm-deepseek-api-key / dsh-llm-pi-ai / dsh-llm-retry
dsh-session / dsh-session-persistence / dsh-session-query / dsh-session-stats
dsh-storage / dsh-storage-json / dsh-storage-domain
dsh-skill / dsh-skill-filesystem
dsh-subagent / dsh-subagent-fork-in-process / dsh-subagent-spawn-in-process
dsh-subprocess-local / dsh-terminal / dsh-terminal-bash
dsh-system-prompt / dsh-time-context / dsh-token-meter
dsh-web / dsh-web-app / dsh-web-fetch-http / dsh-web-search-deepseek
dsh-workspace / dsh-workspace-changes / dsh-working-directory
dsh-user-approval / dsh-user-questions / dsh-settings
```

### 注意：模型相關插件
```
dsh-llm-deepseek          ← DeepSeek 模型接入
dsh-llm-deepseek-account  ← 账号
dsh-llm-deepseek-api-key  ← API key
dsh-llm-pi-ai             ← 另一个模型提供商
dsh-llm-retry             ← 重试策略
```

**关键：模型接入是独立插件 → 换模型不换 harness。**
**这正是「非 DeepSeek 模型也能开发」的架构依据。**

---

## 九、對標準的意涵

### 1. 「插件」和「bundle」是兩種東西

| 類型 | manifest 欄位 | 用途 |
|---|---|---|
| **插件** | `dsh.bundle.patch` | 單一功能擴展 |
| **bundle** | `dsh.profile.bundles` 列表裡 | 一組插件的集合（如 `dsh-web-app`）|

**標準要分開定義。**

### 2. 版本兼容是硬門檻

`engines.dsh` 寫錯 = 裝不上。這必須是插件標準的第一條。

### 3. 模型無關性有架構保證

`dsh-llm-*` 是插件 → harness 本身不知道用哪個模型。
**所以 T0–T4（安裝/列裝/掛載/啟動）天然模型無關。**
只有 T5（工具調用）依賴模型。

### 4. 配置 schema 是官方給的

`--dump-config-schema` 可以直接導出 → 標準可以引用它，不用自己發明。

---

## 十、待補充

| # | 項目 | 怎麼取 |
|---|---|---|
| 1 | 完整 150+ 插件清單 | `dsh --dump-default-config` |
| 2 | `defineTool` 完整簽名 | `dsh-tools/lib/types/*.d.ts` |
| 3 | 官方 docs 原文 | github.com/deepseek-ai/deepseek-harness/docs |
| 4 | profile manifest schema | `dsh.profile` 的 JSON Schema |
