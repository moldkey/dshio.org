# DSHIO/T 00—2026 Terms and Definitions

| | |
|---|---|
| **Standard No.** | DSHIO/T 00—2026 |
| **Title** | Terms and Definitions |
| **Version** | v1.0 (Trial) |
| **Published** | 2026-11-07 |
| **Effective** | 2026-11-07 |
| **Status** | Trial (2026-11-07 → 2027-02-07; becomes current automatically) |
| **Original** | Simplified Chinese |
| **Layer** | Basic standard |

---

> ## 📋 Current scope
>
> **DSHIO currently provides test stages T0–T4. T5 (tool invocation) is written into
> the standard with status "planned".**
>
> T5 involves model cost and nondeterminism and is not yet scheduled. Until it is
> formally included, any report claiming a T5 run is void.

---

## 1 Scope

This standard defines the terms and definitions used throughout the DSHIO standards
system.

It applies to the writing, citation and implementation of all DSHIO standards.

It does not specify technical requirements — it only fixes the meaning of words.
Wherever another DSHIO standard uses a term defined here, it is to be understood as
defined in this chapter.

---

## 2 Normative references

None. **Note:** this standard is the root of the system and references no other standard.

---

## 3 Terms and definitions

### 3.1 System and actors

#### 3.1.1
**DSHIO**
DeepSeek Harness Input/Output

The designation of the body that publishes this standards system. It is not an
official body and not a certification authority; it is the author and maintainer of
the standards.

> **Note:** DSHIO does not own DeepSeek Harness and is not responsible for its
> behaviour. DSHIO standards are third-party generalisations, not an official
> specification.

#### 3.1.2
**DSH**
DeepSeek Harness

The host program this standards system addresses. All version references below mean
the npm package `@deepseek-ai/dsh`.

> **Note:** the name DSH contains "DeepSeek", but its architecture is decoupled from
> the model — see 3.4.1 "model agnosticism".

#### 3.1.3
**DSH version**

The string emitted by `dsh --version`, for example `0.2.2-rc.1`.

**Wherever version determination occurs in this standards system, the output of that
command governs.**

#### 3.1.4
**author**

The natural person or organisation that develops and publishes a DSH plugin.

#### 3.1.5
**user**

A person who installs and runs DSH and its plugins.

#### 3.1.6
**certification**

The act by which DSHIO makes a conformance determination about a plugin according to
a standard, and publishes that determination.

> **Note:** certification is not approval and not a licence — it is "a public record
> of having been tested under specified conditions".

---

### 3.2 Anatomy of a plugin

#### 3.2.1
**plugin**

A software unit that provides one or more capabilities to DSH. Distributed as an npm
package, declaring its position in the config tree via `cordis.patch.yml`.

**Criterion (all three shall hold):**

1. `package.json` contains a `dsh.bundle.patch` field
2. The patch file that field points to exists
3. That patch file's `insert` list contains at least one entry

> **Note:** a package with only a `dsh` field whose patch points at a non-existent
> file is not a plugin — see 3.6.3 "malformed plugin".

#### 3.2.2
**bundle**

A collection of plugins. Appears as a member of the `dsh.profile.bundles` list.

**Difference from a plugin:** a bundle provides no capability itself; it only packages
several plugins together with their configuration.

> **Example:** `@deepseek-ai/dsh-web-app` is a bundle; it contains a set of plugins for
> the web UI, session management, tool execution and so on.

#### 3.2.3
**tool**

A callable function that a plugin exposes to the model. Registered with `defineTool()`.

**Criterion:** registered in `ctx.tools.register()`, with the four fields `name`,
`description`, `parameters` and `execute`.

#### 3.2.4
**service**

An injectable capability that a plugin provides to other plugins. Dependencies are
declared with `inject`.

> **Example:** `dsh-fs` provides file read/write services; `dsh-llm` provides model
> invocation services.

#### 3.2.5
**entry**

One item in the config tree. An element of the `insert` list of `cordis.patch.yml`,
containing fields such as `id`, `name`, `config` and `inject`.

> **Official schema:** the `entry` definition in the output of `dsh --dump-config-schema`.

#### 3.2.6
**patch**

A YAML file describing insert, override or disable operations against the config tree.
The entry patch file of a DSH plugin is conventionally named `cordis.patch.yml`.

---

### 3.3 Runtime environment

#### 3.3.1
**profile**

The set of plugins and their configuration loaded when DSH starts. Corresponds to the
directory `$DSH_HOME/profiles/<name>/`.

**Factory profiles:** `web`, `headless`, `sdk`, `sdk-minimal`, `acp`

> **Note:** `desktop` is a reserved name owned by the Electron host; the CLI does not
> accept it.

#### 3.3.2
**composed config tree**

The entry list obtained after composing all patch layers of a profile by precedence.
Inspect it with `dsh --dump-config`.

#### 3.3.3
**layer precedence**

The order of config composition. Later overrides earlier:

```
1. each bundle's patch, in dsh.profile.bundles order
2. the profile's cordis.patch.yml
3. the home-level $DSH_HOME/cordis.patch.yml
4. the --patch override layer
```

> **Source:** official README lines 42–44.

#### 3.3.4
**sandbox**

An isolated container runtime used to install and run plugins without affecting the
host.

**A sandbox in this standards system shall:**
- be isolated from the host
- be able to limit CPU, memory and network
- be destroyed after each test

---

### 3.4 Model relativity

#### 3.4.1
**model agnosticism**

The property that DSH's core capabilities (install, mount, boot, tool registration) do
not depend on a specific model.

**Basis:** model access is provided by separate plugins (`dsh-llm-deepseek`,
`dsh-llm-pi-ai`, etc.), not by the harness core.

**Implication:** a plugin that installs under a DeepSeek model also installs under any
other model. **Installability is model-independent.**

#### 3.4.2
**model-dependent requirement**

A requirement whose result depends on the model used. Typically:

- tool-call reliability
- prompt compatibility
- token cost

**In this standards system, any verdict on a model-dependent requirement shall name the
model used.**

#### 3.4.3
**model adapter**

A plugin that brings a model provider into DSH. Named `dsh-llm-<provider>`.

> **Example:** `dsh-llm-deepseek`, `dsh-llm-pi-ai`

---

### 3.5 Testing and verdicts

#### 3.5.1
**test stage**

One independent checking step in plugin testing. This standards system defines six:

| Code | Name | Network | What it checks |
|---|---|---|---|
| **T0** | Static check | not needed | structural validity of manifest and patch |
| **T1** | Install | required | whether the package manager can install it into a profile |
| **T2** | List | off | whether it appears in the dependency list |
| **T3** | Mount | off | whether it appears in the composed config tree |
| **T4** | Boot | off | whether the profile starts and stays alive |
| **T5** | Tool invocation | required | whether the model correctly invokes a tool the plugin registers |

**T0–T4 are model-independent stages; T5 is the model-dependent stage.**

#### 3.5.2
**test condition**

The set of external factors that affect the test result. Including but not limited to
DSH version, container image, resource limits, network state and timeout settings.

**The complete list is in DSHIO/T 21.**

#### 3.5.3
**verdict**

The conclusion of a test run. This standards system defines five:

| Verdict | Meaning | Terminates at |
|---|---|---|
| **pass** | All stages passed | T5 |
| **install_failed** | T1 failed | T1 |
| **config_failed** | T3 failed | T3 |
| **installs_but_not_registered** | T3 passed but the entry did not reach the config tree | T3 |
| **boot_failed** | T4 failed | T4 |

> **Note:** a verdict covers only up to the stage that failed; later stages are not run.

#### 3.5.4
**reproducibility**

The property that repeating under identical test conditions yields an identical verdict.

**Requirement:** any publicly published verdict shall be executed at least twice with
consistent results.

#### 3.5.5
**first error**

When a verdict is a failure, the first non-empty line of the test output that locates
the cause.

**Purpose:** to give the author a lead for fixing; it is not used for the verdict itself.

---

### 3.6 Conformance

#### 3.6.1
**conforming**

The state of satisfying all mandatory clauses of a standard.

**Determining body:** the certification authority (currently DSHIO itself).
**Basis:** the technical requirements and test methods of that standard.

#### 3.6.2
**non-conforming**

The state of not satisfying one or more mandatory clauses of a standard.

**Must state:** which clause, under what conditions, in which test it was found.

#### 3.6.3
**malformed plugin**

A package that satisfies part of the "plugin" characteristics but cannot complete
installation. Typical cases:

- has `dsh.bundle.patch` but the file does not exist
- the patch file has a YAML syntax error
- the `insert` list is empty

**Criterion:** identifiable by the T0 static check; no need to enter T1.

#### 3.6.4
**exception**

A publicly registered deviation where a standard's requirement conflicts with reality.

**An exception shall include:** which clause, why, the scope of impact, and the
validity period.

**An unregistered deviation is not an exception; it is non-conformance.**

#### 3.6.5
**reference implementation**

The first instance that fully implements a standard and passes all its clauses.

**Purpose:** to prove the standard is executable.

**Rule:** a standard shall have a reference implementation before publication. A
standard without one may not move from "trial" to "current".

---

### 3.7 Security and permissions

#### 3.7.1
**declared capability**

The required capabilities a plugin declares in its manifest.

**Capability identifiers used by this standards system:**

`fs-read`, `fs-write`, `network`, `shell`, `credentials`, `dynamic-code`, `llm`, `env`

#### 3.7.2
**red line**

A permission combination that is high risk by its mere presence.

**Currently defined:** `network` together with (`shell` or filesystem write) →
constitutes a data-exfiltration path.

#### 3.7.3
**risk level**

A risk classification of a plugin's permission combination. This standards system uses
three levels: `low`, `medium`, `high`.

**The determination method is in DSHIO/T 24.**

---

## 3.x Modal verbs

This standard follows **ISO/IEC Directives Part 2** (and, for Chinese, GB/T 1.1—2020
Annex C) for modal verbs. The mapping is one-to-one:

| Clause type | English | Chinese | Not used |
|---|---|---|---|
| **Requirement** | **shall** | 應 | must, is required to |
| **Requirement (negative)** | **shall not** | 不應 | must not, may not |
| **Recommendation** | **should** | 宜 | ought to, is recommended to |
| **Recommendation (negative)** | **should not** | 不宜 | — |
| **Permission** | **may** | 可 | is allowed to |
| **Permission (negative)** | **need not** | 不必 / 可以不 / 無須 | may not, cannot |
| **Capability / possibility** | **can / cannot** | 能 / 不能 | — |

> **Why not "must".**
> "must" carries the sense of a legal obligation in common-law drafting, the same
> problem "必須" has in Chinese standards. A standard binds a party only after that
> party declares conformance; it does not impose statutory duties. ISO/IEC and
> GB/T both reserve "shall" for requirements and avoid "must".

> **"cannot" is not a requirement.**
> "cannot" states a capability limit. To express "not permitted", write "shall not".


## 4 Abbreviations

| Abbreviation | Full form | Note |
|---|---|---|
| DSH | DeepSeek Harness | The host program |
| DSHIO | DeepSeek Harness Input/Output | The standards publisher |
| MCP | Model Context Protocol | Model context protocol |
| npm | Node Package Manager | Package manager |
