# Security Review — mustangs1552/clock-display @ b53d820

- **Repository:** `mustangs1552/clock-display` (public, default branch `master`)
- **Pinned commit:** `b53d8201eefc9b9a4cc651a7051f3e16c523fe79` (`b53d820`, 2015-10-06, Matthew George)
- **Reviewed tree:** 6 tracked files at the pinned sha — `.gitattributes`, `.gitignore`, `ClockDisplay.java`, `NumberDisplay.java`, `README.TXT`, `lab3Questions_MGeorge.txt`; ~530 lines total, 2 Java classes (~130 LOC of Java).
- **Method:** shallow + blobless clone (`--depth 1 --filter=blob:none`), fetch of the pinned sha, read-only inspection of every tracked file, pattern grep for credential/injection/network indicators, remote ref and repo-metadata check (`gh api repos/...`). No code was executed; no build system exists to run.
- **Date of review:** 2026-10-26 (UTC)

## Verdict

**No security findings. 0 Critical / 0 High / 0 Medium / 0 Low.**

This is a BlueJ textbook exercise (`Objects First with Java`, chapter 3) consisting of two self-contained
in-memory classes. There is no input surface, no I/O, no network access, no persistence, no build system,
no dependency manifest, no CI, no committed binaries or artifacts, and no credential material anywhere in
the tree. There is nothing here that can be exploited: the program's only inputs are integers passed by
the caller through a constructor or `setTime`, and its only output is a `String` field. The findings table
is empty because the code is clean — no findings were manufactured to fill it.

## Findings table

| # | Severity | File:line | Issue | Fix |
|---|----------|-----------|-------|-----|
| — | — | — | *No security findings.* | — |

## Category-by-category results

| Category | Result |
|---|---|
| Hardcoded secrets / credentials / tokens | **None.** No API keys, tokens, passwords, private-key bodies, JWTs, or credential-bearing connection strings in any of the 6 files. `lab3Questions_MGeorge.txt` contains only the author's name/date and course answers — no secrets. Only URL in the tree is the Oracle Java help link in `.gitignore:11`. |
| Injection (SQL / command / path traversal) | **Not applicable / none.** No SQL, no `Runtime.exec`/`ProcessBuilder`, no shell, no file-path construction, no `File`/`Files`/`Paths` use anywhere. |
| Unsafe deserialization | **None.** No `ObjectInputStream`, no JSON/XML/YAML parsing, no reflection, no `Serializable` types. |
| Auth / authorization flaws and insecure defaults | **Not applicable.** No authentication, authorization, session, or privilege concept exists in the codebase. |
| Vulnerable or unpinned dependencies | **None.** No `pom.xml`, `build.gradle`, `ivy.xml`, `MANIFEST.MF`, or vendored jars. `.gitignore:6-9` ignores `*.jar`/`*.war`/`*.ear`, so no binaries are tracked. Zero third-party code: only JDK `int`/`String` primitives are used. |
| Sensitive data exposure in logs / config / committed artifacts | **None.** No logging, no config files, no `.env`, no committed build output (`*.class` ignored at `.gitignore:1`), no IDE state (`*.ctxt`/`*.pkh`/`*.bluej` ignored at `.gitignore:16-18`). `README.TXT:22` explicitly documents that the project "does not implement adequate handling of errors" — an upstream scope statement, not a vulnerability, since there is no untrusted input. |
| CI / workflow / supply-chain surface | **None.** No `.github/`, no workflow files, no deploy scripts, no release automation, no tags, single branch (`master`). |

## Non-security observations (nitpicks — explicitly NOT vulnerabilities)

These are correctness/robustness defects with no security impact, listed only because a code review
surface that omits them would be incomplete. They do not affect the verdict above.

| # | Class | File:line | Observation | Suggested fix |
|---|---|---|---|---|
| N1 | Correctness | `ClockDisplay.java:88` | 12-hour conversion tests `currHr > 12`, so hour 12 (noon) is still labelled `" am"` — 12:05 displays as "12:05 am" instead of "12:05 pm". | Use `currHr >= 12` for the PM branch and map `12` → `12 pm`, `0` → `12 am`. |
| N2 | Correctness | `ClockDisplay.java:38-44` | The 3-arg constructor calls `setTime()` (line 42) *before* assigning `twelveHourForm` (line 43), so `displayString` is first built in 24-hour format. Self-corrects on the next `timeTick()`/`getTime()`, which call `updateDisplay()` again, so no stale output escapes. | Assign `twelveHourForm` before `setTime(hour, minute)`. |
| N3 | Robustness | `NumberDisplay.java:55-60` | `setValue()` silently ignores out-of-range input — a caller gets no error and the object keeps its previous value. Combined with `ClockDisplay.setTime()` (`ClockDisplay.java:63-68`) this can leave `hours` updated and `minutes` unchanged. | Throw `IllegalArgumentException`, or validate both values before assigning either. `README.TXT:22` already acknowledges the missing error handling. |
| N4 | Robustness | `NumberDisplay.java:66-69`, constructor `:22-26` | `increment()` uses `value = (value + 1) % limit`; a `limit` of `0` gives `ArithmeticException` and a negative limit yields negative values. Not reachable — both call sites hardcode 24 and 60. | Reject `rollOverLimit <= 0` in the constructor. |
| N5 | Cosmetic | `ClockDisplay.java:98` | 24-hour mode prints the raw hour (`9:05`, not `09:05`) while minutes are zero-padded via `getDisplayValue()`. | Use `hours.getDisplayValue()` for the hour component in 24-hour mode. |

## Owner exceptions applied

No owner notes were present. `GET repos/mustangs1552/clock-display/contents/HermesReviews/reviewNotes.md`
returned `404 Not Found` on 2026-10-26, so there were no owner exceptions, scope directives, or
known-good-file notes to apply. Nothing was suppressed from this report.

## Scope and limitations

- Review covers only the pinned commit `b53d8201eefc9b9a4cc651a7051f3e16c523fe79`, which is also the
  current tip of `master` (verified via `git ls-remote`); no other branches or tags exist.
- The clone was shallow (`--depth 1`), so repository history before the pinned sha was not inspected for
  historical secrets. Since the pre-history is textbook scaffolding from the book's sample project and no
  live credentials relate to this repo, the residual risk is negligible.
- No dynamic analysis was performed: the project has no entry point, no build configuration, and no
  runtime dependencies.
- No project code, issues, pull requests, branches, or tags were touched. The only write made was the
  commit adding this document under `HermesReviews/`.
