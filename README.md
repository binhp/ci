# ci — AI code verification pipeline

One offline container. Scans AI-written and human code for security and quality problems
before merge. Works with any coding flow and any CI.

> **Status:** this repo has the spec only: [`llm.txt`](llm.txt). A coding agent builds
> the pipeline from it. Nothing has run yet. Trust only what `.ai-review/tests/smoke.sh` proves.

## What it scans

- Secrets: Gitleaks
- Dependencies, SBOM: Trivy
- SAST: Semgrep with local rules
- Lint, quality: ESLint, PMD, SpotBugs, Error Prone, SwiftLint, Ruff, golangci-lint, jscpd
- Quality gate: Sonar-style, on new code
- AI guardrails (G1–G15): fake packages, hidden lint disables, weak tests, CI hardening
- Docs and comments: STE-lite lint with Vale and markdownlint. Warnings only. Never blocks.

Stacks: Java, JS/TS, React, iOS, Python, Go.
Exit codes: `0` clean, `1` findings, `2` error. Reports: SARIF in `.ai-review/artifacts/`.

## Setup

No manual values. The tool finds them.

1. Give `llm.txt` to a coding agent. Example: `claude "Follow llm.txt exactly."`
   Reply `next` after each file. Review each file.
2. Run `.ai-review/resolve.sh`. It finds the latest versions, URLs, digests, checksums.
   It verifies them. It writes `versions.lock.json`. Commit it. Review the diff.
3. Run `.ai-review/build.sh`. It builds and signs the image. It writes `.ai-review/image.digest`.
4. Run `bash .ai-review/tests/smoke.sh`. It must pass.

Needs: Docker, git, bash, jq, gh, cosign. Network is needed for steps 2–3 only.
The scan is offline.

## Run

```bash
./ai-review.sh --full-scan                          # whole repo
./ai-review.sh --diff-only --base-ref origin/main   # committed changes only
./ai-review.sh --lang java,js,react                 # override detection
./ai-review.sh --init-ci                            # wire into CI (dry run)
```

For diff mode: run `git fetch origin main` first. Commit your changes first.

## Add to CI

```bash
./ai-review.sh --init-ci           # detect GitHub, GitLab, other. Print snippet.
./ai-review.sh --init-ci --write   # create one new CI file. Never edit old files.
```

Any CI: fetch base branch, run `./ai-review.sh`, fail on exit 1 or 2, upload SARIF.
Make the check required in branch protection.

## Use with an AI agent

Copy the coding contract (`./ai-review.sh --help`) into `AGENTS.md` or `CLAUDE.md`.
Add: *"Before you finish, run `./ai-review.sh --diff-only`. Fix every finding.
Do not edit lint, CI, or gate config to pass."*

## Limits

- iOS compile check needs macOS. Objective-C and Kotlin: not covered.
- Static analysis does not replace review, tests, or threat modeling.
- Rule names change between tool versions. The agent must check each name.

## License

See [LICENSE](LICENSE).
