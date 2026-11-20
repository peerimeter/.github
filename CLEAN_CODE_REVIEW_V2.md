## 5. V2 implementation status (2026-11-20)

Changes are split across separate PRs, each based on a freshly updated `origin/main`. No local tests or lint runs were performed; GitHub Actions CI owns test validation.

| Finding | Repository | Status | Notes |
| --- | --- | --- | --- |
| D-V2-1 | dashboard | Fixed | PR #14 merged; required CI and CodeRabbit passed. |
| D-V2-2 | dashboard | Fixed | PR #14 merged; required CI and CodeRabbit passed. |
| D-V2-3 | dashboard | Fixed | PR #14 merged; split JSON and void API requests and reused URL normalization. |
| D-V2-4 | dashboard | Fixed | PR #14 merged; expanded compressed JSX. |
| S-V2-1 | service | Fixed | PR #28; required CI passed, CodeRabbit review pending. |
| S-V2-2 | service | Fixed | PR #28; required CI passed, CodeRabbit review pending. |
| S-V2-3 | service | Fixed | PR #28; required CI passed, CodeRabbit review pending. |
| S-V2-4 | service | Fixed | PR #28; required CI passed, CodeRabbit review pending. |
| S-V2-5 | service | Fixed | PR #28; required CI passed, CodeRabbit review pending. |
| A-V2-1 | agent | Fixed | PR #87; CI running. |
| A-V2-2 | agent | Partially fixed | Extracted release confirmation; service-stop and verify/commit phases remain. |
| A-V2-3 | agent | Partially fixed | Shared key validation and reload builder extracted; managed runner refactor remains. |
| A-V2-4 | agent | Fixed | PR #87; daemon command and sync flow extracted into helpers. |
| A-V2-5 | agent | Partially fixed | ID validation, inventory lookup and temp-file writing deduplicated; Windows helper polling remains. |
| A-V2-6 | agent | Partially fixed | Update transaction staging/finalization extracted; `MigrateProfile` decomposition remains. |
| A-V2-7 | agent | Partially fixed | Linux systemd `ExecStart` parser extracted; Windows and macOS decompositions remain. |
| A-V2-8 | agent | Not fixed — invalid finding | Existing tests and E2E linker `-X` injection use the mutable URL variables; converting them to constants would break those seams. |
| A-V2-9 | agent | Fixed | PR #87; removed the duplicate CLI output-width helper. |
| A-V2-10 | agent | Fixed | PR #87; clarified node-registration validation and underlay discovery. |

PRs:

- Dashboard: https://github.com/peerimeter/peerimeter-dashboard/pull/14
- Service: https://github.com/peerimeter/peerimeter-service/pull/28
- Agent: https://github.com/peerimeter/peerimeter-agent/pull/87

Dashboard PR #14 merged after required CI and CodeRabbit passed; its non-required Cloudflare Workers preview build check reported failure. The dashboard local `main` branch was fast-forwarded to the merge. Service PR #28 has auto-merge enabled; its earlier CI failure (unused import) was fixed, and the updated CI is running after an additional CodeRabbit diagnostic fix. Agent PR #87 has auto-merge enabled; all CI jobs passed, and CodeRabbit is pending. The agent's local `go build ./...` and `git diff --check` passed; no local tests or lint were run.
