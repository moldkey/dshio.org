# DSHIO/T 36—2026 Requirements for Testing Organizations

| | |
|---|---|
| **Standard No.** | DSHIO/T 36—2026 |
| **Title** | Requirements for Testing Organizations |
| **Version** | v1.0 (Trial) |
| **Published** | 2026-11-08 |
| **Effective** | 2026-11-08 |
| **Status** | Trial (2026-11-08 → 2027-02-08; becomes current automatically) |
| **Original** | Simplified Chinese |
| **Layer** | Management standard |

---

> ## 📋 Current scope
>
> **This standard is in effect. No application, no approval, no fee.**
>
> **Any organization or individual that meets the conditions in this standard may
> issue DSHIO test reports.**
>
> **DSHIO does not currently offer certification.** DSHIO/T 32 remains
> "Trial · not in effect".

---

## 1 Scope

This standard specifies the conditions that organizations issuing DSHIO test reports
must satisfy, and what such reports must contain.

**This standard does not specify:** certification, accreditation, admission, or registration.
**There is no gate. There are only conditions.**

---

## 2 Design principle: no gatekeeping

### 2.1 Why not an accreditation regime

Traditional testing-lab accreditation (apply → review → certificate → annual audit) has three problems:

| Problem | Consequence |
|---|---|
| Cost of review | Only large organizations can afford it; small labs and individuals are excluded |
| Accreditation becomes a form of power | Who gets accredited may be influenced by non-technical factors |
| De facto monopoly | The fewer accredited labs, the weaker the independence of results |

**DSHIO does not want this.**

### 2.2 Instead: publish the conditions, make results reproducible

```
Anyone can test
    ↓
but must follow DSHIO/T 20 methods and DSHIO/T 21 conditions
    ↓
must publish the environment fingerprint and full evidence
    ↓
anyone — the author, another lab, a passer-by — can reproduce or challenge
```

**Trust comes not from *who* ran the test, but from *how* it was run and whether the evidence is public.**

### 2.3 Bad results get eliminated by comparison

If a lab tests incorrectly:

- The author can compare against another lab's report
- Anyone can re-run under the same conditions
- Disagreements are recorded publicly

**No prior review needed. Post-hoc comparison is enough.**

---

## 3 Terms and definitions

Terms from DSHIO/T 00—2026 apply. **Additional definitions:**

### 3.1 Testing organization
An organization or individual that issues DSHIO test reports.
**No DSHIO approval is required; self-declaration of conformance with this standard suffices.**

### 3.2 Test report
A report produced by applying the methods of DSHIO/T 20 under the conditions of
DSHIO/T 21, in the format specified in clause 5 of this standard.

### 3.3 Self-declaration
A public statement by a testing organization that it meets all conditions in clause 4.
**DSHIO does not verify the truth of that statement; the public does.**

### 3.4 Report equivalence
Two reports are directly comparable when their environment fingerprints match and
they cover the same clauses.

---

## 4 Conditions a testing organization must meet

### 4.1 Mandatory conditions

**Meeting all conditions is sufficient to issue reports. No application, no authorisation.**

| # | Condition | How to demonstrate |
|---|---|---|
| 1 | Execute all stages T0–T4 of DSHIO/T 20 | List each stage in the report |
| 2 | Test conditions comply with DSHIO/T 21 clause 5 | Report includes the fingerprint |
| 3 | Report includes all mandatory fields of DSHIO/T 21 clause 7 | The report itself |
| 4 | Retain the raw output of every stage | Publicly reachable evidence address |
| 5 | Run twice and reach the same verdict | Repeat record in the report |
| 6 | Publish a conflict-of-interest statement | See 4.2 |

### 4.2 Conflict-of-interest statement

**A testing organization must publicly state:**

1. Whether it develops or sells DSH plugins
2. Whether it has a commercial relationship with the plugin's author
3. Whether it charges fees contingent on the verdict

**If any applies:**

- It must be marked prominently in the report
- **The related plugin should be retested by another organization**
  (not mandatory — but without such a retest, readers judge credibility for themselves)

> **Why declare rather than recuse.**
> We have no authority to forbid anyone from testing any plugin.
> But we can require that **the relationship is written on the report.**
> A reader who sees "this report was issued by the plugin author's employer"
> knows what to make of it.

### 4.3 Fees

**DSHIO charges nothing for testing and takes no share of testing revenue.**

How a testing organization charges its clients is its own business. However:

- It must not pay DSHIO anything related to a report (DSHIO charges nothing)
- It must not charge under the name of "DSHIO authorisation" (there is no such thing)

---

## 5 Report contents

Per DSHIO/T 21 clause 7. **Additional requirements:**

| Field | Requirement |
|---|---|
| **Tester identification** | Name plus a reachable URL |
| **Self-declaration** | Explicitly state "this organization self-declares conformance with DSHIO/T 36" |
| **Conflict of interest** | Per 4.2 |
| **Raw evidence address** | Publicly reachable |
| **Environment fingerprint** | Complete, including the `perm` component (DSHIO/T 21 §5.2) |

**The report must state explicitly:**

> This is a test report. **It does not constitute certification.**
> DSHIO does not currently offer certification.

---

## 6 Public list

### 6.1 Nature of the list

DSHIO maintains a **public list of testing organizations** at dshio.org.

**This is not an accreditation list. It is an information directory.**

- Listing ≠ approval
- Not listed ≠ prohibited
- No approval needed to be listed; self-declaration suffices
- DSHIO does not endorse anyone on the list

### 6.2 How to be listed

Any organization or individual may submit the following to DSHIO:

```
Organization name:
Website:
Fingerprint of the most recent complete run:
Conflict-of-interest statement:
```

DSHIO lists it within 5 working days. **No technical review is performed.**

### 6.3 Removal

**Removal occurs only when:**

1. The organization asks
2. No valid report for 12 consecutive months
3. A report is proven inconsistent with the actual run (public evidence required)

**Removal does not affect the validity of reports already issued.**

---

## 7 Conflicting reports

### 7.1 Same plugin, same version, two reports disagree

1. Both are published
2. **Do not pick the favourable one**
3. Readers judge for themselves, or request a third-party retest
4. DSHIO records the disagreement as input to revising DSHIO/T 20

> **Disagreement itself is valuable.** It may expose ambiguity in the test method.

### 7.2 When a verdict is invalid

Any of the following renders the verdict invalid:

| Situation | Reason |
|---|---|
| Fingerprint does not match DSHIO/T 21 | Not comparable |
| Only one run recorded | Not reproducible |
| Raw evidence not published | Not auditable |
| Conflict of interest concealed | Integrity defect |

---

## 8 Responsibility

### 8.1 The report belongs to its issuer

**Whoever issues a report is responsible for its accuracy.**

| Party | Responsible for | Not responsible for |
|---|---|---|
| **DSHIO** | Correctness and executability of the standard itself | **The truth of any test report** |
| **Testing organization** | The accuracy and completeness of reports it issues | Other organizations' reports |
| **Author / reader** | Choosing which report to believe | — |

### 8.2 DSHIO does not endorse

**DSHIO assumes no responsibility for any test report, including:**

- Not guaranteeing the report's content is true
- Not guaranteeing the lab followed the conditions
- Not liable for any consequence of the report
- Not endorsing anyone merely by listing them

**When a reader sees a report, it should be clear:**

> "**X said this** — DSHIO did not."

### 8.3 Why the line is drawn here

If DSHIO endorsed reports, it would have to review every report — which returns us to
the accreditation regime we rejected.

Moreover, DSHIO has no capacity to verify every report.
**Claiming otherwise would be a lie.**

**So we do only the one thing we can be responsible for: writing the standard clearly.**

### 8.4 What to do about a false report

Anyone may publicly point out that a report is inconsistent with actual behaviour,
**with reproducible evidence**.

DSHIO will:

1. Record the dispute
2. If the evidence holds, remove the organization from the public list
3. **Not arbitrate between the organization and the author**

DSHIO has no arbitration authority and does not want one.

---

## 9 Relationship to other standards

```
DSHIO/T 00  Terms and definitions
      ↓
DSHIO/T 20  General rules for testing  ── what to do, what to observe, how to judge
DSHIO/T 21  Test conditions            ── under what conditions
      ↓
DSHIO/T 36  Requirements for testing organizations (this document) ── who does it
      ↓
DSHIO/T 37  Submission format  ── how to publish results
      ↓
DSHIO/T 32  Certification rules ── not in effect
```

**This standard answers "what conditions must a report satisfy", not "how to test" (T20/T21).**

**This standard does not concern certification.** Certification is a separate matter,
and is not currently offered.

---

## Appendix A (informative) Currently public testing organizations

| Organization | URL | Stages covered | Self-declared |
|---|---|---|---|
| deepseekbase.com | https://deepseekbase.com | T0–T4 | 2026-11-08 |

**Conflict-of-interest statement:** the operator of deepseekbase.com also develops
the `@liangjiehappy/*` plugin series. Per 4.2, reports for those plugins are marked accordingly.

> **This list is information, not approval.**
> Anyone else meeting clause 4 may also issue reports — no need to notify us.

---

## Appendix B (informative) How to start issuing reports

```
1. Read DSHIO/T 20 (how to test) and DSHIO/T 21 (under what conditions)
2. Prepare an environment meeting DSHIO/T 21 clause 5
3. Run T0–T4 per T20, retain all output
4. Write the report per DSHIO/T 21 clause 7
5. Publish the report and raw evidence
6. State in the report: "self-declares conformance with DSHIO/T 36"
7. (Optional) submit your details to the public list at dshio.org
```

**There is no step 8. No one's approval is required.**

---

## Revision record

| Version | Date | Change | Proposed by | Participants | Conflict statement |
|---|---|---|---|---|---|
| v1.0 (Trial) | 2026-11-08 | First release | DSHIO | DSHIO | deepseekbase.com, operated by DSHIO, is the first public testing organization; this standard accredits no one, so no preference is conferred |
