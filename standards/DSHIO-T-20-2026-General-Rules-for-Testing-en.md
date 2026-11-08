# DSHIO/T 20—2026 General Rules for Testing

| | |
|---|---|
| **Standard No.** | DSHIO/T 20—2026 |
| **Title** | General Rules for Testing |
| **Version** | v1.0 (Trial) |
| **Published** | 2026-11-07 |
| **Effective** | 2026-11-07 |
| **Status** | Trial (2026-11-07 → 2027-02-07; becomes current automatically) |
| **Original** | Simplified Chinese |
| **Layer** | Method standard |

---

> ## 📋 Current scope
>
> **T0–T4 are implemented. T5 is written into the standard with status "planned".**
>
> Until T5 is formally included, any report claiming a T5 run is void.
> **This notice prevails over any T5-related provision elsewhere in this standard.**

---

## 1 Scope

This standard specifies the stage breakdown of DSH plugin testing, and for each stage
its purpose, operations, observation targets and verdict rules.

It applies to any person or system that makes a verdict on a plugin according to the
DSHIO standards.

**It does not specify test conditions** — the fixed values for environment, resources,
network and timeouts are in DSHIO/T 21.

> **Division of labour:** this standard answers "what to do, what to look at, how to
> judge"; DSHIO/T 21 answers "under what conditions".

### 1.1 Statement of implementation independence

This standard does not require a particular test tool or framework. Any implementation
that produces the same observations is conforming.

It defines an **observation contract**, not a script.

---

## 2 Normative references

| Standard | Title | Clauses referenced |
|---|---|---|
| **DSHIO/T 00—2026** | Terms and Definitions | 3.2.1, 3.2.5, 3.3.2, 3.3.4, 3.5.1, 3.5.3, 3.5.4, 3.6.3 |
| **DSHIO/T 21—2026** | Test Conditions | All |
| **DSHIO/T 10—2026** | Plugin Standard | Chapters 5 and 6 |

---

## 3 Terms and definitions

The terms defined in DSHIO/T 00—2026 apply. **Additional definitions:**

### 3.1 stage

One independent checking step in a test. The six stage codes are T0 to T5.

### 3.2 stage termination

The behaviour of not executing later stages once a stage's verdict is a failure.

**Rationale:** later stages are meaningless when their premises do not hold, and they
would waste resources.

### 3.3 observation target

The source of fact a stage uses for its verdict, e.g. "the command exit code" or
"a particular line in the output".

### 3.4 stage evidence

The raw output of the stage, used to support the verdict and to permit review.

**Evidence for all stages must be retained in full**; retaining only the verdict is
not acceptable.

---

## 4 Stage overview

| Stage | Name | Network | Model dependent | Purpose |
|---|---|---|---|---|
| **T0** | Static check | not needed | No | structural validity |
| **T1** | Install | on | No | can the package manager install it |
| **T2** | List | **off** | No | does it appear in the dependency list |
| **T3** | Mount | **off** | No | does it reach the config tree |
| **T4** | Boot | **off** | No | can the profile start and stay alive |
| **T5** | Tool invocation | on | **Yes** | can the model invoke it correctly |

**T0–T4 are model-independent stages.** Basis: model access is provided by separate
plugins, not by the harness core.

**The order T0 → T1 → T2 → T3 → T4 must be followed.** T5 is optional after T4 passes.

---

## 5 T0 Static check

### 5.1 Purpose

Decide whether the plugin's structure is valid, without touching a runtime environment.

### 5.2 Prerequisites

None. T0 needs no container and no network access to DSH.

**Inputs required:**
- the plugin's registry metadata (`package.json`)
- the content of the patch file

### 5.3 Operations

1. Fetch the plugin metadata from a public registry and determine the target version
2. Read the value of `dsh.bundle.patch` from the metadata
3. Fetch that patch file from a public CDN
4. Execute the checks in the order of DSHIO/T 10 Annex A

### 5.4 Observation targets

| Clause | Observation target |
|---|---|
| 5.1.1 | `$.dsh.bundle.patch` exists and is non-empty |
| 5.1.2 | CDN returns non-empty content |
| 5.1.3 | YAML parsing raises no exception |
| 5.1.4 | The type and structure of the parsed result |
| 5.1.5 | The `id`/`name` of each item in `insert` |
| 5.2.1 | `$.engines.dsh` or `$.peerDependencies` |
| 5.2.2 | Relationship between the semver range and the baseline version |
| 5.2.3 | `main` or `exports` exists |
| 5.3.1 | The contents of the `files` array |
| 5.3.2 | The values of the four dependency fields |
| 5.5.3 | The permission declaration set |

### 5.5 Verdict

| Situation | Verdict |
|---|---|
| Metadata cannot be fetched | **No verdict**; report "unresolvable" |
| Any critical clause non-conforming | `conforming_static = false` |
| No critical non-conformance | `conforming_static = true` |

**T0 does not produce verdicts such as pass/install_failed.** Its output is a
clause-by-clause result plus one boolean.

### 5.6 Termination rule

**T0 does not stop later stages on failure.** A statically non-conforming plugin may
still proceed to T1–T4 — the static check can be wrong, and the actual run is the
final authority.

**However the tool should warn:** with a critical static non-conformance the sandbox
test will most likely fail, so fixing first is advisable.

---

## 6 T1 Install

### 6.1 Purpose

Decide whether the package manager can correctly resolve and install the plugin into a
DSH profile.

### 6.2 Prerequisites

- a clean profile (no pre-existing plugin dependencies)
- network available (the package must be downloaded)
- resource limits per DSHIO/T 21 clause 5.1

### 6.3 Operation

```
dsh plugin --profile web add <package-spec>
```

`package-spec` is an npm package name (optionally with a version) or `github:owner/repo`.

### 6.4 Observation targets

| Observation | Basis for the verdict |
|---|---|
| Command exit code | `0` = success |
| The added-dependency line in the output | `+ <package> <version>` |
| The profile's `package.json` | `dependencies` contains the package |

**All three must agree.** Exit code 0 with the dependency not written is not success.

### 6.5 Verdict

| Situation | Verdict |
|---|---|
| Exit code 0 and the dependency was written | **pass**, proceed to T2 |
| Exit code non-zero | **`install_failed`**, terminate |
| Timeout | **`install_failed`**, first error recorded as timeout |

### 6.6 First error

On failure, the first non-empty line of the output that locates the cause must be
extracted as `first_error`.

**Excluded:** pure JSON lines, box-drawing lines (`╭`, `│`, `╰`), progress lines
(`Progress:`).

**Purpose:** to give the author a lead for fixing; it is not used for the verdict.

---

## 7 T2 List

### 7.1 Purpose

Decide whether the installed plugin appears in the profile's dependency list.

### 7.2 Prerequisites

- T1 passed
- **network off** (`--network none`)

### 7.3 Operation

```
dsh plugin --profile web list
```

### 7.4 Observation target

Whether the output's dependencies block lists the target package and its version.

### 7.5 Verdict

| Situation | Verdict |
|---|---|
| Output contains the target package | **pass**, proceed to T3 |
| Output does not contain it | **`install_failed`**, terminate |

> **Note:** a T2 failure is recorded as `install_failed` rather than a separate
> verdict. Rationale: a T2 failure means installation did not really complete, which
> has the same practical consequence as a T1 failure.

---

## 8 T3 Mount

### 8.1 Purpose

Decide whether the plugin's entry reaches the composed config tree.

**This is the dividing line between "installed" and "actually in effect".** Many
plugins install and list fine, yet their entry never reaches the config tree and the
functionality therefore never activates.

### 8.2 Prerequisites

- T2 passed
- **network off**

### 8.3 Operation

```
dsh --profile web --dump-config
```

### 8.4 Observation targets

| Observation | Note |
|---|---|
| The output contains the plugin's entry `id` | the id declared in the patch's `insert` |
| The entry's `name` matches the package name | the module specifier |

**The accurate way to decide the entry exists:** in the output YAML, find
`- id: <entry-id>` and check that the `name:` below it is the expected module name.

**Do not merely grep for the package name** — the name may appear in unrelated config
values.

### 8.5 Verdict

| Situation | Verdict |
|---|---|
| Entry exists and name matches | **pass**, proceed to T4 |
| Install and list succeeded, but the entry does not exist | **`installs_but_not_registered`**, terminate |
| The command itself fails or times out | **`config_failed`**, terminate |

### 8.6 What `installs_but_not_registered` means

This is the most easily overlooked, and the most confusing, failure mode:

- the user sees "installed successfully"
- the plugin is in the plugin list
- yet the functionality simply does not work

**Common causes:**
1. The patch's `insert` is overridden or disabled by a later profile layer
2. The `id` collides with an existing entry, causing an override rather than an addition
3. The patch file was written but contains no `insert` item

---

## 9 T4 Boot

### 9.1 Purpose

Decide whether the profile can start after loading the plugin, and stays alive after
starting.

### 9.2 Prerequisites

- T3 passed
- **network off**
- process-count limit (see DSHIO/T 21 clause 5.1.5)

### 9.3 Operation

```
dsh --profile web
```

After starting, observe for **12 seconds**; the process must stay alive throughout.

### 9.4 Observation targets

| Observation | Basis for the verdict |
|---|---|
| Whether the process exits within 12 s | exit = failure |
| Whether the output contains a fatal error | present = failure |
| Whether there is a URL line (web profile) | present = startup success marker |

**Startup success marker for the web profile:** the output contains a
`dsh web: http://...` line.

### 9.5 Verdict

| Situation | Verdict |
|---|---|
| Did not exit within 12 s and the success marker is present | **pass** |
| Exited within 12 s | **`boot_failed`**, terminate |
| Timeout (see DSHIO/T 21 clause 5.1.6) | **`boot_failed`** |

### 9.6 Why the network must be off

**Two reasons:**

1. **Reproducibility** — with the network on, a plugin may depend on content returned
   by an external service, making the result vary with that service's state
2. **Security** — even a malicious plugin has nowhere to send data

**Additionally:** with the network off, any lingering network request fails and is
exposed, which is itself a valuable observation.

---

## 10 T5 Tool invocation (model-dependent)

### 10.1 Purpose

Decide whether the model can correctly understand and invoke a tool the plugin
registers.

### 10.2 Prerequisites

- T4 passed
- the plugin registers at least one tool
- a model is available
- test conditions conform to DSHIO/T 21 chapter 6

### 10.3 Operations

1. Construct a natural-language request that clearly requires invoking the target tool
2. Record the full system prompt and the full test prompt
3. Execute under a fixed model and fixed generation parameters
4. Repeat 3 times

### 10.4 Observation targets

| Observation | Basis for the verdict |
|---|---|
| Whether the model invoked the target tool | the tool name appears in the invocation record |
| Whether the arguments are reasonable | required arguments present, values within the expected range |
| Whether a wrong tool was invoked | compare against the available tool list |

### 10.5 Verdict

**The report must state which criterion was used:**

| Criterion | Definition | Use |
|---|---|---|
| **All** | 3 of 3 correct invocations | certification |
| **Majority** | at least 2 of 3 | evaluation |

**A T5 verdict must name the model.** A T5 report that says only "pass" without
naming the model is void.

### 10.6 Why T5 is not part of T0–T4

Model stochasticity makes T5 results non-deterministic. Mixing it with the
deterministic stages would make the whole verdict appear unreliable.

**The benefit of separation:** users can clearly distinguish "this plugin installs"
(deterministic) from "this model can use this plugin correctly" (probabilistic).

---

## 11 Verdicts and propagation

### 11.1 Verdict codes

| Code | Meaning | Terminates at |
|---|---|---|
| `pass` | All executed stages passed | T4 or T5 |
| `install_failed` | T1 or T2 failed | T1/T2 |
| `config_failed` | The T3 command failed | T3 |
| `installs_but_not_registered` | T3 passed but the entry did not reach the config tree | T3 |
| `boot_failed` | T4 failed | T4 |

### 11.2 Propagation rule

**Once a stage's verdict is a failure, later stages are not executed and their
results are recorded as "not executed".**

"Not executed" must not be treated as "failed", nor as "passed".

### 11.3 Unstable

**When repeated executions yield different verdicts:**
1. Record the verdict as `unstable`
2. Retain the full evidence of every repetition
3. Must not be used as a basis for certification

**Common sources of instability:**
- network fluctuation (T1)
- dependency version drift
- timeouts caused by insufficient resources

---

## 12 Evidence requirements

### 12.1 Evidence that must be retained

| Stage | Evidence |
|---|---|
| T0 | metadata, the raw patch file, the clause-by-clause results |
| T1 | full stdout/stderr, exit code, duration |
| T2 | full stdout, exit code |
| T3 | the full `--dump-config` output (relevant sections may be excerpted, with context retained) |
| T4 | the full boot log, liveness duration |
| T5 | the full conversation record of every attempt, tool invocation records |

### 12.2 Purpose of evidence

1. Support the verdict conclusion
2. Permit third-party review
3. Let the author locate the problem

**A test report that retains only verdicts and no evidence is not reviewable and must
not be used as a basis for certification.**

---

## 13 Interface with DSHIO/T 21

Every stage of this standard depends on the conditions defined in DSHIO/T 21.

**This standard defines the observation contract; DSHIO/T 21 defines the fixed values
that make the observations comparable.**

---

## Annex A (normative) Stage execution checklist

| # | Stage | Precondition | Operation | Pass criterion |
|---|---|---|---|---|
| 1 | T0 | none | fetch metadata + patch, run clause checks | no critical non-conformance |
| 2 | T1 | clean profile, network on | `dsh plugin --profile web add <pkg>` | exit 0 and dependency written |
| 3 | T2 | T1 passed, network off | `dsh plugin --profile web list` | target package listed |
| 4 | T3 | T2 passed, network off | `dsh --profile web --dump-config` | entry id present and name matches |
| 5 | T4 | T3 passed, network off | `dsh --profile web`, observe 12 s | alive 12 s with success marker |
| 6 | T5 (optional) | T4 passed, model available | fixed prompt × 3 | per chosen criterion |

---

## Annex B (informative) Verdict distribution reference

| Verdict | Typical share |
|---|---|
| `pass` | most plugins that follow the manifest and patch conventions |
| `install_failed` | chiefly YAML errors, version mismatches, missing files |
| `installs_but_not_registered` | patch structure problems (see 8.6) |
| `boot_failed` | initialisation exceptions |

---

## Revision record

| Version | Date | Change | Proposed by | Participants | Conflict-of-interest statement |
|---|---|---|---|---|---|
| v1.0 (Trial) | 2026-11-07 | First release | DSHIO | DSHIO | The test platform operated by DSHIO is the sole implementer of this standard; the observation contract here is the one that platform implements, and is publicly verifiable. |
