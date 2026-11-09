# ci — AI code verification pipeline

One offline container. Scans AI-written and human code before merge on 4 axes: security,
reliability, maintainability, tests. Works with any coding flow and any CI.

> **Status:** this repo has the spec, [`llm.txt`](llm.txt), and the scan image file,
> [`.ai-review/Dockerfile`](.ai-review/Dockerfile). A coding agent builds the other files from
> the spec. The image does not build until they exist. Trust only what `.ai-review/tests/smoke.sh` proves.

## What it scans

- Secrets: Gitleaks
- Dependencies, SBOM: Trivy
- SAST: Semgrep with local rules
- Lint, quality: ESLint, PMD, SpotBugs, Error Prone, SwiftLint, Ruff, golangci-lint, jscpd
- Quality gate: Sonar-style, on new code. One status per axis. Any failed axis fails the run.
- AI guardrails (G1–G15): fake packages, hidden lint disables, weak tests, CI hardening
- Docs and comments: STE-lite lint with Vale and markdownlint. Warnings only. Never blocks.

Stacks: Java, JS/TS, React, iOS, Python, Go.
Exit codes: `0` clean, `1` findings, `2` error. Reports: SARIF and `summary.json` in
`.ai-review/artifacts/`.

In CI, rules and config come from the image and the base branch, not from the pull
request (PR). Limits apply. See `llm.txt` section TRUST MODEL.

## Setup

No manual values. The tool finds them.

1. Give `llm.txt` to a coding agent. Example: `claude "Follow llm.txt exactly."`
   Reply `next` after each file. Review each file.
   To build fewer stacks, add `stacks=java,js` to the prompt.
2. Run `.ai-review/resolve.sh`. It finds the latest versions, URLs, digests, checksums.
   It verifies them. It writes `versions.lock.json`. Commit it. Review the diff.
3. Run `.ai-review/build.sh`. It builds [`.ai-review/Dockerfile`](.ai-review/Dockerfile) with the
   values from `versions.lock.json`. It signs the image and writes `.ai-review/image.digest`.
   Then it finds your CI and asks one question: add the scan to CI? See [Add to CI](#add-to-ci).
4. Run `bash .ai-review/tests/smoke.sh`. It must pass.

Host tools: you install nothing by hand. Each script runs `.ai-review/doctor.sh` first.
It finds your OS and package manager. It installs missing tools after one `[y/N]` prompt.
Rules and limits: `llm.txt` section P.

```bash
.ai-review/doctor.sh --plan    # show missing tools, change nothing
.ai-review/doctor.sh --yes     # install with no prompt (CI, agents)
```

Windows: run `wsl --install` in an admin PowerShell first. Then use the Ubuntu shell.
Network is needed for steps 2–3 only. The scan is offline.

### Use Apple container (macOS)

The pipeline also runs on [Apple container](https://github.com/apple/container) 1.6.0 or later.
It needs macOS 26 or later on Apple silicon. Rules: `llm.txt` section 7 RUNTIME.

1. Set the runtime: `export AI_REVIEW_RUNTIME=container`.
2. Run `.ai-review/doctor.sh`. If `container` is missing, it installs the Apple-signed package.
3. Do the 4 setup steps above. `build.sh` uses `container build`.

Default `AI_REVIEW_RUNTIME=auto` uses Docker first, then Apple container.

> **Warning:** The Apple container runtime has no no-new-privileges, no seccomp filter and
> no pids cgroup. The scan prints this warning on each run. Use Docker in CI.

## Build the image

`build.sh` runs the build. Do not build by hand: each tool needs 3 build args from the lock file.
Rules: `llm.txt` section 1. The build context is `.ai-review/`.
The Dockerfile has a default `BASE_IMAGE` pinned by digest. `resolve.sh` keeps it equal to the lock file.

The build needs these files in `.ai-review/`. If one is missing, the build stops.

- `versions.lock.json`, `.cache/trivy/`, `.cache/data/`, `.cache/npm/`, `.cache/pypi/` (from `resolve.sh`)
- `pipeline.sh`, `guardrails.sh`, `quality-gate.json`, `.semgrepignore`
- `lint/`, `rules/`, `tests/`

The Dockerfile installs the tools for all stacks: Java, JS/TS, React, iOS, Python, Go.
The image runs as user `10001`, not root.

## Run

```bash
./ai-review.sh --full-scan                          # whole repo
./ai-review.sh --diff-only --base-ref origin/main   # committed changes only
./ai-review.sh --lang java,js,react                 # override detection
./ai-review.sh --init-ci                            # wire into CI (dry run)
AI_REVIEW_RUNTIME=container ./ai-review.sh --full-scan   # Apple container
```

For diff mode: run `git fetch origin main` first. Commit your changes first.

## Add to CI

`build.sh` asks this for you. To do it by hand later:

```bash
./ai-review.sh --init-ci           # detect GitHub, GitLab, other. Print snippet.
./ai-review.sh --init-ci --write   # create one new CI file. Never edit old files.
```

Any CI: fetch base branch, run `./ai-review.sh`, fail on exit 1 or 2, upload SARIF.
Make the check required in branch protection. `--init-ci` prints the CODEOWNERS lines.

A weekly publish job refreshes versions and the Trivy database. It builds, signs and
pushes the image. Then it opens a PR. A human merges it.

Existing repo with old findings: run a full scan, then `./ai-review.sh --update-baseline`
one time. Commit `.ai-review/baseline.json`. New findings still fail.

## Use with an AI agent

Copy the coding contract (`./ai-review.sh --help`) into `AGENTS.md` or `CLAUDE.md`.
Add: *"Before you finish, run `./ai-review.sh --diff-only`. Fix every finding.
Do not edit lint, CI, or gate config to pass."*

## Limits

- iOS compile check needs macOS. Objective-C and Kotlin: not covered.
- Apple container: local use only. CI uses Docker. A local build has no provenance and no SBOM.
- Static analysis does not replace review, tests, or threat modeling.
- Rule names change between tool versions. Smoke case R5 checks each name.
- On GitHub, a PR can edit its own workflow file. Only CODEOWNERS review and an org
  ruleset stop that. The tool prints them. It cannot apply them.
- Fork PRs and private repos without GitHub Advanced Security: no SARIF upload.
  Artifacts and the exit code stay.
- The scan runs no tests. Pass the CI coverage report with `--coverage`. Without it, the
  tests axis is `not_evaluated`.
- Diff mode misses a finding on an unchanged line that a change breaks. The weekly full
  scan finds it.
- A PR that adds a Java dependency needs a cache seed from a trusted branch.
- A PR that adds any dependency must also add it to `.ai-review/allowed-deps.txt`.
  A code owner reviews it. The scan cannot check its age or history offline.
- Type checks need installed dependencies. Install them before the scan, with scripts off.
- The publish PR needs a GitHub App token, or the required check does not start on it.
- Library and architecture choices (for example "Redux Toolkit only") are advisory.
  They do not fail the run.

## License

See [LICENSE](LICENSE).
