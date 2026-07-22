
# Contribution 2: Deallocate single-precision output arrays (`q_sf_s` family) in post_process

**Contribution Number:** 2, 
**Student:** Sarthak Sharma, 
**Issue:** https://github.com/MFlowCode/MFC/issues/1569, 
**Status:** Phase I Complete.

----

## Why I Chose This Issue

MFC's post_process step leaks memory on every run that uses single-precision output. The single-precision output arrays `q_sf_s`, `q_root_sf_s`, and `cyl_q_sf_s` are allocated when `precision == precision_single` (in `src/post_process/m_data_output.fpp`), but the module's finalizer, `s_finalize_data_output_module`, only deallocates the working-precision triple — so the single-precision arrays are never freed. It matters because post_process is run routinely to convert simulation output, and a leak on every single-precision run wastes memory and can matter for large jobs or repeated invocations. This is a real, maintainer-tracked bug with a clearly bounded fix: free the three single-precision arrays in the finalizer under the same condition that allocates them.

I chose this issue because:

1. It sits in the Fortran solver/post_process source, and my first merged contribution (#1623) already involved this codebase and toolchain, so I have MFC building locally and can move straight into Phase II.
2. It has an unambiguous definition of "done" and a fully documented mechanism: the maintainer's report names the allocation sites and the finalizer that omits the matching deallocations, so I can plan against concrete acceptance criteria rather than a vague goal.
3. It is genuinely wanted and already decided — the maintainer labels it a tracked cleanup, so there is no pending design or "is this a real issue" question to resolve before I can start.
4. It is a real defect (a memory leak) rather than a cosmetic change, which is a deliberate step up from my first contribution while staying well within a bounded, single-file-area scope. It also builds directly on the allocate/deallocate-pairing theme I worked with in my first issue.

From studying issue #1569, I understand my contribution needs to: (1) locate every allocation in the `q_sf_s` family that is guarded by the single-precision path in `m_data_output.fpp`; (2) add the matching deallocations to `s_finalize_data_output_module`, guarded by the same `precision == precision_single` condition so the free path mirrors the allocation path exactly; and (3) verify no single-precision output array is left unfreed and that working-precision runs are unaffected. I plan to confirm the leak first (for example, with a memory/leak-sanitizer build or an allocate/deallocate audit on a `--single` precision post_process run) and post that evidence before changing code. I have left a comment on the issue introducing myself, noting my prior MFC contribution, and outlining this plan.

----

## Understanding the Issue

### Problem Description

[Phase II — In your own words, what's broken or missing?]

### Expected Behavior

[Phase II — What should happen?]

### Current Behavior

[Phase II — What actually happens?]

### Affected Components

[Phase II — Which parts of the codebase are involved?]

----

## Reproduction Process

### Environment Setup

[Phase II — Notes on setting up your local development environment: challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

* **Commit showing reproduction:** [Link to commit in your fork]
* **Screenshots/logs:** [If applicable]
* **My findings:** [What you discovered during reproduction]

----

## Solution Approach

### Analysis

[Phase II — Your analysis of the root cause]

### Proposed Solution

[Phase II — High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]

1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist: does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

----

## Testing Strategy

### Unit Tests

* [ ] Test case 1: [Description]
* [ ] Test case 2: [Description]
* [ ] Test case 3: [Description]

### Integration Tests

* [ ] Integration scenario 1
* [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

----

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

* **Files modified:** [List]
* **Key commits:** [Links to important commits]
* **Approach decisions:** [Why you chose certain approaches]

----

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description — much of the content above can be adapted]

**Maintainer Feedback:**

* [Date]: [Summary of feedback received]
* [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

----

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

----

## Resources Used

* [Link to helpful documentation]
* [Tutorial or Stack Overflow post that helped]
* [GitHub issues or discussions that helped]
