# DSHIO/T 21—2026 Test Conditions

| | |
|---|---|
| **Standard No.** | DSHIO/T 21—2026 |
| **Title** | Test Conditions |
| **Version** | v1.0 (Trial) |
| **Published** | 2026-11-07 |
| **Effective** | 2026-11-07 |
| **Status** | Trial (2026-11-07 → 2027-02-07; becomes current automatically) |
| **Original** | Simplified Chinese |
| **Layer** | Method standard |

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
> **paid service**, and the opinion is advisory only.
>
> **This notice prevails over any T5-related provision elsewhere in this standard.**

---

## 1 Scope

This standard specifies the environmental, resource, network and timing conditions
under which each stage of DSH plugin testing takes place.

It applies to any person or system that makes a verdict on a plugin according to the
DSHIO standards.

It does not specify the test methods themselves — see DSHIO/T 20.

> **Why this standard exists.**
> A test result without fixed conditions is one person's impression on one machine
> on one day. With fixed conditions, the same report can be reproduced, challenged
> and audited by someone else.

---

## 2 Normative references

The clauses of the following document become clauses of this standard by virtue of
the reference.

| Standard | Title | Clauses referenced |
|---|---|---|
| **DSHIO/T 00—2026** | Terms and Definitions | 3.1.3 (DSH version), 3.3.4 (sandbox), 3.5.1 (test stage), 3.5.4 (reproducibility), 3.4.1 (model agnosticism) |

---

## 3 Terms and definitions

The terms defined in DSHIO/T 00—2026 apply. **Additional definitions:**

### 3.1 fixed condition

An environmental factor that must not change during a test run. Changing any one of
them constitutes a new test, and its result must not be compared with the original.

### 3.2 variable condition

A condition that is permitted to change within a single test run. Variable conditions
must be recorded item by item in the test report.

### 3.3 test environment fingerprint

A set of values sufficient to uniquely identify the test environment, expressed as a
string, for comparison purposes.

> **Example:** `dsh=0.2.2-rc.1;img=dsh-sandbox:5.0@sha256:...;node=24.11.1;pnpm=9.15.9`

---

## 4 Condition classification

### 4.1 Two classes of test condition

| Class | Stages | Model dependent |
|---|---|---|
| **Model-independent conditions** | T0, T1, T2, T3, T4 | No |
| **Model-dependent conditions** | T5 | Yes |

**Basis:** DSHIO/T 00—2026 clause 3.4.1 — DSH install, mount and boot do not depend on
a specific model; model access is provided by separate plugins.

**Implication:** a T0–T4 verdict holds under any model; a T5 verdict holds only under
the named model.

---

## 5 Model-independent conditions (T0–T4)

### 5.1 Fixed condition list

**The following conditions must be fixed during T0–T4 and must be written into the
test report.**

#### 5.1.1 DSH version

| Item | Requirement |
|---|---|
| Value form | The raw output of `dsh --version` |
| Granularity | **Must be exact to the pre-release identifier** (e.g. `0.2.2-rc.1`, not `0.2.2` or `0.2.x`) |
| Used for | Whether the string matches the plugin's `engines.dsh` range |

**Current baseline version:** `0.2.2-rc.1`

> **Note:** a report that states a range (e.g. "the 0.2.2 series") is not reproducible
> and cannot be used as a basis for certification.

#### 5.1.2 Container image

| Item | Requirement |
|---|---|
| Image name | `dsh-sandbox:5.0` |
| Must record | image name + **digest** (`sha256:...`) |
| Base system | derived from `node:20-slim` |

**Must be fixed inside the image:**

| Component | Version |
|---|---|
| Node.js | `24.11.1` (installed at `/opt/node24`) |
| pnpm | `9.15.9` |
| DSH | `0.2.2-rc.1` |

> **Why the digest matters.** A tag can be overwritten — same name, different content.
> Without a digest you cannot reproduce the run a year later.

#### 5.1.3 Resource limits

| Resource | Limit | Stages |
|---|---|---|
| Memory | `768 MB` | T1, T2, T3, T4 |
| CPU | `1 core` | T1, T2, T3, T4 |
| Processes | `128` | T4 |
| Disk | unlimited (container overlay) | all |

**T0 (static check) needs no container and has no resource limits.**

#### 5.1.4 Network state

| Stage | Network | Option |
|---|---|---|
| T0 | not needed | — |
| **T1 install** | **network required** | default |
| **T2 list** | **network must be off** | `--network none` |
| **T3 mount** | **network must be off** | `--network none` |
| **T4 boot** | **network must be off** | `--network none` |

> **Why T2–T4 must have no network.**
> During install a plugin can fetch anything from the network. With the network off,
> any lingering request fails and is exposed; and even a malicious plugin has nowhere
> to send data. This is a security design as well as a reproducibility design.

#### 5.1.5 Permissions and isolation

| Item | T1 | T2 | T3 | T4 |
|---|---|---|---|---|
| `--user node` | ✅ | ✅ | ✅ | ✅ |
| `--cap-drop ALL` | — | — | ✅ | ✅ |
| `--security-opt no-new-privileges` | — | — | ✅ | ✅ |
| `--pids-limit 128` | — | — | — | ✅ |

**Note:** T1 needs to install, so the ordinary privileges pnpm requires are kept.
From T2 onward they are tightened progressively.

#### 5.1.6 Timeouts

| Stage | Timeout | Verdict on timeout |
|---|---|---|
| T1 install | `180 s` | `install_failed`, first error recorded as timeout |
| T2 list | `90 s` | `install_failed` (a list failure counts as an install failure) |
| T3 mount | `90 s` | `config_failed` |
| T4 boot | `90 s` (including the 12 s liveness observation) | `boot_failed` |

**The 12-second liveness observation at T4:** after the profile starts it must stay
alive for 12 consecutive seconds without exiting to pass.

> **Where 12 seconds comes from.** Long enough to cover the common failure window of
> plugin initialisation, config loading and the first event-loop turn; longer would
> markedly slow batch testing, and in practice no failure mode has been observed that
> only appears after 12 seconds.

#### 5.1.7 Working directory and profile

| Item | Value |
|---|---|
| HOME inside the container | `/home/node` |
| DSH_HOME | `/home/node/.dsh` |
| Test profile name | `web` |
| Profile initialisation | automatically from the factory template on first use |
| State before the test | **must be clean** (no pre-existing plugin dependencies, no pre-existing patch) |

**What "clean" means:** in the profile directory, `package.json`'s `dependencies` has
no entries other than the target plugin of this test.

#### 5.1.8 Command sequence

**The command sequence for T1–T4 must be identical, verbatim:**

```bash
# T1 install (network on)
dsh plugin --profile web add <package-spec>

# T2 list (network off)
dsh plugin --profile web list

# T3 mount (network off)
dsh --profile web --dump-config

# T4 boot (network off, observe 12 s)
dsh --profile web
```

**No insertion, omission or reordering is permitted.**

#### 5.1.9 Repeat count

| Situation | Minimum runs | Consistency requirement |
|---|---|---|
| Verdict published publicly | **2** | The two verdicts must be exactly the same |
| Internal batch testing | 1 | — |

**When the two verdicts differ:**
1. Record the verdict as `unstable`
2. Both full outputs must be recorded
3. An unstable result must not be used as a basis for certification

---

### 5.2 Test environment fingerprint under model-independent conditions

**Format:**

```
dsh=<DSH version>;img=<image name>@<digest>;node=<node version>;pnpm=<pnpm version>;mem=<memory>;cpu=<CPU>
```

**Example:**
```
dsh=0.2.2-rc.1;img=dsh-sandbox:5.0@sha256:abc123...;node=24.11.1;pnpm=9.15.9;mem=768m;cpu=1
```

**Same fingerprint = same conditions.** When fingerprints differ, results must not be
compared directly.

---

## 6 Model-dependent conditions (T5)

### 6.1 Why it needs separate rules

T5 tests "whether the model correctly invokes a tool the plugin registers".
The result of this stage **depends on both the plugin and the model**; changing the
model can reverse the conclusion entirely.

**Therefore:** a T5 verdict is only meaningful when bound to a specific model.

### 6.2 Fixed conditions

**In addition to all conditions in chapter 5, T5 must also fix the following:**

#### 6.2.1 Model identifier

| Item | Requirement |
|---|---|
| Must record | model ID **and version identifier** |
| Prohibited | vendor name only (e.g. "DeepSeek") or series name only (e.g. "V4") |
| Example | `deepseek-v4.1-flash-0731` |

#### 6.2.2 Generation parameters

| Parameter | Must record |
|---|---|
| temperature | ✅ |
| max_tokens | ✅ |
| top_p (if used) | ✅ |
| tool calling enabled | ✅ |

**A T5 report that does not record temperature and max_tokens is not reproducible.**

#### 6.2.3 Prompts

| Item | Requirement |
|---|---|
| System prompt | record in full |
| Test prompt | record in full |
| Prompt language | record (Chinese / English / other) |

> **Note:** every character of a prompt affects model behaviour. Writing only
> "ask it to call the tool" does not constitute a reproducible condition.

#### 6.2.4 Tool-calling configuration

| Item | Requirement |
|---|---|
| Tool count (total tools available in the same round) | must record |
| Parallel tool calling enabled | must record |
| Tool-selection strategy (if configurable) | must record |

> **Why record the total tool count.** The more candidate tools, the higher the chance
> the model picks wrong. "Pick correctly among 3 tools" and "among 200" are not the
> same difficulty.

#### 6.2.5 Repeat count

| Situation | Minimum runs |
|---|---|
| Publicly published T5 verdict | **3** |
| Internal evaluation | 1 |

**Pass criteria (choose one; the report must state which):**

| Criterion | Definition | Use |
|---|---|---|
| **All** | 3 of 3 correct invocations | for certification |
| **Majority** | at least 2 of 3 correct | for evaluation, not for certification |

> **Why the T5 bar is higher than T0–T4.** Models are stochastic; passing twice could
> be luck.

### 6.3 Model-dependent test environment fingerprint

```
<model-independent fingerprint>;model=<model ID>;temp=<temperature>;maxtok=<max output>;tools=<total tools>
```

**Example:**
```
dsh=0.2.2-rc.1;img=dsh-sandbox:5.0@sha256:abc...;node=24.11.1;pnpm=9.15.9;mem=768m;cpu=1;model=deepseek-v4.1-flash-0731;temp=0;maxtok=4096;tools=7
```

---

## 7 Mandatory fields of a test report

### 7.1 All tests

| Field | Mandatory |
|---|---|
| Plugin identity (npm name or `github:owner/repo`) | ✅ |
| Test environment fingerprint | ✅ |
| Test time (including time zone) | ✅ |
| Duration of each stage | ✅ |
| Verdict | ✅ |
| First error (if failed) | ✅ |
| Test executor identity | ✅ |

### 7.2 Additional mandatory fields for T5

| Field | Mandatory |
|---|---|
| Model-dependent environment fingerprint | ✅ |
| Full system prompt | ✅ |
| Full test prompt | ✅ |
| Invocation record for every attempt | ✅ |
| Which pass criterion was used | ✅ |

---

## 8 Handling condition deviation

### 8.1 What counts as deviation

Changing the value of any fixed condition in chapter 5 or chapter 6 constitutes a
deviation.

### 8.2 Requirements for deviation

**Deviation is not prohibited, but it must:**

1. Be marked explicitly in the report as "condition deviation"
2. List item by item what deviated, what it was changed to, and why
3. **The verdict must not be compared with undeviated reports**
4. **Must not be used as a basis for certification**

### 8.3 Common deviations

| Deviation | Meaningful | Note |
|---|---|---|
| Changing the DSH version | ✅ yes | That is exactly what is being tested (cross-version compatibility) |
| Changing the container image | ⚠️ careful | Different environment; results not comparable |
| Raising memory | ⚠️ | May let a plugin pass that would otherwise OOM |
| T2–T4 with network on | ❌ **not accepted** | Breaks the security premise; result void |
| Reducing the repeat count | ❌ not accepted (for certification) | See 5.1.9 and 6.2.5 |
| Not recording model parameters (T5) | ❌ **not accepted** | Result not reproducible |

---

## 9 Stability of conditions

### 9.1 Monitoring condition drift

**At least once every 30 days, verify the following against this standard:**

| Item | Verification method |
|---|---|
| Image digest | `docker inspect --format='{{.Id}}' dsh-sandbox:5.0` |
| Node version | `node --version` inside the container |
| pnpm version | `pnpm --version` inside the container |
| DSH version | `dsh --version` inside the container |

**When drift is found:**
1. Mark old test results as "conditions changed"
2. Re-test key samples under the new conditions
3. Update this standard or restore the conditions

### 9.2 Why monitor

If image tag `5.0` is rebuilt, the Node version under the same tag may have changed.
**Without monitoring, test results silently lose comparability.**

---

## Annex A (normative) Condition quick-reference

| Item | T0 | T1 | T2 | T3 | T4 | T5 |
|---|---|---|---|---|---|---|
| Container | not needed | ✅ | ✅ | ✅ | ✅ | ✅ |
| Network | — | on | **off** | **off** | **off** | on |
| Memory | — | 768m | 768m | 768m | 768m | 768m |
| CPU | — | 1 | 1 | 1 | 1 | 1 |
| `--user node` | — | ✅ | ✅ | ✅ | ✅ | ✅ |
| `--cap-drop ALL` | — | — | — | ✅ | ✅ | ✅ |
| `no-new-privileges` | — | — | — | ✅ | ✅ | ✅ |
| `--pids-limit` | — | — | — | — | 128 | 128 |
| Timeout | 30s | 180s | 90s | 90s | 90s | 300s |
| Repeat runs | 1 | 2 | — | — | — | 3 |
| Model parameters | — | — | — | — | — | **mandatory** |

**Note:** T2–T4 are not repeated separately; they are part of the same sequence that
follows T1. "Repeat runs" refers to how many times the whole sequence is executed.

---

## Annex B (informative) Relationship to other standards

```
DSHIO/T 00  Terms and Definitions
      ↓ referenced by
DSHIO/T 21  Test Conditions (this document)
      ↓ referenced by
DSHIO/T 20  General Rules for Testing  ← what each stage does
      ↓ referenced by
DSHIO/T 10  Plugin Standard            ← what a plugin must achieve
      ↓ referenced by
DSHIO/T 32  Certification Rules        ← how conformance is certified
```

**This standard answers only "under what conditions", not "how to test" or
"what counts as passing".**

---

## Revision record

| Version | Date | Change | Proposed by | Participants | Conflict-of-interest statement |
|---|---|---|---|---|---|
| v1.0 (Trial) | 2026-11-07 | First release | DSHIO | DSHIO | The test platform operated by DSHIO is the sole implementer of this standard; the conditions specified here are the conditions that platform currently runs under, and are publicly verifiable. |
