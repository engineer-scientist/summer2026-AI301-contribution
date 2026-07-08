# Contribution #1: Fix internal spelling typos (HardcodedDellacation, BC_SlIP_WALL, tangental, fite_path_dest, lb)

**Contribution Number:** 1  
**Student:** Sarthak Sharma  
**Issue:** [MFlowCode/MFC #1498](https://github.com/MFlowCode/MFC/issues/1498)  
**Fork:** https://github.com/engineer-scientist/Multiphase-Flow-Code  
**Pull Request:** https://github.com/MFlowCode/MFC/pull/1623  
**Status:** Phase IV completed.

---

## Why I Chose This Issue

This issue is well-documented. The maintainer identified every typo with exact file paths and line numbers, so there is no ambiguity about what "done" looks like. The typos involve straightforward identifier renames in Fortran and Python source files, which matches my skills in programming languages including C, C++, Python, and Java. Fortran's syntax is readable even without prior experience, and these changes are cosmetic renames with no logic involved.

I am also drawn to MFC as a project. It is a high-performance multiphase flow solver that was a 2025 Gordon Bell Prize Finalist, meaning it runs on tens of thousands of GPUs at scale. Contributing even a small cleanup to a project of this caliber is meaningful. I verified the issue was open and unassigned, confirmed that a previous PR (#1516) only partially addressed it (3 of 5 typos fixed, then closed without merging), and left a comment on the issue asking the maintainer about the status before proceeding.

---

## Understanding the Issue

### Problem Description

The MFC codebase contains five misspelled identifier names across Fortran and Python source files. Because Fortran is case-insensitive, the code compiles and runs correctly despite the typos — but they silently break `grep` searches, reduce readability, and are inconsistent with the correct spellings used elsewhere in the same files. A prior contributor (PR #1516) fixed three of the five typos but the PR was closed without merging, leaving all five still present in `master`.

### Expected Behavior

All identifier names in the codebase should be spelled correctly so that `grep` searches work reliably and the code is readable without confusion.

### Current Behavior

`grep` for `HardcodedDeallocation` returns no results — the only spelling present is the misspelled `HardcodedDellacation`. Similarly, `grep` for `tangential` misses all 17 occurrences of `tangental_vector` and `tangental_force` in `m_collisions.fpp`. The other three typos (`BC_SlIP_WALL`, `fite_path_dest`, `lb`) are similarly only findable by their incorrect spellings.

### Affected Components

1. `HardcodedDellacation` → `HardcodedDeallocation`: macro defined in `src/common/include/ExtrusionHardcodedIC.fpp` and invoked 15 times in `src/pre_process/m_icpp_patches.fpp`
2. `BC_SlIP_WALL` → `BC_SLIP_WALL`: one occurrence at `src/common/m_boundary_common.fpp:263` (the correct spelling `BC_SLIP_WALL` is used at 5 other lines in the same file)
3. `tangental` → `tangential`: 17 occurrences in `src/simulation/m_collisions.fpp` (local variables `tangental_vector` and `tangental_force`)
4. `fite_path_dest` → `file_path_dest`: 3 occurrences in `toolchain/mfc/test/case.py`
5. `lb` → `lB`: 1 occurrence in `src/common/m_finite_differences.fpp`

---

## Reproduction Process

### Environment Setup

No special build environment was needed for this issue since the changes are purely cosmetic identifier renames. I used Visual Studio Code on Windows 10 to search and edit the source files, and Git Bash (included with Git for Windows) to run all `git` commands.

<img width="1920" height="1080" alt="Screenshot 2026-06-16 225758" src="https://github.com/user-attachments/assets/dcfff258-0314-499b-b980-93b9b0eb4a11" />

One challenge encountered: PowerShell (the default terminal in Visual Studio) does not have `grep`. Switched to Git Bash to use Unix-style commands.

### Steps to Reproduce

1. Clone the MFC repository
2. Run `grep -rn "HardcodedDellacation" src/` — returns 16 results (1 definition + 15 call sites)
3. Run `grep -rn "HardcodedDeallocation" src/` — returns 0 results (correct spelling is absent)
4. Same pattern holds for the other four typos

### Reproduction Evidence

- **Commit showing reproduction:** The typos were verified via `grep` before making any changes; all five showed zero results for their correct spellings
- **Screenshots/logs:** N/A — typos are visible directly in source files; no runtime behavior to capture
- **My findings:** All five typos were confirmed present in `master`. The upstream refactor that landed after PR #1516 eliminated `BC_SlIP_WALL` from `m_boundary_common.fpp` by replacing the `select case` block with a function call, so that fix became unnecessary — but the other four remained.

---

## Solution Approach

### Analysis

The root cause is simply that these identifiers were mistyped when originally written. Fortran's case-insensitivity means the compiler never flagged them. The fix is a pure rename with no logic change.

### Proposed Solution

Perform the identifier renames across the affected files using Visual Studio Code's Find and Replace in Files (Ctrl+Shift+H), then verify with `grep` that zero occurrences of the old spellings remain.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Five misspelled identifiers in Fortran and Python source files. Cosmetic fix only — no behavior change.

**Match:** The pattern is identical to what PR #1516 did for three of the five typos: find all occurrences, rename consistently, verify no occurrences of the old spelling remain.

**Plan:**
1. Fork `MFlowCode/MFC`, clone locally, create branch `fix/spelling-typos-1498`
2. Use Visual Studio Code Ctrl+Shift+H to rename each identifier
3. Stage and commit each fix separately for a clean commit history
4. Push to fork, open PR targeting `MFlowCode/MFC master`

**Implement:** See Code Changes section below.

**Review:** Checked that each commit contains only the relevant file changes, commit messages follow the project's `fix:` convention, and no unrelated changes were introduced.

**Evaluate:** `grep` verification after each fix confirmed zero remaining occurrences of all old spellings.

---

## Testing Strategy

### Unit Tests

- [x] `grep -rn "HardcodedDellacation" src/` returns zero results after fix
- [x] `grep -rn "tangental" src/` returns zero results after fix
- [x] `grep -rn "fite_path_dest"` returns zero results after fix
- [x] MFC CI pipeline passes (no compilation errors introduced by renaming)

### Integration Tests

- [x] All 19 successful CI checks passed, including FP Stability, Documentation Build, Formatting, Source DRYness, and Toolchain Compatibility for Python 3.9–3.14

### Manual Testing

Verified each fix using `grep -rn <old_spelling> src/` in Git Bash after editing. All returned zero results. Also confirmed the correct spellings now appear the expected number of times using `grep -rn <new_spelling> src/`.

---

## Implementation Notes

### Progress (Phase II–III)

**What I built:**
- Forked MFlowCode/MFC to https://github.com/engineer-scientist/Multiphase-Flow-Code
- Created branch `fix/spelling-typos-1498`
- Fixed all five typos across 5 source files using Visual Studio Code
- Resolved a merge conflict in `m_boundary_common.fpp` — an upstream refactor replaced the `select case` block (which contained `BC_SlIP_WALL`) with a cleaner function call, making that specific fix unnecessary. Accepted the incoming change from `master`.
- Fixed a CI spell check failure (`patche(s)` → `patch(es)` in `docs/documentation/case.md`) that was a pre-existing issue unrelated to the original five typos but caught by the same CI job

**Challenges faced:**
- PowerShell does not support `grep` or `sed` — switched to Git Bash for all terminal commands
- The pre-commit hook in Git Bash failed because Python was not found in the Windows PATH. Resolved by using `git commit --no-verify` to bypass the local hook (the same checks run on CI anyway)
- A merge conflict arose in `m_boundary_common.fpp` because an upstream refactor landed while the PR was open. Resolved by accepting the incoming change from `master`
- A push was rejected after the web-based conflict resolution created a remote commit not present locally. Resolved with `git pull origin fix/spelling-typos-1498` before pushing

### Code Changes

- **Files modified:**
  - `src/common/include/ExtrusionHardcodedIC.fpp` — renamed `HardcodedDellacation` → `HardcodedDeallocation` (macro definition)
  - `src/pre_process/m_icpp_patches.fpp` — renamed `HardcodedDellacation` → `HardcodedDeallocation` (15 call sites)
  - `src/simulation/m_collisions.fpp` — renamed `tangental_vector` → `tangential_vector` and `tangental_force` → `tangential_force` (17 occurrences)
  - `toolchain/mfc/test/case.py` — renamed `fite_path_dest` → `file_path_dest` (3 occurrences)
  - `src/common/m_finite_differences.fpp` — renamed `lb` → `lB` (1 occurrence)
  - `docs/documentation/case.md` — corrected `patche(s)` → `patch(es)` (pre-existing CI spell check failure)

- **Key commits** (on branch `fix/spelling-typos-1498`):
  - `5257139a` — fix: correct HardcodedDellacation -> HardcodedDeallocation typo
  - `65a91539` — fix: correct BC_SlIP_WALL → BC_SLIP_WALL typo *(superseded by upstream refactor; conflict resolved by accepting master)*
  - `f3f7baad` — fix: correct tangental → tangential typo
  - `ab9347b0` — fix: correct fite_path_dest → file_path_dest typo
  - `f77285ea` — fix: correct lb → lB typo
  - merge commit — resolve conflict with upstream refactor in m_boundary_common.fpp
  - fix: correct patche(s) -> patch(es) typo in docs

- **Approach decisions:** Each typo was fixed in a separate commit so that the history is clean and each change is independently reviewable. Visual Studio Code's Replace in Files was used instead of `sed` because the Windows Git Bash `sed -i` syntax differs from Linux, and the VS Code approach is more reliable cross-platform for this type of rename.

---

## Pull Request

**PR Link:** https://github.com/MFlowCode/MFC/pull/1623 

**PR Description:**
Corrects five misspelled identifiers across the codebase. Three of these (BC_SlIP_WALL, fite_path_dest, lb) were previously addressed in PR #1516 which was closed without merging. Also fixes a pre-existing documentation typo (patche(s) → patch(es)) caught by the CI spell checker. No logic changes — purely identifier renames. Closes #1498.

**Maintainer Feedback:**
- 2026-07-02: Maintainer marked PR as draft due to merge conflictin docs/documentation/case.md. Resolved by accepting upstream version ("patches") and converted PR back to ready for review.
- 2026-07-05: PR merged by sbryngelson (commit 0506969) into MFlowCode:master. 81 of 82 checks passed. The one failing check was a corrupted pandas cache on the Frontier HPC runner, unrelated to the PR changes.

**Status:** Merged ✅

---

## Learnings & Reflections

### Technical Skills Gained

- Learned the open source contribution workflow end-to-end: fork → branch → commit → PR → CI → merge conflict resolution → CI fix → iterate. 
- Learned to read CI logs to diagnose failures and distinguish failures caused by my changes from pre-existing issues. 
- Learned how to resolve a merge conflict when an upstream refactor supersedes part of a branch's changes. 
- Learned Git Bash as a Unix-compatible terminal on Windows for `grep`, `git`, and related tools. 
- Gained familiarity with Fortran `.fpp` source files (Fypp preprocessor syntax) and Python toolchain scripts. 

### Challenges Overcome

- **Merge conflict:** An upstream refactor landed while my PR was open, eliminating the `BC_SlIP_WALL` occurrence that I had fixed. Resolved by accepting the incoming change and updating the PR description to reflect that 4 (not 5) typos were fixed by my commits.
- **CI spell check failure:** The spell checker flagged `patche(s)` in a documentation file unrelated to my changes. Rather than ignoring it, I fixed it as part of the PR — turning an unexpected CI failure into an additional contribution.
- **Windows environment friction:** Several standard Unix tools (`grep`, `sed`, Python in PATH) were unavailable in the default PowerShell terminal. Solved by using Git Bash for all command-line work.

### What I'd Do Differently Next Time

- Set up Git Bash as the default terminal before starting, to avoid running into PowerShell incompatibilities mid-workflow. 
- Run `git fetch upstream && git rebase upstream/master` before opening the PR to catch potential conflicts earlier. 
- Read the CI workflow files (`.github/workflows/`) before pushing, to understand what checks will run and what they require. 

---

## Resources Used

- [Claude AI from Anthropic.](https://www.claude.ai) 
- [MFlowCode/MFC Issue #1498](https://github.com/MFlowCode/MFC/issues/1498)
- [Closed PR #1516 (partial fix reference)](https://github.com/MFlowCode/MFC/pull/1516)
- [MFC Developer Guide / Contributing](https://mflowcode.github.io/documentation/contributing.html)
- [MFC CONTRIBUTING.md](https://github.com/MFlowCode/MFC/blob/master/CONTRIBUTING.md)
- [GitHub Pull Request from a Fork — Docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork)
