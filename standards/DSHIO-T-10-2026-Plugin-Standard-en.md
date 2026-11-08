# DSHIO/T 10—2026 Plugin Standard

| | |
|---|---|
| **Standard No.** | DSHIO/T 10—2026 |
| **Code** | DSHIO-PLUGIN |
| **Title** | Plugin Standard |
| **Version** | v1.0 (Trial) |
| **Published** | 2026-11-07 |
| **Effective** | 2026-11-07 |
| **Status** | Trial (2026-11-07 → 2027-02-07; becomes current automatically) |
| **Original** | Simplified Chinese |
| **Layer** | Product standard |
| **Supersedes** | DSHIO-DEV (retired) |

---

> ## 📋 Current scope
>
> **This standard defines six test stages (T0–T5). Current implementation status:**
>
> | Stage | Name | Status |
> |---|---|---|
> | T0 | Static check | ✅ Available |
> | T1 | Install | ✅ Available |
> | T2 | List | ✅ Available |
> | T3 | Mount | ✅ Available |
> | T4 | Boot | ✅ Available |
> | **T5** | **Tool invocation** | 🕐 **Planned** |
>
> **Why T5 is only planned:** it involves model cost and nondeterminism
> (see DSHIO/T 32 clause 4.3). It is written into the standard but not yet
> scheduled. Until it is formally included, any report claiming a T5 run is void.
>
> **Manual-judgement clauses (5.5.1, 5.5.2) are the author's own responsibility.**
> DSHIO does not issue verdicts on them. Producing an opinion on them is a
> **paid service**, and the opinion is advisory only — it does not change
> the conformance verdict.
>
> **This notice prevails over any T5-related provision elsewhere in this standard.**

---

## 1 Scope

This standard specifies the classification, technical requirements, test methods,
inspection rules and marking of DSH plugins.

It applies to every plugin distributed as an npm package that can be installed
into a DSH profile with `dsh plugin`.

It does not apply to:

- bundles — see DSHIO/T 11
- the DSH core packages themselves
- ordinary npm packages that do not provide `dsh.bundle.patch`

### 1.1 Statement of implementation independence

**This standard does not require any particular development tool, programming
language, build process or helper library.**

Anything that satisfies every technical requirement is conforming, however it is
implemented. Conformance is judged by **observable runtime behaviour**, not by
source structure.

Concretely:

1. `@deepseek-ai/dsh-tools` is **not** mandatory. `defineTool()` is merely a
   convenient way to author a tool; a hand-written object that satisfies the entry
   contract is equally valid.
2. TypeScript is **not** mandatory. Official plugins are written in TS because the
   official project uses TS, not because the standard requires it.
3. The official build pipeline is **not** mandatory. The standard constrains the
   structure of the artefact, not how it is produced.
4. pnpm is **not** mandatory. `dsh plugin` currently delegates to pnpm, but the
   standard only requires that dependencies resolve correctly.
5. A specific Cordis version is **not** mandatory. The plugin only has to mount
   and run on the target DSH version.

**Judgement is black-box: install, list, mount, boot, tool invocation.
The author's source code is not read.**

---

## 2 Normative references

The clauses of the following documents become clauses of this standard by virtue
of the reference. For dated references, only the dated edition applies.

| Standard | Title | Clauses referenced |
|---|---|---|
| **DSHIO/T 00—2026** | Terms and Definitions | 3.1.3, 3.2.1, 3.2.5, 3.3.3, 3.5.1, 3.5.3, 3.6.1, 3.7.1, 3.7.2 |
| **DSHIO/T 21—2026** | Test Conditions | All (chapter 6 of this standard references it directly) |
| **DSHIO/T 20—2026** | General Rules for Testing | Chapter 3 (T0–T5 definitions) |

**Non-normative references (informative only):**

- The official DSH README (layer precedence, `engines.dsh` evaluation rules)
- The entry/patch JSON Schema emitted by `dsh --dump-config-schema`

---

## 3 Terms and definitions

The terms defined in DSHIO/T 00—2026 apply. **Additional definitions:**

### 3.1 baseline version

The DSH version currently used by DSHIO for conformance decisions.
The current baseline is `0.2.2-rc.1`.

**The baseline version is updated as DSH releases. The update procedure is in
DSHIO/T 35.**

### 3.2 static layer

Checks that can be decided without installing. Corresponds to stage T0 of
DSHIO/T 20.

The static layer examines three kinds of object: `package.json`, the patch file,
and the entry structure.

### 3.3 dynamic layer

Checks that require an actual install and run. Corresponds to stages T1–T5.

### 3.4 red-line combination

A permission combination that is high risk by its mere presence.

One is currently defined: `network` declared together with (`shell` or `fs-write`).

Rationale: these two capability classes together form a complete path for
"read local data and send it out", and the user cannot see the actual direction
of flow from the declaration.

---

## 4 Classification

### 4.1 By capability provided

| Type | Criterion | Typical |
|---|---|---|
| **Tool** | Registers at least one tool | `dsh-tool-*` |
| **Service** | Provides capability to other plugins via `inject` | `dsh-fs`, `dsh-llm` |
| **UI** | Provides Web client components | `dsh-client-ui-*` |
| **Adapter** | Brings an external system into DSH | `dsh-llm-<provider>` |
| **Configuration** | Provides only a patch, no runtime logic | market bundles |

**A plugin may belong to several classes at once.**

### 4.2 By model dependence

| Type | Meaning | Effect on judgement |
|---|---|---|
| **Model-independent** | Functionality does not depend on a specific model | T0–T4 suffice |
| **Model-dependent** | Behaviour depends on model capability | Also requires T5, and the verdict shall name the model |

**Basis:** model access is provided by separate plugins (`dsh-llm-*`), not by the
harness core, so install, mount and boot are inherently model-independent.

### 4.3 By permission requirement

| Class | Permissions | Risk |
|---|---|---|
| **No permission** | Computation, UI only | low |
| **Read-only** | `fs-read`, `llm` | low |
| **Network** | `network` | depends on combination |
| **Write** | `fs-write` | depends on combination |
| **Shell** | `shell` | depends on combination |
| **Red line** | `network` + (`shell`/`fs-write`) | **high** |

---

## 5 Technical requirements

**Every clause in this chapter is mandatory**, except those marked "(recommended)",
which are advisory.

### 5.1 Structural requirements

#### 5.1.1 Plugin declaration

`package.json` **shall** contain a `dsh.bundle.patch` field whose value is the
path of the patch file relative to the package root.

```json
{
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

**Criterion:** the field exists and its value is a non-empty string.

#### 5.1.2 Patch file exists

The file referenced by `dsh.bundle.patch` **shall** exist inside the published package.

**Criterion:** the file can be fetched from a public CDN (unpkg / jsdelivr / npmmirror).

**Common failure:** the path differs from the declaration; the `files` array omits
the patch file.

#### 5.1.3 Patch file is valid YAML

The patch file **shall** be parseable by a YAML 1.2 parser.

**Criterion:** `yaml.safe_load()` does not raise.

**Most common mistake:** a value starting with `@` is not quoted.

```yaml
# wrong
name: @scope/pkg
# right
name: "@scope/pkg"
```

In YAML, `@` is a reserved indicator; unquoted it fails to parse.

#### 5.1.4 Patch structure

The top level of the patch file **shall** be an array containing at least one item
with an `insert` field.

```yaml
- insert:
    - id: my-plugin
      name: "my-package"
```

**Criterion:** the top level is a list; at least one item is a dict containing `insert`.

#### 5.1.5 Entry completeness

Every entry in the `insert` list **shall** contain a non-empty string `id` and a
non-empty string `name`.

| Field | Requirement |
|---|---|
| `id` | Non-empty string, unique within that patch layer |
| `name` | Non-empty string, a module specifier |

**Criterion:** iterate every dict in every `insert` array and check both fields.

#### 5.1.6 Optional entry fields

The following fields are optional; when present they **shall** conform to the
official schema:

| Field | Type | Purpose |
|---|---|---|
| `config` | any | Plugin configuration |
| `group` | boolean/null | Allow patch indexing |
| `disabling` | boolean/null/`!!js` | Disable the entry |
| `inject` | array/object/null | Dependency injection |
| `intercept` | object/null | Interception |
| `isolate` | object/null | Isolation |

**Basis:** the `entry` definition emitted by `dsh --dump-config-schema`.

### 5.2 Version and entry-point requirements

#### 5.2.1 Version range declaration (recommended)

`package.json` **should** contain an `engines.dsh` field declaring the compatible
DSH version range.

**When absent:** judge by the version ranges of `@deepseek-ai/*` packages in
`peerDependencies`. If neither is present, treat as unconstrained (any DSH version
accepted) and record it as a suggested improvement only.

```json
{
  "engines": { "dsh": ">=0.2.0-rc.2" }
}
```

#### 5.2.2 Version range includes the baseline version

The range declared in `engines.dsh` **shall** include the current baseline version.

**Criterion:** use semver rules to decide whether `0.2.2-rc.1` falls inside the range.

**Common mistake:** hard-coding an upper bound.

```json
// wrong: excludes 0.2.2-rc.1
"engines": { "dsh": ">=0.2.0-rc.2 <0.2.2-0" }

// right: lower bound only
"engines": { "dsh": ">=0.2.0-rc.2" }
```

**Basis:** the official README states that both installation and profile startup
validate `engines.dsh` against the runtime version; an incompatible plugin needs an
explicit exact-version exemption.

#### 5.2.3 Code entry point

`package.json` **shall** declare a code entry point: `main` or `exports`, at least one.

**Pure configuration plugins** (the "configuration" class in 4.1) may have an empty
`apply()`, but a usable entry is still mandatory — otherwise the install check fails.

```json
{
  "type": "module",
  "main": "lib/index.js"
}
```

#### 5.2.4 Module type consistency (recommended)

ESM packages **should** declare `"type": "module"` rather than rely on `.mjs`
extensions.

### 5.3 Packaging requirements

#### 5.3.1 `files` array completeness

If a `files` array is declared, it **shall** include:

- `lib` (or the directory containing the entry file)
- the patch file

**Criterion:** check whether `files` has an item starting with `lib`, and whether it
covers the value of `dsh.bundle.patch` (compare after stripping a leading `./`).

**When `files` is not declared:** all files are included and this clause passes.

#### 5.3.2 No local-path dependencies

The values of the following fields **shall not** start with `file:` or `/`:

`dependencies`, `optionalDependencies`, `peerDependencies`, `dsh`

**`devDependencies` is excluded** — development dependencies are not published with
the package and are not installed by consumers.

**Common mistake:** writing an internal tool as `"my-kernel": "file:./kernel"` inside
`dependencies`, which makes installation fail for consumers.

#### 5.3.3 Package size (recommended)

Installed `node_modules` size **should** be below 50 MB.

**Note:** no hard limit is imposed, but excessive size slows installation markedly.
For reference, official plugins are typically 1–20 MB.

### 5.4 Behavioural requirements (require sandbox testing)

#### 5.4.1 Installable

The plugin **shall** be resolvable and installable into a DSH profile from the public
npm registry or a specified Git repository.

**Criterion:** stage T1. See DSHIO/T 20 and DSHIO/T 21.

#### 5.4.2 Listable

After installation the plugin **shall** appear in the profile's dependency list.

**Criterion:** stage T2.

**Note:** this stage runs with networking disabled.

#### 5.4.3 Mountable

The plugin's entry **shall** appear in the output of `dsh --dump-config`.

**Criterion:** stage T3. Runs with networking disabled.

**Failure verdict:** when `install_ok=true` but `registered=false`, the verdict is
`installs_but_not_registered`.

#### 5.4.4 Bootable

After loading the plugin the profile **shall** start and stay alive for at least
12 consecutive seconds.

**Criterion:** stage T4. Runs with networking disabled.

#### 5.4.5 Tool contract (tool plugins)

Every registered tool **shall** satisfy:

| Field | Requirement |
|---|---|
| `name` | Non-empty, globally unique, lowercase with underscores or hyphens |
| `description` | Non-empty, see 5.5.1 |
| `parameters` | Conforms to the JSON Schema object convention |
| `execute` | A function |

**Criterion:** read the tool registration code statically, or observe the tool list
during stage T5.

### 5.5 Quality requirements

#### 5.5.1 Tool description states when to use it

Every tool's `description` **shall** let someone unfamiliar with the plugin decide
**under what circumstances the tool should be called**.

**Non-conforming examples:**
- only the function name: `"Searches plugins"`, `"Gets price"`
- only a description of the parameters

**Conforming example:**
- "Look up DeepSeek model prices. Returns the official price table plus per-million-token
  quotes from third-party providers, sorted cheapest first, with quantisation notes.
  Use when asked how much DeepSeek costs or which provider is cheapest."

**How it is judged:** **human judgement required.** Guiding questions: could someone
who does not know this plugin decide from the description whether to use it? Can they
tell when to use this rather than something else?

#### 5.5.2 Declared permissions match behaviour

The permission set a plugin declares **shall**:
- cover the capabilities it actually uses (no under-reporting)
- not declare capabilities it does not use (no over-reporting)

**How it is judged:** **human judgement required.** Guiding questions: declared
`network` but never makes a request? Declared `fs-write` but only reads?
Over-reporting erodes trust; under-reporting causes runtime failure.

#### 5.5.3 No red-line combination

Declaring `network` together with (`shell` or `fs-write`) is a red line.

**Criterion:** read the permission declaration statically.

**Exception:** where genuinely necessary, the README **shall** state:
1. why network is needed
2. why write or shell is needed
3. the direction of data flow

**A registered exception is still high risk, but is not non-conforming.**

---

## 6 Test methods

### 6.1 Static layer (T0)

**No container required.** Run `deepseekbase_check` or an equivalent implementation.

| Clause | Method |
|---|---|
| 5.1.1 | Read registry metadata, inspect `$.dsh.bundle.patch` |
| 5.1.2 | Fetch `{pkg}@{ver}/{patch}` from a CDN |
| 5.1.3 | pyyaml `safe_load` |
| 5.1.4 | Parsed result is a list with an item containing `insert` |
| 5.1.5 | Iterate `insert`, check `id`/`name` are non-empty |
| 5.1.6 | Compare against the official schema |
| 5.2.1 | Read `$.engines.dsh`; if absent read `$.peerDependencies` |
| 5.2.2 | semver range check against baseline `0.2.2-rc.1` |
| 5.2.3 | Check `main` or `exports` |
| 5.3.1 | Check `files` covers `lib` and the patch |
| 5.3.2 | Check the value prefixes of the four dependency fields |
| 5.5.3 | Read the permission declaration, check the red-line combination |

**A static-layer result has three possible states: conforming / non-conforming /
human judgement required.**

### 6.2 Dynamic layer (T1–T4)

**Shall run in a sandbox conforming to DSHIO/T 21.**

| Clause | Stage | Command |
|---|---|---|
| 5.4.1 | T1 (network on) | `dsh plugin --profile web add <pkg>` |
| 5.4.2 | T2 (network off) | `dsh plugin --profile web list` |
| 5.4.3 | T3 (network off) | `dsh --profile web --dump-config` |
| 5.4.4 | T4 (network off) | `dsh --profile web`, observe for 12 s |

**Judgement follows the stage definitions in DSHIO/T 20.**

### 6.3 Model-dependent layer (T5, model-dependent plugins only)

**A model-related environment fingerprint shall be recorded** (see DSHIO/T 21 chapter 6).

Method: under a named model, use a fixed prompt to request a call to a tool the
plugin registers, repeated 3 times.

**Pass criteria (the report shall state which one was used):**
- all: 3 of 3 correct invocations (for certification)
- majority: at least 2 of 3 (for evaluation; not usable for certification)

### 6.4 Human-judgement layer

**5.5.1 and 5.5.2 can only be judged by a human.**

The output shall be "human judgement required + guiding questions"; it shall not
produce a pass or fail conclusion.

---

## 7 Inspection rules

### 7.1 Inspection types

| Type | Purpose | Scope |
|---|---|---|
| **Static inspection** | Structural validity | All clauses of chapter 6.1 |
| **Type inspection** | Full conformance | Static + dynamic + human |
| **Certification inspection** | Issue a certificate | Full type inspection + repeatability requirements |

### 7.2 Sampling

**This standard does not involve batch sampling.** Each plugin version is an
independent inspection object.

Different versions of the same plugin (including patch versions) are different
objects and shall be judged separately.

### 7.3 Verdict rules

#### 7.3.1 Per-clause verdict

| Status | Meaning |
|---|---|
| **Conforming** | The clause is satisfied |
| **Non-conforming** | The clause is not satisfied |
| **Not applicable** | The clause does not apply to this plugin (reason required) |
| **Human judgement** | Automation cannot decide |

#### 7.3.2 Overall verdict

| Overall conclusion | Condition |
|---|---|
| **Conforming** | No critical or major non-conformance, and no pending human items |
| **Conditionally conforming** | No critical non-conformance, but major non-conformance or a few pending human items |
| **Non-conforming** | Any critical clause non-conforming |

#### 7.3.3 Clause severity

| Severity | Clauses | Impact |
|---|---|---|
| **critical** | 5.1.1–5.1.5, 5.2.2, 5.2.3, 5.3.2, 5.4.1–5.4.4 | If not satisfied, the plugin cannot be installed at all or does not work |
| **major** | 5.3.1, 5.5.1, 5.5.2, 5.5.3 | Non-conforming, though it may still run |
| **minor** | 5.2.1, 5.2.4, 5.3.3 | Suggested improvement |

### 7.4 Repeatability requirements

| Type | Minimum runs | Consistency |
|---|---|---|
| Static inspection | 1 | — |
| Dynamic inspection (public release) | **2** | The two verdicts shall be identical |
| T5 (public release) | **3** | Per the criteria in 6.3 |

**When the two dynamic verdicts differ:**
1. Record as "unstable"
2. Record both full outputs
3. shall not be used as a basis for certification

### 7.5 Re-inspection

A non-conforming plugin may be re-inspected after the author fixes it and publishes
a new version. The new version is a new object and does not inherit the old verdict.

---

## 8 Marking

### 8.1 Declaration of conformity

A plugin that passed inspection may declare in its README:

```
Conforms to DSHIO/T 10—2026 (baseline DSH 0.2.2-rc.1, static + dynamic layer)
Tested: YYYY-MM-DD
Environment fingerprint: dsh=0.2.2-rc.1;img=...;node=...;pnpm=...
```

**The following shall also be declared:**
- the clause scope covered by the inspection (if only the static layer was run,
  this shall be stated)
- the environment fingerprint
- the inspection date

**Prohibited:** writing only "conforms to the DSHIO standard" without version,
scope and environment.

### 8.2 Certification mark

**Not currently in effect.** See DSHIO/T 32 Certification Rules (trial, not yet
implemented).

While v1.0 of this standard is in force, the state "DSHIO certified" does not exist.
Any statement claiming DSHIO certification is false.

### 8.3 Citation format

Cite this standard as:

```
DSHIO/T 10—2026 clause 5.2.2
```

Do not write:
- `DSHIO-DEV` (retired legacy code)
- `DSHIO plugin standard clause 2` (the standard number is mandatory)

---

## Annex A (normative) Static inspection checklist

Execute in this order; abort on any critical non-conformance:

| # | Clause | Severity | Check |
|---|---|---|---|
| 1 | 5.1.1 | critical | `package.json` has `dsh.bundle.patch` |
| 2 | 5.1.2 | critical | The patch file is fetchable |
| 3 | 5.1.3 | critical | YAML parses |
| 4 | 5.1.4 | critical | Top level is a list containing `insert` |
| 5 | 5.1.5 | critical | Every entry has `id` and `name` |
| 6 | 5.2.1 | minor | Has `engines.dsh` or DSH peerDeps |
| 7 | 5.2.2 | critical | Range includes `0.2.2-rc.1` |
| 8 | 5.2.3 | critical | Has `main` or `exports` |
| 9 | 5.3.1 | major | `files` covers lib and the patch |
| 10 | 5.3.2 | critical | No `file:`/absolute paths in dependencies |
| 11 | 5.5.3 | major | No red-line permission combination |
| 12 | 5.5.1 | major | Tool description quality (human) |
| 13 | 5.5.2 | major | Permissions match behaviour (human) |

---

## Annex B (informative) Common failure modes

| Symptom | Clause | Cause | Fix |
|---|---|---|---|
| Won't install, YAML parse error | 5.1.3 | `@`-prefixed value unquoted | `name: "@scope/pkg"` |
| Won't install, version mismatch | 5.2.2 | `engines.dsh` upper bound hard-coded | Lower bound only |
| Won't install, file missing | 5.1.2 / 5.3.1 | `files` omits the patch | Add it to `files` |
| Installs but does nothing | 5.4.3 | Entry never reaches the config tree | Check whether the patch layer is overridden |
| Installs but fails to boot | 5.4.4 | Initialisation throws | Read the first error's stack |
| Consumers cannot install | 5.3.2 | `dependencies` contains `file:` | Use the npm package name |

---

## Annex C (informative) Relationship to other standards

```
DSHIO/T 00  Terms and Definitions
      ↓ referenced by
DSHIO/T 10  Plugin Standard (this document)
      ↓ references
DSHIO/T 20  General Rules for Testing ── what each stage does
DSHIO/T 21  Test Conditions           ── under what conditions
      ↓ referenced by
DSHIO/T 32  Certification Rules       ── how conformance is certified (not in effect)
```

---

## Revision record

| Version | Date | Change | Proposed by | Participants | Conflict-of-interest statement |
|---|---|---|---|---|---|
| v1.0 (Trial) | 2026-11-07 | First release; supersedes DSHIO-DEV | DSHIO | DSHIO | DSHIO itself develops the plugins `@liangjiehappy/dsh-deepseekbase` and `@liangjiehappy/dsh-deepseekbase-market`; this standard applies to both equally. The reference implementation is the former. |
