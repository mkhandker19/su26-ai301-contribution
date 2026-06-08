# Contribution #1: go1.26: remove crypto/rand.Reader argument from all keygen and signing calls

**Contribution Number:** 1
**Student:** Mahin Khandker
**Issue:** https://github.com/letsencrypt/boulder/issues/8540
**Fork:** https://github.com/mkhandker19/boulder
**Status:** Phase I — Complete

---

## Why I Chose This Issue

I chose this issue because it offers a well-scoped, clearly defined task that fits within the 3–4 week timeline of the CodePath AI 301 Open Source Capstone program. The issue involves a Go language modernization task — replacing a now-deprecated `crypto/rand.Reader` argument with `nil` across all keygen and signing call sites in the Boulder codebase. This is a great opportunity to get familiar with a large, real-world Go codebase without needing deep domain expertise upfront.

I also chose this issue because Let's Encrypt's Boulder is one of the most impactful open-source projects in internet security infrastructure. Even a cleanup contribution like this one is meaningful in a security-critical codebase. Through this work, I hope to improve my Go skills, learn how to navigate a large production codebase, and gain experience following a professional open-source contribution process end-to-end.

---

## Understanding the Issue

### Problem Description

As of Go 1.26, all cryptography functions that accept a `rand.Reader` argument will ignore it and instead use a secure internal randomness source. The Boulder codebase currently passes `crypto/rand.Reader` explicitly to every keygen and signing call. Since Go 1.26 makes this argument obsolete, it should be replaced with `nil` at all call sites to keep the code clean and aligned with the new Go standard.

### Expected Behavior

All keygen and signing function calls throughout the Boulder codebase should pass `nil` instead of `crypto/rand.Reader` as the random reader argument, reflecting the Go 1.26 change that makes the argument unnecessary.

### Current Behavior

The codebase passes `crypto/rand.Reader` explicitly to cryptographic functions such as key generation and signing calls. While this is not broken behavior, it is now redundant and should be cleaned up to match Go 1.26 conventions.

### Affected Components

- All files in the Boulder codebase that call keygen or signing functions with a `crypto/rand.Reader` argument
- Primarily affects cryptographic utility code and certificate authority components
- Reference: https://github.com/golang/go/issues/70942

---

## Reproduction Process

### Environment Setup

*To be filled in during Phase II — notes on setting up the Boulder local development environment using Docker Compose, any challenges faced, and how they were resolved.*

### Steps to Reproduce

1. Clone the fork: `git clone https://github.com/mkhandker19/boulder`
2. Search the codebase for `crypto/rand.Reader` in keygen and signing call contexts
3. Observe that all call sites pass `rand.Reader` explicitly where `nil` should now be used

### Reproduction Evidence

- **Commit showing reproduction:** *To be added during Phase II*
- **Screenshots/logs:** *To be added during Phase II*
- **My findings:** *To be added during Phase II*

---

## Solution Approach

### Analysis

The root cause is not a bug but a code modernization need. Go 1.26 changed the behavior of cryptographic functions so that the `rand` argument is ignored in favor of a secure internal source. The fix is purely mechanical — find every call site passing `crypto/rand.Reader` to a keygen or signing function and replace it with `nil`.

### Proposed Solution

Do a codebase-wide search for `crypto/rand.Reader` usage in cryptographic function calls and replace each instance with `nil`. Ensure all existing tests continue to pass after the change.

### Implementation Plan

Using the UMPIRE framework:

**Understand:** The Go 1.26 update makes the `rand.Reader` argument to cryptographic functions redundant. Boulder passes it explicitly everywhere, and this should be cleaned up by replacing all instances with `nil`.

**Match:** This is similar to large-scale codebase refactors done in open-source Go projects when a standard library API changes. The pattern is: find all usages → replace → verify tests pass.

**Plan:**
1. Run `grep -rn "rand.Reader" --include="*.go"` in the Boulder repo to locate all relevant call sites
2. Review each usage to confirm it is a keygen or signing call (not other rand usages)
3. Replace each `rand.Reader` argument with `nil` at all confirmed call sites
4. Run the Boulder test suite to verify nothing is broken
5. Update any related imports that are no longer needed after the change

**Implement:** *Link to branch/commits to be added as work progresses*

**Review:**
- [ ] Does the change follow Boulder's contribution guidelines (tests required, deployability requirements met)?
- [ ] Are there any edge cases where `nil` behaves differently than `rand.Reader`?
- [ ] Are all modified files covered by existing tests?

**Evaluate:** Run the full Boulder test suite locally using Docker Compose and confirm all tests pass with the changes applied.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: Existing keygen tests still pass after replacing `rand.Reader` with `nil`
- [ ] Test case 2: Existing signing tests still pass after the change
- [ ] Test case 3: No new test failures introduced across the full test suite

### Integration Tests

- [ ] Full Boulder integration test suite passes with Docker Compose
- [ ] Certificate issuance flow works end-to-end with the updated code

### Manual Testing

*To be filled in during Phase III — results of manual testing performed.*

---

## Implementation Notes

### Week 1 Progress
*To be filled in — environment setup, initial codebase exploration, locating all relevant call sites.*

### Week 2 Progress
*To be filled in — implementation of changes, running tests, addressing any issues.*

### Week 3 Progress
*To be filled in — finalizing the fix, preparing the PR, self-review.*

### Code Changes

- **Files modified:** *To be filled in during implementation*
- **Key commits:** *To be filled in during implementation*
- **Approach decisions:** *To be filled in during implementation*

---

## Pull Request

**PR Link:** *To be added upon submission*
**PR Description:** *Draft to be added during Phase III*

**Maintainer Feedback:**
- *To be filled in as feedback is received*

**Status:** Not yet submitted

---

## Learnings & Reflections

### Technical Skills Gained
*To be filled in upon completion*

### Challenges Overcome
*To be filled in upon completion*

### What I'd Do Differently Next Time
*To be filled in upon completion*

---

## Resources Used

- [Go 1.26 crypto/rand changes](https://github.com/golang/go/issues/70942)
- [Go 1.26 crypto reader overview](https://antonz.org/go-1-26/#crypto-reader)
- [Boulder CONTRIBUTING.md](https://github.com/letsencrypt/boulder/blob/main/docs/CONTRIBUTING.md)
- [Boulder README / Development Setup](https://github.com/letsencrypt/boulder/blob/main/README.md)
