# summer2026-AI301-contribution
For CodePath's course AI301 "AI Open Source Capstone".

# Contribution 1: Fix internal spelling typos (HardcodedDellacation, BC_SlIP_WALL, tangental, fite_path_dest, lb).

**Contribution Number:** 1498  
**Student:** Sarthak Sharma  
**Issue:** https://github.com/MFlowCode/MFC/issues/1498  
**My fork of the original repository:** https://github.com/engineer-scientist/Multiphase-Flow-Code
**Status:** Phase II completed.

---

## Why I Chose This Issue

This issue is well-documented. The maintainer identified every typo with exact file paths and line numbers, so there is no ambiguity about what "done" looks like. The typos involve straightforward identifier renames in code files, which matches my skills in programming.

I am also drawn to MFC as a project. It is a high-performance multiphase flow solver that was a 2025 Gordon Bell Prize Finalist, meaning it runs on tens of thousands of GPUs at scale. Contributing even a small cleanup to a project of this caliber is meaningful. 

---

## Understanding the Issue

### Problem Description

The MFC codebase (a high-performance multiphase flow solver) contains five internal spelling typos in identifier names across Fortran and Python source files: HardcodedDellacation, BC_SlIP_WALL, tangental, fite_path_dest, and lb. Because Fortran is case-insensitive, these compile and run correctly, but they reduce grep-ability and code readability. 

### Expected Behavior

All identifier names in the codebase should be spelled correctly so that grep searches work reliably and the code is readable without confusion.

### Current Behavior

grep for HardcodedDeallocation returns no results (the only spelling present is the misspelled HardcodedDellacation). Similarly, grep for tangential misses all 17 occurrences of tangental_vector and tangental_force in m_collisions.fpp.

### Affected Components

1. HardcodedDellacation → HardcodedDeallocation
Fypp macro defined at src/common/include/ExtrusionHardcodedIC.fpp:195 (#:def HardcodedDellacation()) and invoked 15 times in src/pre_process/m_icpp_patches.fpp (16 total in src/; the correct spelling appears 0 times). Self-consistent so it works, but misspelled. → rename definition + all 15 call sites.

2. BC_SlIP_WALL → BC_SLIP_WALL at src/common/m_boundary_common.fpp:263
The correct constant BC_SLIP_WALL is used at lines 100, 130, 167, 197, 233; only line 263 has the lowercase-l typo. Fortran is case-insensitive so it compiles and selects the right constant, but a grep for BC_SLIP_WALL misses this site. → line 263 case (BC_SLIP_WALL).

3. tangental → tangential — 17 occurrences, all in src/simulation/m_collisions.fpp (tangental_vector, tangental_force): lines 82, 93, 131, 132, 133, 137, 138, 143, 149, 165, 170, 207, 208, 209, 211, 212, 216. Cosmetic local-variable rename.

4. fite_path_dest → file_path_dest at toolchain/mfc/test/case.py:482

5. lb → lB at src/common/m_finite_differences.fpp:42


---

## Reproduction Process

### Environment Setup

Simple environment like Visual Studio code is sufficient for this issue.

### Steps to Reproduce

The issue is "internal spelling typos", so there is no effect on the software functionality because of it. We just have to rename some mis-spelled variables and ensure that there is consistency throughout the codebase for what those variables point to.

---

## Solution Approach

### Analysis

The issue is caused due to spelling errors in cvertain variables of the code. 

### Proposed Solution

Search and correct the mis-spelled variables in the code.

### Implementation Plan

Will be working on Visual studio Code for easy searching and editing of the codebase.

Examples given in the repository readme: 
1. HardcodedDellacation → HardcodedDeallocation
2. BC_SlIP_WALL → BC_SLIP_WALL at src/common/m_boundary_common.fpp:263
3. tangental → tangential
4. fite_path_dest → file_path_dest

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What I tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What I built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as I work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why I chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What I learned technically]

### Challenges Overcome

[What was hard and how I solved it]

### What I'd Do Differently Next Time

[Reflection on my process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

