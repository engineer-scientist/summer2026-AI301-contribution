# Contribution 2: Validator error message references non-existent parameter `bc_{dir}%Tw_out` (should be `Twall_out`)

**Contribution Number:** 2, 

**Student:** Sarthak Sharma, 

**Issue:** [#1480 — MFlowCode/MFC](https://github.com/MFlowCode/MFC/issues/1480), 

**Fork:** https://github.com/engineer-scientist/Multiphase-Flow-Code, 

**Pull request:** https://github.com/MFlowCode/MFC/pull/1666, 

**Status:** Phase III (Pull Request submitted) — In Progress.

---

## Why I Chose This Issue

This is my second contribution, and I chose it deliberately as a step up in *codebase* complexity while keeping the *change* small and low-risk. MFC (Multi-Component Flow Code) is a large, real-world scientific HPC project — a 2025 Gordon Bell Prize finalist developed at Georgia Tech — so contributing to it means learning to navigate a big, unfamiliar repository, its Python toolchain, and its contribution process, rather than just editing a toy project. The issue is labeled both `bug` and `good first issue`, which made it a good match for building confidence with the fork → branch → pull request → CI workflow on a serious project.

The fix itself is a one-line diagnostic-message correction, so my learning goal here isn't the difficulty of the code change — it's everything *around* it: reading enough of the surrounding validator code to be sure I understand what the message is for, following the project's coding standards and PR template, and correctly linking a pull request to an issue so it closes automatically on merge.

---

## Understanding the Issue

### Problem Description

In `toolchain/mfc/case_validator.py`, the check that validates the exit-side isothermal wall temperature prints an error message that names a parameter which does not exist. The code correctly *reads* the value from `bc_{dir}%Twall_out`, but the error string shown to the user refers to `bc_{dir}%Tw_out`. There is no `Tw_out` parameter anywhere in MFC — the real parameter is `Twall_out`.

### Expected Behavior

When a user sets a non-positive exit wall temperature for an isothermal-out boundary condition, the validator should reject it and tell the user to fix `bc_{dir}%Twall_out` — the actual parameter name they need to change.

### Current Behavior

The validator correctly rejects the non-positive value, but the error message tells the user to fix `bc_{dir}%Tw_out`, a parameter that doesn't exist. This misdirects the fix: the user goes looking for a parameter they can't find. The validation logic itself is correct — only the message text is wrong.

### Affected Components

- **`toolchain/mfc/case_validator.py`** — the isothermal-out wall-temperature positivity check. The value is fetched correctly as `bc_{dir}%Twall_out`, but the `prohibit` call for the positivity check hardcodes the wrong name `bc_{dir}%Tw_out` in its message string. (The parallel `Twall_in` branch prints its name correctly, which is what the fix should match.)
- **`toolchain/mfc/params/definitions.py`** — where `Twall_out` is actually defined (`"Temperature of the exit-side isothermal wall."`), confirming the correct name. This file is reference-only; no change here.

---

## Reproduction Process

### Environment Setup

I worked on a Windows laptop using **WSL (Windows Subsystem for Linux)** with **VS Code** connected via the WSL extension, so the editor UI runs on Windows while all files and git operations happen inside the Linux environment.

Key setup decisions and challenges:
- **Used WSL rather than PowerShell**, because MFC's tooling (`./mfc.sh`) is a bash script intended for Linux/WSL.
- **Cloned the repo inside the WSL home directory** (not under `/mnt/c/...`) to avoid Windows↔Linux line-ending (CRLF vs LF) conversions that would otherwise pollute the diff with phantom changes.
- Added the upstream remote so I can keep my fork in sync:
  ```bash
  git clone https://github.com/engineer-scientist/Multiphase-Flow-Code.git
  cd Multiphase-Flow-Code
  git remote add upstream https://github.com/MFlowCode/MFC.git
  ```

### Steps to Reproduce

Because this is a diagnostic-message-only bug, the primary reproduction is by **code inspection** of the affected path; the runtime scenario that would surface the message is also described below.

Code-inspection reproduction:
1. Open `toolchain/mfc/case_validator.py` and locate the isothermal-out wall-temperature check.
2. Observe that the value is fetched as `bc_{dir}%Twall_out`, but the positivity-check `prohibit` message string contains `bc_{dir}%Tw_out`.
3. Confirm `Tw_out` is not a real parameter by searching the codebase (see findings below).

Runtime scenario that triggers the message (optional verification):
1. Create a case that enables an isothermal-out boundary condition (`bc_{dir}%isothermal_out`).
2. Set the corresponding `bc_{dir}%Twall_out` to a non-positive value (e.g. `0` or a negative number).
3. Run the case validator. **Observed result:** the validation correctly fails, but the printed error names `bc_{dir}%Tw_out` instead of `bc_{dir}%Twall_out`.

### Reproduction Evidence

- **Commit showing reproduction:** [Not applicable — diagnostic-only bug reproduced by code inspection. If you ran the runtime scenario, link the case/commit here.]
- **Screenshots/logs:** None required.
- **My findings:** Searching the codebase for `Tw_out` returned exactly one occurrence — the incorrect message string. Searching for `Twall_out` confirmed it is the real, defined parameter (`toolchain/mfc/params/definitions.py`) and that it is used correctly for the value fetch on the line directly above the buggy message. This matches the issue author's report that the bug is a message-string typo with no effect on validation logic.

---

## Solution Approach

### Analysis

The root cause is an inconsistency introduced when the message string was written: the code fetches the value using the correct parameter name (`Twall_out`) but the human-readable message hardcodes an abbreviated name (`Tw_out`) that was never a real parameter. The neighboring `Twall_in` branch prints its parameter name correctly, so the fix is simply to bring the `Twall_out` message into line with both the fetch above it and the `Twall_in` branch beside it.

### Proposed Solution

Change the single occurrence of `bc_{dir}%Tw_out` in the positivity-check message to `bc_{dir}%Twall_out`. No logic changes.

### Implementation Plan

Using the UMPIRE framework (adapted):

**Understand:** The validator's error message for a non-positive exit wall temperature names a non-existent parameter (`Tw_out`), misdirecting users to fix something that doesn't exist. The correct name is `Twall_out`.

**Match:** The parallel inlet branch (`Twall_in`) already prints its parameter name correctly, and the line immediately above the bug already fetches the value using the correct `Twall_out` name. The fix just makes the message consistent with these existing correct patterns.

**Plan:**
1. Locate the incorrect string by searching for `Tw_out` in `toolchain/mfc/case_validator.py` (more reliable than trusting a line number, which can drift between commits).
2. Change `bc_{dir}%Tw_out` to `bc_{dir}%Twall_out` in that message only.
3. Verify with `git diff` that exactly one line changed and no line-ending noise was introduced.

**Implement:** Branch `fix/issue-1480` on my fork: https://github.com/engineer-scientist/Multiphase-Flow-Code/tree/fix/issue-1480

**Review:** Self-review against the MFC developer guide — single-purpose change, message consistent with the `Twall_in` branch, no unrelated edits, diff is a single line.

**Evaluate:** Confirm the change is output-neutral (message text only) so existing golden-file tests still pass, and confirm `Tw_out` no longer appears anywhere in the codebase.

---

## Testing Strategy

### Unit Tests

- [ ] N/A — no new unit test warranted. This is a message-string-only change with no effect on control flow or numerical output. MFC's test suite is golden-file based; because the change is output-neutral, existing tests cover it and no golden files change.

### Integration Tests

- [ ] N/A — no integration behavior changes; the validation logic is unchanged.

### Manual Testing

- Confirmed via search that `Tw_out` appeared exactly once (the incorrect message) and no longer appears after the change.
- Confirmed `Twall_out` is the parameter actually defined in `toolchain/mfc/params/definitions.py`.
- Reviewed `git diff` to confirm a single changed line with no unrelated or line-ending changes.

---

## Implementation Notes

### Week [X] Progress

I located the string, made and pushed the change, opened the pull request. Confirmed that the PR targeted `master`.

### Code Changes

- **Files modified:** `toolchain/mfc/case_validator.py` (one line)
- **Key commits:** https://github.com/engineer-scientist/Multiphase-Flow-Code/tree/fix/issue-1480
- **Approach decisions:** Located the string by searching for `Tw_out` rather than editing by line number, since line numbers had likely drifted from the commit (`40dde5e`) the issue was filed against. Matched the corrected message to the existing `Twall_in` branch for consistency.

---

## Pull Request

**PR Link:** https://github.com/MFlowCode/MFC/pull/1666

**PR Description:**

> The isothermal-out wall-temperature positivity check in `toolchain/mfc/case_validator.py` printed a parameter name that does not exist. The value is correctly fetched as `bc_{dir}%Twall_out`, but the positivity-check error message referred to `bc_{dir}%Tw_out`. The actual parameter is `Twall_out` (defined in `toolchain/mfc/params/definitions.py`); there is no `Tw_out` parameter. This changes the message string to `bc_{dir}%Twall_out`, matching the parallel `Twall_in` branch. Diagnostic-message-only — no validation logic is affected, and the change is output-neutral / golden-file-safe.
>
> Closes #1480.

**Maintainer Feedback:**

- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** Awaiting review

---

## Learnings & Reflections

### Technical Skills Gained

Navigating a large scientific HPC codebase, using VS Code with WSL, avoiding CRLF/LF diff noise, reading a validator's control flow to understand a message in context, and correctly linking a PR to an issue with a closing keyword.

### Challenges Overcome

Ensuring the edit produced a clean single-line diff, and confirming the PR targeted the default branch (`master`) so the `Closes #1480` keyword would actually auto-close the issue.

### What I'd Do Differently Next Time

Set up `git config` for line endings before the first edit, or try running the validator on a small case to capture before/after screenshots of the message.

---

## Resources Used

- [MFC Contributing guide](https://mflowcode.github.io/documentation/contributing.html) — coding standards and PR process
- [MFC Getting Started](https://mflowcode.github.io/documentation/md_getting-started.html) — Windows/WSL setup
- [GitHub Docs: Linking a pull request to an issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue) — closing keywords and how auto-close works
- [Issue #1480](https://github.com/MFlowCode/MFC/issues/1480) — original report with file/line references
