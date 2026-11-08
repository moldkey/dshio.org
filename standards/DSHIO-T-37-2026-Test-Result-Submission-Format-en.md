# DSHIO/T 37—2026 Test Result Submission Format

| | |
|---|---|
| **Standard No.** | DSHIO/T 37—2026 |
| **Title** | Test Result Submission Format |
| **Version** | v1.0 (Trial) |
| **Published** | 2026-11-08 |
| **Effective** | 2026-11-08 |
| **Status** | Trial (2026-11-08 → 2027-02-08; becomes current automatically) |
| **Original** | Simplified Chinese |
| **Layer** | Management standard |

---

> ## 📋 Current scope
>
> **This standard is in effect.**
> Submission repository: [github.com/moldkey/dsh-conformance](https://github.com/moldkey/dsh-conformance)
>
> **The validator checks completeness, not truth.**
> Whether a submission is accurate is the responsibility of whoever submitted it.

---

## 1 Scope

This standard specifies the directory layout, required files, field requirements and
validation rules for submitting DSH plugin test results.

**Applies to:** any organization or individual submitting test results per DSHIO/T 36.

**Does not apply to:** certification. DSHIO does not currently offer certification.

---

## 2 Why the submission format is specified

### 2.1 Without a format, results cannot be compared

If two labs write reports differently — different fields, different granularity — a reader
cannot tell whether a difference comes from the plugin or from the writing.

**Once the format is fixed, "pass" means the same thing for every submitter.**

### 2.2 The format must be machine-checkable

Reviewing every submission by hand is not realistic.
**It must be checkable by a script, with a predictable outcome.**

### 2.3 But only completeness can be checked

The validator does not run tests and does not judge accuracy. It answers one question:

> "Is this submission complete and internally consistent?"

**Accuracy is the submitter's responsibility (DSHIO/T 36 §8).**

---

## 3 Directory layout

```
results/
  v<baseline-dsh-version>/
    <plugin-slug>/
      README.md
      result.json
      TESTER.yaml
```

### 3.1 Version directory

| Item | Requirement |
|---|---|
| Format | `v<major>.<minor>.<patch>[-prerelease]` |
| Value | The **exact output** of `dsh --version`, prefixed with `v` |
| Forbidden | Ranges, `latest`, `x.y.x` |
| Example | `v0.2.2-rc.1` |

### 3.2 Plugin slug

| Item | Requirement |
|---|---|
| Character set | `a-z0-9._-` |
| Conversion | `@scope/pkg` → `scope-pkg`; `pkg` → `pkg` |
| Case | Lowercase only |

### 3.3 One version per directory

**Different versions of the same plugin are different directories.**
Do not put multiple versions in one directory.

---

## 4 Required files

### 4.1 Three files, all mandatory

| File | Purpose |
|---|---|
| `README.md` | Human-readable reproduction instructions |
| `result.json` | Machine-readable full raw output |
| `TESTER.yaml` | Submitter declaration |

**No other files may be present.** Extra files make it unclear what was reviewed.

### 4.2 Required fields in `result.json`

| Field | Type | Requirement |
|---|---|---|
| `plugin` | string | npm package name |
| `version` | string | Exact plugin version tested |
| `tested_at` | string | `YYYY-MM-DD HH:MM:SS` |
| `timezone` | string | e.g. `+08:00` |
| `baseline_dsh` | string | Must equal the directory version (without `v`) |
| `standard` | string | Must mention `DSHIO/T 20` and `DSHIO/T 21` |
| `fingerprint` | string | Environment fingerprint, see 4.3 |
| `stages` | object | Five stages, see 4.4 |
| `verdict` | string | One of five values, see DSHIO/T 00 §3.5.3 |
| `repeat` | object | `{count, verdicts, consistent}` |

### 4.3 Environment fingerprint

Must follow DSHIO/T 21 §5.2:

```
dsh=<version>;img=<image>@<digest>;node=<version>;pnpm=<version>;mem=<mem>;cpu=<cpu>[:perm=<permissions>]
```

**Six mandatory components:** `dsh`, `img`, `node`, `pnpm`, `mem`, `cpu`.
`perm` is recommended.

### 4.4 Stage records

`stages` must contain five keys: `static`, `install`, `list`, `config`, `boot`.

Each stage object must include at least:

```json
{"exit": 0, "duration": 3.4, "stdout": "...", "stderr": "..."}
```

**Raw output may be truncated, but truncation must be marked:**

```json
{"stdout_truncated": true, "stdout_bytes_total": 481203}
```

### 4.5 `repeat` requirements

| Field | Requirement |
|---|---|
| `count` | ≥ 2 (DSHIO/T 10 §7.4) |
| `verdicts` | The verdict of each run |
| `consistent` | Whether the runs agreed |

**A submission with `consistent: false` must be marked unstable and may not be used for endorsement.**

### 4.6 Required fields in `TESTER.yaml`

| Field | Type | Notes |
|---|---|---|
| `name` | string | Organization name |
| `url` | string | Reachable website |
| `contact` | string | Contact email |
| `scope` | object | `stages` and `plugin_types` |
| `environment` | object | Image, digest, node, pnpm, mem, cpu |
| `conflicts` | object | **See 4.7** |
| `declaration` | string | Self-declaration accepting responsibility |

### 4.7 Conflict of interest (mandatory)

`conflicts` must contain three keys, none optional:

| Key | Type | Notes |
|---|---|---|
| `develops_plugins` | boolean | Whether the tester develops or sells DSH plugins |
| `paid_by_author` | boolean | Whether the tester has a commercial relationship with the author |
| `result_dependent_fees` | boolean | Whether fees depend on the verdict |

**Optional additions:** `plugin_scope`, `notes`.

> **Why declare rather than recuse.**
> We have no authority to forbid anyone from testing any plugin.
> But we can require that **the relationship is written on the submission.**
> A reader who sees "this report was issued by the plugin author's employer"
> knows what to make of it.

### 4.8 `README.md` requirements

Must state:

1. Plugin name and version
2. DSH version used
3. Commands executed
4. Approximate total duration
5. **Deviation from DSHIO/T 21** (state "none" explicitly if there is none)

Without these, a reader cannot reproduce the result.

---

## 5 Validation rules

### 5.1 Twelve checks

| # | Check | Basis |
|---|---|---|
| 1 | Directory is `results/v<ver>/<slug>/` | Path |
| 2 | All three required files present | File listing |
| 3 | `result.json` is valid JSON | Parse |
| 4 | Fingerprint contains six mandatory components | String scan |
| 5 | All five stages present | Key check |
| 6 | Verdict is one of the five values, and no stage failed | Value check |
| 7 | `repeat.count ≥ 2` and `consistent` | Value check |
| 8 | `TESTER.yaml` has all seven required fields | Parse |
| 9 | `conflicts` has all three keys | Parse |
| 10 | `README.md` explains reproduction | Keywords plus length |
| 11 | Baseline matches the directory and is supported | Value check |
| 12 | No extra files in the directory | File listing |

### 5.2 Validator boundaries

**The validator explicitly does not check:**

- Whether the test was actually run
- Whether the output is genuine
- Whether the plugin is any good
- Whether the environment really matched the fingerprint

**These are the submitter's responsibility.**

### 5.3 Handling failure

If any check fails:

1. The submission is labelled `not-verifiable`
2. Each failed check is explained
3. The submitter may correct and resubmit

**`not-verifiable` means "non-conforming format", not "fraudulent".** They are different.

---

## 6 How to submit

### 6.1 One submission per pull request

Each PR submits one plugin version.

**Reason:** with several mixed together, it becomes unclear what was reviewed and what was merged.

### 6.2 Correcting a merged result

Results are immutable once merged.

To correct:

1. Open a new PR
2. Delete the old directory
3. Add the corrected directory
4. Explain in the PR body

**The git history preserves the original. That is the point.**

### 6.3 Bulk submissions

You may first submit a `snapshots/<date>.json` pointing at a downloadable archive,
then add individual directories over time.

**A snapshot is not a conformance result** — it is a pointer.

---

## 7 Supported version range

**The current DSH version plus the two prior versions.**

| Version | Status |
|---|---|
| `0.2.2-rc.1` | ✅ Supported |

The range rolls forward as DSH releases. Submissions outside it are rejected.

---

## 8 Disagreement

### 8.1 Same plugin, same version, two conflicting submissions

1. Both are published
2. **Do not pick the favourable one**
3. Readers may request a third-party retest
4. DSHIO records the disagreement

### 8.2 Adversarial submissions

Anyone may open a "challenge" issue pointing out a problem with a submission,
**with reproducible evidence**.

DSHIO will:

1. Record the dispute
2. If the evidence holds, mark the submission `disputed`
3. **Not arbitrate**

---

## 9 Relationship to other standards

```
DSHIO/T 20  Test methods   ── how to test
DSHIO/T 21  Test conditions ── under what conditions
DSHIO/T 36  Requirements for testing organizations ── who may test
      ↓
DSHIO/T 37  Submission format (this document) ── how to submit
      ↓
DSHIO/T 32  Certification rules ── not in effect
```

---

## Appendix A (normative) Minimal valid submission

```
results/v0.2.2-rc.1/dsh-pet/
├── README.md
├── result.json
└── TESTER.yaml
```

### result.json

```json
{
  "plugin": "dsh-pet",
  "version": "0.4.5",
  "tested_at": "2026-11-08 10:00:00",
  "timezone": "+08:00",
  "baseline_dsh": "0.2.2-rc.1",
  "standard": "DSHIO/T 20, DSHIO/T 21",
  "fingerprint": "dsh=0.2.2-rc.1;img=dsh-sandbox:5.0@sha256:...;node=24.11.1;pnpm=9.15.9;mem=768m;cpu=1;perm=network",
  "stages": {
    "static": {"exit": 0},
    "install": {"exit": 0},
    "list": {"exit": 0},
    "config": {"exit": 0},
    "boot": {"exit": 0}
  },
  "verdict": "pass",
  "repeat": {"count": 2, "verdicts": ["pass", "pass"], "consistent": true}
}
```

### TESTER.yaml

```yaml
name: example.org
url: https://example.org
contact: ci@example.org
scope:
  stages: [T0, T1, T2, T3, T4]
  plugin_types: [tool]
environment:
  image: dsh-sandbox:5.0
  image_digest: sha256:...
  node: "24.11.1"
  pnpm: "9.15.9"
  mem: 768m
  cpu: 1
conflicts:
  develops_plugins: false
  paid_by_author: false
  result_dependent_fees: false
declaration: "Self-declares conformance with DSHIO/T 36."
```

---

## Appendix B (informative) Why this follows Kubernetes

The Kubernetes conformance programme (CNCF) works like this:

1. The vendor runs `sonobuoy run --mode=certified-conformance` itself
2. It opens a PR to `cncf/k8s-conformance` containing `e2e.log`, `junit_01.xml` and `PRODUCT.yaml`
3. A bot verifies 15 technical requirements (files present, directory shape, no failed tests in the log)
4. A human checks policy (membership, trademark)
5. Merging the PR equals certification

**CNCF tests nothing. It requires the raw log to be public, then stamps it.**

We adopt the same model, but lighter:

- No membership requirement
- No fee
- No certification (DSHIO/T 32 is not in effect)

**References:**
- [CNCF Certified Kubernetes](https://www.cncf.io/training/certification/software-conformance/)
- [k8s-conformance instructions](https://github.com/cncf/k8s-conformance/blob/master/instructions.md)

---

## Revision record

| Version | Date | Change | Proposed by | Participants | Conflict statement |
|---|---|---|---|---|---|
| v1.0 (Trial) | 2026-11-08 | First release | DSHIO | DSHIO | deepseekbase.com is the first submitter; this standard applies the same validation rules to it as to any other submitter |
