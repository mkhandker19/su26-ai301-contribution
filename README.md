# Contribution #1: go1.26: remove crypto/rand.Reader argument from all keygen and signing calls

**Contribution Number:** 1
**Student:** Mahin Khandker
**Issue:** https://github.com/letsencrypt/boulder/issues/8540
**Fork:** https://github.com/mkhandker19/boulder
**Status:** Phase IV — Complete (PR Approved, Awaiting Second Reviewer)

---

## Why I Chose This Issue

I chose this issue because it offers a well-scoped, clearly defined task that fits within the 3–4 week timeline of the CodePath AI 301 Open Source Capstone program. The issue involves a Go language modernization task — replacing a now-deprecated `crypto/rand.Reader` argument with `nil` across all keygen and signing call sites in the Boulder codebase. This is a great opportunity to get familiar with a large, real-world Go codebase without needing deep domain expertise upfront.

I also chose this issue because Let's Encrypt's Boulder is one of the most impactful open-source projects in internet security infrastructure. Even a cleanup contribution like this one is meaningful in a security-critical codebase. Through this work, I hope to improve my Go skills, learn how to navigate a large production codebase, and gain experience following a professional open-source contribution process end-to-end.

---

## Understanding the Issue

### Problem Description

As of Go 1.26, all cryptography functions that accept a `rand.Reader` argument will ignore it and instead use a secure internal randomness source. The Boulder codebase currently passes `crypto/rand.Reader` explicitly to every keygen and signing call. Since Go 1.26 makes this argument obsolete, it should be replaced with `nil` at all call sites to keep the code clean and aligned with the new Go standard.

### Expected Behavior

After maintainer feedback, the correct expected behavior is narrower than originally understood: only `rsa.GenerateKey` and `ecdsa.GenerateKey` calls in test files should pass `nil`. Signing calls (`CreateCertificate`, `CreateCertificateRequest`, `CreateRevocationList`, `Sign`, `SignPKCS1v15`) still require `rand.Reader` because they use randomness functionally, not just as an ignored argument.

### Current Behavior

The codebase passes `crypto/rand.Reader` explicitly to `GenerateKey` calls in test files. These are safe to replace with `nil` since Go 1.26 ignores the argument for key generation.

### Affected Components

- Test files across `ca/`, `cmd/admin/`, `cmd/ceremony/`, `cmd/cert-checker/`, and others
- Only `rsa.GenerateKey` and `ecdsa.GenerateKey` call sites in test files
- Reference: https://github.com/golang/go/issues/70942

---

## Reproduction Process

### Environment Setup

**Prerequisites installed:**

| Tool | Notes |
|------|-------|
| Go 1.26.4 | Install from [go.dev/dl](https://go.dev/dl). Verify with `go version`. |
| Git 2.x+ | Verify with `git --version`. |
| VS Code | Used with the official Go extension (`golang.go`). |

**Fork & Clone:**

```bash
# Clone your fork (not the original repo)
git clone https://github.com/mkhandker19/boulder.git
cd boulder

# Add the original repo as upstream for future syncing
git remote add upstream https://github.com/letsencrypt/boulder.git

# Verify remotes
git remote -v
# origin    https://github.com/mkhandker19/boulder.git (fetch)
# origin    https://github.com/mkhandker19/boulder.git (push)
# upstream  https://github.com/letsencrypt/boulder.git (fetch)
# upstream  https://github.com/letsencrypt/boulder.git (push)
```

**Create the working branch:**

```bash
git checkout main
git pull origin main
git checkout -b fix-issue-8540
git push origin fix-issue-8540
```

---

### Steps to Reproduce

> This is a code-pattern issue, not a runtime bug. Reproduction means confirming the old `rand.Reader` pattern still exists in test GenerateKey calls.

1. Navigate to the cloned repo root in the VS Code terminal.

2. Run a broad search for all `rand.Reader` usages:
   ```bash
   git grep "rand.Reader"
   ```

3. Narrow the search to key generation calls specifically:
   ```bash
   git grep "GenerateKey(rand.Reader"
   ```

4. Confirm the issue is still open and unresolved at:
   https://github.com/letsencrypt/boulder/issues/8540

**Expected result:** Multiple matches appear across test files confirming the old pattern is present.

---

### Reproduction Evidence

**Branch:** https://github.com/mkhandker19/boulder/tree/fix-issue-8540

Running `git grep "rand.Reader"` on branch `fix-issue-8540` returned matches across multiple test files, confirming the issue is present.

**Screenshot of terminal output:**

![git grep rand.Reader output](./reproduction_screenshot.png)

---

## Solution Approach

### Analysis

After maintainer review, the scope was narrowed. The fix only applies to `GenerateKey` calls in test files — not signing calls, not production code, not `CreateCertificate`. This is because only `rsa.GenerateKey` and `ecdsa.GenerateKey` fully ignore the `rand` argument in Go 1.26. Signing and certificate functions still use randomness functionally.

### Proposed Solution

Replace `rand.Reader` with `nil` only in `rsa.GenerateKey` and `ecdsa.GenerateKey` calls in test files. Remove any `crypto/rand` imports that become unused. Leave all other call sites unchanged.

### Implementation Plan (UMPIRE)

**Understand:**
Go 1.26 makes the `rand` argument to `GenerateKey` functions redundant. However, signing functions and certificate creation functions still use randomness functionally. The fix is scoped to `GenerateKey` calls in test files only.

**Match:**
Similar to targeted API cleanup PRs in large Go codebases. Scope: find all `rsa.GenerateKey` and `ecdsa.GenerateKey` usages in test files → replace `rand.Reader` with `nil` → clean up unused imports.

**Plan:**
1. Run `git grep "GenerateKey.*rand.Reader\|rand.Reader.*GenerateKey"` to find all affected test files.
2. Replace `rand.Reader` with `nil` only in those specific calls.
3. Remove unused `"crypto/rand"` imports from affected files.
4. Run `go test ./...` to confirm no new failures.
5. Verify vendor is untouched.

**Implement:**
> ✅ Complete. Changes made on branch `fix-issue-8540` and submitted as PR [#8802](https://github.com/letsencrypt/boulder/pull/8802).

**Review:**
- [x] Read `CONTRIBUTING.md` in boulder for PR format requirements
- [x] PR description references the issue (`Closes #8540`)
- [x] Only `GenerateKey` `rand.Reader` arguments in test files are replaced
- [x] All imports compile cleanly with no `imported and not used` errors
- [x] Vendor directory untouched

**Evaluate:**

| Check | Command | Expected Result |
|-------|---------|-----------------|
| No unused imports | `go build ./...` | Zero unused import errors |
| Key packages pass | `go test github.com/letsencrypt/boulder/goodkey` etc. | All pass |
| Vendor untouched | `git diff upstream/main -- vendor/` | Zero output |
| Production code unchanged | `git grep "nil" non-test files` | No GenerateKey/Sign nil in production |

---

## Testing Strategy

### Unit Tests

- [x] `github.com/letsencrypt/boulder/goodkey` — passes
- [x] `github.com/letsencrypt/boulder/goodkey/sagoodkey` — passes
- [x] `github.com/letsencrypt/boulder/privatekey` — passes
- [x] No new test failures introduced by our changes — remaining failures are pre-existing Windows/pkcs11 environment issues

### Integration Tests

- [ ] Full Boulder integration test suite — requires Docker/Linux environment; not runnable on Windows locally
- [ ] Certificate issuance flow end-to-end — deferred to CI on PR submission

### Manual Testing

Ran `go test` on all directly affected packages locally — all passed. Ran `go build ./... 2>&1 | grep "imported and not used" | grep -v vendor` — zero output. Ran `git diff upstream/main -- vendor/` — zero output confirming vendor is untouched. Ran `go test ./... 2>&1 | tee test_output.txt` twice to confirm no new failures from our changes.

---

## Implementation Notes

### Week 1 Progress
Selected issue: `go1.26: remove crypto/rand.Reader argument from all keygen and signing calls` in the letsencrypt/boulder repo. Reviewed the issue, understood the scope, and completed Phase I documentation.

### Week 2 Progress
Set up the local development environment in VS Code. Forked and cloned the boulder repo. Created working branch `fix-issue-8540`. Confirmed `origin` points to fork and added `upstream` pointing to the original boulder repo. Ran `git grep "rand.Reader"` and confirmed the issue is present across 11+ files. Phase II complete.

### Week 3 Progress
Implemented the initial fix on branch `fix-issue-8540`. Used `sed` via Git Bash to bulk-replace `rand.Reader` with `nil`. Restored `vendor/` after discovering third-party files were incorrectly modified. Removed unused `"crypto/rand"` imports. Fixed edge cases in `core/util.go` and `privatekey/privatekey.go`. Committed and pushed to the working branch. Phase III complete.

### Week 4 Progress
Opened PR [#8802](https://github.com/letsencrypt/boulder/pull/8802). Received maintainer feedback from `aarongable` and `jsha` requesting several changes:
- Restore a comment in `crl_test.go` that was accidentally modified
- Fix import ordering in `core/util.go`
- Remove all remaining unused `crypto/rand` imports
- Revert vendor directory changes
- Reduce scope: only replace `rand.Reader` with `nil` in `GenerateKey` calls in test files — signing and certificate functions still need `rand.Reader`

Addressed all feedback across multiple rounds of review. Rebased branch against upstream multiple times to resolve merge conflicts. Ran full local test suite to verify changes before each push.

Additional feedback received — `aarongable` noted there were still many files with unused `crypto/rand` imports, and that test output files (`test_output.txt`, `test_output_2.txt`) had been accidentally committed. Removed both test output files and removed remaining unused `crypto/rand` imports from `cmd/admin/cert_test.go`, `cmd/admin/key_test.go`, `issuance/cert_test.go`, `sa/sa_test.go`, and `test/load-generator/state.go`. Verified locally with `go test ./...` and `go build ./...` — zero unused import errors. Pushed and awaiting CI approval from maintainer. `aarongable` subsequently approved the PR after fixing a broken rebase on their end. PR now has one approval and is awaiting a second reviewer before merge.

### Code Changes

- **Files modified:** Test files across `ca/`, `cmd/admin/`, `cmd/ceremony/`, `cmd/cert-checker/`, and test helper files
- **Key commits:**
  - [`9a83fddfa`](https://github.com/mkhandker19/boulder/commit/9a83fddfa) — initial replacement
  - Multiple follow-up commits addressing reviewer feedback
- **Branch:** [`fix-issue-8540`](https://github.com/mkhandker19/boulder/tree/fix-issue-8540)
- **Approach decisions:**
  - Scope reduced to `GenerateKey` calls in test files only after maintainer clarification
  - Vendor restored with `git checkout upstream/main -- vendor/`
  - Used Claude Code and GitHub Copilot to assist with targeted fixes while reviewing all changes manually
  - Ran `go test ./... 2>&1 | tee test_output.txt` twice before each push to verify locally

---

## Pull Request

**PR Link:** [https://github.com/letsencrypt/boulder/pull/8802](https://github.com/letsencrypt/boulder/pull/8802)

**PR Description:** Replaced `rand.Reader` with `nil` in `rsa.GenerateKey` and `ecdsa.GenerateKey` calls in test files, in preparation for Go 1.26 compatibility. Removed resulting unused `"crypto/rand"` imports. Production code and signing calls are unchanged.

**Maintainer Feedback:**
- `aarongable` requested: restore modified comment, fix import ordering, remove unused imports, revert vendor changes, reduce scope to GenerateKey test calls only
- `jsha` requested: omit `CreateCertificate` from this PR as its rand usage depends on the signing algorithm
- All feedback addressed across multiple rounds of iteration
- `aarongable` fixed a broken rebase on their end, then approved the PR with comment: "Had to fix this up again -- looks like there was a broken rebase instead of a merge -- but it LGTM now."

**Status:** Approved by `aarongable` ✅ — awaiting second reviewer approval before merge

---

## Learnings & Reflections

### Technical Skills Gained
This project significantly improved my understanding of the Go programming language. I learned how Go's standard library cryptographic functions work, specifically how `ecdsa.GenerateKey`, `rsa.GenerateKey`, `rsa.SignPKCS1v15`, `ecdsa.Sign`, and `x509.CreateCertificate` accept a random reader argument and how Go 1.26 changed that behavior. I also got hands-on experience navigating a large, real-world Go codebase, removing unused imports, running `gofmt`, and using `go build` and `go test` to validate changes. Working in Git Bash on Windows while using Go tooling gave me practical experience bridging cross-platform development challenges.

### Challenges Overcome
The hardest part was genuinely understanding the issue before touching any code. At first, "replace `rand.Reader` with `nil`" seemed simple, but understanding *why* Go 1.26 made this change — and more importantly, *which* usages of `rand.Reader` should be replaced versus which ones should be kept — required careful reading of the Go proposal, the issue thread, and the codebase itself. Through maintainer feedback I learned that signing functions and certificate creation still use randomness functionally, so the fix is narrower than initially understood.

### What I'd Do Differently Next Time
I would be more careful about scoping my bulk find-and-replace commands to exclude the `vendor/` directory from the start. I would also run `go test ./...` locally and review the logs before every push rather than relying on CI to catch errors. The maintainer specifically called this out and it would have saved several rounds of feedback.

---

## Resources Used

- [Go 1.26 crypto/rand changes](https://github.com/golang/go/issues/70942)
- [Go 1.26 crypto reader overview](https://antonz.org/go-1-26/#crypto-reader)
- [Boulder CONTRIBUTING.md](https://github.com/letsencrypt/boulder/blob/main/docs/CONTRIBUTING.md)
- [Boulder README / Development Setup](https://github.com/letsencrypt/boulder/blob/main/README.md)
